# Custom Components - Change Events

In the previous example we defined a custom component type that renders a torus with a color that we can dynamically update. Often, we'd like to be able to observe changes to the values of our component's properties, so in this next example we'll learn how to make those properties fire change events.

The source code for our next example is shown below, and you can also run it live [here](http://xeoengine.org/examples/index.html#extending_customComponent_changeEvents).


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

Now we'll instantiate our custom component:

```js
var coloredTorus = new XEO.ColoredTorus({
    color: [0.3, 1.0, 0.3]
});
```

Then we'll attach a change listener to its ````color```` property, which log updates to that property to a element in the page:

```js
coloredTorus.on("color", function (color) {    
    log("coloredTorus.color = " + JSON.stringify(color, null, 4));
});

function log(msg) {    
    document.getElementById("log").innerHTML = msg;
}
```

Finally, we'll periodically set the property to a random color, which will fire our change listener:

```js
setInterval(function () {
    coloredTorus.color = [Math.random(), Math.random(), Math.random()];
}, 1000);
```


