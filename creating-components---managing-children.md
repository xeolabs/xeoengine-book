# Creating Components - Managing Children

```javascript
XEO.ColoredTorus = XEO.Component.extend({

    type: "XEO.ColoredTorus",

    _init: function (cfg) {

        this._torus = this.create(XEO.Entity, {
            geometry: this.create(XEO.TorusGeometry, {
                radius: 2,
                tube: 0.6
            }),
            material: this.create(XEO.PhongMaterial, {
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

    _destroy: function () {
        // Nothing to do here! The child geometry, material 
        // and entity components will be  automatically destroyed 
        // along with this ColoredTorus.
    }
});
```

