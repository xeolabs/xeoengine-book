# Creating Components

You can extend xeoEngine's component classes to create your own custom component types. The basic Component class has various template methods which you can override to hook into xeoEngine.


````javascript
XEO.ColoredTorus = XEO.Component.extend({

    // Class name
    type: "XEO.ColoredTorus",

    // Constructor
    _init: function (cfg) {

        this._torus = new XEO.Entity({
            geometry: new XEO.TorusGeometry({ radius: 2, tube:.6 }),
            material: new XEO.PhongMaterial({
                diffuse: [0.5, 0.5, 0.5]
            })
        });

        this.color = cfg.color;
    },

    // Public properties 
    _props: {

        // The color of this ColoredTorus.
        color: {
            set: function (color) {
                this._torus.material.diffuse = color;
            },
            get: function () {
                return this._torus.material.diffuse;
            }
        }
    },

    // Destructor
    _destroy: function () {
        this._torus.geometry.destroy();
        this._torus.material.destroy();
        this._torus.destroy();
    }
});
````

Instantiate a ColoredTorus (in the default XEO.Scene)

````javascript
var coloredTorus = new XEO.ColoredTorus({
    color: [0.3, 1.0, 0.3]
});
````

Periodically set the color of the ColoredTorus to a random value


````javascript
setInterval(function () {
    coloredTorus.color = [Math.random(), Math.random(), Math.random()];
}, 1000);

````
