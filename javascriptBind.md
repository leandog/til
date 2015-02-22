# javascript .bind(this)

Today I learned a bit more about the bind function.

In general, I have not used the bind function much before now.  And most of that usage has been:

1. calling a function doesn't work as expected
1. add bind
1. it now works

Here's a sequence that left me puzzled.  We had to keep adding bind.  Why?

```javascript
Page.prototype.refreshData = function () {
    return Service.fetch().then(function (data) {
        this.data =  data;
    }.bind(this));
};

Page.prototype.waitForData = function () {
    setTimeout(function () {
        var oldData = this.data;
        this.refreshData().then(function () {
            if (_.isEqual(_.pluck(oldData, 'id'), _.pluck(this.data, 'id'))) {
                this.waitForData();
            }
        }.bind(this));
    }.bind(this), 10000);
};
```

If you were to inline that refresh function, then the code is:
1.  Fetch from service and store result
1.  Check to see if the data has changed
1.  Wait some more if it hasn't.

MDN has a straightforward explanation of [bind](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind):

>The bind() method creates a new function that, when called, has its this keyword set to the provided value, with a given sequence of arguments preceding any provided when the new function is called.

The first part is what we use bind for, the second part of the definition probably has implications for currying, which is a bit further that I want to dive at the moment.

So, in that code snippet, the part that I have no trouble groking is that the setTimeout() function called needs ```this``` redefined.  ```setTimeout()``` is short for ```window.setTimeout()```

```javascript
Page.prototype.waitForData = function () {
    setTimeout(function () {
        var oldData = this.data;
        //other stuff
    }.bind(this), 10000);
};
```

So in order to get data off my page-object I need to change ```this``` from ```window``` to ```page```.  Otherwise I get the wonderful ```undefined is not a function```.  _This brings back the cold sweats I used to get from "Segmentation fault", but it's getting [better](https://twitter.com/addyosmani/status/569157136137134081)._

So what about ```refreshData()```?

```javascript
Page.prototype.refreshData = function () {
    return Service.fetch().then(function (data) {
        this.data =  data;
    }.bind(this));
};
```

When I see this code, I see a closure.  Go get me an input and I'm going to do something with it.  How could ```this``` possibly be anything but the object that asked for the input?

So I head to the javascript console.  If I remove the bind:
```javascript
Page.prototype.refreshData = function () {
    return Service.fetch().then(function (data) {
        console.log(this);
        debugger;
        this.data =  data;
    });
};
```

```this``` looks suspiciously promise-like.  In fact it's probably a ```jqXHR``` or the equivalent.

Ok, so in this case, ```this``` is the object that we're calling ```then``` on.  Of course!

I think most of the confusion here comes from when I've written closures in other OO languages.  I expect everything in the "class" to have implicit knowledge of ```this```.  But of course javascript is not OO, it's just close enough in places to __blow your mind__.

Ok, two mysteries solved.  But what about that waiting function:

```javascript
Page.prototype.waitForData = function () {
    setTimeout(function () {
        var oldData = this.data;
        this.refreshData().then(function () {
            if (_.isEqual(_.pluck(oldData, 'id'), _.pluck(this.data, 'id'))) {
                this.waitForData();
            }
        }.bind(this));
    }.bind(this), 10000);
};
```

We assigned ```this``` to ```page``` at the ```setTimeout``` closure.  We assigned ```this``` to ```page``` when we called ```refreshData()```.  Do we really have to assign ```this``` again in this second chained closure?

Here's the secret.  ```bind()``` is not some magical dynamic run-time thing.  It's not overriding ```this``` on the fly.  The value of ```this``` is baked-in when the function/closure is __defined__, not when it's executed.  So again in this instance, we called ```then``` on a promise-like object and that promise is ```this``` inside the closure unless we hard-wire it to something else using the ```bind()``` function.

### What's the point?

```bind()``` appeared in ECMAScript 5 and opens up some interesting options.  Back in the old days, we would usually do one of two things to get around the "where is ```this```?" problem.  We would identify some silly ```var that = this;``` and make sure ```that``` was accessible to the calling context.  Or we would make everything namespaced and globally accessible.  The latter felt like a better solution, and I think you could argue for debugging that it has distinct advantages.  But in this age where everyone and their great-aunt has written a javascript library, being able to redefine ```this``` on someone else's function and avoid polluting the global namespace is pretty cool.
