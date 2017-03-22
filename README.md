# JS inheritance patterns

All these examples use *Object.assign()* for on-the-fly modifications at the time of object creation. IE was late to pick up *Object.assign()*, but there is a polyfill for it.

___Direct object modification___

This is the simplest, clearest way. The only problem is some inefficiency in memory usage, and perhaps some slowness in object construction and garbage collection?

```javascript
function new_animal(params) {
    var animal = {};
    animal.x = 100;
    animal.y = 100;

    animal.move = function (dx, dy) {
        this.x += dx;
        this.y += dy;
    }

    animal.locate = function () {
        console.log([this.x, this.y]);
    }

    Object.assign(animal, params);
    return animal;
}

// ------------------------

function new_duck(params) {
    var duck = new_animal();

    duck.speak = function () {
        console.log("quack");
    }

    Object.assign(duck, params);
    return duck;
}

// ------------------------

var daffy = new_duck({x: 5});
daffy.move(20, 2);
daffy.locate();
daffy.speak();
```

___True object-based inheritance with Object.create()___

Here the prototype chain is clear and explicit, as long as you understand what *Object.create()* actually does, which is create an empty object that has its hidden *[[prototype]]* property point at the specified thing. So the hidden *[[prototype]]* of *daffy* points at *Duck*, and the hidden *[[prototype]]* of *Duck* points at *Animal*.

An attempt to use a property of *daffy* will go up this chain, if necessary. Thus, calling *daffy.move()* works.

```javascript
var Animal = Object.create(null);
Animal.x = 100;
Animal.y = 100;

Animal.move = function (dx, dy) {
    this.x += dx;
    this.y += dy;
}

Animal.locate = function () {
    console.log([this.x, this.y]);
}

// ------------------------

var Duck = Object.create(Animal);
Duck.speak = function () {
    console.log("quack");
}

// ------------------------

function make(base, params) {       // Just to reduce verbiage.
    return Object.assign(Object.create(base), params);
}

var daffy = make(Duck, {x: 5});
daffy.move(20, 2);
daffy.locate();
daffy.speak();
```

___Old-school pseudo-classes___

When *new Foo()* is called, the keyword *this* points at the new object. The hidden *[[prototype]]* property of the new object is automatically pointed at *Foo.prototype* (an ordinary object, and **not** the actual *[[prototype]]* of Foo). This object *Foo.prototype* starts off as simply **{constructor: Foo}**, though this property is not enumerable.

To make the *Duck* subclass, we make a function *Duck()* and overwrite its property *Duck.prototype* to be a new object that has its hidden *[[prototype]]* property point at *Animal.prototype*. The call to *Object.create()* makes this happen. We have to explicitly set *Duck.prototype.constructor*, otherwise accessing *daffy.constructor* will go up the chain to *Duck*, fail to find it, and go further up the chain to *Animal.prototype*, find it, and return the (wrong) answer, *Animal*.

Anyway, the final *daffy* object has its hidden *[[prototype]]* property point at the object *Duck.prototype* (an ordinary object), which has its hidden *[[prototype]]* property point at *Animal.prototype* (an ordinary object).

An attempt to use a property of *daffy* will go up this chain, if necessary. Thus, calling *daffy.move()* works.

Clear as mud.

```javascript
function Animal(params) {
    this.x = 100;
    this.y = 100;
    Object.assign(this, params);
}
Animal.prototype.move = function (dx, dy) {
    this.x += dx;
    this.y += dy;
}
Animal.prototype.locate = function () {
    console.log([this.x, this.y]);
}

// ------------------------

function Duck(params) {
    Animal.call(this);              // Runs Animal() with its "this" set to our "this".
    Object.assign(this, params);
}
Duck.prototype = Object.create(Animal.prototype);   // Now Duck.prototype.[[prototype]] points at Animal.prototype
Duck.prototype.constructor = Duck;
Duck.prototype.speak = function () {
    console.log("quack");
}

// ------------------------

var daffy = new Duck({x: 5});
daffy.move(20, 2);
daffy.locate();
daffy.speak();
```

___ES6 classes___

This is really just syntactic sugar for the above pseudo-classes.

```javascript
class Animal {
    constructor(params) {
        this.x = 100;
        this.y = 100;
        Object.assign(this, params);
    }
    move(dx, dy) {
        this.x += dx;
        this.y += dy;
    }
    locate() {
        console.log([this.x, this.y]);
    }
}

// ------------------------

class Duck extends Animal {
    constructor(params) {
        super();
        Object.assign(this, params);
    }
    speak() {
        console.log("quack");
    }
}

// ------------------------

var daffy = new Duck({x: 5});
daffy.move(20, 2);
daffy.locate();
daffy.speak();
```
