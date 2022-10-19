# What makes your web app faster? (part 4, Progressive Enhancements)

# What makes your web app faster? (part 4, Browser And Progressive Enhancements)


If you are coming to this story directly, i’ll suggest to take a look at [part 1](/tej-writes/what-makes-your-web-app-faster-part-1-databases-c3899c402d72), [part 2](/tej-writes/what-makes-your-web-app-faster-part-2-caching-f24cc9a960e5) and [Part 3](/tej-writes/what-makes-your-web-app-faster-part-3-client-app-4ec0d4fde428) of this series.

So, last time i left you saying,

> “Be indistinguishable from native apps.”

As a web developers we were successful in convincing user to leave there Native Desktop Apps for a web app, Now only few people use an native email client over Gmail. But we are still not able to make user leave there native mobile app for a mobile web app.

This is because a native app feels more comfortable smooth and responsive than a web app. A native app has Push Notification for engagement, Device storage for offline access and much more. But this is changing quickly as web can do more things, check [_what web can do today_](https://whatwebcando.today/).



## * Progressive Web App

PWA (Progressive Web App) is the next step in making web feel more native to a device.

PWA give you access to native capabilities such as:

1.  Offline Storage (Service Worker)
2.  Push Notification (Service Worker)
3.  Install as Native (App Manifest)
4.  Open app links (App Manifest)

> To use PWA you must be using https

PWA has 2 main Service Worker and Web App Manifest

## [1\. Service Worker:](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API)

> Service workers essentially act as proxy servers that sit between web applications, the browser, and the network (when available). They are intended, among other things, to enable the creation of effective offline experiences, intercept network requests and take appropriate action based on whether the network is available, and update assets residing on the server. They will also allow access to push notifications and background sync APIs.

To read more about service worker goto [_MDN_](http://Service workers essentially act as proxy servers that sit between web applications, the browser, and the network (when available). They are intended, among other things, to enable the creation of effective offline experiences, intercept network requests and take appropriate action based on whether the network is available, and update assets residing on the server. They will also allow access to push notifications and background sync APIs.).

## [2\. Web App Manifest:](https://developer.mozilla.org/en-US/docs/Web/Manifest)

> The web app manifest provides information about a web application in a JSON text file, necessary for the web app to be downloaded and be presented to the user similarly to a native app (e.g., be installed on the homescreen of a device, providing users with quicker access and a richer experience). PWA manifests include its name, author, icon(s), version, description, and list of all the necessary resources (among other things).

To read more about web app manifest goto [_MDN_](https://developer.mozilla.org/en-US/docs/Web/Manifest).

* * *

## Other Enhancements:

As we know javascript executes all operations in a single thread. It becomes a problem while going more intensive tasks such as image processing. To overcome this situation we can transfer that task to [**_Web Worker_**](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API) which will execute that task in a background thread. You cannot touch any DOM element from Web Worker and only transfer data using messages. To read more about Web Worker goto [_MDN_](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API).