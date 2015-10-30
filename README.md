#Object Oriented JavaScript

Since JavaScript is a functional programming language, you don't have classes and instances like an object-oriented language has in the conventional sense. In order to simulate classes we use what are called constructors.

##Constructors

Constructors are just functions that you use with the ```new``` keyword to make new objects. You have already seen several built-in constructors in JavaScript:

Object, Array, Function, etc.. (Notice they are always capitalized!)

```javascript
var str = new String("Electroencephalographically");
var arr = new Array(10);  // arr = [ , , , , , , , , , ]
var obj = new Object({ name: "David", age: "unknown" });
```

They are used to define objects with similar properties, kind of like classes in an object-oriented language. Let's define a class called "Person" using a function declaration:

```javascript
function Person () {};
```
Equivalently (almost), we could define it this way.. as a function expression:
```javascript
var Person = function () {};
```
The only difference is when using a function declaration, the function gets hoisted to the top of its scope and is accessible in its local and its parent's scope.. 

Now that we've got a constructor function (class), we can make instances of it:
```javascript
var person1 = new Person();
var person2 = new Person;
```
When you have no parameters to pass into the constructor, the parentheses are optional.. but NOT the ```new``` keyword (more on that later)!

Even though we don't explicitly return anything from our constructor function, ```person1``` and ```person2``` are both instances of the ```Person``` type:
```javascript
console.log(person1 instanceof Person);  // true
console.log(person2 instanceof Person);  // true
console.log(person1.constructor) // [Function: Person]
```
Because we used the ```Person``` constructor to create them, ```instanceof``` returns ```true``` for both ```person1``` and ```person2``` ... You can also check the type of an instance using the ```constructor``` property. Every object is created with a ```constructor``` property that contains a reference to the function that created it. For *generic* objects (those created with an object literal or the ```Object``` constructor, it points to ```Object```:
```javascript
var obj1 = { size: 10, empty: false };
var obj2 = new Object({ size: 12, empty: true});
obj1.constructor === Object  // true
obj2.constructor  // [Function: Object]
obj1.constructor === obj2.contructor // true
```

Let's spice things up a bit with a constructor that actually takes some parameters..
```javascript
function Person(name, age) {
  this.name = name;
  this.age = age;
  this.sayName = function() {
    console.log(this.name + " is my name.");
  };
}
```
This ```Person``` constructor accepts two named parameters, ```name``` and ```age```, and assigns them to properties on the ```this``` object with the same names. It also adds a method called ```sayName()``` to the object. The ```this``` object is automatically created by using the ```new``` keyword when calling the constructor, and it is an instance of the constructor's type. It is implicitly returned by the function, so there's no need to include a ```return``` statement within your constructor function. If you do, and the return value is an object, it will return the object instead of the newly created instance of the class. If the return value is a primitive, it will be ignored and the new instance will be returned.

Now you can use the ```Person``` constructor to create objects that are initialized with ```name``` and ```age``` properties:
```javascript
var person1  = new Person("Billy", 12);
var person1 = new Person("Jad", 34);

console.log(person1.name);    // 'Billy'
console.log(person2.name);    // 'Jad'
console.log(person1.age);     //  12

person1.sayName();            // 'Billy is my name.'
person2.sayName();            // 'Jad is my name.'
```

Now that you can create a bunch of objects with the same properties and methods, you might think constructors are a great way to reduce code redundancy. However, for each instance you create, a copy of the properties are made, too. For example, even though the ```sayName``` function is no different between each instance, a separate copy is made for every object instance! If you had 1,000 instances of the ```Person``` class, you'd have 1,000 copies of that function, doing the same exact thing. Wouldn't it be way more efficient if all the instances could share just one copy of that method? Well yes, it would, and that is why JavaScript has **Prototypes**..

##Prototypes
JavaScript Prototypes are like blueprints for objects. Almost every function (besides some of the built-in ones) has a ```prototype``` property that is used when creating new instances, and every instance of the same type shares access to the properties of the prototype. Maybe an example using an instance of the built-in ```Object``` constructor function will make this clearer:
```javascript
var band = { name: "Reverse Peristalsis" };

band.constructor === Object  // true
```
Even though we've only defined one property on this object (the ```name``` property), look at all the properties it already has:
```
band.__defineGetter__      band.__defineSetter__      band.__lookupGetter__
band.__lookupSetter__      band.__proto__             band.constructor
band.hasOwnProperty        band.isPrototypeOf         band.propertyIsEnumerable
band.toLocaleString        band.toString              band.valueOf

band.name   
```
The twelve properties above that I didn't define are there because they are defined on the built-in ```Object.prototype``` object, and since ```band``` is an instance of ```Object```, it automatically has access to all of those properties. You can use the ```in``` operator to find out if a property exists on an object or its prototype. If you ever need to figure out which properties are part of the prototype and which were defined within the constructor, the ```hasOwnProperty()``` method can be used to determine that:
```javascript
console.log("name" in band);                                    // true
console.log(band.hasOwnProperty("name"));                       // true
console.log("hasOwnProperty" in band);                          // true
console.log(band.hasOwnProperty("hasOwnProperty"));             // false
console.log(Object.prototype.hasOwnProperty("hasOwnProperty")); // true
```
So ```in``` returns ```true``` for both prototype properties *and* own properties, and ```hasOwnProperty()``` only returns true on own properties, or properties defined within the constructor. Can you create a function that determines whether a given property is a prototype property?

```javascript
function hasPrototypeProperty(object, name) {
  return name in object && !object.hasOwnProperty(name);
}
```
###Using Prototypes with Constructors
Since prototype properties are shared between all instances of a class or constructor function, we can define a property on the prototype of the constructor and it will be available to all instances created after its definition. It's much more efficient to put a method on the prototype and then use ```this``` to access the current instance, instead of defining the method within the constructor:
```javascript
function Person(name, age) {
  this.name = name;
  this.age = age;
}

Person.prototype.sayName = function() {
  console.log(this.name + " is my name.");
};

var person1 = new Person("David", 100);
var person1 = new Person("Melissa", 99);

person1.sayName(); // 'David is my name.'
person2.sayName(); // 'Melissa is my name.'
```
The prototype of an object is stored internally in the [[Prototype]] property, or ```__proto__``` in most JS engines. This property is a reference, not a copy, so if you change the prototype at any point in time, the changes occur on all instances of the class.
```javascript
function Person(name) {
  this.name = name;
}

var person1 = new Person("Lynne");

'sayName' in person1 // false

Person.prototype.sayName = function() {
  console.log(this.name + " is my name.");
}

'sayName' in person1 // true
person1.sayName();  // 'Lynne is my name.'
```

You can even alter built-in Object prototypes.. say you want to implement a ```sum``` method on the Array prototype:
```javascript
Array.prototype.sum = function() {
  return this.reduce(function(a, b) {
    return a + b;
  });
}

var result = [1,2,3,4,5].sum();
console.log(result); // 15
```
Even though it's possible, I wouldn't recommend altering built-in object prototypes in a production environment... This would make default behaviors unpredictable, especially for other coders trying to user your code.

##Summary
Constructors are just normal functions that are called with the ```new``` operator to generate objects with the same properties. In this way, they are like classes in strictly object oriented languages. You can identify an object's constructor accessing the ```constructor``` property or using the ```instanceof``` operator.
Every function has a ```prototype``` property that defines any properties shared by objects created with the same constructor. Shared methods and primitive value properties are typically defined on the prototype, with all other properties defined within the constructor. An example of a prototype property shared by all instances is the ```constructor``` property itself.

# Prototypal Inheritance

## Classes & Prototypes

JavaScript has an unusual system for implementing
inheritance. JavaScript's version of inheritance is called
**prototypal inheritance**, and it differs from the **classical
inheritance** that we are familiar with from Ruby.

When you call any property on any JavaScript object, the interpreter
will first look for that property in the object itself; if it does not
find it there, it will look in the object's prototype (which is pointed to
by the object's internal `__proto__` property).

If it does not find the property in the prototype, it will recursively
look at the prototype's `__proto__` property to continue up the
*prototype chain*. How does the chain stop?
`Object.prototype.__proto__ == null`, so eventually the chain ends.

It is for this reason that we call `Object` the "top level class" in
JavaScript.

Inheritance in JavaScript is all about setting up the prototype
chain. Let's suppose we have `Animal`s and we'd like to have `Dog`s
that inherit from `Animal` and `Poodle`s that inherit from `Dog`.

Well, we know that we'll instantiate each of these constructor style:

```javascript
function Animal () {};
function Dog () {};
function Poodle () {};

var myAnimal = new Animal();
var myDog = new Dog();
var myPoodle = new Poodle();
```

* `myAnimal`'s `__proto__` references `Animal.prototype`.
* `myDog`'s `__proto__` references `Dog.prototype`.
* `myPoodle`'s `__proto__` references `Poodle.prototype`.

Great, but `Animal`, `Dog`, and `Poodle` don't yet relate to each
other in any way. How can we link them up?

Well, we know that we want `Poodle.prototype` to reference
`Dog.prototype`, and we want `Dog.prototype` to reference
`Animal.prototype`. And when we say reference, we really mean that we
want the `__proto__` property to point to the correct prototype
object. In particular, we want:

* `Poodle.prototype.__proto__ == Dog.prototype`
* `Dog.prototype.__proto__ == Animal.prototype`

`__proto__` is a special property: you can't set it yourself. Only
JavaScript can set `__proto__`, and the only way to get it to do it is
through constructing an object with the `new` keyword.

So this is one way we can hook up the classes into a prototypal
inheritance chain:

```javascript
function Animal () {};

function Dog () {};

// Replace the default `Dog.prototype` completely!
// `Dog.prototype.__proto__ == Animal.prototype`.
Dog.prototype = new Animal();

function Poodle() {};
// Likewise with `Poodle.prototype`.
Poodle.prototype = new Dog();

var myAnimal = new Animal();
var myDog = new Dog();
var myPoodle = new Poodle();
```

*Voila.* Prototypal inheritance.

Any method that is defined for `Animal.prototype` will be accessible
to all `Dog`s and `Poodle`s instances, because of the recursive search
through the prototype chain. Likewise, any property of `Dog.prototype`
will be accessible to `Poodle`s.

## Problems With Naive Prototypal Inheritance

Okay, you may have noticed something odd with these `prototype`s. In
setting up a prototype (say, set `Dog.prototype = new Animal()`), we
construct an instance of the super class.

Doesn't that seem weird? We create an `Animal` instance (to set up
`Dog.prototype`), before we instantiate any `Dog`s. Why? The `Animal`
object instance stored in `Dog.prototype` is sort of a "fake"
`Animal`; is it really right to create this? It doesn't feel right
that **defining** a new type of `Animal` should involve **constructing
an instance** of `Animal`.

