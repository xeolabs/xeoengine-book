# Custom Components

We can extend xeoEngine's component classes to create our own custom component types. The basic [Component](http://xeoengine.org/docs/classes/Component.html) class has various template methods which we can override in our component class to hook it into xeoEngine.

Let's start simple, with a custom component that renders a torus with a color that we can dynamically update. Then in subsequent sections, we'll create versions of the component that plug into the various other bits of xeoEngine component functionality.

### Defining the Component

The source code for our first component is shown below. You can also run it live [here](http://xeoengine.org/examples/index.html#extending_customComponent_basic).

````javascript
XEO.ColoredTorus = XEO.Component.extend({

    // Class name
    type: "XEO.ColoredTorus",

    // Optional component ID
    id: "myColoredTorus", 

    // Constructor
    _init: function (cfg) {

        this._super(cfg); // Call base class constructor

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
                this._torus.material.diffuse = color || [0.5, 0.5, 0.5];
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

The ````type```` property is the name of the class. xeoEngine uses this to register instances of the the component, and also when it needs to serialize and deserialize them as JSON (which we'll look at in one of the following sections).

The ````_init()```` method is the component's constructor. The first thing we usually do in this method is call the super class' constructor with a call to  ````this._super(cfg)````. Then we create our component's internal bits.

The ````_props```` section defines the component's public properties. On our component, we have just the ````color```` property, which is both readable and writable since we defined a getter and a setter for it. Note how we set that property in the constructor, to whatever value is supplied in the constructors ````cfg```` parameters object, if any. If that value is undefined, then see how the setter method sets it to the default value ````[0.5, 0.5, 0.5]````.

Finally, the ````_destroy```` mathod is the component's destructor. In our case, we're destroying the various child components that we created in the constructor. There's a way to create those child components so that they are destroyed automatically along with our component, which we'll also look at in one of the  following sections.

### Instantiating the Component

Let's instantiate our new component class: 

````javascript
var coloredTorus = new XEO.ColoredTorus({
    color: [0.3, 1.0, 0.3]
});
````

That's going to call our ````_init()```` method, passing in an initial value for ````color````, which gets passed into the setter method for that property.

Since we didn't explicitly provide a [Scene](http://xeoengine.org/docs/classes/Scene.html) for our component, xeoEngine will create it within its default Scene, which it instantiates on-demand as soon as it's needed. Let's get that Scene off our component, then find our component by its type within that Scene:

````javascript
var scene = coloredTorus.scene;
var coloredTorii = scene.types["XEO.ColoredTorus"];
var myColoredTorus = coloredTorii["myColoredTorus"];
````

### Updating the Component

We can then update the value of that property like so:

````javascript
coloredTorus.color = [0.5, 0.5 , 1.0];
````

### Destroying the Component

Lastly, to destroy our component:

````
coloredTorus.destroy();
````