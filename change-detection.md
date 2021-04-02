# Change detection

Links: 
- https://www.mokkapps.de/blog/the-last-guide-for-angular-change-detection-you-will-ever-need/
- https://angular.io/api/core/ChangeDetectionStrategy
- https://angular.io/api/core/ChangeDetectorRef
- https://blog.angular-university.io/how-does-angular-2-change-detection-really-work/
- https://blog.angular-university.io/onpush-change-detection-how-it-works/

![Angular change detection](https://www.mokkapps.de/static/208f64143869f05d2219a1e9735b9ab3/15ec7/change-detector-ref.jpg)


## Default change detection


![Angular change detection](https://d33wubrfki0l68.cloudfront.net/43c03578c42f2333b28e9f2a6ab03b6d856f3f23/a7bdc/cf7351e3976cdc3041cadce5367fc318/cd-cycle.gif)


By default, Angular uses the ChangeDetectionStrategy.Default change detection strategy. This default strategy checks every component in the component tree from top to bottom every time an event triggers change detection (like user event, timer, XHR, promise and so on). This conservative way of checking without making any assumption on the component’s dependencies is called dirty checking. It can negatively influence your application’s performance in large applications which consists of many components.

## On-Push change detection


![Angular change detection](https://d33wubrfki0l68.cloudfront.net/ed2650c2a4930cee5d17ddc92b0448031bb8e7fd/a240b/e9c151a1260485a277c6928b5f19019b/cd-on-push-cycle.gif)


## How is change detection implemented?
Angular can detect when component data changes, and then automatically re-render the view to reflect that change. But how can it do so after such a low-level event like the click of a button, that can happen anywhere on the page?

To understand how this works, we need to start by realizing that in Javascript the whole runtime is overridable by design. We can override functions in String or Number if we so want.

##Browser Async APIs supported
The following frequently used browser mechanisms are patched to support change detection:

all browser events (click, mouseover, keyup, etc.)
setTimeout() and setInterval()
Ajax HTTP requests
In fact, many other browser APIs are patched by Zone.js to transparently trigger Angular change detection, such as for example Websockets. Have a look at the Zone.js test specifications to see what is currently supported.

One limitation of this mechanism is that if by some reason an asynchronous browser API is not supported by Zone.js, then change detection will not be triggered. This is, for example, the case of IndexedDB callbacks.

This explains how change detection gets triggered, but once triggered how does it actually work?


## How does the default change detection mechanism work?
This method might look strange at first, with all the strangely named variables. But by digging deeper into it, we notice that it's doing something very simple: for each expression used in the template, it's comparing the current value of the property used in the expression with the previous value of that property.

If the property value before and after is different, it will set isChanged to true, and that's it! Well almost, it's comparing values by using a method called
looseNotIdentical(), which is really just a === comparison with special logic for the NaN case (see here).

## Async Pipe
The built-in AsyncPipe subscribes to an observable and returns the latest value it has emitted.

Internally the AsyncPipe calls markForCheck each time a new value is emitted, see its source code:

private _updateLatestValue(async: any, value: Object): void {
  if (async === this._obj) {
    this._latestValue = value;
    this._ref.markForCheck();
  }
}
As shown, the AsyncPipe automatically works using OnPush change detection strategy. So it is recommended to use it as much as possible to easier perform a later switch from default change detection strategy to OnPush.

You can see this behavior in action in the async demo.
