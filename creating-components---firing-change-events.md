# Custom Components - Change Events

Our previous two custom component examples rendered a torus object, with a color property that we were able to dynamically update. Often, we'd like to observe updates on component properties, so in this next example we'll learn how to make them fire change events whenever they're updated.

Let's take the previous example and add a line of code to fire a "color" event each time the ````color```` property is updated.

[[Run this example](http://xeoengine.org/examples/index.html#extending_customComponent_changeEvents)]

```javascript
XEO.ColoredTorus = XEO.Component.extend({

    type: "XEO.ColoredTorus",

    id: "myColoredTorus", // Optional

    _init: function (cfg) {
        this._torus = new XEO.Entity({
            geometry: new XEO.TorusGeometry({ radius: 2, tube:.6 }),
            material: new XEO.PhongMaterial({
                diffuse: [0.5, 0.5, 0.5]
            })
        });

        this.color = cfg.color;
    },

    _props: {
        color: {
            set: function (color) {
                this._torus.material.diffuse = color || [0.5, 0.5, 0.5];

                // Fire change events for the color property
                this.fire("color", this._torus.material.diffuse);
            },
            get: function () {
                return this._torus.material.diffuse;
            }
        }
    }
});
```

Now, to show how it works, let's instantiate our component:

```js
var coloredTorus = new XEO.ColoredTorus({
    color: [0.3, 1.0, 0.3]
});
```

Next, we'll attach a change listener to the ````color```` property, which will log value updates to an element in the page:

```js
coloredTorus.on("color", function (color) {    
    log("coloredTorus.color = " + JSON.stringify(color, null, 4));
});

function log(msg) {    
    document.getElementById("log").innerHTML = msg;
}
```

Finally, we'll periodically set the property to a random color, which will fire our change listener each time:

```js
setInterval(function () {
    coloredTorus.color = [Math.random(), Math.random(), Math.random()];
}, 1000);
```

In the spirit of the [Open/Closed Principle](https://en.wikipedia.org/wiki/Open/closed_principle), pretty much everything in xeoEngine fires change events when updated, however your custom components don't neccessarily need to.
