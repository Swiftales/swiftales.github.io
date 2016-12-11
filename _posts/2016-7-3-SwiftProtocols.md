---
layout: post
title: Swift protocols with 'get set' and 'get' properties.
---

`Protocols` in swift defines a blueprint of methods, properties, and other requirements that suit a particular task or piece of functionality. The protocol can then be adopted by a class, structure, or enumeration to provide an actual implementation of those requirements. Any type that satisfies the requirements of a protocol is said to conform to that protocol.

Lets get started with some fictitous example code.

This is how we can declare and conform to a simple protocol in Swift.

```swift
protocol VoiceAssistant {
    
}

struct Siri: VoiceAssistant {
    
}

```

## Properties in protocols

A protocol is set of rules. Any conforming type has to follow all the rules. As a part of rules, a protocol can require any conforming type to provide a property with a particular name and type. The conforming type can fulfill this requirement either by using a stored property or a computed property. To give some clarity, conforming type means the types (structs, classes, enums) which are conforming to a protocol. In the above code sample, `VoiceAssistant` is a protocol and `Siri` is the conforming type. Lets add some property requirement in `VoiceAssistant`.

**NOTE: Property requirements in a protocol are always declared as variable properties because they are declared as computed properties.**

```swift
protocol VoiceAssistant {
    var name: String {get}
}

```
There are different ways by which the conforming types can fulfill the property requirement of a protocol.

```swift
struct Siri: VoiceAssistant {
    let name = "Siri"
}

struct Alexa: VoiceAssistant {
    var name = "Alexa"
}

struct Cortana: VoiceAssistant {
    let assistantName = "Cortana"
    
    var name: String {
        get {
            return assistantName
        }
    }
}

```

Notice that even though inside protocols, properties are declared with var keyword only, we can use `let` for fulfilling `get` property requiremnet inside the conforming types. The only requirement of `VoiceAssistant` is to provide a name, and all conforming types are free to provide it using a `let` or a `var`. 

Below is an example of get set property requirement in a protocol.

```swift
protocol VoiceAssistant {
    var name: String {get}
    var voice: String {get set}
}

// Fulfilling get set propety requirement:
struct Siri: VoiceAssistant {
    let name = "Siri"
    var voice = "Vicky" 
}

struct Cortana: VoiceAssistant {
    let assistantName = "Cortana"
    var assistantVoice = "Vicky"
    var name: String {
        get {
            return assistantName
        }
    }

    var voice: String {
        get {
            return assistantVoice
        }
        set {
            assistantVoice = newValue
        }
    }
}

```

If a protocol requires a property to be gettable and settable, that property requirement **cannot** be fulfilled by a constant (let) stored property or a read-only computed property. 

```swift
protocol VoiceAssistant {
    var name: String {get}
    var voice: String {get set}
}

struct Siri: VoiceAssistant {
    let name = "Siri"
    let voice = "Voice" // Compilation Error: Candidate is not settable, but protocol requires it.
}

```

## How *get* properties are different from *get set* properties in a protocol?

In this section, I will try to explain the difference between get propeties and get set properties.

A get property does not mean that the conforming type has to declare that propety as get only. Take below example:


```swift
protocol VoiceAssistant {
    var name: String {get}
    var voice: String {get set}
    var version: String {get}
}

struct Siri: VoiceAssistant {
    // Conforming to `VoiceAssistant`
    let name = "Siri" 
    var voice = "Vicky"
    var version = "1.0" 
}

var myVoiceAssistant = Siri()
myVoiceAssistant.voice = "Samantha"
myVoiceAssistant.version = "2.0"

```

As we can see, even though `version` is a get property requirement in the protocol, we are stil able to perform both get and set operations on `version` of `myVoiceAssistant`. 

![Image alt](/assets/posts/Swift_Protocol_Get_Set/984.jpg "confused")

## Explanation
By making `version` as get property in `VoiceAssistant`, we are asking the conforming types to provide a getter method with name `version`. `VoiceAssistant` is not stopping the conforming types to set the property `version`. 

The reason we are able to set `version` of `myVoiceAssistant` is that since we have not explicity specified the type of `myVoiceAssistant`, its type is being inferred as `Siri`. So we are actually setting `version` of `Siri` and not of `VoiceAssistant`. `version` is decalred as `var` in `Siri`, thats why we can set `version` in `myVoiceAssistant`.

If we explicitly set the type of `myVoiceAssistant` to be `VoiceAssistant`, then compiler wont allow us to set the `version`, as `VoiceAssistant` only guarantees that a getter with `version` exists and has no knowledge about the setter. Its totally upto the conforming types to provide a setter or not for a get-only property requirement of a protocol.  


```swift
protocol VoiceAssistant {
    var name: String {get}
    var voice: String {get set}
    var version: String {get}
}


struct Siri: VoiceAssistant {
    let name = "Siri" // Conforming to `VoiceAssistant`
    var voice = "Vicky"
    var version = "1.0" 
}

var myVoiceAssistant: VoiceAssistant = Siri()
myVoiceAssistant.voice = "Samantha"
myVoiceAssistant.version = "2.0" // Compilation Error: cannot assign to property: 'version' is a get-only property

```

I was always confused in the beginning that if we can set the properties which are marked as get in protocol, then how is it different from get set property. Later I realised that we are actualy using setter given by the conforming types. And I find this behaviour very useful in encapsulation. If you have also had this confusion, then I hope that this post might have made your head feel little lighter by now.

Happy Swifting.🃏