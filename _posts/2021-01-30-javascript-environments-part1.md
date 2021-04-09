---
layout: post
title: Javascript Environments 1 - Execution Context and Environment Records
categories: [Javascript]
---

Have you ever wondered how Javascript engines implement scope? Where bindings live? Or how closures really work? At the bottom of these questions are **environments**. In my experience, gaining an understanding of environments gives much needed clarity to several important concepts, especially scope and closures. It becomes apparent  'odd' behaviour of Javascript that causes frustration becomes apparent 

This journey presents challenges however - since Javascript is a continually evolving language it's not always clear whether a learning resource might be referring to outdated mechanisms. Consulting the ECMAScript specification documents themselves can be also confusing, as much of the behind the scenes behaviour results from the need to ensure backward compatibility, and makes no sense without a knowledge of the history of the language. 

In this series, I attempt to give an up to date explanation of environments, highlighting the most important concepts, and pointing out the legacy bits. In the first part, we'll introduce the basic concepts along with some examples, without going too deeply into specifics.

### Definitions

First, let's make sure we're all on the same page. When we talk about "scope" we are referring to the mechanism by which Javascript determines which variables are defined at a particular part of the program.

You've probably heard that Javascript has **lexical scope**. Put simply, this just means that the scope of a variable can be determined by looking at its location in the source code. 

We'll be using Javascript code to model the concepts we're examining. We aren't aiming to model the exact *implementation* of these concepts as they would appear in Javascript engines, but rather to understand the underlying *behaviours* that arise. Let's get started!

```javascript
let a = 1;
```

What's happening here? We may say something like "a `let` variable named `a` is declared and initialized with to the value 1." We might add that `a` becomes a globally scoped variable - it becomes accessible from within the global scope and within any inner scopes that might also be present. 

Where is this binding actually stored? And how does Javascript know that `a` identifies a variable with value 1 in global scope, but might identify a completely different variable in another scope? The answer is simple: "environment records". Before we can discuss them however, we'll need to introduce another concept: execution contexts. 

### Execution Context

Whenever Javascript code is executing, it is doing so within an **execution context**. Think of this as a device used to track the state necessary for executing the program at each particular moment. Execution contexts are stored in an **execution context stack**, also known as a "call stack", given its name by the fact that functions establish a new execution context each time they are called. The execution context that is currently active at any point in time is referred to as the **running execution context**, and it is the top execution context in the stack.

When a function is called and a new execution context is established, it is pushed onto the top of the stack and becomes the running execution context. When the function returns, its execution context is removed from the stack and the previous context is restored as the running execution context

We could construct a model of the execution context stack as an array object. Let's say we call a function `outerFn` from within global scope. From within `outerFn` we call another function `innerFn`. 

```javascript
function outerFn() {
  return innerFn();
}
function innerFn() {
  return true;
}
outerFn();
```

Our model of the execution context stack would look like this:

```javascript
const ExecutionContextStack = [];

// A global execution context is created and pushed onto the stack:
const globalExecutionContext = new ExecutionContext();
ExecutionContextStack.push(globalExecutionContext)
// The running execution context is globalExecutionContext.

/* outerFn is called here */

// A a new execution context is created and pushed onto the stack:
const outerFnExecutionContext = new ExecutionContext();
ExecutionContextStack.push(outerFnExecutionContext)
// The running execution context is now outerFnExecutionContext.

/* innerFn is called here, from inside outerFn */

// Yet another execution context is created and pushed onto the stack:
const innerFnExecutionContext = new ExecutionContext();
ExecutionContextStack.push(innerFnExecutionContext)
// The running execution context is now innerFnExecutionContext

// innerFn returns, its execution context is removed from the stack:
ExecutionContextStack.pop();
// The running execution context is back to outerFnExecutionContext

// outerFn returns, its execution context is removed from the stack:
ExecutionContextStack.pop();
// The running execution context is back to globalExecutionContext.
```

Don't worry about the exact implementation of the `ExecutionContext` object or everything it might hold - it's an abstraction of an abstraction. The important thing is to understand that:
- At any point during execution, there is always an execution context - the "running execution context", which is at the top of the stack (and in our model, the last item of the array).
- A function call will create a new execution context which is pushed onto the top of the stack where it becomes the running execution context while the function executes. It is removed from the stack when the function returns. 

We're now going to focus in on one important kind of property that each execution context holds - environment records.

### Environment Records

Associated with each execution context are **environment records**. These hold data and specify methods which allow javascript to manage the state of the current context. 

Each environment record contains at least the following:
- A mapping of all the bindings that are present within the scope of that particular environment.
- A reference to an outer environment record. 
- Methods used for managing state - including those for creating, initializing, and reassigning bindings

The reference to an outer environment is what implements the nested scope of Javascript, allowing identifiers to be resolved by traversing environments outwards from an inner environment.

