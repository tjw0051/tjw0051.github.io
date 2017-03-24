---
layout:     post
title:      "Classes and Events in node.js"
subtitle:   "Exporting classes, receiving events, emitting events."
date:       2017-01-08 12:30:00
author:     "Tom"
header-img: "img/2017-01-08_cover.jpg"
header-mask: 0.5
catalog: true
tags:
    - Javascript
    - node.js
    - Classes
    - Events
---

# Overview

Since the release of ECMAScript 6 it is now possible to create classes in a more conventional object-oriented way. A previous method of creating 'classes' in Javascript was to define a function as a class, with variables stored in the object via `this` and sub-functions to the 'class' or to the prototype, like so:

```javascript
function CarClass (constructorVariable) {
    this.internalVariable = constructorVariable;

    function myMethod = function(methodParameter) {
        return this.internalVariable;
    }
}

CarClass.prototype.protoMethod = function() {
    return this.internalVariable;
}
```

The same class defined in ES6 could look like this:

```javascript
class CarClass {
    constructor(constructorVariable) {
        this.internalVariable = constructorVariable;
    }

    myMethod(methodParameter) {
        return this.internalVariable;
    }
```

For a more detailed description of classes in ES6, check out [Mozilla's very helpful Javascript Reference](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Classes).

I had a situation recently where I had to send and receive events from a class which turned out to be more confusing than I initially thought. Being rather new to Javascript but familiar with more object-oriented languages, I made some incorrect assumptions about how classes work in Javascript and spent a good amount of time scouring stack overflow for the wrong questions.

Without further ado, here's some quick snippets on how to implement some common event features with classes.

---

# Sending Events from Classes

This is pretty straightforward and not unlike many other languages. Firstly lets get the events module:

```javascript
const EventEmitter = require('events').EventEmitter;
```

Next we create our class and give it a constructor. Its important to note that if you create a constructor and leave it empty, it will throw errors. If you don't need a constructor don't include one:

```javascript
class Car {
    constructor(name) {
        // Don't leave this empty
        this.name = name;
        
    }
}
```

To send events from our class we need to extend the EventEmitter class:

```javascript
class Car extends EventEmitter {
```

For extending the Event Emitter we also need to call the constructor of the superclass in our constructor:

```javascript
class Car extends EventEmitter {
    constructor(name) {
        super();
        this.name = name;
    }
}
```

Finally, lets add another method to the class and make it fire off an event:

```javascript
class Car extends EventEmitter {
    constructor(name) {
        super();
        this.name = name;
    }

    openDoor() {
        this.emit('doorOpen', name);
    }
}
```

When the class is instantiated and the `openDoor()` method is called, the class will emit the 'doorOpen' event, passing the name of the vehicle that was defined in the constructor.

We also need to make our class accessible by other scripts. At the end of the script, add the following line to export the module:

```javascript
module.exports = Car;
```

To receive events from our class, make a new script and import the module we just created:

```javascript
var Car = require('./Car.js');

var myCar = new Car('mclaren');
```

Next we create a handler to fire when the `openDoor()` module is called:

```javascript
var Car = require('./Car.js');

var myCar = new Car('mclaren');

myCar.on('doorOpen', function(name) {
   console.print('Door has been opened'); 
});
```

Now we can open the car door, triggering the callback:

```javascript
myCar.openDoor();
```

# Receiving Events inside a Class

Another situation I encountered was trying to receive events within a class. A good place to set these up (unless we want to create callbacks from a method) is in the constructor:

```javascript
class Car extends EventEmitter {
    constructor(name) { // mclaren
        super();
        this.name = name;

        var testCar = new Car('ferrari');
        testCar.on('doorOpen', function(name) {
           console.print('Door has been opened'); 
        });
    }
```

Now when the doorOpen event is fired from the testCar, the code within our callback will be executed.

# Referencing Class Variables inside Closure

Say we wanted to print the name of the car from inside the testCar.on callback. Intuition says the way to do this is:

```javascript
 constructor(name) { // mclaren
        super();
        this.name = name;

        var testCar = new Car('ferrari');
        testCar.on('doorOpen', function(name) {
           console.print('Door has been opened on: ' + this.name); 
        });
    }
```

However, this won't print the name of the enclosing vehicle, because the scope of `this` is limited to within the closure, and cannot access the outside class. Therefore we need to store a reference to the enclosing class so that we can access it from within the closure:

```javascript
 constructor(name) { // mclaren
        super();
        this.name = name;

        var self = this;

        var testCar = new Car('ferrari');
        testCar.on('doorOpen', function(name) {
           console.print('Door has been opened on: ' + self.name); 
        });
    }
```

Now the name of the enclosing class (mclaren) will be printed to the console. This is also sometimes referred to as the this-that trick.

—— Tom 2017.01
