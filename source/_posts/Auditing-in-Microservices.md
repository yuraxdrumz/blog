---
title: Auditing in Microservices
tags:
  - Software Architecture
  - Microservices
  - Auditing
  - Event Sourcing
  - Golang
category: Architecture
date: 2021-07-19 19:41:33
thumbnail: images/audit.png
---

In software, auditing means tracking user or system activities for various needs, such as business or security. An example would be - `user X tried to access resource Y`.
When I encuntered issues with auditing at my current company, I looked for solutions online, most of which, were either vague, lackluster or plain simple. That is why, after having implemented and dealt with auditing at scale, I would like to share my thoughts. In this post, I will show the various methods for implementing auditing together with code examples and pros and cons.

## Preface

All code examples will be written in Golang.

## Auditing With Logs

The simplest form of audit is to log the event that happened from the business logic.
Afterwards, it is possible to aggregate logs and send them to an ELK service for parsing and viewing.

```golang

type User struct {
    name string `json:"name"`
    id string `json:"id"`
}

func (g *Groups) AddUserToGroup(user *User, groupId uuid.UUID) error {
    err := g.validateUser(user)
    if err != nil {
        return err
    }

    err := g.validateGroupId(groupId)
    if err != nil {
        return err
    }

    err := g.db.AddUserToGroup(user)
    if err != nil {
        return err
    }

    // log user added to group for various purposes
    g.logger.info("user %s was added to group %s", user.Name, groupId)
}
```

### Pros

- Easy to implement
- Easy to ship to any 3rd party service

### Cons

- Can take a while until the logs are scraped, parsed and shipped
- Writing many operations to stdout will cause a performance hit

## Auditing With Databases

Developers will usually have some sort of database like MongoDB or PostgreSQL that they use, which can be used for audits as well. It is also possible to have a separate database for the auditing.

```golang

type User struct {
    name string `json:"name"`
    id string `json:"id"`
}

func (g *Groups) AddUserToGroup(user *User, groupId uuid.UUID) error {
    err := g.validateUser(user)
    if err != nil {
        return err
    }

    err := g.validateGroupId(groupId)
    if err != nil {
        return err
    }

    err := g.db.AddUserToGroup(user)
    if err != nil {
        return err
    }

    // add user to audit db, can be same db as main application or another db specifically for auditing
    err := g.auditDb.AddUserToGroup(user)
    if err != nil {
        g.logger.error("failed to add audit for userAddedToGroup, user=%s", user)
    }

    g.logger.info("user %s was added to group %s", user.Name, groupId)
}
```

### Pros

- Main application db can be used for the auditing as well, if the service's scale is low, which saves in operational costs and maintainance
- Audits can be exposed as an API for customers, marketing and various teams in the organization.

### Cons

- Not scalable if you use the same database as your application
- Tougher to manage in a microservices environment as each service should have its own database, meaning no single owner of the audit data, unless you make an audit service, which we will discuss later in the post.

## Auditing With Event Sourcing And CQRS

From microservices.io [1, 2]
> Event sourcing persists the state of a business entity such an Order or a Customer as a sequence of state-changing events. Whenever the state of a business entity changes, a new event is appended to the list of events. Since saving an event is a single operation, it is inherently atomic. The application reconstructs an entity’s current state by replaying the events.

CQRS - Command Query Responsibility Segregation [3]
> At its heart is the notion that you can use a different model to update information than the model you use to read information

To keep things simple, with event sourcing, your business logic fires commands that in turn generate events that are appended to the store (append only database that is usually fast for writes), and each time a new event is appended, it is also published for the appropriate consumers to react.

Event sourcing goes hand in hand with CQRS, due to events needing some sort of snapshot (a normalized view), to allow for fast reads. If we were to query only from the append only store, we would need to start from a known state, extract all relevant events and apply each one of them to the known state, which is very slow. With CQRS, we store a snapshot of the latest data and when a query comes, we are able to return the snapshot instead of reconstructing the state.

Back to our example, we fire the `AddUserToGroup` Command, which generates the `UserAddedToGroup` event. Afterwards, the group consumer receives the `UserAddedToGroup` event, and reacts accordingly by populating the new data in a normalized way for easier querying.

Command side

