---
layout: post
title: Statements & Expressions
categories: [Tutorials,Javascript]
published: false
---

This article is part of a series that takes some concept and . It is not meant to be an exhaustive or accurate, but rather useful.
It is assumes a knowledge of language basics, 


## Values
What are values?

What are primitive types - 

In javascript, primitive values are 'wrapped' with helper objects. 
When you call a method on a primitive value, Javascript automatically converts that value into the object of its type, and thus is able to access the prototype methods.

```javascript
"abcd".length;
// returns 4
```



### Immutability

Primitive values are immutable. They cannot be changed. I like to think of primitive values as concrete entities that already exist. For example, values of the Number type could be represented as a deck of cards, with each card having an integer written on it. If you have the card with "42" written on it, the only way to get "43" is to pick up a new card - you can't change "42" into "43".

Another aspect of the immutability of primitive values is that you cannot modify or add to their properties:
```javascript
"abcd".length = 0; // Gives an error in strict mode, otherwise silently fails
"abcd".length;
// returns 4
```

Object values (such as Arrays, Functions and Objects) are mutable. 

## Expressions

We can think of expressions as any units of code that resolves or evaluates to a value. Wherever there is an expression, you can think of Javascript as evaluating it to a value, and replacing the expression with that value at that point in the code.


Values are expressions.

Pay attention to where JS expects expressions. eg in for loop

x++ vs ++x

### Statements
As you're probably aware, statements

We can also think of them as units of code that *don't* resolve to a value, as opposed to expressions.

The implicit return of a statement is, you guessed it, `undefined`. This behavior is most often relevant when dealing with functions that omit the `return` keyword.


### Statement Expressions



### Grouping

Statements are grouped in blocks surrounded by curly braces `{ }`. 

Expressions can be grouped in parentheses `( )`, and the expression separator is a comma: `,` .

One interesting behavior of creating multiple expressions with the expression separator, is that the interpreter will evaluate all expressions but return the *last* expression.

This allows for some useful shorthand 


You may have used the expression grouping operator to give control over the order of operations in complicated expressions. It's also useful when we need to force the interpreter to see the enclosed code as an expression, where there is a possibility 

For example, in arrow shorthand notation the function body can be either an expression to revaluate and return *or* a statement block. What happens if we want to use the shorthand, but return an object literal?


```javascript
myArray.map([key, val]) => { key : value };
```
To fix it, we can use the expression grouping operator to force the interpreter to see the body as an expression:

```
if (expression) statement;
for (statement; expression; expression)
```

### Summary

### Going further

##


Functions vs IIFE


```javascript
array.filter(elem => { elem });
```

This produces an error because 


Pay close attention to when language structures are expecting a statement or an expression.

Implicit vs explicit return.

Taking a for loop for example:

Where it gets confusing is expression-statements.

```javascript
for (let index = 0; index < array.length; index++) {

}
```

However, it can be run without. 

```javascript
for (let count = 1; count++, count < 5; console.log('hello!'));
```

```javascript
for (let count = 1; (console.log(count), count++) );
```

The following is possible:

```javascript
let x;
let y = 2 + (x = 2);
```
There are actually two things happening at `(x = 2)`. The variable `x` is being assigned to the value `2`. Since it's also an expression, a value is returned, so the value of `x` is returned.
The overall expression then becomes `2 + 2`, which is evaluated to 4.
This is the final value which is assigned to the variable `y`.


### Implicit Returns

The `return` keyword is often one of the first things we learn. It's also important to understand the *implicit* return values as well.
We know that expressions return a value. What do statements return?

If you've ever used a browser console or other Javascript REPL, you've probably seen something like the following:
```
>  let x = 0;
<- undefined
```

This is most often of importance when dealing with functions. Without a return statement, all functions implicitly return `undefined`.

```

```


