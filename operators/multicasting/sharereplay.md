# shareReplay

#### signature: `share(bufferSize?: number, windowTime?: number, scheduler?I IScheduler): Observable`

## Share source and replay specified number of emissions on subscription.

---

:bulb: share is like [multicast](multicast.md) with a `ReplaySubject` and
refCount!

---

### Why use `shareReplay`?

You generally want to use `shareReplay` when you have side-effects or taxing
computations that you do not wish to be executed amongst multiple subscribers.
The difference between `shareReplay` and [`share`](./share.md) is late
subscribers (subscriptions after a value has been emitted) can recieve a
specified number of emitted values on subscription. For instance, suppose you
have an observable that emits the last successful url. In the first example we
are going to use [`share`](./share.md):

```js
// simulate url change with subject
const routeEnd = new Subject<{data: any, url: string}>();
// grab url and share with subscribers
const lastUrl = routeEnd.pipe(
  pluck('url'),
  share()
);
// initial subscriber required
const initialSubscriber = lastUrl.subscribe(console.log)
// simulate route change
routeEnd.next({data: {}, url: 'my-path'});
// nothing logged
const lateSubscriber = lastUrl.subscribe(console.log);
```

In the above example we may want access to the last url upon any subscription.
We can accomplish this goal with `shareReplay`:

```js
// simulate url change with subject
const routeEnd = new Subject<{data: any, url: string}>();
// grab url and share with subscribers
const lastUrl = routeEnd.pipe(
  pluck('url'),
  shareReplay(1)
);
// initial subscriber required
const initialSubscriber = lastUrl.subscribe(console.log)
// simulate route change
routeEnd.next({data: {}, url: 'my-path'});
// logged: 'my path'
const lateSubscriber = lastUrl.subscribe(console.log);
```

Note that this is the same behavior you would see if you subscribed a
`BehaviorSubject` or `ReplaySubject` to the `lastUrl` stream, then subscribed to
that `Subject`:

```js
// simulate url change with subject
const routeEnd = new Subject<{data: any, url: string}>();
// instead of using shareReplay, use ReplaySubject
const shareWithReplay = new ReplaySubject();
// grab url and share with subscribers
const lastUrl = routeEnd.pipe(
  pluck('url')
)
.subscribe(val => shareWithReplay.next(val))
// simulate route change
routeEnd.next({data: {}, url: 'my-path'});
// subscribe to ReplaySubject instead
// logged: 'my path'
shareWithReplay.subscribe(console.log);
```

In fact, if we dig into the source code we can see almost exactly that:

(
[source](https://github.com/ReactiveX/rxjs/blob/b25db9f369b07f26cf2fc11714ec1990b78a4536/src/internal/operators/shareReplay.ts#L26-L37)
)

```js
subject = new ReplaySubject() < T > (bufferSize, windowTime, scheduler);
subscription = source.subscribe({
  next(value) {
    subject.next(value);
  },
  error(err) {
    hasError = true;
    subject.error(err);
  },
  complete() {
    isComplete = true;
    subject.complete();
  }
});
```

<div class="ua-ad"><a href="https://ultimateangular.com/?ref=76683_kee7y7vk"><img src="https://ultimateangular.com/assets/img/banners/ua-leader.svg"></a></div>

### Examples

##### Example 1: Multiple subscribers sharing source

( [Stackblitz](https://stackblitz.com/edit/typescript-qfhryg?file=index.ts) )

```js
```

### Additional Resources

* [shareReplay](http://reactivex.io/rxjs/class/es6/Observable.js~Observable.html#instance-method-shareReplay)
  :newspaper: - Official docs

---

> :file_folder: Source Code:
> [https://github.com/ReactiveX/rxjs/blob/master/src/internal/patching/operator/shareReplay.ts](https://github.com/ReactiveX/rxjs/blob/master/src/internal/patching/operator/shareReplay.ts)