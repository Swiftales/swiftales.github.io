---
layout: post
title: Protocols and associatedtypes.
---

One of the most promising feature in `Swift` is `associatedtype`. Prior to `Swift 2.2`, the keyword used to define a placeholder type inside protocol was `typealias`. But this was very confusing as `typealias` in protocols means something else than in other places. 

Check out [this](https://github.com/apple/swift-evolution/blob/master/proposals/0011-replace-typealias-associated.md "0011") proposal in swift evolution for more details.

## What is an associatedtype? 
An associated type gives a placeholder name to a type that is used as part of the protocol. The actual type to use for that associated type is not specified until the protocol is adopted. Associated types are specified with the `associatedtype` keyword.

Lets see an example.

```swift
protocol AutoMobileServiceCenter {
    associatedtype VehicleType
    func wash(vehicle: VehicleType)
    func changeEngineOil(vehicle: VehicleType)
    }

//A struct representing GeneralMotors cars. 
struct GeneralMotors {}
//A struct representing Volkswagen cars.
struct Volkswagen {}
```
In above example protocol, we defined an associatedtype `VehicleType`. It means that its just a placeholder type and the actual type will be provided by the conforming entity. 


Lets extend our example to see how can we provide an actual type by conforming to `AutoMobileServiceCenter` protocol.

```swift
struct GeneralMotorsServiceCenter: AutoMobileServiceCenter {
    typealias VehicleType = GeneralMotors
    func wash(vehicle: VehicleType) {
        
    }
    func changeEngineOil(vehicle: VehicleType) {
    
    }
}
```

In above snippet, we created a `GeneralMotorsServiceCenter`, which conforms to `AutoMobileServiceCenter`. It has fulfilled the requirement of providing an actual type to assocaitedtype by declaring a `typealias` as `GeneralMotors`. 

The actual type can also be inferred from other functions of the protocol like below example.

```swift
struct VolkswagenServiceCenter: AutoMobileServiceCenter {
    func wash(vehicle: Volkswagen) {
        
    }
    func changeEngineOil(vehicle: Volkswagen) {
    
    }
}
```

In above example, the actual type of `VehicleType` can be inferred as `Volkswagen` from the parameters of `wash` and `changeEngineOil` functions. 

**Note:** If we return different types in both the functions, then complier will throw an error.

We can also tell an `associatedtype` to conform to a different protocol.

Like in above example, we wouldn't want someone to pass a Tree for servicing. 

Lets create another protocol to restrict users to pass only vehicles.

```swift
protocol Vehicle {}

protocol AutoMobileServiceCenter {
    associatedtype VehicleType: Vehicle
    func wash(vehicle: VehicleType)
    func changeEngineOil(vehicle: VehicleType)
}
```

This way, we are making sure that all service center should pass only vehciles for servicing. Compiler will complain if the actual type does not confrom to `Vehicle`. 

We need to modify our structs as below.

```swift
struct GeneralMotors: Vehicle {}
struct Volkswagen: Vehicle {}
```

This way we can restrict the type of an `associatedtype`. 

**Caveat:** Having a protocol constraint in `associatedtype` has one caveat. We can not fulfill `associatedtype` requirement with a some protocol when `associatedtype` has a protocol constraint.

Consider below example, which looks fine but does not compile.

```swift
protocol Vehicle {}

protocol AutoMobileServiceCenter {
    associatedtype VehicleType: Vehicle
    func wash(vehicle: VehicleType)
    func changeEngineOil(vehicle: VehicleType)
}

protocol Car: Vehicle {}

protocol Bike: Vehicle {}

struct GeneralMotors: Car {}

struct Volkswagen: Car {}

struct GenericCarSericeCenter: AutoMobileServiceCenter {
    func wash(vehicle: Car)
    func changeEngineOil(vehicle: Car)
}
```

Even though `Car` protocol conforms to `Vehicle`, the compilers throws an error 
```
Inferred type 'Car' (by matching requirement 'wash') is invalid: does not conform to 'Vehicle'
```

If we remove `Vehicle` constraint from `associatetype` in `AutoMobileServiceCenter`, then using `Car` as actual type compiles with no issues.

I have also filed a [radar](https://bugs.swift.org/browse/SR-1581) for this issue.

Lastly, we can also use **generics** to fulfill `associatedtype` requirement. We will now see how can we ultimately make a generic car serivce center. :)

```swift
protocol Vehicle {}

protocol AutoMobileServiceCenter {
    associatedtype VehicleType: Vehicle
    func wash(vehicle: VehicleType)
    func changeEngineOil(vehicle: VehicleType)
}

protocol Car: Vehicle {}

protocol Bike: Vehicle {}

struct GeneralMotors: Car {}

struct Volkswagen: Car {}

struct GenericCarSericeCenter<T>: AutoMobileServiceCenter where T: Car {
    func wash(vehicle: T)
    func changeEngineOil(vehicle: T)
}
```


Hope we now have good understanding of `associatedtype`. Many people, including myself, find it bit confusing but its a very powerful technique in protocol oreinted design.


In next blog, we will try to build UITableView's generic datasource which can be re-used for different dataset using the concept of `associatedtype`.

Happy Swifting.🃏