```golang

type User struct {
    name `json:"name"`
    id `json:"id"`
}

type Event struct {
    id string `json:"id"`
    event_type string `json:"event_type"`
    aggregate_id string `json:"aggregate_id"`
    aggregate_type string `json:"aggregate_type"`
    payload string `json:"payload"`
}

func (g *Gateway) AddUserToGroup(user *User, groupId uuid.UUID) error {
    err := g.validateUser(user)
    if err != nil {
        return err
    }

    err := g.validateGroupId(groupId)
    if err != nil {
        return err
    }

    userPayload, err := json.Marshal(user)
    if err != nil {
        return err
    }

    event := &Event{
        id: uuid.NewV4(),
        event_type: "UserAddedToGroup",
        aggregate_id: groupId.String(),
        aggregate_type: "group",
        payload: string(userPayload)
    }

    // append the event to the store
    err := g.appendOnlyStore.createEvent(user)
    if err != nil {
        return err
    }

    // send to pubsub
    go g.pubsub.PublishEvent(event)

    g.logger.info("event %+v was published", event)
}
```

Query Handler

```golang

func main() {
    groupHandler := NewGroupHandler(...dependencies)
    groupListener := NewGroupListener()
    groupListener.Subscribe("group.UserAddedToGroup", groupHandler.handler)
}

type Group struct {
    id string `json:"id"`
    name string `json:"name"`
    users []User `json:"users"`
}

func (g *groupHandler) handler(event string) {
    ev, err := json.Unmarshal(event)
    if err != nil {
        g.logger.error("failed to unmarshal event %s", event)
        return
    }

    // add in a normalized way for the read model to easily query
    err := g.queryStore.AddUserToGroup(ev.aggregate_id, ev.payload)
}

// later on when an API request will be made for group with users
func (g *groupHandler) GetGroup(groupId uuid.UUID) (*Group, error) {
    return g.queryStore.GetGroup(groupId)
}
```

### Pros

- We get audit out of the box for all operations
- We are asynchronous from the get go, which can help with scale issues down the line

### Cons

- The event store is difficult to query since it requires typical queries to reconstruct the state of the business entities, unless we ALSO save the data in a normalized way (CQRS), like in our group handler example. This increases operational costs as well as adds complexity to the developers.
- Unfamiliar programming style for most developers

## Auditing With Dedicated Micro-Service

Have a single service for auditing, which is responsible for managing all audit data. All other services can communicate with it either synchronously (HTTP) or asynchronously (Publish/Subscribe, Queues, gRPC).

```golang

type User struct {
    name string `json:"name"`
    id string `json:"id"`
}

type Audit struct {
    id string `json:"id"`
    category string `json:"category"`
    audit_type string `json:"audit_type"`
    description string `json:"description"`   
}

func (g *Groups) AddUserToGroup(user *User, groupId uuid.UUID) error {
    err := g.validateUser(user)
    if err != nil {
        return err
    }

    group, err := g.groups.GetGroup(groupId)
    if err != nil {
        return err
    }

    err := g.db.AddUserToGroup(user)
    if err != nil {
        return err
    }

    newAudit := &Audit{
        id: uuid.NewV4(),
        audit_type: "UserAddedToGroup",
        // can be user/system/group, depends on your needs
        category: "user",
        description: fmt.Sprintf("User %s was added to Group %s", user.Name, group.Name)
    }

    err := g.auditService.CreateAudit(newAudit)
    if err != nil {
        return err
    }

    // log user added to group for various purposes
    g.logger.info("user %s was added to group %s", user.Name, group.Name)
}
```

### Pros

- Easy to incorporate to an existing architecture.

### Cons

- New audits must be explicitly created with a call to audit service, unlike the implicit nature of Event Sourcing

## Summary

| Audit Type        | Complexity | Scfalablity  | Ease Of Integration With Existing Architecture |
|-------------------|------------|-------------|----------------------------------------|
| Logging           | Low        | Low         | High                                   |
| Database          | Low        | Medium      | Medium/High                                 |
| Event Sourcing    | High       | High        | Low                                    |
| Dedicated Service | Low        | Medium/High | Medium/High                            |
|                   |            |             |                                        |

Each of the implementations above has its pros and cons and you should always start with the simpler solution that can be implemented with as little effort as possible. If you are lucky enough to grow with your company to larger business needs and scale, you should consider the more scalable approaches, which are also more challenging. Regardless of what you choose, always try to write your code modular and consistent, like I explained in my previous posts, Ports and Adapters [4] and Clean Architecture [5].

## Bibliography

1. [event-sourcing](https://docs.microsoft.com/en-us/azure/architecture/patterns/event-sourcing)
2. [event-sourcing-microservices](https://microservices.io/patterns/data/event-sourcing.html)
3. [cqrs](https://martinfowler.com/bliki/CQRS.html)
4. [Ports and Adapters](/2020/02/01/Implementing-Ports-and-Adapters/)
5. [Clean Architecture](/2019/06/11/Implementing-Clean-Architecture/)