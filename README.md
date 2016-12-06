## Advanced JavaScript concepts
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
Another common use case of `bind()` is to set the context of the keyword `this` for a function that will be called at a later point in time. Very commonly this happens when dealing with asynchornous code. Let's have a look at a more complex example:
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

### 4. `new` and `this`
When the `new` keyword is used the value of the keyword `this` is set to an empty object and returned from the function that is used with the `new` keyword. Here is an example (we'll comer `new` in more depth later): 
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
If we want to avoid repeating the same code, we need to have a way of declaring a blueprint and then using it to create new instances. Other programming languages have classes, but in JavaScript we use functions and objects to mimick classes and related behaviour (the new JavaScript version ES6 does have classes). Here is how we set a constructor function and then create an new object from it:
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
