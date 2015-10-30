#Object Oriented JavaScript

Since JavaScript is a functional programming language (everything is a function!), you don't have classes and instances like an object-oriented language has in the conventional sense. In order to simulate classes we use what are called constructors.

##Constructors

Constructors are just functions that you use with the ```new``` keyword to make new objects. You have already seen several built-in constructors in JavaScript:

Object, Array, Function, etc.. (Notice they are always capitalized!)

```javascript
var str = new String("Electroencephalographically");
var arr = new Array(10);
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

##Inheritance
We've seen how to create objects and simulate class-like behavior in JavaScript - the necessary first-step in understanding object oriented programming in JavaScript. The second step is learning about inheritance, which in JavaScript is called *Prototypal Inheritance*.

