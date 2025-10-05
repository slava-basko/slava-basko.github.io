---
layout: post
title: "Functional Programming and PHP"
tags: [php, functional]
---

Let's talk about PHP and functional programming. Why might it be a thing.

I always try to learn something new because this is the part of me. I love learning, it helps brain to be fit. As part of this process, I used to try Go, Python, Java, etc. More or less, they are all about the same.
It's like learning Spanish and Italian. They are both Latin, use similar structures, grammar, syntax, etc. You get some profit of knowing both languages, but not the same as you know Latin and Cyrillic, for example.
Cyrillic is something completely different. This shows how else one can communicate. It opens completely new horizons.

After a little bit of googling, I found functional programming. Eventually, I ended up with Scheme and Haskell, and later primary with Haskell. Ohh, that was a real struggle even in understanding what was going on in a simple Haskell function because this language was exotic to my procedural brain. I believe it is far easier for a newcomer to grasp Haskell than for a someone who get used to writing procedural code.

Today I want to focus not on the specific functional programming language, but on the paradigms, rules, principles from the functional programming, like:
* Purity
* Immutability
* First-Class Functions
* Composition
* Currying
* Declarative Programming

The first three might sound familiar to any modern procedural developer. I will not go thoroughly through all the concepts, but instead, I want to show what they are all about briefly.

### Purity

Function that has **both input and output**, with no [side effects](https://en.wikipedia.org/wiki/Side_effect_(computer_science)){:target="_blank"}.
Why do I emphasize on both input and output? Because most likely your function has side effects if there is no input or output.
```php
// Pure function
function concat($separator, $a, $b) {
    return $a . $separator . $b;
}

// Not pure, because relying on global constant
function concat($a, $b) {
    return $a . DEFAULT_SEPAEATOR . $b;
}
```

Pure functions are much easier to test. You don't need to mock anything! You pass arguments, you get the result.
Pure functions are predictable and straightforward.

### Immutability
Data, once created, cannot be changed. Instead of modifying existing data, new data is created based on the old data.
We don't need to do anything special in most cases to stick this rule because `passed by value` is a default PHP behavior.
Except [objects](https://www.php.net/manual/en/functions.arguments.php#functions.arguments.by-reference){:target="_blank"}, they a `passed by reference`.
```php
function formatEmail(ValueObject $valueObject) {
    $value = $valueObject->getValue();
    
    // do what you need with $value
    
    return new ValueObject($value);
}
```
Obviously, immutability saves as from the unexpected results at the end.

### First-Class Functions
This fancy phrase means that the functions like any other data type, such as numbers or strings.
You can assign them to variables, pass them as arguments to other functions, and return them from functions.

I bet you are familiar with this technique and using it on a daily basis. For example, collections:
```php
$collection = new Collection([1, 2, 3]);

$filtered = $collection->filter(function($element) {
    return $element > 1;
}); // [2, 3]
```
Collection::filter() accepts function as argument.

Here is another example of how first-class functions could be useful:
```php
sort(ascend(prop('age')), [
     ['name' => 'Emma', 'age' => 70],
     ['name' => 'Peter', 'age' => 78],
     ['name' => 'Mikhail', 'age' => 62],
]); // [['name' => 'Mikhail', 'age' => 62], ['name' => 'Emma', 'age' => 70], ['name' => 'Peter', 'age' => 78]]
```
First-class functions allow us to write more expressive code. Like this example: function `prop` returns a new 
function that is passed to `ascend` that returns a new function that is passed to `sort`.

### Composition
We can build more complicated functions out of simpler ones.
```php
function plus1($value) {
    return $value + 1;
}

function power($value) {
    return $value * $value;
}

$powerPlus1 = compose('plus1', 'power');
// We created the new function $powerPlus1 that powers the value and that adds 1 to it.

$powerPlus1(3); // 10
```
Composition is like you are playing with Lego blocks. You can create a new, more complex structure based 
on smaller bricks.

### Currying
Currying is the technique of translating a function that takes multiple arguments into a sequence of families of functions.
```php
function add($a, $b, $c) {
     return $a + $b + $c;
};

$curryiedAdd = curry('add');
$curryiedAdd(1, 2, 3); // 6
$curryiedAdd(1)(2)(3); // 6
$curryiedAdd(1)(2, 3); // 6
$curryiedAdd(1, 2)(3); // 6
```
Function `add` execution will be triggered when all three arguments are supplied. A new function will be returned 
that expects the rest of the arguments in case not all arguments were provided.

This allows us to write things like in the `First-Class Functions` section.

### Declarative Programming
Usually, you write your code in a way that you describe to the computer how you want to achieve results. 
The declarative way allows you to write code in a way that you are talking to the computer about what you want 
to achieve.
```php
$totalQty = compose(sum, pluck('qty'))([
    [
        'description' => 't-shirt',
        'qty' => 2,
        'value' => 20
    ],
    [
        'description' => 'jeans ',
        'qty' => 1,
        'value' => 30
    ],
    [
        'description' => ' boots',
        'qty' => 1,
        'value' => 40
    ]
]);
```
You are telling your computer to `Take all 'qty' properties and sum them up`. You are not bothered by how to extract 
property and how to sum up values.

### But all these small functions are procedural under the hood!

Yes, you’re right. We are writing in PHP at the end. Functional code is just a higher-level abstraction. 
At the core, all programs — even in languages like Haskell — reduce to procedural execution, because computers 
are inherently procedural machines.

The idea is that you are creating a lot of small, stateless Lego bricks. Then, using them, you can build more 
and more complex functions. You don't like the resulting structure? No problem! Just break it apart and create a 
new one with the same bricks.