Two side-effects of this weirdness are that:

0. The `Animal` constructor will be called before we create any `Dog`s,
0. Creating a `Dog` instance will never run the `Animal`'s constructor
   function.

Let's see what this means in the real world:

```javascript
function Animal (name) {
  this.name = name;
};

Animal.prototype.sayHello = function () {
  console.log("Hello, my name is " + this.name);
};

function Dog () {};
// Hey wait, doesn't Animal need a name?
Dog.prototype = new Animal();

Dog.prototype.bark = function () {
  console.log("Bark!");
};

// We're not even going to run `Animal`'s constructor, so why bother
// passing the name?
var dog1 = new Dog("James");

// `this.name` is `undefined`
dog1.sayHello();
```

## The Surrogate Trick

Let's summarize our goals:

0. `Dog.prototype.__proto__` **must be** `Animal.prototype` so that we
   can call `Animal` methods on a `Dog` instance.
0. Constructing `Dog.prototype` **must not** involve calling the
   `Animal` constructor function.

The classic trick to accomplish this is to introduce a third class,
traditionally called the **surrogate**. Let's see the trick:

```javascript
function Animal (name) {
  this.name = name;
};

Animal.prototype.sayHello = function () {
  console.log("Hello, my name is " + this.name);
};

function Dog () {};

// The surrogate will be used to construct `Dog.prototype`.
function Surrogate () {};
// A `Surrogate` instance should delegate to `Animal.prototype`.
Surrogate.prototype = Animal.prototype;

// Set `Dog.prototype` to a `Surrogate` instance.
// `Surrogate.__proto__` is `Animal.prototype`, but `new
// Surrogate` does not invoke the `Animal` constructor function.
Dog.prototype = new Surrogate();

