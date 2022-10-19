# Create your own Event Emitter in JS

In my  [previous post](https://blog.tejpratapsingh.com/rxjs-at-its-basic-ck1mqannv00e68zs1f5qixkzj), i wrote about how we can design you app using Observer Pattern.

Now we will try something simpler and which works across our application and not bound to a `Object`.

It has some interesting use cases like listening to Route Change so you can change page header and show logged in user info Or listening an event when next page is loaded.

Event emitter can be achieved with 3 simple steps:

1. Register event
3. Trigger event
2. Unregister that event

___
So, let's Start:

### Initialisation
First thing first, we need a global variable to hold all events and their callbacks like this:
```javascript
// events property will hold our events and all the listeners it has.
this.events = {};
```

And if you are using ES6 classes, we have to declare this class as singleton, to do that we simply create a global variable and check if variable is initialised or not.
```javascript
let eventManagerInstance = null;
export default class EventManager {
  constructor() {
    if (!eventManagerInstance) {
      console.log("event inttialised");
      eventManagerInstance = this;
      // events hold our events and their callbacks
      this.events = {};
      // eventTypes can be used to keep track of all the events that can be used by this event manager.
      this.eventTypes = {
        ROUTE_CHANGED: "changeRoute",
        SCROLL_DIRECTION_CHANGED: "scrollDirectionChanged",
        SCROLL_LOAD_NEXT_PAGE: "scrollLoadNextPage"
      };
    }
    return eventManagerInstance;
  }
}
```

### Register event in Event Manager
To register a event we need 2 parameters, a unique name for event and a callback function.

```javascript

/**
   * Register to a event
   *
   * @param {string} eventName Name of event to register for
   * @param {Function} fn callback function
   */
  on(eventName, fn) {
    console.log("event On: " + eventName);
    this.events[eventName] = this.events[eventName] || [];
    this.events[eventName].push(fn);
  }
```

### Trigger that event
After we set up a event listener we can send data via triggers like this,
```javascript
/**
   * Trigger a event with data
   *
   * @param {string} eventName Event to call
   * @param {Object} data optional data to pass
   */
  trigger(eventName, data) {
    console.log("event triggered: " + eventName);
    if (this.events[eventName]) {
      this.events[eventName].forEach(function(fn) {
        fn(data);
      });
    }
  }
```

### Unregister that event
Once we are done listening to a particular event we should be able to unregister.
```javascript
  /**
   * Un-Register from a event
   *
   * @param {string} eventName Name of event to de-register from
   * @param {Function} fn callback function to be removed
   */
  off(eventName, fn) {
    console.log("event Off: " + eventName);
    if (this.events[eventName]) {
      for (var i = 0; i < this.events[eventName].length; i++) {
        if (this.events[eventName][i] === fn) {
          this.events[eventName].splice(i, 1);
          break;
        }
      }
    }
  }
```
___
Here is the full implementation of Event Manager class.
```javascript
// This class is implemented as SINGLETON(singleton)
let eventManagerInstance = null;

export default class EventManager {
  constructor() {
    if (!eventManagerInstance) {
      console.log("event inttialised");
      eventManagerInstance = this;
      this.events = {};
      this.eventTypes = {
        ROUTE_CHANGED: "changeRoute",
        SCROLL_DIRECTION_CHANGED: "scrollDirectionChanged",
        SCROLL_LOAD_NEXT_PAGE: "scrollLoadNextPage"
      };
    }
    return eventManagerInstance;
  }

  /**
   * Register to a event
   *
   * @param {string} eventName Name of event to register for
   * @param {Function} fn callback function
   */
  on(eventName, fn) {
    console.log("event On: " + eventName);
    this.events[eventName] = this.events[eventName] || [];
    this.events[eventName].push(fn);
  }

  /**
   * Un-Register from a event
   *
   * @param {string} eventName Name of event to de-register from
   * @param {Function} fn callback function to be removed
   */
  off(eventName, fn) {
    console.log("event Off: " + eventName);
    if (this.events[eventName]) {
      for (var i = 0; i < this.events[eventName].length; i++) {
        if (this.events[eventName][i] === fn) {
          this.events[eventName].splice(i, 1);
          break;
        }
      }
    }
  }

  /**
   * Trigger a event with data
   *
   * @param {string} eventName Event to call
   * @param {Object} data optional data to pass
   */
  trigger(eventName, data) {
    console.log("event triggered: " + eventName);
    if (this.events[eventName]) {
      this.events[eventName].forEach(function(fn) {
        fn(data);
      });
    }
  }
}
```
___
### How to use EventManager class
Using event manager is simple, you just have to subscribe to a event like this:
```javascript
// Get eventManager instance
this.eventManager = new EventManager();

// Declare Event Callback Function, so we can unregister it as sometime.
this.eventCallbackhandler = function(eventData) {
  console.log(eventData);
};

// Subscribe to ROUTE_CHANGED event
this.eventManager.on(
  this.eventManager.eventTypes.ROUTE_CHANGED,
  this.eventCallbackhandler
);
```

To Trigger ROUTE_CHANGED from different component, we just have to
```javascript
emitEvent() {
  const eventManager = new EventManager();
  eventManager.trigger(eventManager.eventTypes.ROUTE_CHANGED, {
    eventSource: "EventEmitterComponent"
  });
}
```
___

Here is a working sample on  [codesandbox.io](https://codesandbox.io/s/js-event-manager-xnimu) 


%[https://codesandbox.io/embed/js-event-manager-xnimu?fontsize=14]
