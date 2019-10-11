## Creating a Simple Router for you web app

### Routing:

> Routing is the process of selecting a path for traffic in a network or between or across multiple networks.

Im pretty sure that you all know what Router is and what it does. But, how to make one?

It's 4 basic steps:

- Add click handler to every anchor tag `<a>`
```javascript
document.addEventListener(
        "click",
        function(event) {
          if (event.target.tagName === "A") {
            event.preventDefault();
            var href = event.target.href;
            console.log(href);
          }
 }, false);
``` 

- On click handler, make get request to `href` of `<a>` and load html to current page.
```javascript
var xhr = new XMLHttpRequest();
xhr.onreadystatechange = function() {
  if (xhr.readyState === XMLHttpRequest.DONE) {
    history.pushState([], href, href);
    // Here we are replacing current page html using new page html
    document.getElementById("app").innerHTML = xhr.responseText;
  }
};
xhr.open("GET", href, true);
xhr.send(null);
``` 

- Push new page to [history](https://developer.mozilla.org/en-US/docs/Web/API/Window/history)
```javascript
// Add new history entry
history.pushState([], title, href);
``` 
- Add a navigation handler to know when user pressed back and forward on browser.
```javascript
// Listen to browser history event
window.addEventListener("popstate", function(e) {
  // Download new page's html and set it to current page 
  var xhr = new XMLHttpRequest();
  xhr.onreadystatechange = function() {
    if (xhr.readyState === XMLHttpRequest.DONE) {
      document.getElementById("app").innerHTML = xhr.responseText;
    }
  };
  xhr.open("GET", window.location.pathname, true);
  xhr.send(null);
});
``` 

Here is code sample on [codesandbox](https://swhhe.csb.app/ "(target|_blank)")

%[https://codesandbox.io/s/simple-router-example-swhhe?autoresize=1&fontsize=14&hidenavigation=1]

Above example can easily be implemented as a `<Link/>` Component in react.

```javascript
import React, { Component } from 'react';

export default class Link extends Component {

    constructor(props) {
        super(props);
        this.state = props;

        this.linkClicked = this.linkClicked.bind(this);
    }

    componentWillReceiveProps(nextProps) {
        // You don't have to do this check first, but it can help prevent an unneeded render
        if (nextProps.href !== this.state.href) {
            this.setState({ href: nextProps.href });
        }
        if (nextProps.title !== this.state.title) {
            this.setState({ title: nextProps.title });
        }
    }

    linkClicked(event) {
        if (!this.state.href || this.state.href.length <= 0) {
            event.preventDefault();
            return false;
        }
        if (typeof window != 'undefined') {
            event.preventDefault();
            history.pushState(this.state.title, this.state.title, this.state.href);
            window.title = this.state.title;
            history.pushState([], this.state.href, this.state.href);
            // Here we are replacing current page html using new page html
            // Set new state for your component here
        }
    }

    render() {
        return (
            <a href={this.state.href} title={this.state.title} style={this.state.style} className={this.state.className} onClick={this.linkClicked}>
                {this.props.children}
            </a>
        );
    }
}
``` 
