## Advanced JavaScript concepts
These are my notes on advanced JS topics such as "this", prototype, closures and etc.

## Keyword `this`

`this` is determined by how a function is called ("execution context"). What `this` refers to can be assessed using 4 rules (global, object/implicit, explicit, new).

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
This is bad practice, never do this!

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
How about nested objects? Well, this is where the story gets a little complicated. Below we have a similar situation:

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
But this is not what we wanted. How can we fix this? That is where `call()`, `apply()` and `bind()` methods come in. More on these below.

### 3. Explicit `this` (call, apply, bind)

With explicit binding we can choose what the value of `this` will be. So let's have a look at how we can solve the problem above with `call()`.

#### `Call()`

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

#### `apply()`

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
```
#### `bind()`
`bind()` works just like `call()` but instead of calling the function right away it returns a function definition with the keyword `this` set to the value of the `thisArg`. So when is `bind()` useful? One common use case is when we do not know all of the arguments that will be passed to a function. It means we do not want to invoke the function right away, we just want to return a new function with some of the parameters set. It is called "partial application". 
```
// with bind we do not need to know all the arguments up front

var elieCalc = inga.addNumbers.bind(elie, 1, 2) // function() {}...
elieCalc(3, 4) // Elie just calculated 10
```
Another common use case of `bind()` is to set the context of the keyword `this` for a function that will be called at a later point in time. Very commonly this happens when dealing with asynchronous code. Let's have a look at a more complex example:
```
var inga = {
  firstName = "Inga",
  sayHi: function() {
    setTimeout(function() {
      console.log("Hi " + this.firstName)
    }, 1000)
  }
}
inga.sayHi() // Hi undefined (1 second later)
```
So it might come as a suprise that the result of `inga.sayHi()` would be Hi undefined. This is because setTimeout is set on the window object even though it is inside a declared object. To correct this situation we can explicitly assign `this` to the declared object with `bind()` like so:
```
var inga = {
  firstName = "Inga",
  sayHi: function() {
    setTimeout(function() {
      console.log("Hi " + this.firstName)
    }.bind(this), 1000)
  }
}
inga.sayHi() // Hi Inga (1 second later)
```
This situation is a good example when `call()` and `apply()` would be no good because we want to call the method at a later point in time with setTimeout, in our case 1 second later. 

Note that ES6 version has fat arrow functions (=>) which solve the problem of `this` in some cases.

### 4. `new` and `this`
When the `new` keyword is used the value of the keyword `this` is set to an empty object and returned from the function that is used with the `new` keyword. Here is an example (we'll cover `new` in more depth later): 
```
function Person(firstName, lastName) {
  this.firstName = firstName;
  this.lastName = lastName;
}
var inga = new Person("Inga", "Vaiciakauskaite");

inga.firstName // "Inga"
inga.lastName // "Vaiciakauskaite"
```
## OOP and `new` keyword
If we want to avoid repeating the same code, we need to have a way of declaring a blueprint and then using it to create new instances. Other programming languages have classes, but in JavaScript we use functions and objects to mimick classes and related behaviour (the new JavaScript version ES6 does have classes). Here is how we set a constructor function and then create a new object from it:
```
// constructor function

function House(bedrooms, bathrooms, numSqM) {
  this.bedrooms = bedrooms;
  this.bathrooms = bathrooms;
  this.numSqM = numSqM;
}

// creating an object from the constructor function with the "new" keyword 
var firstHouse = new House(2, 2, 120)

firstHouse.bedrooms // 2
firstHouse.bathrooms // 2
firstHouse.numSqM // 120
```
So what does exactly the `new` keyword do? These are its main functions:

1. First it creates an empty object
2. It then sets the keyword `this` to be that empty object
3. It adds the implicit `return this` to the end of the function so that the object can be returned from the function
4. It adds a property onto the empty object called "__proto__" which links the prototype property on the constructor function to the empty object (more on this later).

### Multiple constructors

So let's imagine that we have two similar constructor functions in our app:
```
function Car(make, model, year) {
  this.make = make;
  this.model = model;
  this.year = year;
  this.numWheels = 4;
}

