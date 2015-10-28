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
This ```Person``` constructor accepts two named parameters, ```name``` and ```age```, and assigns them to properties on the ```this``` object with the same names. It also adds a method called ```sayName()``` to the object. The ```this``` object is automatically created by using the ```new``` keyword when calling the constructor, and it is an instance of the constructor's type. It is implicitly return by the function, so there's no need to inlude a ```return``` statement within your constructor function. If you do, and the return value is an object, it will return the object instead of the newly created instance of the class. If the return value is a primitive, it will be ignored and the new instance will be returned.

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

