# JS inheritance patterns

All these examples use Object.assign, which IE was late to pick up, but there is a polyfill for it.

__Simple(ish) inheritance__

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

__Object.create() based inheritance__

```javascript
var animal = Object.create(null);
animal.x = 100;
animal.y = 100;

animal.move = function (dx, dy) {
    this.x += dx;
    this.y += dy;
}

animal.locate = function () {
    console.log([this.x, this.y]);
}

// ------------------------

var duck = Object.create(animal);
duck.speak = function () {
    console.log("quack");
}

// ------------------------

var daffy = Object.assign(Object.create(duck), {x: 5});
daffy.move(20, 2);
daffy.locate();
daffy.speak();
```

__Old-school "new" keyword__

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