Dog.prototype.bark = function () {
  console.log("Bark!");
};
```

This is better, because it avoids improperly creating an `Animal`
instance when setting `Dog.prototype`. However, we still have the
problem that `Animal` won't be called when constructing a `Dog`
instance.

Now you might be wondering why we don't just call `Dog.prototype = Animal.prototype`
We don't do this because that means any function added to Dog would also be added to Animal.

Let's make one final tweak to the previous solution:

```javascript
function Dog (name, coatColor) {
  // call super-constructor function on **the current `Dog` instance**.
  Animal.call(this, name);

  // `Dog`-specific initialization
  this.coatColor = coatColor;
}
```

This pattern is especially useful if the superclass (`Animal`) has a
lot of initialization code. You could have copy-and-pasted the
`Animal` constructor code into `Dog`'s constructor, but calling the
`Animal` constructor itself is obviously much DRYer.

Note that we write `Animal.call` and not `new Animal`. Inside the
`Dog` constructor, we don't want to construct a whole new `Animal`
object. We just want to run the `Animal` initialization logic **on the
current `Dog` instance**. That's why we use `call` to call the
`Animal` constructor, setting `this` to the current `Dog` instance.

## Bonus: Object.create

You can accomplish the above without the surrogate using ECMAScript 5's
`Object.create` method, which returns a new object with its `__proto__`
set to its first argument. Here's the modified code:

```javascript
function Animal (name) {
  this.name = name;
}

