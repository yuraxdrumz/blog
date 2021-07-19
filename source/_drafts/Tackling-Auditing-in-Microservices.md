---
title: Tackling Auditing in Microservices
tags:
---
## Use Case (Audit Issues No One Talks About)

We provide zero trust network access at my current company. Any organization can run a docker container, which we provide, inside its vLANs [5] and then they gain access to my company's SaaS UI, which allows administrators to limit users to various resources in the organization, with full visibility in a secure fashion, all without using VPNs. Working with zero trust access means auditing is a first class citizen. Each time a user or an administrator (a person who manages its users), does an operation, it must be available for viewing with as much detail as possible.

### Asynchronous, Synchronous Or Both ?

Due to various security and compliance concerns, our proxies, which are responsible for the ZTNA [6] solution my company provides, are completely isolated from the cluster where all the users, applications, audit and 2x more services reside. This means, I had to decouple the sending and receiving of audits for the proxies.
The services that did have access to the audit service called it directly in a sychronous manner, while the proxies received a secret and a key for an SNS topic in AWS on startup, which they sent the audits to in an asynchronous manner.

The audit service knew to listen both on a network port for direct communication as well as asynchronously poll an SQS queue, which was subscribed to the SNS topic. When receiving an audit synchronously, the audit service would save the audit in MongoDB and publish it to the SNS like the proxies would with a filter policy to avoid consuming the audit again via the asynchronous route.

### How Many Structs Do I Keep ?

Before the audit refactor, each audit message had its own golang struct.
Each message sent from the proxies was read from the SQS as json, unmarshalled to its appropriate Golang struct, marshalled back to bson and then saved to MongoDB.

This solution posed several issues:

1. Each new audit must be added as a new struct both on the proxies as well as the audit service.
2. MongoDb is a main-follower topology, meaning all writes will go to the main db and all reads will occur in the followers. With the audits, we were heavy on writes and had very few reads.
3. Reading the data back was a pain, as we had to extract the bson from MongoDB, derive what type of bson it is, create the appropriate struct for it and populate it. This meant, a large switch case, which had to be modified with each addition of an audit.

Insert example

```golang

const (
    UserLoggedIn = iota + 1
    AppRemovedFromGroup
)

type AuditType struct {
    AuditType string `json:"audit_type" bson:"audit_type"`
}

type BaseAudit struct {
    Id string `json:"id" bson: "id"`
    Name string `json:"name" bson: "name"`
    CreatedAt string `json:"created_at" bson: "created_at"`
    ProcessedAt string `json:"processed_at" bson: "processed_at"`
    Description string `json:"description" bson: "description"`
    AuditType AuditType `json:",inline" bson: ",inline"`
}

type UserLoggedIn struct {
    BaseAudit string `json:",inline" bson:",inline"`
    UserId string `json:"user_id" bson: "user_id"`
    Organization string `json:"organization" bson: "organization"`
}

type AppRemovedFromGroup struct {
    BaseAudit string `json:",inline" bson:",inline"`
    AppId string `json:"app_id" bson: "app_id"`
    GroupId string `json:"group_id" bson: "group_id"`
    Organization string `json:"organization" bson: "organization"`
    Initiator string `json:"initiator" bson: "initiator"`
}

func (a *Audit) ReadMessage() error {
    message := a.sqs.ReadMessage(...opts)
    
    auditType := &AuditType{}

    err := json.Unmarshal(auditType)
    if err != nil {
        return err
    }

    switch auditType.AuditType{
        case *UserLoggedIn:
            userLoggedIn := &UserLoggedIn{}
            err := json.Unmarshal(userLoggedIn)
            if err != nil {
                return err
            }
        case *AppRemovedFromGroup:
            appRemovedFromGroup := &AppRemovedFromGroup{}
            err := json.Unmarshal(appRemovedFromGroup)
            if err != nil {
                return err
            }       
    }
}

```

Reading

```golang

func (a *Audit) GetAudits() ([]interface{}, error) {
    var audits []interface{}
    retrieved, err := a.auditStore.GetAudits()
    for _, doc := range retrieved {
        docType, ok := doc["type"]
        if !ok {
            return nil, errors.New(fmt.Sprintf("bson doc must have a type, doc=%+v", doc))
        }
        switch docType {
            ...
        }
    }
    return audits
}
```

As you can see, it was a nightmare to maintain.

Because we had about 40 types of audit logs and all of them had the BaseAudit plus a few fields, I decided to unify all audit types to a single audit struct. That way, the sending and receiving sides can send any string value in the audit_type field and no further changes are needed. Upon inserting or querying the audit data, if a field doesn't exist, it will be omitted.

