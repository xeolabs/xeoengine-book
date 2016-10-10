# Creating a New Component Type 

We can extend xeogl's component classes to create our own custom component types. The basic [Component](http://xeogl.org/docs/classes/Component.html) class has various template methods which we can override in our component class to hook it into xeogl.

Let's start simple, with a custom component that renders a torus with a color that we can dynamically update. Then in subsequent sections, we'll create versions of the component that plug into the various other bits of xeogl component functionality.

## Creating the Component

The source code for our first component is shown below. You can also run it live [here](http://xeogl.org/examples/index.html#extending_customComponent_basic).

````javascript
xeogl.ColoredTorus = xeogl.Component.extend({

    // Class name
    type: "xeogl.ColoredTorus",

    // Optional component ID
    id: "myColoredTorus", 

    // Constructor
    _init: function (cfg) {

        this._super(cfg); // Call base class constructor

        this._torus = new xeogl.Entity({
            geometry: new xeogl.TorusGeometry({ radius: 2, tube:.6 }),
            material: new xeogl.PhongMaterial({
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

The ````type```` property is the name of the class. xeogl uses this to register instances of the the component, and also when it needs to serialize and deserialize them as JSON (which we'll look at in one of the following sections).

The ````_init()```` method is the component's constructor. The first thing we usually do in this method is call the super class' constructor with a call to  ````this._super(cfg)````. Then we create our component's internal bits.

The ````_props```` section defines the component's public properties. On our component, we have just the ````color```` property, which is both readable and writable since we defined a getter and a setter for it. Note how we set that property in the constructor, to whatever value is supplied in the constructors ````cfg```` parameters object, if any. If that value is undefined, then see how the setter method sets it to the default value ````[0.5, 0.5, 0.5]````.

Finally, the ````_destroy```` mathod is the component's destructor. In our case, we're destroying the various child components that we created in the constructor. There's a way to create those child components so that they are destroyed automatically along with our component, which we'll also look at in one of the  following sections.

#### Instantiating the Component

Let's instantiate our new component class: 

````javascript
var coloredTorus = new xeogl.ColoredTorus({
    color: [0.3, 1.0, 0.3]
});
````

That's going to call our ````_init()```` method, passing in an initial value for ````color````, which gets passed into the setter method for that property.

Since we didn't explicitly provide a [````Scene````](http://xeogl.org/docs/classes/Scene.html) for our component, xeogl will create it within its default Scene, which it instantiates on-demand as soon as it's needed. Let's get that Scene off our component, then find our component by its type within that Scene:

````javascript
var scene = coloredTorus.scene;
var coloredTorii = scene.types["xeogl.ColoredTorus"];
var myColoredTorus = coloredTorii["myColoredTorus"];
````

#### Updating the Component

We can then update the value of that property like so:

````javascript
coloredTorus.color = [0.5, 0.5 , 1.0];
````

#### Destroying the Component

Lastly, to destroy our component:

````
coloredTorus.destroy();
````

## Managing Children

The component in the previous example] needed to keep references to the child [Entity](), [Geometry]() and [PhongMaterial]() components that it created in its constructor, so that it could destroy them in its destructor.

To make that less clunky, the [Component]() base class provides a convenient ````create()```` method which we can use to create child components that will be automatically destroyed when our component is destroyed.

Take a look at the following example, which modifies the previous example to use the ````create()```` method.  See how we no longer need to destroy child components in the ````_destroy```` method (the destructor), which we don't even need anymore. 

To keep code examples simple, we're going to use the ````create()```` method in the rest of our custom component examples.

[[Run this example](http://xeogl.org/examples/index.html#extending_customComponent_childCleanup)]


```javascript
xeogl.ColoredTorus = xeogl.Component.extend({

    type: "xeogl.ColoredTorus",

    _init: function (cfg) {

        this._torus = this.create(xeogl.Entity, {
            geometry: this.create(xeogl.TorusGeometry, {
                radius: 2,
                tube: 0.6
            }),
            material: this.create(xeogl.PhongMaterial, {
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

## Firing Events

Our previous two custom component examples rendered a torus object, with a color property that we were able to dynamically update. Often, we'd like to observe updates on component properties, so in this next example we'll learn how to make them fire change events whenever they're updated.

Let's take the previous example and add a line of code to fire a "color" event each time the ````color```` property is updated.

[[Run this example](http://xeogl.org/examples/index.html#extending_customComponent_changeEvents)]

```javascript
xeogl.ColoredTorus = xeogl.Component.extend({

    type: "xeogl.ColoredTorus",

    id: "myColoredTorus", // Optional

    _init: function (cfg) {
        this._torus = new xeogl.Entity({
            geometry: new xeogl.TorusGeometry({ radius: 2, tube:.6 }),
            material: new xeogl.PhongMaterial({
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
var coloredTorus = new xeogl.ColoredTorus({
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

In the spirit of the [Open/Closed Principle](https://en.wikipedia.org/wiki/Open/closed_principle), pretty much everything in xeogl fires change events when updated, however your custom components don't neccessarily need to.


## Scheduling Tasks

```javascript
// Define the ResizableTorus component class

xeogl.ResizableTorus = xeogl.Component.extend({

    type: "xeogl.ResizableTorus",

    _init: function (cfg) {

        this._torus = new xeogl.Entity({
            geometry: new xeogl.Geometry(),
            material: new xeogl.PhongMaterial({
                diffuse: [0.5, 0.5, 1.0]
            })
        });

        this.radius = cfg.radius
    },

    // This executes the task that this ResizableTorus schedules to the xeogl task queue,
    // in this case to (re)build its torus-shaped geometry. Don't worry about the contents
    // of this function. It's just something computationally intensive that needs to be scheduled
    // to execute asynchronously via the task queue so that it doesn't disrupt the smoothness of
    // xeogl's "game loop".

    _update: function () {

        // Build torus geometry from _radius

        var tube = this._radius / 2;
        var radialSegments = 20;
        var tubeSegments = 20;
        var arc = Math.PI * 2;
        var positions = [];
        var normals = [];
        var uvs = [];
        var indices = [];
        var u, v, centerX, centerY, centerZ = 0, x, y, z, vec, i, j;
        for (j = 0; j <= radialSegments; j++) {
            for (i = 0; i <= tubeSegments; i++) {
                u = i / tubeSegments * arc; v = j / radialSegments * Math.PI * 2;
                centerX = this._radius * Math.cos(u); centerY = this._radius * Math.sin(u);
                x = (this._radius + tube * Math.cos(v) ) * Math.cos(u);
                y = (this._radius + tube * Math.cos(v) ) * Math.sin(u);
                z = tube * Math.sin(v);
                positions.push(x); positions.push(y); positions.push(z);
                uvs.push(1 - (i / tubeSegments)); uvs.push(1 - (j / radialSegments));
                vec = xeogl.math.normalizeVec3(xeogl.math.subVec3([x, y, z], [centerX, centerY, centerZ], []), []);
                normals.push(vec[0]); normals.push(vec[1]); normals.push(vec[2]);
            }
        }
        var a, b, c, d;
        for (j = 1; j <= radialSegments; j++) {
            for (i = 1; i <= tubeSegments; i++) {
                a = (tubeSegments+1)*j+i-1;
                b = (tubeSegments+1)*(j-1)+i-1;
                c = (tubeSegments+1)*(j-1)+i;
                d = (tubeSegments+1)*j+i;
                indices.push(a); indices.push(b); indices.push(c);
                indices.push(c); indices.push(d); indices.push(a);
            }
        }

        // Update the Geometry component

        this._torus.geometry.positions = positions;
        this._torus.geometry.normals = normals;
        this._torus.geometry.uv = uvs;
        this._torus.geometry.indices = indices;
    },

    _props: {

        // The radius of this ResizableTorus
        radius: {
            set: function (radius) {
                this._radius = radius || 1;

                // Schedule our _update() method to the xeogl task queue,
                // to execute at the next available opportunity
                this._scheduleUpdate();
            },

            get: function () {
                return this._radius;
            }
        }
    }
});

// Instantiate a ResizableTorus (in the default xeogl.Scene)
var resizableTorus = new xeogl.ResizableTorus({
    radius: 1
});

// Periodically increase the radius of the ResizableTorus
setInterval(function () {
    resizableTorus.radius += 0.2;
}, 2000);
```


## Making Serializable

````javascript
xeogl.ColoredTorus = xeogl.Component.extend({

    type: "xeogl.ColoredTorus",

    _init: function (cfg) {

        this._torus = new xeogl.Entity({
            geometry: new xeogl.TorusGeometry({ 
                radius: 2, 
                tube:.6 
            }),
            material: new xeogl.PhongMaterial({
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
