---
layout: post
title: Swift type inference and UIColor convenience initialiser.
---

The Swift language and its type system incorporate a number of popular language features, including object-oriented programming via classes, function and operator overloading, subtyping, and constrained parametric polymorphism. Swift makes extensive use of type inference, allowing one to omit the types of many variables and expressions.


```swift
let count = 3 // type is inferred as `Int`
let name = "Vishal" // type is inferred as `String`

```

Type can also be inferred by the return type of the a function.


```swift

func countOfApples() -> Int {
	return 10
}

let count = countOfApples() // type is inferred from the return type of `countOfApples` as `Int`

```

## Struggle with UIColor

Lets say, we have an Objective C project. Our designer gave us r, g, b values for changing background to dark and light depending on user's preference.

```
Dark RGB: (0, 0, 0)

Light RGB: (254, 248, 220)

```
Lets go ahead and implement dark background.

```swift

- (void)setDarkBackground {
    [self.view setBackgroundColor: [UIColor colorWithRed:0/255 green:0/255 blue:0/255 alpha:1]];
}

```

![Image alt](/assets/posts/Type_Inference_UIColor/dark.png "Dark")

Seems perfect, even though there is a mistake in the implementation. We will ignore it as everything is working fine as of now.

![Image alt](/assets/posts/Type_Inference_UIColor/thumbsup.png "Thumbs Up")


Now lets implement light background **quickly**.

```swift

- (void)setLightBackground {
    [self.view setBackgroundColor: [UIColor colorWithRed:254/255 green:248/255 blue:220/255 alpha:1]];
}

```
Lets see how our light background looks.

![Image alt](/assets/posts/Type_Inference_UIColor/dark.png "Dark")

Where is the light? 

![Image alt](/assets/posts/Type_Inference_UIColor/wtf.png "WTF")


## Debug
```swift
(UIColor *)colorWithRed:(CGFloat)red
                    green:(CGFloat)green
                     blue:(CGFloat)blue
                    alpha:(CGFloat)alpha
```
The method expects `CGFloat`. Since, we **quickly** implemented light background, we forgot to add decimal in the r, g, b values. `Objective C` can not infer that the result of the expression should be `CGFloat`. The expression `254/255` returns result as `NSInteger` which is `0` because in almost all languages, result of integer division is an integer. The method in `setLightBackground`, therefore, is computed as:

```swift

- (void)setLightBackground {
    [self.view setBackgroundColor: [UIColor colorWithRed:0 green:0 blue:0 alpha:1]]; //this is actually a black color.
}

```
To make sure it works correctly, we have to make sure that we add a decimal in the division expression so that it performs floating point division and returns correct value.

```swift

- (void)setLightBackground {
    [self.view setBackgroundColor: [UIColor colorWithRed:254/255.0 green:248/255.0 blue:220/255.0 alpha:1]];
}

```

## Using Swift type inference

In `Swift`, we don't need to pass in a decimal number and the type is inferred by the type of argument in the function. The same method which was causing issue in `Objective C`, will give the desired result in `Swift`. Since we are passing the result of the expression to a function which expects `CGFloat`, the type is inferred as `CGFloat` and it performs floating point division.

```swift

func setLightBackground() {
        view.backgroundColor = UIColor(red: 254/255, green: 248/255, blue: 220/255, alpha: 1)
    }

```
![Image alt](/assets/posts/Type_Inference_UIColor/light.png "Light")

## UIColor convenience initialiser

We still need to do the division by 255 while creating a `UIColor`. Wouldn't it be nice if we just need to pass the r, g, b values in the function to create a `UIColor`.

I usually add below extension in my projects which makes creating `UIColor` very convenient.


```swift

extension UIColor {
    convenience init(_ red: CGFloat, _ green: CGFloat, _ blue: CGFloat, _ alpha: CGFloat = 1) {
        self.init(red: red/255, green: green/255, blue: blue/255, alpha: alpha)
    }
}

```

The division is taken care by the implementation of the function. The value of `alpha` is by default 1. We don't need to pass the value of `alpha` if its 1.

Usage of this convenience initialiser.

```swift

func setLightBackground() {
        view.backgroundColor = UIColor(254, 248, 220)
    }

```

As we can see, how succinct it has become to create a `UIColor`. Share your thoughts and experience with `Swift`'s powerful feature of type inference.

Happy Swifting.🃏