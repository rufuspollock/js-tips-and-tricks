Javascript tips and tricks I've found useful. Contributions welcome (just
visit the source, make sure you are logged in and hit edit - or fork and
patch).

# Here beginneth the tips

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

if (typeof variable === undefined) {
  // do something
}


