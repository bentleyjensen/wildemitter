# WildEmitter - A lightweight event emitter that supports wildcard handlers

## What's an event emitter?
If you've ever listened for a click event in a browser you've used an emitter. But, user interaction isn't the only thing that can trigger an event worth listening to. You can also make other objects capable of emitting events. That's what wildemitter is for. You can extend your objects with it so that you can emit events from them and register handlers on them. This pattern helps you write more re-usable code becaause your objct doen't have to know how it's going to be used. It can simply emit events any time something happens that other code *may* be interested in.

You'll see this type of pattern a lot in node.js. Where lots of things in the standard libraries inherit from EventEmitter and emit various events to indicate progress, errors, completion, etc.

So, why make another one? Aren't there others already?

Well, yes there are, but not quite what I wanted. This one is largely based on the emitter in @visionmedia's UIKit. So, much props to TJ for that. But there were a few more things I wanted. Specifically the following:
- Super lightweight
- Support for browser/node.js (browser use requires a CommonJS wrapper of some kind, like Stitch or Browserify)
- Support for wildcard handlers (`*` or `something*`)
- Support for grouping registered handlers and aun unbinding them all by their group name. This is really handy when, for example, you want unbind all handlers associated with a given "sub-page" within a single page app. 

##How do I use it?

```js
var Emitter = require('./wildemitter');

// the example object we're making
function Fruit(name) {
    this.name = name;
    
    // call emitter with this context
    Emitter.call(this);
}

// inherit from Emitter
Fruit.prototype = new Emitter;

// a function that emits an events when called
Fruit.prototype.test = function () {
    this.emit('test', this.name);
};

// set up some test fruits
var apple = new Fruit('apple');

apple.on('*', function () {
    console.log('"*" handler called', arguments);
});

apple.on('te*', function () {
    console.log('"te*" handler called', arguments);
});

apple.on('test', function () {
    console.log('"test" handler called', arguments);
});

// calling the method that emits events.
apple.test();

// it should write the following the log:
/*
"test" handler called { '0': 'apple' }
"*" handler called { '0': 'test', '1': 'apple' }
"te*" handler called { '0': 'test', '1': 'apple' }
*/

// this will remove any handlers explicitly listening for 'test' events.
apple.off('test');

// calling our method again would this time only call the two wildcard handlers
// producing the following output
/*
"*" handler called { '0': 'test', '1': 'apple' }
"te*" handler called { '0': 'test', '1': 'apple' }
*/

// grouped handlers example, we'll create another fruit
var orange = new Fruit('orange');

// In this case "today" is the name of the group.
// here we'll bind some handlers that all pass 'today'
// as the group name
orange.on('test', 'today', someHandler);
orange.on('someOtherEvent', 'today', someHandler);
orange.on('*', 'today', someHandler);

// we can now unbind all three of those handlers like this
orange.releaseGroup('today');
```

## Including
Emitters are often something you want to be able to include in another lib. There's also file called wildemitter-bare.js that doesn't have any export mechanism.

## Testing
You can run the tests with `nodeunit` by running: `nodeunit test.js`

## Credits

Written by [@HenrikJoreteg](http://twitter.com/henrikjoreteg) inspired by TJ's emitter component. 

Contributors:

- [Lance Stout](http://github.com/legastero)

##License
MIT

If you like this follow [@HenrikJoreteg](http://twitter.com/henrikjoreteg) on twitter.
