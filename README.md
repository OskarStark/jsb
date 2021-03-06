jsb README
=======================

Latest Release: [![GitHub version](https://badge.fury.io/gh/DracoBlue%2Fjsb.png)](https://github.com/DracoBlue/jsb/releases)

Build-Status: [![Build Status](https://travis-ci.org/DracoBlue/jsb.png?branch=master)](https://travis-ci.org/DracoBlue/jsb)

Official Site: <http://dracoblue.net/>

jsb is copyright 2010-2016 by DracoBlue <http://dracoblue.net>

What is Jsb?
--------------------

Jsb is very extendable Toolkit to inject Javascript Behaviour into
rendered HTML without Inline Javascript.

Requirements:

* Firefox 3+, Safari 5+, Chrome, Opera, IE7+
* (optional) requirejs - for on demand and subfolder loading

How does it work?
-----------------

The idea behind jsb is pretty simple. Put a class (jsb_) on all
elements which should be enriched/enhanced by javascript. Additionally
put a class `jsb_`*keyword* on the element to define which behaviour
should be applied to the element.

Each behaviour can register on such *keyword* by using this

    jsb.registerHandler('keyword', KeywordBehaviour);

method. As soon as the dom is loaded

    jsb.applyBehaviour(window.document);

is executed. You might even overwrite your Request.HTML method to do the
same.

Example
-------

Include the jsb into your website with the following meta
tag (before you define any behaviours):

    <script type="text/javascript" src="js/jsb.js"> </script>

Additionally add this one:

    <script type="text/javascript" src="js/Example.js"> </script>

Now create a new file `js/Example.js`

    Example = function(dom_element, options) {
       dom_element.textContent = 'I am loaded with name: ' + options.name;
    };

    jsb.registerHandler('Example', Example);

If you want to use requirejs integration, create it like this (no `registerHandler` necessary!):

    define("Example", [], function()
    {
        "use strict";

        var Example = function(dom_element, options) {
           dom_element.textContent = 'I am loaded with name: ' + options.name;
        };

        return Example;
    });

Now add somewhere in your html code the following:

    <span class="jsb_ jsb_Example" data-jsb="{&quot;name&quot;:&quot;Jan&quot;}" >Are you loaded?</span>

or with single attribute quotes:

    <span class="jsb_ jsb_Example" data-jsb='{"name": "Jan"}'>Are you loaded?</span>

When you execute the html page now, the text "Are you loaded?" won't display,
but will be replaced with 'I am loaded with name: Jan'.

It is also possible to use the query string syntax:

    <span class="jsb_ jsb_Example" data-jsb="name=Jan&amp;param1=one" >Are you loaded?</span>

Check out the generator functions for your favorite programming language.

If you want to have special data for one class, you might use `data-jsb-ClassName`.

    <span class="jsb_ jsb_Example" data-jsb-Example="{&quot;name&quot;:&quot;Jan&quot;}">Are you loaded?</span>

Foldernames must be replaced with dashes, so: `view/ui/Gui` becomes to `data-jsb-view-ui-Gui`.

Since jsb 2.0 it's also possible to put multiple classes on one element:

    <span class="jsb_ jsb_Example jsb_OtherExample" data-jsb="name=Jan&amp;param1=one" data-jsb-OtherExample="only=for-other">Are you loaded?</span>


Why an Extra jsb_-Class?
---------------------

One could expect to use `class="jsb_Example"` instead of `class="jsb_ jsb_Example"`.
But this is necessary, since searching for all elements which have a class `jsb_*`
is way slower then using the built in methods to search for one class `jsb_`.

You can use one of the Generators (or build your own) to make generation of those
tags easier.

Generator-Helpers
-----------------

### PHP

``` php
<?php
/**
 * @example <pre>
 *    <span class="jsb_ jsb_Example" data-jsb="<?php echo jsbOptions(array('name' => 'Jan')); ?>"></span>
 * </pre>
 */
function jsbOptions(array $options = array()) {
    return htmlspecialchars(json_encode($options));
}
?>
```

Advanced: Using with requirejs and bower
----------------------------------------

If you want to avoid to include all those script tags:

    <script type="text/javascript" src="js/Example.js"> </script>

for your behaviours, you may use requirejs.

Install jsb with bower:

``` console
$ bower install jsb
```

Inject a config to tell requirejs, where jsb lives in bower_components folder and
afterwards apply all behaviours on `document.body`:

     <script type="text/javascript" src="js/requirejs.js"> </script>
     <script>
         requirejs.config({
             baseUrl: './js/', // if your files live in the /js/ folder
             paths: {
                 jsb: './bower_components/jsb/jsb'
             }
         });

         require(['jsb'], function() {
             jsb.applyBehaviour(document.body);
         });
     </script>
     <script type="text/javascript" src="js/jsb.js"> </script>

Create a new file (`js/Example.js`), but don't include it with `<script>` into the head:

    define("Example", [], function()
    {
        "use strict";

        var Example = function(dom_element, options) {
           dom_element.textContent = 'I am loaded with name: ' + options.name;
        };

        return Example;
    });

And now just include your Behaviours in HTML, e.g.:

    <span class="jsb_ jsb_Example" data-jsb="name=Jan&amp;param1=one" />Are you loaded?</span>

If jsb notices, that the handler "Example" is not yet registered. Internally it will call `require` for "Example" and use the result
with `jsb.registerHandler`. Afterwards is the handler for "Example" defined.

This is very good if you want to keep the global namespace clean (since `var Example` defines a local variable). It's
also very nice, if you only want to load the element on demand!

You can even use sub folders of any required depth: Put the file into `js/mymodule/Example.js` and call it from html with
`class="jsb_ jsb_mymodule/Example.js"`.

Advanced: Communication between instances
-----------------------------------------

If you get used to `jsb`, you'll noticed that you have the need to communicate
between multiple jsb_-objects.


### jsb.fireEvent(`name`, `[values = {}, sticky = false`)

Since 1.3.0 jsb ships with a very simple (by design) event system. It is
framework independent and works with simple channel identifier and a
json-object as value.

    jsb.fireEvent('HoneyPot::CLICKED', {"name": "Bob", "times": 2});

This should be fired by a Js-Behaviour which needs to say something, instead
of global variables and direct call. This enables you to use dependency
injection if you keep the channel identifier the same.

If you set `sticky` to `true` or use the `jsb.fireStickyEvent` alias, you can retrieve multiple events with the same
name with `jsb.whenFired`.

### jsb.on(`name`, `[filter, ]` `callback`)

You can listen to that event, too:

    jsb.on(
        'HoneyPot::CLICKED', // identifier
        function(values, event_name) { // callback
            alert('The user ' + values.name + ' clicked it already ' + values.times);
        }
    );

It's even possible to filter for a filter-object when listening:

    jsb.on(
        'HoneyPot::CLICKED', // identifier
        {"name": "Bob"}, // filter everything with name = Bob
        function(values, event_name) { // callback
            alert('The user ' + values.name + ' clicked it already ' + values.times);
        }
    );

You may also use RegExp as channel identifier when calling `jsb.on`:

    jsb.on(
        /^HoneyPot.*$, // identifier which starts with HoneyPot*
        function(values, event_name) { // callback
            alert('The user ' + values.name + ' clicked it already ' + values.times + ' with event name: ' + event_name);
        }
    );

### jsb.off(`name`, `callback`)

Event handlers can be removed by passing the exact same name/regex and Function object to `jsb.off`.

    var counter = 0;
    var handler = function(){
        counter++
    };
    jsb.on('OFF_TEST', handler);
    jsb.fireEvent('OFF_TEST'); //counter is now 1
    jsb.off('OFF_TEST', handler);
    jsb.fireEvent('OFF_TEST'); //counter is still 1 because the listener was removed before the second event fired.

Alternatively `jsb.on` returns a function that can be called without any parameters and will remove the name/handler pair that was registered by `jsb.on` in that call.

    var counter = 0;
    var handler = function(){
        counter++
    };
    var off = jsb.on('OFF_TEST', handler);
    jsb.fireEvent('OFF_TEST'); //counter is now 1
    off();
    jsb.fireEvent('OFF_TEST'); //counter is still 1 because the listener was removed before the second event fired.

### jsb.whenFired(`name`, `[filter, ]` `callback`)

If the event may be triggered before your jsb class is loaded, you can use `jsb.whenFired`. Afterwards it behaves
the same like `jsb.on`.

    var counter = 0;
    jsb.fireEvent('MASTER_READY', { "key": "value"});
    jsb.whenFired(/^MASTER_READY$/, function(values, event_name) {
        /*
         * Will be called IMMEDIATELY because the event
         * was already fired.
         */
        counter++;
    });
    jsb.fireEvent('MASTER_READY', { "key": "value"});
    // counter is now 2!

If you use `fireStickyEvent` in favor of `fireEvent`, it's also possible to use whenFired for multiple events with the same name-

Advanced: Using with nodejs
----------------------------------------

If you want to run mocha tests or want to use the event system of jsb in nodejs, you can install jsb as npm package, too!

``` console
$ npm install node-jsb --save
```

In your source, you might use it like this:

``` javascript
var jsb = require('node-jsb');
jsb.on('Event::NAME', function() {
  console.log('Hi!');
});
jsb.fireEvent('Event::NAME');
```

Resources
----------

* Blog: <http://dracoblue.net/c/jsb/>
* Issue Tracker: <http://github.com/DracoBlue/jsb/issues>

Changelog
---------

See CHANGELOG.md

Thanks
-------
Thanks to hoffigk and graste for the discussions and feedback!

Contributors
------------
* Lars Laade https://github.com/larslaade
* Leon Weidauer https://github.com/lnwdr
* Steffen Gransow https://github.com/graste
* Benny Görlitz https://github.com/axten

License
--------

Jsb is licensed under the terms of MIT. See LICENSE for more information.
