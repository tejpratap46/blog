---
title: "How to use Bitwise while setting properties"
datePublished: Tue May 02 2023 11:24:22 GMT+0000 (Coordinated Universal Time)
cuid: clh66mc6b000009jjgv0c58ic
slug: how-to-use-bitwise-while-setting-properties
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1683026543755/6db2ae9a-dc64-4d52-859a-7f86a6b6957f.png
tags: programming-blogs, kotlin, bitwise

---

Sometimes we are faced with a situation, where a parameter can have multiple values, or even it can have combination of those values.

We have a few choices here, either we create multiple fields for each property, which is hard to scale but easier to understand.

Here is a code sample for that:

```kotlin
fun main() {
    order(true, true, false)
}

fun order(hasOrderedCheeseBurger: Boolean, hasOrderedFries: Boolean, hasOrderedSoda: Boolean) {
    println("Has Ordered CHEESEBURGER : ${hasOrderedCheeseBurger}"); // true
    println("Has Ordered FRIES : ${hasOrderedFries}"); // true
    println("Has Ordered SODA : ${hasOrderedSoda}"); // false
}
```

As you can see, we had to send 3 different parameters to know what this user had ordered and imagine, if, in future we need to add another item, let's say `SANDWICH` then we need to add another parameter.

---

Now, instead of passing multiple parameters, we can use bit manipulation to combine multiple parameters into a single parameter.

```kotlin
val CHEESEBURGER = 2; // 00000010
val FRIES = 4;        // 00000100
val SODA = 8;         // 00001000

fun main() {
    val ordered = CHEESEBURGER or FRIES;
    
    order(ordered)
}

fun order(what: Int) {
    val hasOrderedCheeseBurger = what and CHEESEBURGER == CHEESEBURGER;
    val hasOrderedFries = what and FRIES == FRIES;
    val hasOrderedSoda = what and SODA == SODA;
    println("Has Ordered CHEESEBURGER : ${hasOrderedCheeseBurger}"); // true
    println("Has Ordered FRIES : ${hasOrderedFries}"); // true
    println("Has Ordered SODA : ${hasOrderedSoda}"); // false
}
```

Here, bitwise allows us to add as many values as we need into a single parameter.