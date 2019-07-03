---
title: Implementing Clean Architecture
category: Architecture
date: 2019-06-11 21:31:56
tags: 
  - Node.js
  - Typescript 
  - Software Architecture
  - Clean Architecture
  - Uncle Bob
---
Last year, I had the opportunity to design a new project at work and since I had just finished reading Robert Martin's *Clean Architecture* I thought to myself, why not implement it on that project ? One of the reasons, except my usual *I have to implement this cool thing right away!* was that working on legacy systems in the company was accompanied with the good ol' *big ball of mud* code. The purpose of this post is to show you how can one implement *Clean Architecture* in practice and still understand it years from now, whether you work alone or in a team. Everything shown will be written in *Typescript* on *Node.js* using *Object Oriented* programming paradigm. All of the code will be available [here](https://github.com/yuraxdrumz/clean-architecture-example)

# Disclaimer
Some of the things that I am going to write and show are my personal experiences and opinions, you may have read Robert Martin's *Clean Architecture* and thought, interpreted or implemented otherwise. All the architectures have the same goals in the end. 

## Core Idea
The idea behind *Clean Architecture* is that we have layers. Each layer is encapsulated by a higher level layer and the only way to communicate between the layers is with *The Dependency Rule*. 


## The Dependency Rule
 states that source code dependencies can only point inwards, meaning each layer can be dependant on the layer beneath it, but never the other way around. 
## Entities and Use-Cases
The core of this architecture are your entities, which represent your classes/types/interfaces/basic methods.
## Use-Cases
A layer above the entities layer is your use-cases. Use-cases are your application specific business rules, for example, if we are talking about a shopping cart, then `addToCart` will be a use case, because it needs to recieve a type `product` and, for examples sake, check warehouse for availability and then insert new data to a DB and return response. Do not couple your use-cases to some input or output, instead pass a contract (interface) of some type in the constructor and pass the implementation itself at higher layers.
## Repository Pattern
For database interactions it is recommended to use the *Repository Pattern* which encapsulates all your database interactions through an abstraction layer. The repository pattern does give you a bit freedom to replace databases with ease, but this rule only applies when your interactions are basic CRUD operations! If you have many to many relationships which require a graph database, switching to mongodb at the repository layer will not help you much as it is not built for that purpose, so take interactions into consideration at design level! 
## Interface Adapters
After the use-cases layer we have the Interface Adapters layer. Here, you convert your data from the form most convenient for entities and use cases, into the form most convenient for whatever persistence framework is being used, like the database, web or whatever you like. I like to call it, the implementations layer.

## Frameworks and Drivers
The last layer is the Frameworks and Drivers. Here you call all of your dependencies that abide the contracts you defined in your use-cases. That way you can replace dependencies without the use-cases knowing anything about it, according to the L in S.O.L.I.D, which is called the Liskov substitution principle.

## Liskov Substitution Principle
Liskov's substitution principle states that if a system is using a type **T** which is an implementation of type **S** and we switch the implementation to type **Z** which is also of type **S** , the behaviour of the program should not change.

A small diagram to illustrate our layers, notice the arrows only pointing inward!
Taken from:
[https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)

![](./Implementing-Clean-Arhitecture/CleanArchitecture.jpg)
 

## Clean Architecture In Practice
lets build something overused, like a shopping cart. We will first decide what are our use cases and from that we would be able to conclude an initial data model - our entities. Later on, we will create Interface Adapters(implementations) and at the final layer we will simply glue all of our dependencies and implementations and see how clean architecture could benefit us in future projects. Last but not least, we will show how easy it is to switch implementations from a web server to a command line interface.

## What are our use-Cases
Because we chose a shopping cart, our use-cases will be pretty straight forward - `addToCart` and `removeFromCart`. Lets say `addToCart` needs to check our warehouse which is an external service, afterwards it will need to insert to our DB. `removeFromCart` will update warehouse and afterwards delete from our DB. After deciding on our business rules (use-cases), we can create an inital data model.


### Defining the contracts (interfaces)


`src\entities\interfaces\addToCart.ts`
```typescript
import AddedToCart from '../types/AddedToCart'
import Product from '../types/Product'
export default interface IAddToCart {
  add(item: Product): Promise<AddedToCart>
}
```
`src\entities\interfaces\removeFromCart.ts`
```typescript
import RemovedFromCart from '../types/RemovedFromCart'
import Product from '../types/Product'
export default interface IRemoveFromCart {
  remove(item: Product): Promise<RemovedFromCart>
}
```
`src\entities\interfaces\cartRepository.ts`
```typescript
import AddedToCart from '../types/AddedToCart'
import RemovedFromCart from '../types/RemovedFromCart'
import Product from '../types/Product'

export default interface ICartRepository {
  add(item: Product): Promise<AddedToCart>
  remove(item: Product): Promise<RemovedFromCart>
}
```
`src\entities\interfaces\warehouse.ts`
```typescript
import Product from '../types/Product'
import ItemInWareHouse from '../types/ItemInWareHouse'

export default interface IWarehouse {
  checkItemInWarehouse(item: Product): Promise<ItemInWareHouse>
  returnItemToWarehouse(item: Product): Promise<ItemInWareHouse>
}
```
### Defining the types

`src\entities\types\AddedToCart.ts`
```typescript
type AddedToCart = boolean
export default AddedToCart
```

`src\entities\types\RemovedFromCart.ts`
```typescript
type RemovedFromCart = boolean
export default RemovedFromCart
```
`src\entities\types\Product.ts`
```typescript
type Product = {}
export default Product
```
`src\entities\types\ItemInWareHouse.ts`
```typescript
type ItemInWareHouse = boolean
export default ItemInWareHouse
```

### Defining the use-cases

Note how we expect to receive implementations of the warehouse and cartRepository interfaces in the code snippet below. The implementations themselves could be anything as long as they implement our interfaces.
The implementations will be passed in the constructor as well as other 3rd party dependencies/modules. If we used/imported the dependencies/implementations directly, we would not adhere to the `Clean Architecture` dependency rule (arrows we saw in the diagram above) as we would create a dependency of a higher level module to a lower level detail, which is against the `inwards dependency`, also called `dependency inversion`. 

`src\use-cases\addToCart.ts`
```typescript
import AddedToCart from '../entities/types/AddedToCart'
import Product from '../entities/types/Product'

import IWarehouse from '../entities/interfaces/warehouse'
import ICartRepository from '../entities/interfaces/cartRepository'
import IAddToCart from '../entities/interfaces/addToCart'

abstract class AddToCart implements IAddToCart {
  protected cartRepository: ICartRepository
  protected warehouseService: IWarehouse
  constructor(cartRepository: ICartRepository, warehouseService: IWarehouse){
    this.cartRepository = cartRepository
    this.warehouseService = warehouseService
  }

  async add(item: Product): Promise<AddedToCart> {
    const isItemInWarehouse = await this.warehouseService.checkItemInWarehouse(item)
    if(isItemInWarehouse){
      await this.cartRepository.add(item)
      return true
    }
    return false
  }

}

export default AddToCart
```

`src\use-cases\removeFromCart.ts`
```typescript
import RemovedFromCart from '../entities/types/RemovedFromCart'
import Product from '../entities/types/Product'


import IRemoveFromCart from '../entities/interfaces/removeFromCart'
import IWarehouse from '../entities/interfaces/warehouse'
import ICartRepository from '../entities/interfaces/cartRepository'

abstract class RemoveFromCart implements IRemoveFromCart {
  protected cartRepository: ICartRepository
  protected warehouseService: IWarehouse
  constructor(cartRepository, warehouseService){
    this.cartRepository = cartRepository
    this.warehouseService = warehouseService
  }

  async remove(item: Product): Promise<RemovedFromCart> {
    await this.cartRepository.remove(item)
    const isItemReturned = await this.warehouseService.returnItemToWarehouse(item)

    if(isItemReturned){
      await this.cartRepository.remove(item)
      return true
    }
    return false
  }

}

export default RemoveFromCart
```

### Defining the implementations
Now lets first create implementations of add / remove / cartRepository and warehouse for a web server.

`src\implementations\addToCart\web.ts`

```typescript
import AddToCart from '../../use-cases/addToCart'

class ConcreteAddToCart extends AddToCart {
  async receiveFileFromWeb(request, response){
    if(request && request.body && request.body["item"]){
      const isAdded = await this.add(request.body["item"]["item"])
      response.json(isAdded)
    } else {
      throw new Error("body is missing required field item")
    }
  }
}
export default ConcreteAddToCart
```

`src\implementations\addToCart\web.ts`
```typescript
import RemoveFromCart from '../../use-cases/removeFromCart'

class ConcreteRemoveFromCart extends RemoveFromCart {
  async removeFileFromWeb(request, response){
    if(request && request.body && request.body["item"]){
      const isRemoved = await this.remove(request.body["item"]["item"])
      response.json(isRemoved)
    } else {
      throw new Error("body is missing required field item")
    }
  }
}

export default ConcreteRemoveFromCart
```

I added a console implementation of cartRepository, which simply logs to stdout.

`src\implementations\cartRepository\console.ts`
```typescript
import ICartRepository from '../../entities/interfaces/cartRepository'
import Product from '../../entities/types/Product'
import AddedToCart from '../../entities/types/AddedToCart'

class ConcreteCartRepository implements ICartRepository {
  async add(item: Product): Promise<AddedToCart> {
    console.log('adding item to database')
    return true
  }
  async remove(item: Product): Promise<AddedToCart> {
    console.log('removing item to database')
    return false
  }
}

export default ConcreteCartRepository
```
I added a console implementation of warehouse, which simply logs to stdout.

`src\implementations\warehouse\console.ts`
```typescript
import IWarehouse from '../../entities/interfaces/warehouse'
import Product from '../../entities/types/Product'
import AddedToCart from '../../entities/types/AddedToCart'

class ConcreteWarehouse implements IWarehouse {
  async checkItemInWarehouse(item: Product): Promise<AddedToCart> {
    console.log('adding item to warehouse')
    return true
  }
  async returnItemToWarehouse(item: Product): Promise<AddedToCart> {
    console.log('returning item to warehouse')
    return false
  }
}

export default ConcreteWarehouse
```
We chose a web implementation for `addToCart` and `removeFromCart` and a console implementation for `warehouse` and `cartRepository`.
We wrapped each use-case with a handler which will be part of a web server, in other words, we prepared the data in this layer for the next layer to use, which is the `frameworks and drivers` layer.

### Defining the frameworks and drivers


Now, the last glue layer looks like this:
We could make it prettier, but I will leave that to you, after we learn this cool new architecture!

`src\frameworks-drivers\web.ts`
```typescript
import express from 'express'
import bodyParser from 'body-parser'

import CartRepositoryImpl from '../implementations/cartRepository'
import WarehouseImpl from '../implementations/warehouse'
import AddToCartWebImpl from '../implementations/addToCartWeb'
import RemoveFromCartWebImpl from '../implementations/removeFileWeb'

const app = express()
const cartRepo = new CartRepositoryImpl()
const warehouse = new WarehouseImpl()
const addToCartInstance = new AddToCartWebImpl(cartRepo, warehouse)
const removeFromCartInstance = new RemoveFromCartWebImpl(cartRepo, warehouse)

app.use(bodyParser.json())

app.post('/item', async (req,res,next)=>{
  try{
    await addToCartInstance.receiveFileFromWeb(req, res)
  }catch(e){
    next(e)
  }
})

app.delete('/item', async (req,res,next)=>{
  try{
    await removeFromCartInstance.removeFileFromWeb(req, res)
  }catch(e){
    next(e)
  }
})

app.listen(process.env.PORT, ()=>{
  console.log(`listening on port ${process.env.PORT}`)
})
```

We initiated all dependencies, created all instances and passed everything along, if all interfaces are adhered, the code will compile and we can run `web.ts`. Try and run this example and send a POST to `/item` and a DELETE to `/item`, you will see our warehouse and database console implementations writing to stdout like we planned to.

## Creating another implementation
Now, lets create a command line interface implementation for `addToCart` and `removeFromCart`. We will leave the `warehouse` and `cartRepository` as is but you can play with them as you wish.

lets create a new `cli.ts` under `implementations\addToCart`
`src\implementations\addToCart\cli.ts`
```typescript
import AddToCart from '../../use-cases/addToCart'

class ConcreteAddToCart extends AddToCart {
  async receiveFileFromCli(product){
    if(product){
      const isAdded = await this.add(product)
      console.log(`isAdded: ${isAdded}`)
    } else {
      console.error("commandLineOptions require product to be passed")
    }
  }
}
export default ConcreteAddToCart
```

`src\implementations\removeFromCart\cli.ts`
```typescript
import RemoveFromCart from '../../use-cases/removeFromCart'

class ConcreteRemoveFromCart extends RemoveFromCart {
  async removeFileFromCli(product){
    if(product){
      const isRemoved = await this.remove(product)
      console.log(`isRemoved: ${isRemoved}`)
    } else {
      console.error("commandLineOptions require product to be passed")
    }
  }
}
export default ConcreteRemoveFromCart
```

Now, all we have left to do is simply create a new `cli.ts` under `frameworks-drivers` and we have a different implementation altogether.

`src\frameworks-drivers\cli.ts`
```typescript
import program from 'commander'

import CartRepositoryImpl from '../implementations/cartRepository/console'
import WarehouseImpl from '../implementations/warehouse/console'
import AddToCartCliImpl from '../implementations/addToCart/cli'
import RemoveFromCartCliImpl from '../implementations/removeFromCart/cli'

program
  .option('-p, --product <name>', 'product name')
  .option('-a, --add', 'action')
  .option('-r, --remove', 'action')

program.parse(process.argv);

/**
 * to run this from typescript first run npm run dist and then node dist\index.js -p "test product name" -a or node dist\index.js -p "test product name" -r
 */

(async ()=>{
  const cartRepo = new CartRepositoryImpl()
  const warehouse = new WarehouseImpl()
  const addToCartInstance = new AddToCartCliImpl(cartRepo, warehouse)
  const removeFromCartInstance = new RemoveFromCartCliImpl(cartRepo, warehouse)

  if(!program.product){
    throw new Error("-p is required")
  }
  if(program.add){
    await addToCartInstance.receiveFileFromCli(program.product)
  } else if (program.remove){
    await removeFromCartInstance.removeFileFromCli(program.product)
  } else {
    throw new Error("-a or -r are required")
  }

})()
```

## Summary
We first looked at what *Clean Architecture* is, as defined by Robert Martin, afterwards we saw what each layer does and provided detailed examples. At last, we created a new implementation without touching the core business rules (addToCart, removeFromCart).
This kind of architecture encourages seperation of concerns out of the box, gives a guideline on how to structure your code, defines strict interactions between layers and allows rapid changes regardless of the size of your codebase. Implementing this in production has taught me a lot about how to build better software and expanded my toolbox.