function Motorcycle(make, model, year) {
  this.make = make;
  this.model = model;
  this.year = year;
  this.numWheels = 2
}
```
This isnt't what we would call DRY code. So here is how we can refactor it combining the two constructor functions (we will have tu use either `call()` or `apply()`:
```
// The Car function looks the same
function Car(make, model, year) {
  this.make = make;
  this.model = model;
  this.year = year;
  this.numWheels = 4;
}
// in the Motorcycle function we invoke the Car function to avoid code duplication
// we also have to explicitly say what the 'this' value needs to be

function Motorycle(make, model, year) {
  Car.call(this, make, model, year);
  this.numWheels = 2;
}
```
This looks pretty neat. Here we set the value of `this` to be the object created from the Motorcycle constructor function rather than the one created from the Car function. 
We can do the same thing with `apply()` and the code will look even more sleek: 
```
function Motorcycle(make, model, year) {
  Car.apply(this, arguments);
  this.numWheels = 2;
}
```
The `arguments` keyword is a special keyword. It is a list of all of the arguments that are passed to a function. Of course, we could have written it likes this too: `Car.apply(this, [make, model, year])`. 

## Prototypes

Every constructor function has a property on it called "prototype" which is an object. It can have properties and methods attached to it like any other object. This prototype object has a property on it called "constructor" which points back to the constructor function. Anytime an object is created using the `new` keyword, a property called "__proto__" gets created which links the object and the prototype proterty of the constructor function. This means that methods and properties attached to the prototype object are shared and accessible by any object that is created from that constructor function. Let's look at some code that illustrates this tricky relationship: 
```
// this is the constructor function

function Person(name) {
  this.name = name;
}

// this is an object created from the Person constructor

var inga = new Person("Inga");

// since we used the new keyword, we have established a link between the object and the prototype property
// we can access that using __proto__

inga.__proto__ === Person.prototype; // true
```
### Prototype chain

The prototype property is an object which can have methods and properties placed on it. These methods and proeprties are shared and accessible by any object that is created from that constructor function when the `new` keyword is used. To continue with the previous code example: 
```
function Person(name) {
  this.name = name;
}
var inga = new Person("Inga");

Person.prorotype.isInstructor = true;

inga.isInstructor; // true
```
We have access to the properties of the prototype through `__proto__`. Here we see the prototype chain at play.
Prototype chain means that JavaScript will look at an object and see if the method or property you are looking for exists and if not it will go to that object's `__proto__` property and repeat until there is not another `__proto__` to look at. 

How to use prototypes? 

Let's say we have a code like this:
``` 
function Person(name) {
  this.name = name;
  this.sayHi = function() {
    return "Hi " + this.name;
  }
}

var inga = new Person("Inga");
inga.sayHi(); // Hi Inga
```
This code works but it is inefficient because every time that the Person object is created we have to define this function on that object. So when we make 1 million objects from the constructor we are adding the `sayHi` property one million times. That is not very efficient. It would be nice if we could just define it once and have it accessible from every object created from the constructor. And that is exactly what placing methods on the prototype object lets us do. So let's refacture our code a little using prototype object: 
```
function Person(name) {
  this.name = name;
}

Person.prototype.sayHi = function() {
  return "Hi " + this.name;
}

var inga = new Person("Inga");
inga.sayHi(); // Hi Inga
```
This code is much more effcient and makes use of best practices in OOP. We should always strive to place properties and methods that we want to share among all of our objects created from a constructor in the prototype as these properties and methods only need to be defined once and not redefined for every single object.

## Closures

A closure is a function that makes use of variables defined in outer functions that have previously returned. Here is an example of closure:
```
function outer() {
  var data = "closures are ";
  return function inner() {
    var innerData = "awesome";
    return data + innerData;
  }
}

outer()() // "closures are awesome"
```
Usually local variables created inside a function are not accessible outside of it. But in closures, variables created in an outer function can be accessed inside of an inner function when the outer function has already returned. 
A closure exists only when an inner function makes use of variables defined in an outer function that has returned. If the inner function does not make use of any of the external variables all we have is a nested function. 
You would use a closure to create private variables and to isolate your logic. 