Animal.prototype.sayHello = function () {
  console.log("Hello, my name is " + this.name);
};

function Dog (name, coatColor) {
  Animal.call(this, name);
  this.coatColor = coatColor;
};

// Set Dog.prototype to a new object whose
// __proto__ points to Animal.prototype.
Dog.prototype = Object.create(Animal.prototype);

Dog.prototype.bark = function () {
  console.log("Bark!");
};
```

ECMAScript 5 adds nice new features like this, but I want you to
understand JS very deeply, so we're not going to use `Object.create`.
Also, IE8 doesn't have this feature, so `Object.create` is sadly not
backward compatible.

But the future is bright!

## Exercises

### `inherits`

We learned how to implement class inheritance using a
`Surrogate`. There are a number of steps:

* Define a `Surrogate` class
* Set the prototype of the `Surrogate` (`Surrogate.prototype =
  SuperClass.prototype`)
* Set `Subclass.prototype = new Surrogate()`
* Set `Subclass.prototype.constructor = Subclass`

Write a `Function#inherits` method that will do this for you. Do not use
`Object.create` for this, you should deeply understand what the `new` keyword
does and how the `__proto__` chain is constructed. This
will help you in Asteroids today:

```javascript
function MovingObject () {};

function Ship () {};
Ship.inherits(MovingObject);

function Asteroid () {};
Asteroid.inherits(MovingObject);
```

How would you test `Function#inherits`? A few specs to consider:

0. You should be able to define methods/attributes on `MovingObject`
   that ship and asteroid instances can use.
0. Defining a method on `Asteroid`'s prototype should not mean that a
   ship can call it.
0. Adding to `Ship`/`Asteroid`'s prototypes should not change
   `MovingObject`'s prototype.
0. The `Ship` and `Asteroid` prototypes should not be instances of
   `MovingObject`.




