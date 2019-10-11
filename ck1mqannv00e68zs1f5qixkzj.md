## RxJS at its basic

## ReactiveX or Reactive Extensions
 [RxJS](https://github.com/ReactiveX/rxjs)  is javascript implementation of Reactive Extension, it allows you to use  [Observer pattern](https://en.wikipedia.org/wiki/Observer_pattern)  in your web app.

> RxJS is much more than what we are going to learn here, that is why the name **(RxJS at its basic)**

At its basic Observer pattern lets you *subscribe* to changes in *subject* with the help of *events*.

First we will focus on `Subscribers` and `Events`:

### Subscriber:
Object has a property named `subscribers` to manage subscribers who want to listen to changes in our object.
```javascript
let subscribers= []
``` 
When a subscriber arrives, object adds it to `subscribers` and when a subscriber wants to unsubscribe, subject should remove subscriber from his `subscribers`

### Event:
When data of subject is changed or when subject wants to inform changes to its subscribers it triggers an event. When an event is triggered, it is duty of a subject to inform every subscriber about the event. To do that we just have to iterate over all subscribers and execute their callbacks.

```javascript
this.subscribers.forEach(function(fn) {
  fn(data);
});
```

So let's come to the final step of a Observable Pattern and that is `Subject`
### Subject:
A subject which just an Object which informs its changes to all of its subscribers.

Here is an example of a general Observable Subject will extend.

```javascript
export default class ObservableSubject {
  constructor() {
    this.subscribers = [];
  }

  getAllSubscribers() {
    return this.subscribers;
  }

  /**
   * Register to a event
   *
   * @param {Function} fn subscriber callback function
   */
  subscribe(fn) {
    this.subscribers.push(fn);
  }

  /**
   * Un-Register from a event
   *
   * @param {Function} fn callback function to be removed
   */
  unsubscribe(eventName, fn) {
    if (this.subscribers) {
      for (var i = 0; i < this.subscribers.length; i++) {
        if (this.subscribers[i] === fn) {
          this.subscribers.splice(i, 1);
          break;
        }
      }
    }
  }

  /**
   * Trigger a event with data
   *
   * @param {Object} data optional data to pass
   */
  trigger(data) {
    if (this.subscribers) {
      this.subscribers.forEach(function(fn) {
        fn(data);
      });
    }
  }
}
``` 

So, let's make a observable User which will extent `ObservableSubject`

```javascript
import ObservableSubject from "./ObservableSubject";

export default class User extends ObservableSubject {
  setName(name) {
    this.name = name;
    this.trigger({
      user: {
        name: this.name,
        email: this.email
      }
    });
  }

  getName() {
    return this.name;
  }

  setEmail(email) {
    this.email = email;
    this.trigger({
      user: {
        name: this.name,
        email: this.email
      }
    });
  }

  getEmail() {
    return this.email;
  }
}
``` 

And this concludes a basic RxJs implementation.

Complete example is over at  [codesandbox](https://codesandbox.io/s/js-event-observer-bvjdd) 


%[https://codesandbox.io/embed/js-event-observer-bvjdd?autoresize=1&expanddevtools=1&fontsize=14&hidenavigation=1&module=%2Fsrc%2FObservableSubject.js]