```golang

const (
    UserLoggedIn = iota + 1
    AppRemovedFromGroup
)

type AuditType struct {
    AuditType string `json:"audit_type" bson:"audit_type"`
}

type Audit struct {
    Id string `json:"id,omitempty" bson: "id,omitempty"`
    Name string `json:"name,omitempty" bson: "name,omitempty"`
    CreatedAt string `json:"created_at,omitempty" bson: "created_at,omitempty"`
    ProcessedAt string `json:"processed_at,omitempty" bson: "processed_at,omitempty"`
    Description string `json:"description,omitempty" bson: "description,omitempty"`
    AuditType AuditType `json:",inline" bson: ",inline"`
    UserId string `json:"user_id,omitempty" bson: "user_id,omitempty"`
    Organization string `json:"organization,omitempty" bson: "organization,omitempty"`
    AppId string `json:"app_id,omitempty" bson: "app_id,omitempty"`
    GroupId string `json:"group_id,omitempty" bson: "group_id,omitempty"`
    Organization string `json:"organization,omitempty" bson: "organization,omitempty"`
    Initiator string `json:"initiator,omitempty" bson: "initiator,omitempty"`
}

func (a *Audit) ReadMessage() error {
    message := a.sqs.ReadMessage(...opts)
    
    audit := &Audit{}

    err := json.Unmarshal(audit)
    if err != nil {
        return err
    }

    err := a.db.Save(audit)
    if err != nil {
        return err
    }
}

```

Reading

```golang

func (a *Audit) GetAudits() ([]*Audit, error) {
    var audits []*Audit
    retrieved, err := a.auditStore.GetAudits()
    if err != nil {
        return nil, err
    }

    for _, doc := range retrieved {
        audits = append(audits, doc.ToAudit())
    }
    return audits, nil
}
```

### Enrichment

When creating an audit like the one below, an administrator expects to see something like `System configuration under site X changed from Y to Z` or `User X accessed resource Y`.

```golang
type Audit struct {
    Id string `json:"id,omitempty" bson: "id,omitempty"`
    Name string `json:"name,omitempty" bson: "name,omitempty"`
    CreatedAt string `json:"created_at,omitempty" bson: "created_at,omitempty"`
    ProcessedAt string `json:"processed_at,omitempty" bson: "processed_at,omitempty"`
    Description string `json:"description,omitempty" bson: "description,omitempty"`
    AuditType AuditType `json:",inline" bson: ",inline"`
    UserId string `json:"user_id,omitempty" bson: "user_id,omitempty"`
    Organization string `json:"organization,omitempty" bson: "organization,omitempty"`
    AppId string `json:"app_id,omitempty" bson: "app_id,omitempty"`
    GroupId string `json:"group_id,omitempty" bson: "group_id,omitempty"`
    Organization string `json:"organization,omitempty" bson: "organization,omitempty"`
    Initiator string `json:"initiator,omitempty" bson: "initiator,omitempty"`
}
```

The issue here, is, we only have `x_id` fields without the actual data.
This meant, for each administrator that requested to see the audit log, we needed to enrich each user_id, app_id, etc. from their respective micro-services.

Another issue was that with each audit sent via the SNS, there are several consumers, apart from the audit service. For instance, we allow customers to receive the audits as they happen to any endpoint / S3 bucket, which posed serious performance issues to the micro-services, which were part of the data enrichment process.
For each new consumer subscribed to the SNS topic, the services received more queries. At first a redis cache was added to help with the performance issues.

The issue above really reminds me of querying in event sourcing, where without CQRS we must reconstruct the state all over again for each query.

The possible solutions to mitigate the issue were:

1. Keep a snapshot db for the audits, similar to CQRS. But, because we send the audit to several consumers at once, keeping a snapshot db would not really help for consumers other than the audit service.

2. Pass the data enriched. This solution increases the amount of data we send, but removes the load on the micro-services needed for data enrichment afterwards. On top of that, we can add any number of consumers and all will receive data already enriched.

I went with passing the data enriched. Because of the nature of the product, all places that call the audit service in a synchronous manner already have the data at hand, either by querying in advance or by receiving it from another service.

The last piece of the puzzle was the proxies, which do not have access to any of the services that own the data needed.
On each user connection, the proxies ask our control plane, if the user is allowed. The control plane, which is where all of our services reside returns a rule to the proxy, which is an instruction on what the user is allowed to do and where he/she is allowed to access. I added all the audit data on the rule, which the proxy then sent to SNS together with its data.

## Summary

We saw several implementation options for auditing together with a summary table that shows the implementation complexity, scalability and integration with existing architecture levels. Afterwards, I introduced a real world use case, I had the chance to implement. We saw how we can integrate with the audit service both synchronously and asynchronously. Afterwards, we saw how a unified audit struct helped with introduction of new audits and maintenance of existing ones. At last, we saw how enriching the data beforehand, removed a lot of stress from the system and future proofed new consumers that subscribe to the auditing topic.

## Bibliography

1. [logging-levels](https://sematext.com/blog/logging-levels)
2. [event-sourcing](https://docs.microsoft.com/en-us/azure/architecture/patterns/event-sourcing)
3. [event-sourcing-microservices](https://microservices.io/patterns/data/event-sourcing.html)
4. [cqrs](https://martinfowler.com/bliki/CQRS.html)
5. [vLAN](https://en.wikipedia.org/wiki/Virtual_LAN)
6. [ZTNA](https://www.gartner.com/en/information-technology/glossary/zero-trust-network-access-ztna-)
