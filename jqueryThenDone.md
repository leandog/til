# jquery then vs done

So in a function like this, I thought `then` and `done` were pretty much equivalent.

```javascript
Page.prototype.refreshData = function () {
    return Service.fetch().then(function (data) {
        console.log("I got data!", data);
    });
};
```

`then` does have the feature that you can specify both the success and failure handler at once:

```javascript
Page.prototype.refreshData = function() {
    return Service.fetch().then(function() {
      console.log("Much success");
    }, function() {
      console.log("KTHXBYE");
    });
};
```

But that's lazy, and potentially less clear than:
```javascript
Page.prototype.refreshData = function() {
    return Service.fetch().done(function() {
      console.log("Much success");
    }).fail(function() {
      console.log("KTHXBYE");
    });
};
```

And naturally we chain promises around, because we're 1337 functional programmers

```javascript
Page.prototype.refreshData = function() {
    return Service.fetch()
        .then(function(data) {
          return new Model(data);
        }, function(error) {
            displayError(error);
        }).then(function(model) {
            updateView(model);
        });
};
```

Ugh.  I've got this crummy error handling stuff in the middle.  Let's use `done` and `fail` instead.

```javascript
Page.prototype.refreshData = function() {
    return Service.fetch()
        .done(function(data) {
          return new Model(data);
        }).done(function(model) {
           updateView(model);
        }).fail(function(error) {
            displayError(error);
        });
};
```

Uh oh.  That doesn't work.

It turns out that `then` and `done` do have one key difference.  `done` will not modify the value the future callers in the chain receive.  In that way it doesn't matter what order you chain `done` callbacks in, they all get the value passed in from the initial promise (in this case `data` from `fetch()`).  `then` DOES modify the value that promises after that one receive (in this case `model` from `new Model()`).

So I will use `then` when I mean to produce logical call chains, and `done`/`fail` when I mean to produce temporal call chains (or just separate two actions).
