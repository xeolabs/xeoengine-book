# Implementation Notes

This document is aimed at developers who want to know a little more about the internal workings of [xeogl](http://xeogl.org).

<img src="http://xeogl.org/assets/images/blackboard.jpg" width="100%">

## API Design
````xeogl```` is abstract and data-driven on the outside, while performant on the inside. Like [SceneJS](http://scenejs.org), ````xeogl````'s design philosophy is to always try to ["program to the interface"](https://en.wikipedia.org/wiki/Encapsulation_(computer_programming)) (ie. its API), without sacrificing performance.  

Therefore, like many 3D engines, ````xeogl````'s implementation is layered like an onion, with a user-friendly *logically-oriented* design at its highest layer (the scene graph API) and a WebGL-friendly *performance-oriented* design at its lowest layer (the renderer). 

Consequently, much of ````xeogl````'s implementation is about efficiently synchronizing state between those layers, using a few common real-time rendering techniques, which I'll briefly summarize here.

## Components and Properties

As described on the [website](http://xeogl.org#concepts), a ````xeogl```` scene is a soup containing various components that are tied together into drawables by [Entities](http://xeogl.org/docs/classes/Entity.html).

<img src="http://xeogl.org/assets/images/conceptScene.png">

````xeogl```` components expose their state as *properties*, rather than via accessor methods. This is largely sugar, to make the API feel a bit lighter to code with. It also has benefits for documentation, where we document a single property instead of a pair of accessor methods. It's also great for API cleanliness, where we have a neat semantic mapping of property names to their update events.

````Javascript
myRotate.angle += 0.5;

myRotate.on("angle", function (angle) {
    this.log("angle updated: " + angle);
});
````

TODO: A caveat about not directly-modifying array properties.

The snippet below shows how properties are defined on ````xeogl```` component classes. In this example, we're extending the [Component](http://xeogl.org/docs/classes/Component) 
 base class to create a custom MyComponent subclass that has a single "foo" property. The getter and setter functions 
 we provide for the property are internally translated into a [Object.defineProperty](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty) 
 call to define the property on the subclass.    

````javascript
var xeogl.MyComponent = xeogl.Component.extend({

    //..
    
    _props: {
    
        foo: {
            set function(value) {
                if (this._foo !== value) { // 1
                    this._foo = value; // 2
                    this.sheduleTask(...); // 3
                    this.fire("foo", this._foo); // 4
                }
            },
            get function() {
                return this._foo;
            }
        }
    },
    
    //...
}
````
Pretty much every property setter in ````xeogl```` will do the following: 

 1. reject redundant updates, 
 2. update the property value,
 3. possibly schedule some tasks for the engine to process on the next frame, and  
 4. fire a change event, which always has the same name as the property.

## Performance

### Lazy evaluation
````xeogl```` lazy-computes things only when they are actually needed. An example of this are the boundaries of [Geometry's](http://xeogl.org/docs/classes/Geometry) and [Entities](http://xeogl.org/docs/classes/Entity). When you get,
say, a worldBoundary from an Entity, you get a [Boundary3D](http://xeogl.org/docs/classes/Boundary3D) instance that fires "update" events whenever it´s extents change as a result of the Entityś 
geometry or transforms updating, but it´s not until you 
reference the Boundary3D´s **aabb** or **obb** properties that it actually builds its extents from the current state of the geometry and transforms.

### Batching
Many updates in ````xeogl````, such as input, matrix calculations or canvas size updates, are batched and processed on the next game loop. Because we're generally interested in the most recent of these sorts of updates, this helps reduce the amount of unnecessary work done.

### Caching
````xeogl```` caches as much as it can, in order to avoid garbage collection and re-computation.

The snippet below, from the [````Scene````](http://xeogl.org/docs/classes/Scene.html) class' [pick](http://xeogl.org/docs/classes/Scene.html#method_pick) method, is an example of how ````xeogl```` caches variables in a closure: 

```` javascript
xeogl.Scene = xeogl.Component.extend({

    //...
    
    pick: (function () {        

        var tempVec2 = xeogl.math.vec2();
        //..

        return function (params) {
            params = params || {};
            params.canvasPos = params.canvasPos || tempVec2;

            //...
        }

    })()
});
````

We also cache callbacks wherever practical. In ````xeogl````, any method that accepts a callback also accepts an optional scope on which to execute the callback. As shown in the snippet below, this lets us define a callback as a class member, so that we can reuse the callback function instead of redefining it each time we make the call. The snippet below is taken from the [CameraFlight](http://xeogl.org/docs/classes/CameraFlight.html) component, which flies a [Camera] (http://xeogl.org/docs/classes/Camera.html) to a specified position. On each scene "tick" event, 
we're updating the camera position to animate it. 


```` javascript  
xeogl.CameraFlight = xeogl.Component.extend({

    /** Initiates a camera flight animation
     */
    flyTo: function (params, callback, scope) {

        //...

        this.scene.on("tick", this._update, this);
    },

    _update: function () {
        // Update the camera animation
        //...
    }
});
````

### Instancing

TODO

### Task Queuing
When a ````xeogl```` component needs to perform some non-trivial task, such as generating some geometry or a matrix, it will push the task to a FIFO queue for execution on the next frame. On each frame, ````xeogl```` pops and executes as many tasks as possible within a fixed per-frame time budget, leaving behind in the queue any tasks that didn’t get a chance to run within the budget, to be processed on subsequent frames. 

Care was taken when setting the size of the time budget. When too small, the task queue tends to grow large with the backlog of tasks, and interactivity gets sluggish with all the late processing. When too big, the FPS will stutter whenever there is a surge of tasks and ````xeogl```` attempts to execute too many of them on each frame.     

Tasks may be scheduled on ````xeogl```` using the ````scheduleTask```` method, like so:

```` javascript
 var callback = function() { ... }; // Callback to perform the task
 var scope = this; // Scope on which to execute the callback

 xeogl.scheduleTask(callback, scope);
````

Using this public API method, the application layer can push its own tasks to the task queue, if desired, to have them executed within the per-frame time budget, alongside as the ````xeogl```` components' tasks.

### Render Graph Compilation
````xeogl````'s scene graph is simple and logical to use, however that would make it inefficient to 
 traverse directly in order to render each frame, especially if we're trying to sustain 60FPS. ````xeogl```` deals with this by dynamically 
 compiling the scene graph to an internal *render graph* that's very efficient to traverse and render. The render graph 
 is a list of nodes, each corresponding to an [Entity](http://xeogl.org/docs/classes/Entity). The nodes contains chunks of WebGL state changes (uniforms, buffers, draw calls etc) 
 and are batched and sorted so as to apply them in the most optimal order possible. As the scene graph changes, ````xeogl```` internally keeps 
  the render graph synchronized accordingly.

#### State Sorting
The state chunk sorting mentioned earlier is crucial to performance. The sort order chosen for ````xeogl```` is intended to make WebGL do the 
   least amount of thrashing when binding and unbinding buffers for [Textures](http://xeogl.org/docs/classes/Texture) and [Geometries](http://xeogl.org/docs/classes/Geometry), while at the same time enforcing the *render binning* set up by [Layer](http://xeogl.org/docs/classes/Layer.html) and [Stage](http://xeogl.org/docs/classes/Stage.html) components, 
   as well as the *transparency binning* (ie. to render all opaque nodes before transparent nodes for alpha blending) set up by [Modes](http://xeogl.org/docs/classes/Modes.html) components. 
    
The snippet below, taken from WebGL renderer at the core of ````xeogl```` as of writing this page, assigns sort keys to nodes within the 
  render graph. Hopefully that gives some insight into how sort keys are generated. When rendering the nodes, the state chunks 
  of each Entity will then be applied in the sorted order, as determined by the sort keys. 
       

```` javascript
    /**
     * Generates state sorting keys on render graph nodes
     */
    xeogl.renderer.Renderer.prototype._makeStateSortKeys = function () {
        var node;
        for (var i = 0, len = this._nodeListLen; i < len; i++) {
            node = this._nodeList[i];
      
            node.sortKey =
                ((node.stage.priority + 1) * 10000000000000000)
                + ((node.modes.transparent ? 2 : 1) * 100000000000000)
                + ((node.layer.priority + 1) * 10000000000000)
                + ((node.program.id + 1) * 100000000)
                + ((node.material.id + 1) * 10000)
                + node.geometry.id;
            }
        }
    };
````

(Work in progress)