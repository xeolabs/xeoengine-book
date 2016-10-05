# Custom Components - Managing Children

The component in the [previous custom component example]() needed to keep references to the child [Entity](), [Geometry]() and [PhongMaterial]() components that it created in its constructor, so that it could destroy them in its destructor.

To make that less clunky, the [Component]() base class provides a convenient ````create()```` method which we can use to create child components that will be automatically destroyed when our component is destroyed.

Take a look at the following example, which modifies the previous example to use the ````create()```` method.  See how we no longer need to destroy child components in the ````_destroy```` method (the destructor), which we don't even need anymore. 

To keep code examples simple, we're going to use the ````create()```` method in the rest of our custom component examples.

[[Run this example](http://xeoengine.org/examples/index.html#extending_customComponent_childCleanup)]


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

