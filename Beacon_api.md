# Beacon API
> A brief explanation about the browser's Beacon API

Modern browsers are increasingly venturing into the native interaction with the hardware, they offer us an small, but helpful [list of APIs](https://developer.mozilla.org/en-US/docs/Web/API) to have fun.

Some of them have a more native interactions, and some of them are more like utilities that can help developers.

This is the case for the **Beacon API**


## The Problem

It's very likely you'll find yourself, at some point of your web developer career, working with **analytics.** This type of scripts often need to do something (eg. reporting) when the `document` triggers an `unload` event, normally would be a call to a server maybe to let know that a user just left the page just before checkout (ouch!)

The first piece of code that comes to my mind when I encounter this scenario looks like this:

```js
window.onunload = function userLeave() {
  // Do the http request to my analytics server
}
```

Well, turns out that browsers, ignore asynchronous http calls that happen during an `unload` event.

Again, first solution that comes to my mind: **let's make it synchronous**. Well, although it works, doing a synchronous action inside the `unload` event will force the browsers to **wait** until this event has completely finish, therefore the next page will feel slow to load.

There are other techniques to handle this scenario, but they all involves a **bad code practice** and affect the user navigation

## Send a Beacon! 🗼🗼

To solve this issue, the browsers offers us the `Beacon API`.

The `navigator` object has a method called `sendBeacon()`, this method receives 2 parameters: `url` and `options`.

The `url` parameter represents the url of the server you want to send the request, and the `options` parameter (**optional**) can be a simple **String**, or if you want to send more complex data, you could also pass an `ArrayBufferView`, `Blob`, `DOMString`, or `FormData`.

This function schedules an http **POST** request using the parameters you passed in (url and options), it returns `true` if the http call was successfully scheduled, returns `false` otherwise

```js
window.onunload = function userLeave() {
  const crashData = new FormData()
  const url = 'https://my-analytics-server/'

  crashData.append('userID', '123');

  navigator.sendBeacon(url, crashData)
}
```

> * Beacon requests use  only HTTP POST and do not require a response.
>
> * Beacon requests are guaranteed to be initiated before the page unloads.

You should check out all the [examples](https://developer.mozilla.org/en-US/docs/Web/API/Beacon_API/Using_the_Beacon_API) from the docs to see how this can help you in many other cases!


Thanks for reading 🤚
