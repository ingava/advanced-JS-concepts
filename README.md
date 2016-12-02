# Advanced JavaScript concepts
These are my notes on advanced JS topics such as "this", prototype, closures and etc.

Keyword `this`

`this` is determined by how a junction is called ("execution context"). What `this` refers to can be determined using 4 rules (global, object/implicit, explicit, new).

### 1. Global `this`

When `this` is not inside of a declared object it will point to the window object. If it is declared inside of a function it will also point to the window object. 

``` 
console.log(this) // window

function whatIsThis() {
  return this
}

function variablesInThis() {
  // since the value of this is the window object
  // all we are doing here is creating a global variable (generally, you wouldn't want to do this)
  this.person = "Elie"
}

console.log(person) // Elie

whatIsThis() // window
``` 
#### "Strict Mode"

When Strict Mode is enabled ("use strict"), the value of the keyword `this` when inside of a function is `undefined`; it is not the global object. This means that if we try to attach properties onto it (like in the function above), we get a TypeError since we can't attach properties onto `undefined`. This stops us from accidentally creating global variables and allows us to use JavaScript best practices.

#### A Side note

In the below function variable "person" will not be accessed from outside of the function. 

```
function makePerson() {
  var person = "Elie"
}
console.log(person) // person is not defined
```
If we declare a variable inside the function without "var", it will be created in the global scope. Like this:
```
function makePerson() {
  person = "Elie"
}
console.log(person) // "Elie"
```
@his is bad practice, never do this!

### 2. Object/implicit `this`

When the keyword `this` is inside of a declared object, the value of it will be the closest parent object.
Let's have a look at this example:

```
// strict does not make any difference here

var person = {
  firstName: "Elie",
  sayHi: function(){
    return "Hi" + this.firstName
  },
  determineContext: function() {
    return this === person
  }
}

person.sayHi() // "Hi Elie"
person.determineContext() // true
```
How about nested objects? Well, this is where the story gets a little complicated. Below we have a similar silly situation:

```
var person = {
  firstName: "Elie",
  sayHi: function(){
    return "Hi" + this.firstName;
  },
  determineContext: function() {
    return this === person;
  },
  dog: {
    sayHello: function() {
      return "Hello" + this.firstName;
    },
    determineContext: function() {
      return this === person;
    }
  }
}
```
Remeber the rule "closest parent object"? So in this example `person.dog.sayHello()` will return `"Hello undefined` and `person.dog.determineContext` will be `false`. 
But this is not what we wanted. How can we fix this? And that is where `call`, `apply` and `bind` methods come in. More on these below.

### 3. Explicit `this` (call, apply, bind)

With explicit binding we can choose what the value of `this` will be. So let's have a look at how we cal solve the problem that we have above with call.

#### Call()

Here we have the same nested object:
```
var person = {
  firstName: "Elie",
  sayHi: function(){
    return "Hi" + this.firstName;
  },
  determineContext: function() {
    return this === person;
  },
  dog: {
    sayHello: function() {
      return "Hello" + this.firstName;
    },
    determineContext: function() {
      return this === person;
    }
  }
}
```
But if we want `this` to point to the person object and not the dog object, we bind it like so:
```
person.dog.sayHello.call(person) // "Hello Elie"
person.dog.determineContext.call(person) // true
```
Mind that we are not invoking `sayHello` or `determineContext` methods as we don't have parenthesis there. We are only binding `this` to the person object.
The `call()` method takes an infinite number of parameters. The first one is what we want the keyword `this` to refer to. In our case it is the `person` object. The first argument is also often called the `thisArg`. The other arguments are the ones that we want to pass to the method.

#### apply()

`apply()` method is very similar to call(). The only difference is that it takes only two arguments - thisArg and an array of arguments that we want to pass to a method. Let's have a look at an example:

```
var inga = {
  firstName: "Inga",
  sayHi: function() {
    return "Hi " + this.firstName
  },
  addNumbers: function(a, b, c, d) {
    return this.firstName + " just calculated " + (a + b + c + d);
  }
}
var elie = {
  firstName: "Elie"
}

inga.sayHi() // Hi Inga
inga.sayHi.apply(elie) // Hi Elie

// let's add the numbers

inga.addNumbers(1, 2, 3, 4) // Inga just calculated 10
inga.addNumbers.call(elie, 1, 2, 3, 4) // Elie just calculated 10
inga.addNumbers.apply(elie, [1, 2, 3, 4]) // Elie just calculated 10

