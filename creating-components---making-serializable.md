# Custom Components - Making Serializable

````javascript
XEO.ColoredTorus = XEO.Component.extend({

    type: "XEO.ColoredTorus",

    _init: function (cfg) {

        this._torus = new XEO.Entity({
            geometry: new XEO.TorusGeometry({ 
                radius: 2, 
                tube:.6 
            }),
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
            },
            get: function () {
                return this._torus.material.diffuse;
            }
        }
    },
    
    _getJSON: function () {
        return {
            color: this._torus.material.diffuse
        };
    }
});
````