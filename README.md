# object-oriented-javascript

> Course summary from Udacity.

### Object decorator pattern

Code to create many car objects

```js
var amy = {loc: 1}
move(amy); // takes any car like object
var ben = {loc: 9}
move(ben); // takes any car like object
```

Augment an existing object to be treated like a car.

```js
var amy = carlike({}, 1);
move(amy); // takes any car like object
var ben = carlike({}, 9);
move(ben); // takes any car like object

function carLike(obj, loc) {
  obj.loc = loc;
  return obj;
}
```

But functions can be properties too on any object hence:

```js
var amy = carlike({}, 1);
amy.move();
var ben = carlike({}, 9);
ben.move();

var move = function() {
  this.loc++;
};


function carLike(obj, loc) {
  obj.loc  = loc;
  obj.move = move;
  return obj;
}
```

*Note: All the objects above share the same move function.

Alternatively we can move the global function inside the function:

```js
function carLike(obj, loc) {
  obj.loc  = loc;
  obj.move = function() {
    this.loc++;
  };
  return obj;
}
```

The disadvantage here is memory wastage because of the new function object
create for every move method. The author says that we can gain slight advantage
by using `obj` instead of `this` because each function has unique access to
the closure scope created.

### Conclusion

The decorator pattern is used to augment functionality to the existsing object.

## Functional Classes

The only difference is that decorator needs an object whereas in this case it creates
one by itself.

```js
var amy = Car(1);
amy.move();
var ben = Car(9);
ben.move();

var Car = function(loc) {
  var obj = {loc: loc};
  obj.move = move;
  return obj;
};

var move = function() {
  this.loc++;
};

// But here we are writing `move` at many places.

var Car = function(loc) {
  var obj = {loc: loc};
  extend(obj, Car.methods);
  return obj;
};
Car.methods = function() {
  move: funciton() {
    this.loc++;
  }
};

```

### Conclusion

> Any function that produces an object which has same behavior and properties
> then that function could be called class.

## Prototypal Class

Instead of copying methods on each instance we can make the use of facility
provided by javascript which is any missing property lookup is delegated to
the linked object called the prototype of the object.

```js
var Car = function(loc) {
  //var obj = {loc: loc}; its delegation is not helpful
  var obj = Object.create(Car.methods);
  obj.loc = loc;
  //extend(obj, Car.methods); now, we can get rid of it now
  return obj;
};
Car.methods = function() {
  move: funciton() {
    this.loc++;
  }
};
```

The above pattern is so common, the language has provided us support for it
int the form of `prototype`.

```js
Car.prototype.move = funciton() {
  this.loc++;
};
```

Remember we are still creating new objects via: `var amy = Car(1)`

### Conclusions

Amy's prototype is `Car.prototype`, it means failed property lookups from Amy
is delegated to the `Car.prototype` object.

There is a `constructor` property on the `Car.prototype` which points back to
the `Car` function object thus creating a circular reference.

Operator `instanceof` checks if right hand side's `prototype` object can be
can be found in the left hand side operand's prototype chain.

## Psuedoclassical Pattern

There are still repetitive code while creating objects, can we use language
features to remove repetitive code?

```js
var Car = function(loc) {
  var obj = Object.create(Car.prototype); // repetitive code
  obj.loc = loc;
  return obj;                             // repetitive code
};

Car.prototype.move = funciton() {
  this.loc++;
};
```

By adding `new` keyword during function invocation our function gets new powers
known as `Constructor` mode, running in this mode the JS engine does the work:

```js
var Car = function(loc) {
  //this = Object.create(Car.methods);
  this.loc = loc;
  //return this;                       
};

var amy = new Car(1);
```

### Conclusion

Apart from the syntactic differences between the previous two patterns, the
Javascript engines optimizes code while running in `Psuedoclassical` mode.

## The big picture

All object has some similarities and differences between them. In `Psuedoclassical`
pattern this concept becomes pretty clear, the similarities are stored as the
property of the `prototype` object.

The function body contains the differences in instances.

In the `Functional` pattern this difference is not that clear.

### Superclass and Subclass

We now know how to create a fleet of similar objects but what if we need another
category of objects which has a slightly different behaviour? We could do the
following:

```js
var amy = Van(1);
amy.move();
var ben = Van(9);
ben.move();
var cal = Car(2);
cal.move();
cal.call();

var Van = function(loc) {
  var obj  = {loc: loc};
  obj.move = function() {
    obj.loc++;
  };
  obj.grab = function() {/*...*/};
  return obj;
};

var Cop = function(loc) {
  var obj  = {loc: loc};
  obj.move = function() {
    obj.loc++;
  };
  obj.call = function() {/*...*/};
  return obj;
};
```

But we can see that we are repeating a lot of code, can we refactor common
logic?

```js
var Car = function(loc) {
  var obj  = {loc: loc};
  obj.move = function() {
    obj.loc++;
  };
  return obj;
};

var Van = function(loc) {
  var obj  = Car(loc);
  obj.grab = function() {/*...*/};
  return obj;
};

var Cop = function(loc) {
  var obj  = Car(loc);
  obj.call = function() {/*...*/};
  return obj;
};
```

### Psuedoclassical Subclasses

How to achieve the above behavior using the `Psuedoclassical` pattern?

```js
var zed = new Car(3);
zed.move();

var amy = new Van(9);
amy.move();
amy.grab();

var Car = function(loc) {
  this.loc;
};
Car.prototype.move = funciton() {
  this.loc++;
};

var Van = function(loc) {
  // this = Object.create(Van.prototype); // behind the scenes
  //new Car(loc);         // a new instance is created as a side effect, bad option :(
  Car.call(this, loc);
};
```

Till now we have seen how to inherit differntiated code. What if we want to
inherit similar code?

```js
Van.prototype = Car.protoype;
```

Above approach has a flaw. What if we need to add some common
behvior to the Van.prototype?
That would be very hurting because all the instances of Van as
well as Car would have the same behavior, hence the correct
approach should be:

```js
Van.prototype = Object.create(Car.protoype);
Van.prototype.grab = function() { /*...*/ };
```

Since we replaced the original `Van.prototype` object with the
new one we lost some important properties on it too.

```js
Van.prototype.constructor = Van;
```
