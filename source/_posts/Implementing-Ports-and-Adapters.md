---
title: Implementing Ports and Adapters
tags:
  - Ports and Adapters
  - Golang
  - Hexagonal
  - Software Architecture
category: Architecture
date: 2020-02-01 18:15:21
---


There are different architectures that allow you to keep focus on your business domain and allow for fast paced development and changes. Examples would be: Clean Architecture, Onion Architecture and Ports and Adapters (also called hexagonal).
[In my previous post](/2019/06/11/Implementing-Clean-Architecture/), I talked about Clean Architecture and how it helps get your code more modular and developer friendly after a somewhat short learning curve. After joining a new team, I noticed Clean Architecture did not really settle and the team found it to be somewhat over abstracted, so I decided to play around with a variation of Ports and Adapters.
In this post I will show what is Ports and Adapters and how I implemented it in Golang. Github repo is available [here](https://github.com/yuraxdrumz/ports-and-adapters-golang).

### Disclaimer
Some of the things that I am going to write and show are my personal experiences and opinions.

### Core Idea
Ports and Adapters architecture divides a system into several loosely-coupled interchangeable components. The components communicate with each other through abstracted API's with the use of interfaces and their implementations.
This approach is an alternative to the traditional layered architecture, where components are divided into layers. There are no layers in Ports and Adapters, only ports <- adapters, meaning there are no restrictions on how to structure the applications, only that an adapter relies on a port and the business logic port relies on other ports

### Business Logic
All of your business specific use cases.
Example: upon adding a user to a board, send an email, save the user in the database and grant the user permissions to view the board from an auth service.


### Ports
The interfaces to all of the components in your system. There are two kinds of ports: driven and driver.

#### Driven Ports
Interfaces that your application business logic uses for its needs. 

#### Driver Ports
Interfaces for the outside world, a.k.a API

### Adapters
Implementations of our ports. They can be either driven or driver, depending on the port we use

#### Driven Adapter
Example: Service-To-Service adapter, for when we need to request some data from another service in our business logic use case

#### Driver Adapter
Example: GUI adapter, for when we need to convert events triggered by a GUI app to events defined by other ports

### Prerequisites
Because Ports and Adapters does not define a specific folder structure, I will not focus on it, but you can take a reference from the structure in the github repository.
One thing to note is I replaced the `driven` ports with the name `out` and the driver ports with the name `in`, as it confused developers that are new to Ports and Adapters
I like to name my packages as `package/ports`, and the adapters, like `package/console.go`, `package/web.go`, meaning there are two adapters, one is console and one is web for a specific port, that way it is easier to know what is implemented where.
Regarding the structs, I prefer putting them in each package as a sub package, in order to avoid cyclic import dependencies. `package/ports/...`

### Ports and Adapters In Practice
lets build a shopping cart. Our cart will have two use cases, `addToCart` and `removeFromCart`. Let's say that we need a warehouse service and a database to save the changes.

We can create one port for all the use cases regarding the cart, or we can do a port per use case in the cart. It is up to you and the level of abstraction you seek.
I chose to put all the cart use cases under one port.

#### Ports In Practice

##### Driven ports

Our warehouse package ports will look like this:
`warehouse/ports.go`

```go
package warehouse

type Port interface {
	CheckIfAvailable(itemID string) (bool, error)
	RemoveItemFromWarehouse(itemID string) (bool, error)
}

```

Our cart repository package ports will look like this:
`cartRepository/ports.go`

```go
package cartrepository

import (
	"github.com/yuraxdrumz/ports-and-adapters-golang/internal/app/cart/structs"
)

type Port interface {
	AddItemToDB(item *structs.Item) (string, error)
	RemoveItemFromDB(itemID string) (bool, error)
}
```

I like to use the repository design pattern to abstract database interaction. 
I already wrote about it [here](/2019/06/11/Implementing-Clean-Architecture/#Repository-Pattern).


Let's see an example of the port for the cart.
`cart/ports.go`
```go
package cart

import "github.com/yuraxdrumz/ports-and-adapters-golang/internal/app/cart/structs"

type Port interface {
	Add(item structs.Item) error
	Remove(itemID string) error
}


```

##### Driver Ports
`driver/ports.go`
```go
package driver

// Port - how an adapter can use the app
type Port interface {
	Run()
}

```

Now that we finished the ports part, lets look at the adapters.
We will first create simple adapters, like in memory repository adapter and a console warehouse adapter so that we can start using the app.
Afterwards, we will create an sql adapter to show how Ports and Adapters come to our advantage.

#### Adapters In Practice
##### Driven Adapters

`warehouse/console.go`
```go
package warehouse

import (
	"errors"
)

// our console warehouse struct with all the adapters it uses, in this case with the console adapter we dont need any dependencies
type ConsoleWarehouse struct {}

// a new console warehouse factory
func NewConsoleWarehouse() *ConsoleWarehouse {
	return &ConsoleWarehouse{}
}

// our implementation of the port for console warehouse
func (w *ConsoleWarehouse) CheckIfAvailable(itemID string) (bool, error) {
	if itemID == "0" {
		return false, errors.New("item is not available")
	}
	return true, nil
}

// our implementation of the port for console warehouse
func (w *ConsoleWarehouse) RemoveItemFromWarehouse(itemID string) (bool, error) {
	if itemID == "0" {
		return false, errors.New("item cannot be removed from warehouse, please check again later")
	}
	return true, nil
}
```

`cartRepository/inMemory.go`
```go
package cartrepository

import (
	"errors"
	"github.com/yuraxdrumz/ports-and-adapters-golang/internal/app/cart/structs"
)

// in memory repository adapter, we dont have any dependencies on other adapters so its empty
type InMemoryRepository struct {}

// new in memory repository factory
func NewInMemoryRepository() *InMemoryRepository {
	return &InMemoryRepository{}
}

// our implementation of the port
func (r *InMemoryRepository) AddItemToDB(item *structs.Item) (bool, error) {
	if item.Id == "0" {
		return false, errors.New("cannot add item to db")
	}
	return true, nil
}

// our implementation of the port
func (r *InMemoryRepository) RemoveItemFromDB(itemID string) (bool, error) {
	if itemID == "0" {
		return false, errors.New("item cannot be removed, please check again later")
	}
	return true, nil
}
```

Lets see the cart adapter with Add implemented

`cart/cart.go`

```go

package cart

import (
	"github.com/yuraxdrumz/ports-and-adapters-golang/internal/app/cart/structs"
	"github.com/yuraxdrumz/ports-and-adapters-golang/internal/pkg/adapters/out/cartRepository"
	"github.com/yuraxdrumz/ports-and-adapters-golang/internal/pkg/adapters/out/warehouse"
	"sync"
)

type Cart struct {
	wh   warehouse.Port
	repo cartrepository.Port
	mutex sync.Mutex
}

func NewCart(wh warehouse.Port, repo cartrepository.Port) *Cart {
	return &Cart{
		wh:   wh,
		repo: repo,
	}
}

func (c *Cart) Add(item *structs.Item) error {
	c.mutex.Lock()
	defer c.mutex.Unlock()
	
	isAvailable, err := c.wh.CheckIfAvailable(item.Id)
	if err != nil {
		return err
	}
	if isAvailable {
		_, err = c.repo.AddItemToDB(item)
		if err != nil {
			return err
		}
	}
	return nil
}

func (c *Cart) Remove(itemID string) error {
	return nil
}
```

At this point we have our cart, warehouse and repository ports and adapters implemented, all these ports are driven, because our app uses them internally.
Now, we need a driver adapter.

Let's create a cli adapter which receives the item, id and description from the cli.

##### Driver Adapter
`driver/cli.go`

```go
package driver

import (
	"github.com/sirupsen/logrus"
	"github.com/yuraxdrumz/ports-and-adapters-golang/internal/app/cart"
	"github.com/urfave/cli/v2"
	"github.com/yuraxdrumz/ports-and-adapters-golang/internal/app/cart/structs"
	"os"
)

// CliAdapter - struct with necessary ports to run
type CliAdapter struct {
	ca cart.Port
}

// NewCliAdapter - create a new instance of NewCliAdapter with passed implementations
func NewCliAdapter(ca cart.Port) *CliAdapter {
	return &CliAdapter{ca: ca}
}

// Run - initializes cli adapter run
func (in *CliAdapter) Run() {
	app := &cli.App{
		Name: "cart",
		Usage: "handle cart from cli",
		Flags: []cli.Flag{
			&cli.StringFlag{
				Name: "item",
				Usage: "item to add",
				Required: true,
			},
			&cli.StringFlag{
				Name: "description",
				Usage: "description for the item",
				Required: true,
			},
			&cli.StringFlag{
				Name: "id",
				Usage: "item id",
				Required: true,
			},
		},
		Action: func(c *cli.Context) error {

			item := &structs.Item{
				Name:        c.String("item"),
				Id:          c.String("id"),
				Description: c.String("description"),
			}
			err := in.ca.Add(item)
			if err != nil {
				logrus.WithField("error", err.Error()).Error("couldn't add item")
				return nil
			}
			logrus.WithFields(logrus.Fields{
				"name": item.Name,
				"description": item.Description,
				"id": item.Id,
			}).Info("Added new item")
			return nil
		},
	}
	err := app.Run(os.Args)

	if err != nil {
		logrus.Fatal(err)
	}
}

```

Now lets glue everything together in `main.go`

`main.go`

```go
package main

import (
	"github.com/yuraxdrumz/ports-and-adapters-golang/internal/app/cart"
	driver "github.com/yuraxdrumz/ports-and-adapters-golang/internal/pkg/adapters/in"
	"github.com/yuraxdrumz/ports-and-adapters-golang/internal/pkg/adapters/out/warehouse"
	"github.com/yuraxdrumz/ports-and-adapters-golang/internal/pkg/adapters/out/cartRepository"
)

func main() {
	// declare all ports
	var ca cart.Port
	var wh warehouse.Port
	var re cartrepository.Port
	var in driver.Port

	wh = warehouse.NewConsoleWarehouse()
	re = cartrepository.NewInMemoryRepository()
	ca = cart.NewCart(wh, re)
	in = driver.NewCliAdapter(ca)

	in.Run()
}

```

Now, if we run `go run main.go`, we will get

```
❯ go run main.go
NAME:
   cart - handle cart from cli

USAGE:
   main [global options] command [command options] [arguments...]

COMMANDS:
   help, h  Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --item value         item to add
   --description value  description for the item
   --id value           item id
   --help, -h           show help (default: false)
FATA[0000] Required flags "item, description, id" not set 
exit status 1


```

Lets add an item with id 0 that is supposed to fail and with an id of 1 that is supposed to succeed

```
❯ go run main.go --item bicycle --description "used as trasnport" --id 1
INFO[0000] Added new item                                description="used as trasnport" id=1 name=bicycle
❯ go run main.go --item bicycle --description "used as trasnport" --id 0
ERRO[0000] couldn't add item                             error="item is not available"

```

It worked!

Now, let's create a new sql adapter for the repository

`cartRepository/sqlite.go`
```go
package cartrepository

import (
	"database/sql"
	"errors"
	"github.com/sirupsen/logrus"
	"github.com/yuraxdrumz/ports-and-adapters-golang/internal/app/cart/structs"
	_ "github.com/mattn/go-sqlite3"
)

// in memory repository adapter, we dont have any dependencies on other adapters so its empty
type SQLiteRepository struct {
	db *sql.DB
}

// new in memory repository factory
func NewSQLiteRepository() *SQLiteRepository {
	db, err := sql.Open("sqlite3", "database/items.db")
	if err != nil {
		logrus.Fatal("Couldn't initialize sqlite db")
	}
	_, err = db.Exec("create table if not exists items (id string, name text, description text)")
	if err != nil {
		logrus.WithField("error", err.Error()).Fatal("Couldn't create initial items table")
	}
	return &SQLiteRepository{
		db: db,
	}
}

// our implementation of the port
func (r *SQLiteRepository) AddItemToDB(item *structs.Item) (string, error) {
	if item.Id == "0" {
		return "", errors.New("cannot add item to db")
	}
	tx, _ := r.db.Begin()
	stmt, _ := tx.Prepare("insert into items (id, name, description) values (?,?,?)")
	_, err := stmt.Exec(item.Id, item.Name, item.Description)
	if err != nil {
		return "", err
	}
	err = tx.Commit()
	if err != nil {
		return "", err
	}
	return "random id", nil
}

// our implementation of the port
func (r *SQLiteRepository) RemoveItemFromDB(itemID string) (bool, error) {
	if itemID == "0" {
		return false, errors.New("item cannot be removed, please check again later")
	}
	return true, nil
}
```

Lets change our main.go

```go
package main

import (
	"github.com/yuraxdrumz/ports-and-adapters-golang/internal/app/cart"
	driver "github.com/yuraxdrumz/ports-and-adapters-golang/internal/pkg/adapters/in"
	"github.com/yuraxdrumz/ports-and-adapters-golang/internal/pkg/adapters/out/warehouse"
	"github.com/yuraxdrumz/ports-and-adapters-golang/internal/pkg/adapters/out/cartRepository"
)

func main() {
	// declare all ports
	var ca cart.Port
	var wh warehouse.Port
	var re cartrepository.Port
	var in driver.Port

	wh = warehouse.NewConsoleWarehouse()
	//re = cartrepository.NewInMemoryRepository()
	re = cartrepository.NewSQLiteRepository()
	ca = cart.NewCart(wh, re)
	in = driver.NewCliAdapter(ca)

	in.Run()
}

```

Lets run our main again

```
❯ go run main.go --item bicycle --description "used as trasnport" --id 1
INFO[0000] Added new item                                description="used as trasnport" id=1 name=bicycle
❯ go run main.go --item bicycle --description "used as trasnport" --id 0
ERRO[0000] couldn't add item                             error="item is not available"
❯ go run main.go --item bicycle --description "used as trasnport" --id 11
INFO[0000] Added new item                                description="used as trasnport" id=11 name=bicycle

```

It worked and all the data was saved.
By now you should be able to add a new adapter to any port, either driver or driven and interchange them without touching other parts of the system.

A cool thing in Goland is we can easily see who implements our ports and reuse them by calling the adapter we want in `main.go`

![](./goland.png)

### Summary
We saw what is the Ports and Adapters pattern and how it is implemented in Golang.
We created an initial warehouse, cart, cartRepository and driver(in) ports.
Later on, we added each port's adapter(implementation).
Afterwards, we glued everything in main.go.
By Using this architectural pattern we saw how components can interchange on any functionality.


