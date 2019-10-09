## Create Simple Router for you app

### Routing:

> Routing is the process of selecting a path for traffic in a network or between or across multiple networks.

Im pretty sure that you all know what Router is and what is does. But, how to make one?

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
    document.open("text/html");
    document.write(xhr.responseText);
    document.close();
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
      document.open("text/html");
      document.write(xhr.responseText);
      document.close();
    }
  };
  xhr.open("GET", window.location.pathname, true);
  xhr.send(null);
});
``` 


Here is code sample on [codesandbox](https://swhhe.csb.app/ "(target|_blank)")

%[https://codesandbox.io/s/simple-router-example-swhhe?autoresize=1&fontsize=14&hidenavigation=1]