When code is executed that would create a new scope, such as a function or block, before the body of code is executed new environment records are created and linked to the relevant execution context. The identifiers that are in scope within the environment that is to be executed are added to these records, and possibly initialized depending on the type of declaration. This implements the behaviour of **hoisting** which you may be familiar with.

There are different kinds of environment record, and complex rules to determine which variables are created in which record. Ignore all of that for the moment: we're only concerned with understanding the general concept. For now, we'll say that each execution context has a single environment record which holds all of the in-scope variables in that context.

Let's continue using plain Javascript objects to model this!
```javascript
let a = 1;
function addNums() {
  let b = 2;
  return a + b;
}
addNums(); // returns 3
```
When the above script is executed, the top level declarations will be gathered up and added to a new global environment record. This environment record will be associated with the execution context that is created:

```javascript
/* Program state at beginning of execution */
const ExecutionContextStack = [];

/* Environment Records */
const globalEnv = {
  identifiers : {
    a : 1,
    addNums : function {..} // reference to addNums function object
  },
  outerEnv : null
}

/* Execution Contexts */
const globalExecutionContext = new ExecutionContext();
globalExecutionContext.EnvironmentRecord = globalEnv;
ExecutionContextStack.push(GlobalExecContext);
```
Notice that the `outerEnv` reference of `globalEnv` is `null` - the global environment is an environment that has no outer environment.

When `addNums()` is called, the same process is followed as above - this time, the `outerEnv` reference will be set to the environment record where `addNums()` was defined: 

```javascript
/* Program state when addNums is called */

/* Environment Records */ 
const addNumsEnv = {
  identifiers : {
    b : 2
  },
  outerEnv : globalEnv
}

const globalEnv = {
  identifiers : {
    a : 1,
    addNums : function {..} // reference to addNums function object
  },
  outerEnv : null
}

/* Execution Contexts */
const addNumsExecutionContext = new ExecutionContext();
addNumsExecutionContext.EnvironmentRecord = addNumsEnv;
ExecutionContextStack.push(addNumsExecutionContext)
```
Since `globalEnv` is where `addNums` was defined, it becomes the `outerEnv` reference of the environment record of the running execution context when `addNums` is called.

At this point in execution, the stack would look something like the following:
```javascript
ExecutionContextStack = [
  { EnvironmentRecord : globalEnv, /* ..other properties */ },
  { EnvironmentRecord : addNumsEnv, /* ..other properties */ }
]
```

Looking at the above examples, we gain a basic appreciation for how the scope chain works - when resolving a binding, the environment record of the running execution context will be searched first, and if the binding is not present, the outer environment will be searched, and so on. We could imagine some simplified logic that might implement such behaviour:  

```javascript
function resolveBinding(name, env) {
  while (env !== null) {
    return env.identifiers[name] || resolveBinding(name, env.outerEnv);
  }
  return "Reference error - binding not found!"
}

// assuming addNumsEnv is the environment record of the running execution context
resolveBinding("a", addNumsEnv); // returns 1
```
With this, an understanding of **variable shadowing** also emerges. Let's look at another simple example:

```javascript
let a = 5;
let b = 3
const multiply = (a, b) => {
  return a * b;
}
multiply(2, 4); // returns 8.
```
When we call `multiply`, here's what the environment records would look like:
```javascript
/* Environment Records */

const multiplyEnv = {
  identifiers : {
    a : 2,
    b : 4
  },
  outerEnv : globalEnv
}

const globalEnv = {
  identifiers : {
    a : 5,
    b : 3
    multiply : function {..} // reference to multiply function object
  },
  outerEnv : null
}
```
Although `a` and `b` are defined in the `globalEnv` record, being named parameters of `multiply` they are also defined as identifiers in `multiplyEnv`, and since that environment record is searched *before* the `globalEnv` record, `a` and `b` resolve to the values defined in the `multiplyEnv` record, shadowing the definitions in the outer environment. Neat!

One final thing to note: while we've been talking of execution contexts as having associated environment records, the environment records are not *stored* with the execution context. Were it the case, the environment record would be destroyed along with the execution context whenever a function returns. Can you see a problem with that? (Hint: think about closures).

### Summary

We've been looking at a simplified model of how Javascript implements scope with environment records and execution context. We saw that:
- Running code is always done within an associated execution context
- Execution contexts are stored in a stack, with the top of the stack always being the **running execution context**
- Declarations that are in scope within an environment are collected within **environment records**
- Environment records store these identifiers, and a reference to an outer environment
- Each execution context is associated with its own environment record
- Bindings are resolved by searching the environment of the running execution context, and if necessary proceeding up the chain of outer environments until the binding is found or 

So far, we've kept things fairly basic. Now that we have an understanding of the general concept, in part 2 we're going to expand our model of environment records to more closely match reality.




















