# Widget API

You can listen to events from the widget, or even create your own widgets programatically. Below is some documentation on how to do either.

> The widget code is all open source which should help any debugging, you can find the `**WidgetInstance**` class definition [here](https://github.com/gzuidhof/friendly-challenge/blob/master/src/captcha.ts).

## Widget Callbacks
For simple integrations you can specify callbacks directly on the widget HTML element.

```html
<div class="frc-captcha" data-sitekey="<my sitekey>" data-callback="myCallback"></div>

<script>
function myCallback(solution) {
    console.log("Captcha finished with solution " + solution);
}
</script>
```

The callback specified here should be defined in the global scope (i.e. on the `window` object). The callback will get called with one string argument: the `solution`  that should be sent to the server as proof that the CAPTCHA was completed.

There is also `data-callback-error`, which gets called in case there was an error completing the CAPTCHA. This is an experimental feature and should not be depended on for now. The most likely reason for an error here is a network error: either the user's connection has dropped or FriendlyCaptcha's servers are down (never say never). Note that a retry button is present for the user so it may be recoverable.

## Javascript API
For more advanced integrations you will probably can use the **friendly-challenge** Javascript API.

### If you used a script tag
If you added friendly-challenge to your website by using a script tag, a global variable `friendlyChallenge` will be present.

**Example**

```javascript
// Creating a new widget programmatically
const element = document.querySelector("#my-widget");
const myCustomWidget = new friendlyChallenge.WidgetInstance(element, {/* opts, more details in next section */})

// Or to get the widget that was created on script load (null if no widget instance was created)
const defaultWidget = friendlyChallenge.autoWidget;
```

### If you are using it as a library

**Example**

```javascript
import { WidgetInstance } from "friendly-challenge";

function doneCallback(solution) {
    console.log("CAPTCHA completed succesfully, solution:", solution);
    // ... Do something with the solution, maybe use it in a request
}

const element = document.querySelector("#my-widget");
const options = {
    doneCallback: doneCallback;
}
const widget = new friendlyChallenge.WidgetInstance(element, options);

// this makes the widget fetch a puzzle and start solving it.
widget.start()
```


The options object takes the following fields, they are all optional:
* **`startMode`**: string, default `"focus"`. Can be `"auto"`, `"focus"` or `"none"`.
   * `auto`: the solver will start as soon as possible. This is recommended if the user will definitely be submitting the CAPTCHA solution (e.g. there is only one form on the page), this has the best user experience.
   * `focus`: as soon as the form the widget is in fires the `focusin` event the solver starts, or when the user presses the start button in the widget. This is recommended for webpages where only few users will actually submit the form. This is the default.
   * `none`: the solver only starts when the user presses the button or it is programatically started by calling `start()`.

* **`readyCallback`**: function, called when the solver is done initializing and is ready to start.
* **`startedCallback`**: function, called when the solver has started.
* **`doneCallback`**: function, called when the CAPTCHA has been completed. One argument will be passed: the solution string that should be sent to the server.
* **`errorCallback`**: function, called when an internal error occurs. The error is passed as an object, the fields and values of this object are still to be documented and are changing frequently. Consider this experimental.

* **`puzzleEndpoint`**: string, the URL the widget should retrieve its puzzle from. This defaults to FriendlyCaptcha's endpoint, you will only ever need to change this if you are creating your own puzzles.
* **`forceJSFallback`**: boolean, default `false`:  Forces the widget to use the Javascript solver, which is much slower than the WebAssembly solver. Note that it will fallback to the JS solver automatically anyway. Recommended to never set this to true, it does not increase security.

## Questions or issues
If you have any questions about the API or run into problems, the best place to get help is probably the *issues* or *discussions* page on the [github repository](https://github.com/gzuidhof/friendly-challenge/issues).