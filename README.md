# JS inheritance patterns

All these examples use *Object.assign()* for on-the-fly modifications at the time of object creation. IE was late to pick up *Object.assign()*, but there is a polyfill for it.

The first two ways are, in my opinion, probably the best (least confusing) ways to do it. The first doesn't use prototypes at all, and is easy to understand, at the cost of some inefficiency in memory usage. The second uses prototypes in a simple, non-magical way.

__Direct object modification__

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

__True object-based inheritance with Object.create()__

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

__Old-school pseudo-classes__

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
    Animal.call(this);
    Object.assign(this, params);
}
Duck.prototype = Object.create(Animal.prototype);
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

__ES6 classes__

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
