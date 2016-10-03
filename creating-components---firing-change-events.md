# Creating Components - Change Events

````javascript
// Define the ColoredTorus component class

XEO.ColoredTorus = XEO.Component.extend({

    type: "XEO.ColoredTorus",

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
````
````javascript
// Instantiate a ColoredTorus (in the default XEO.Scene)

var coloredTorus = new XEO.ColoredTorus({
    color: [0.3, 1.0, 0.3]
});
````
````javascript
// Periodically set the color of the ColoredTorus to a random value

setInterval(function () {
    coloredTorus.color = [Math.random(), Math.random(), Math.random()];
}, 1000);
````
````javascript

// Log updates to the color of the ColoredTorus

coloredTorus.on("color", function (color) {
    log("coloredTorus.color = " + JSON.stringify(color, null, 4));
});
````
````javascript

// Logs a message to the info element at the top of this page

function log(msg) {
    document.getElementById("log").innerHTML = msg;
}
````