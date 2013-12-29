---
layout: default
title: Home
---

Javascript tips and tricks I've found useful. Contributions welcome -- just
visit the source, make sure you are logged in and hit edit, or, [fork and
pull](https://help.github.com/articles/using-pull-requests).

**Table of Contents**

* Something random here that will be deleted
{:toc}

# Object Operations

## Deep Copy

**Warning**: full deep copy is **hard**. The following are only appropriate for "simple" objects.

### JQuery

Copy (clone) in javascript with JQuery::

    // Shallow copy
    var newObject = jQuery.extend({}, oldObject);

    // Deep copy
    var newObject = jQuery.extend(true, {}, oldObject);

### Underscore

Note underscore *does* not do a deep copy at all -- it will only do copy on
first level attributes::

    _.extend({}, myobject);

### JSON

Deserialize to and from JSON (warning: not at all performant!)::

    var newobj = JSON.parse(JSON.stringify(oldobj));


## Check if a variable is undefined

    // The typeof operator always returns a string!
    if (typeof(variable) === "undefined") {
      // do something
    }

## Simple Inheritance in Javascript

### John Resig's original

From <http://ejohn.org/blog/simple-javascript-inheritance/>

    /* Simple JavaScript Inheritance
     * By John Resig http://ejohn.org/
     * MIT Licensed.
     */
    // Inspired by base2 and Prototype
    (function(){
      var initializing = false, fnTest = /xyz/.test(function(){xyz;}) ? /\b_super\b/ : /.*/;
      // The base Class implementation (does nothing)
      this.Class = function(){};
      
      // Create a new Class that inherits from this class
      Class.extend = function(prop) {
        var _super = this.prototype;
        
        // Instantiate a base class (but only create the instance,
        // don't run the init constructor)
        initializing = true;
        var prototype = new this();
        initializing = false;
        
        // Copy the properties over onto the new prototype
        for (var name in prop) {
          // Check if we're overwriting an existing function
          prototype[name] = typeof prop[name] == "function" && 
            typeof _super[name] == "function" && fnTest.test(prop[name]) ?
            (function(name, fn){
              return function() {
                var tmp = this._super;
                
                // Add a new ._super() method that is the same method
                // but on the super-class
                this._super = _super[name];
                
                // The method only need to be bound temporarily, so we
                // remove it when we're done executing
                var ret = fn.apply(this, arguments);        
                this._super = tmp;
                
                return ret;
              };
            })(name, prop[name]) :
            prop[name];
        }
        
        // The dummy class constructor
        function Class() {
          // All construction is actually done in the init method
          if ( !initializing && this.init )
            this.init.apply(this, arguments);
        }
        
        // Populate our constructed prototype object
        Class.prototype = prototype;
        
        // Enforce the constructor to be what we expect
        Class.prototype.constructor = Class;

        // And make this class extendable
        Class.extend = arguments.callee;
        
        return Class;
      };
    })();

Usage::

    var Person = Class.extend({
      init: function(isDancing){
        this.dancing = isDancing;
      }
    });
    var Ninja = Person.extend({
      init: function(){
        this._super( false );
      }
    });

    var p = new Person(true);
    p.dancing; // => true

    var n = new Ninja();
    n.dancing; // => false 

### Steffen Rusitchka version

    /* Steffen Rusitchka version
     * <http://www.ruzee.com/blog/2008/12/javascript-inheritance-via-prototypes-and-closures>
     * Creative Commons Attribution licensed
     */
    (function(){
      var isFn = function(fn) { return typeof fn == "function"; };
      PClass = function(){};
      PClass.create = function(proto) {
        var k = function(magic) { // call init only if there's no magic cookie
          if (magic != isFn && isFn(this.init)) this.init.apply(this, arguments);
        };
        k.prototype = new this(isFn); // use our private method as magic cookie
        for (key in proto) (function(fn, sfn){ // create a closure
          k.prototype[key] = !isFn(fn) || !isFn(sfn) ? fn : // add _super method
            function() { this._super = sfn; return fn.apply(this, arguments); };
        })(proto[key], k.prototype[key]);
        k.prototype.constructor = k;
        k.extend = this.extend || this.create;
        return k;
      };
    })();

Usage::

    var X = PClass.create({
      val: 1,
      init: function(cv) { 
        this.cv = cv; 
      },
      test: function() { 
        return [this.val, this.cv].join(", "); 
      }
    });

    var Y = X.extend({
      val: 2,
      init: function(cv, cv2) { 
        this._super(cv); this.cv2 = cv2; 
      },
      test: function() { 
        return [this._super(), this.cv2].join(", "); 
      }
    });

    var x = new X(123);
    var y = new Y(234, 567);

## Test whether an object is of a certain type

Test whether it is an Array:

    obj instanceof Array

Another method if constructor is present

    if (output.constructor == Array) {
    }
    else if(output.constructor == Object) {
    }

Test whether it is a string of other simple type:

    typeof obj === 'string'
    typeof obj === 'number'
    typeof obj === 'boolean'


# String functions

## Strip / Trim

Available in most modern browsers at the .trim() function but not available in IE7!

    mystring = mystring.replace(/^\s+|\s+$/g, '');                     

## Contains

Test whether one string contains another. Solution is to use `indexOf`. For example, to test whether myString contains 'levin':

    myString.indexOf('levin') != -1

## Ends With

Does a string end with the given suffix:

    mystring.indexOf(suffix, mystring.length - suffix.length) !== -1;

# Miscellaneous

## Pretty-print JSON

    JSON.stringify({a: "lorem", b: "ipsum"}, null, 4);
    JSON.stringify({a: "lorem", b: "ipsum"}, null, '\t');


## Parse a Query String

Parse a URL query string (?xyz=abc...) into a dictionary.

    parseQueryString = function(q) {
      if (!q) {
        return {};
      }
      var urlParams = {},
        e, d = function (s) {
          return decodeURIComponent(s.replace(/\+/g, " "));
        },
        r = /([^&=]+)=?([^&]*)/g;

      if (q && q.length && q[0] === '?') q = q.slice(1);
      while (e = r.exec(q)) {
        // TODO: have values be array as query string allow repetition of keys
        urlParams[d(e[1])] = d(e[2]);
      }
      return urlParams;
    };

## Format Numbers Nicely

    var formatAmount = function (num) {
      var billion = 1000000000;
      var million = 1000000;
      var thousand = 1000;
      var numabs = Math.abs(num);
      if (numabs > billion) {
        return num / billion + 'bn';
      }
      if (numabs > (million / 2)) {
        return num / million + 'm';
      }
      if (numabs > thousand) {
        return num / thousand + 'k';
      } else {
        return num;
      }
    };

    var formatAmountWithCommas = function (num, decimalPlaces) {
      if (decimalPlaces === undefined) {
        decimalPlaces = 0;
      }
      num = num.toFixed(decimalPlaces);
      x = num.split('.');
      x1 = x[0];
      x2 = x.length > 1 ? '.' + x[1] : '';
      var rgx = /(\d+)(\d{3})/;
      while (rgx.test(x1)) {
        x1 = x1.replace(rgx, '$1' + ',' + '$2');
      }
      return x1 + x2;
    };

# Javscript Module Pattern

Readings:

* [JavaScript Module Pattern: In-Depth by Ben Cherry](http://www.adequatelygood.com/2010/3/JavaScript-Module-Pattern-In-Depth)

### Modules that work in NodeJS and the Browser

Have your normal (browser) JS code then do:

    // export to NodeJS or RequireJS if they are present ...
    var safeExport = function (publicMethods) {
      // NodeJS
      if (typeof module !== 'undefined' && module !== null) module.exports = publicMethods;
      // RequireJS
      if (typeof define === "function") define([], publicMethods);
    };

    // now setup the export
    module = {publicFunc1: publicFunc1, publicFunc2: publicFun2};
    safeExport(module)

