# Importing glTF


|[<img src="http://xeogl.org/assets/images/glTFGearbox.png" width="100%">](http://xeogl.org/examples/#importing_gltf_gearbox)|
|--------------------------|
|xeogl [gearbox visualization](http://xeogl.org/examples/#importing_gltf_gearbox), loaded from [sample model](https://github.com/KhronosGroup/glTF/tree/master/sampleModels/gearbox_assy) provided by the Khronos Group|

|[<img src="http://xeogl.org/assets/images/gltf/glTF_explorer_small.png" width="100%">](http://xeogl.org/examples/#demos_ui_explorer)|[<img src="http://xeogl.org/assets/images/gltf/glTF_flyTo_small.png" width="100%">](http://xeogl.org/examples/#boundaries_flyToBoundary)|[<img src="http://xeogl.org/assets/images/gltf/glTF_buggy_small.jpg" width="100%">](http://xeogl.org/examples/#importing_gltf_buggy)|[<img src="http://xeogl.org/assets/images/gltf/glTF_reciprocatingSaw_small.png" width="100%">](http://xeogl.org/examples/#importing_gltf_ReciprocatingSaw)|[<img src="http://xeogl.org/assets/images/gltf/glTF_duck_small.png" width="100%">](http://xeogl.org/examples/#importing_gltf_duck)|
|----------|---------|----------|-----------|----------|
|[Entity Explorer Demo](http://xeogl.org/examples/#demos_ui_explorer)|[Flying to Boundaries](http://xeogl.org/examples/#boundaries_flyToBoundary)|[Buggy](http://xeogl.org/examples/#importing_gltf_buggy)|[Reciprocating Saw](http://xeogl.org/examples/#importing_gltf_ReciprocatingSaw)|[Duck](http://xeogl.org/examples/#importing_gltf_duck)|

# Introduction

[glTF](https://github.com/KhronosGroup/glTF) (GL Transmission Format) is an efficient and extensible runtime asset 
delivery format that bridges the gap between content authoring tools and WebGL/OpenGL-based viewing applications. 

[xeogl](http://xeogl.org) is an open-source (MIT license) WebGL-based 3D visualization engine which supports glTF as its 
native data format. xeogl intentionally does not (yet) load animations, cameras and lights from glTF. Instead, it 
focuses on loading **static models** while allowing us to access their components via its scene graph API, so that we can 
programmatically rig them with our own tweens, lights, cameras and so on. Also, xeogl (currently) ignores glTF shaders, 
opting instead to rely on its own internally-generated shaders.  
  
In this tutorial, we're going to load the sample gearbox model (see screenshot above) and show how to interact with its elements 
via the xeogl API. We've added IDs to some of the elements within the glTF file so that we can find them within the 
xeogl scene graph.

If you're not already familiar with xeogl, I recommend first 
reading [Getting Started](https://github.com/xeolabs/xeogl/wiki/Getting-Started). Also, you can get finer details on 
each of the xeogl components in the 
[API documentation](http://xeogl.org/docs/). And if you find anything broken or missing, please hit 
the [issue tracker](https://github.com/xeolabs/xeogl/issues).  

## Motivation

* Load CAD parts assemblies from glTF
* Access loaded entities via the xeogl API to dynamically update their transforms, visibility and appearance
* Query and track the Model, World, View and Canvas-space boundaries of entities via the xeogl API  
 
# Techniques

## Loading a glTF Model

Load a glTF file by creating a [Model](http://xeogl.org/docs/classes/Model.html) component:

````javascript
var gearbox = new xeogl.Model({
   id: "gearbox",
   src: "models/gltf/gearbox/gearbox_assy.gltf"
});
````

We've created this particular Model within xeogl's default [Scene](http://xeogl.org/docs/classes/Scene.html), 
since we didn't specify a Scene to its constructor. Internally, xeogl has lazy-created the default Scene at this point, 
along with its HTML canvas, which fills the available space in the page. We use the default Scene in most examples in 
order to keep the code simple. 
  
A Model prefixes its own ID to those of its components. Like all components, the Model's ID is optional, and defaults to 
the value of the Model's ````src```` property. Therefore, providing our own short Model ID keeps those component IDs 
short and easy to use.
  
### Subscribing to load completion 
The Model begins loading the glTF file immediately. To be notified when the Model has loaded, bind a callback (which 
fires immediately if the Model happens to be already loaded):
  
````javascript
gearbox.on("loaded", function() {
        // Model has loaded!
    });
````

### Switching to a different glTF file
To switch a Model to a different glTF file, simply update its ````src```` property:

````javascript
gearbox.src = "models/gltf/buggy/buggy.gltf"
````
Recall that almost everything in xeogl is dynamically editable. 

## Accessing Model Components
Once the Model has loaded, its Scene will contain various [Components](http://xeogl.org/docs/classes/Component.html) 
that represent the elements of the glTF file. We'll now access some of those components by ID, to query and update them 
programmatically. 

Every Component has a reference to its Scene, so we'll get ours from the Model:

````javascript
var scene = gearbox.scene;
````

We can also get our Scene off the XEO namespace, since it happens to be the default Scene:

````javascript
var scene = xeogl.scene;
````
 
### Accessing transforms

Let's reposition one of the Entities in our Model. We'll get the [Transform](http://xeogl.org/docs/classes/Transform.html) that 
positions our target Entity, in this case a gear. Then we'll update its matrix to translate it ten units along the negative Z-axis.

````javascript
var transform = 
    scene.components["gearbox#n274017_gear_53t-node.transform"];
transform.matrix = xeogl.math.translationMat4v([0,0,-10]);
````

Note the format of the Transform's ID: 

````<Model ID>#<glTF node ID>.transform````
 
From left to right, the format contains the Model's ID, the ID of the glTF node that contains the transform, then 
"transform", to distinguish it from the IDs of any other components loaded from elements on the same glTF node. 

### Accessing materials

TODO

### Accessing entities

Let's make our gear Entity invisible. This time we'll get the Entity itself, then update 
its [Visibility](http://xeogl.org/docs/classes/Visibility.html) component: 

````javascript
var gear53 = scene.components["gearbox#n274017_gear_53.entity.0"];
gear53.visibility.visible = false;
````

Note the format of the Entity's ID: ````<Model ID>#<glTF node ID>.entity.<glTF mesh index>````
 
A glTF scene node may contain multiple meshes, and for each of those xeogl will create an individual Entity. As 
before, the part before the hash is the ID of the Model, which is then followed by the ID of the glTF node, then "entity" 
to signify that this is an Entity ID, then finally an index to differentiate the Entity from those loaded from other 
meshes on the same glTF node.
 
When we load multiple Entities from a glTF node, then they will share the same Transform and Visibility components. This 
lets us update their transformation and visibility as a group, as if they were a composite entity that represents 
the glTF node.

### Querying entity boundaries
We can query an Entity's boundary within the Local, World, View and Canvas coordinate systems, and also attach a callbacks (TODO)    

**Local-space** is the 3D coordinate system that's local to the an Entity's [Geometry](http://xeogl.org/docs/classes/Geometry.html), ie. before modelling transforms are 
applied. Each Entity provides its Local boundary as a [Boundary3D](http://xeogl.org/docs/classes/Boundary3D.html), from 
which we can then get OBB and AABB representations: 
````javascript
var localBoundary = gear53.localBoundary;

var aabb = localBoundary.aabb; // Object containing 3D min/max for each axis 
var obb = localBoundary.obb; // Flattened array of eight 3D box vertices 
var center = localBoundary.center; // 3D point
````

Local-space Boundary3Ds automatically update whenever their Entity's Geometries are edited. We can subscribe to 
notifications of their boundary updates like so:

````javascript
localBoundary.on("updated", function() {    
      var aabb = this.aabb;
      var obb = this.obb;
      var center = this.center;
      // ...etc.
  });
````

**World-space** is the 3D coordinate system after modelling transforms are applied, and **View-space** is the 3D 
coordinate system after both modelling and viewing transforms. Entity's also provide boundaries within those as 
Boundary3D's, which work in same way as the Local-space ones we just saw:    

````javascript
var worldBoundary = gear53.worldBoundary;
//... etc

var viewBoundary = gear53.viewBoundary;
//... etc
````

World-space Boundary3Ds automatically update whenever their Entity's Geometries or modelling Transforms are updated. View-space 
Boundary3Ds automatically update whenever their Entity's Geometries, modelling Transforms or associated 
[Camera](http://xeogl.org/docs/classes/Camera.html)'s view transformations are updated. Subscribe to updates on those 
in the same way as shown above for Local-space Boundaries.

**Canvas-Space** is the 2D canvas coordinate system after modelling, viewing and projection transforms are applied. Each 
Entity provides its Canvas boundary as a [Boundary2D](http://xeogl.org/docs/classes/Boundary2D.html), which provides 
a 2D AABB representation, along with the center point.                     
````javascript
var canvasBoundary = gear53.canvasBoundary;

var aabb = canvasBoundary.aabb; // Object containing 2D min/max for each axis  
var center = canvasBoundary.center; // 2D point
````

Boundary2Ds automatically update their boundaries whenever their Entity's Geometry, modelling Transforms, or Cameras 
viewing or projection transforms are updated. As with Boundary3Ds, we can subscribe to their boundary updates like so:

````javascript
canvasBoundary.on("updated", function() {    
     var aabb = this.aabb;
     var center = this.center;
     // ...etc.
  });
````

### Visualizing entity boundaries

To visualize the World-space [Boundary3D](http://xeogl.org/docs/classes/Boundary3D.html) of our 
Entity, simply create an Entity with a [BoundaryGeometry](http://xeogl.org/docs/classes/BoundaryGeometry.html) that's 
attached to our Entity's World-space Boundary3D: 

````javascript
new xeogl.Entity({
    geometry: new xeogl.BoundaryGeometry({
        boundary: gear53.worldBoundary
    }),
    material: new xeogl.PhongMaterial({
        diffuse: [1,0,0],
        lineWidth: 2
    })
});
````

That Entity will render the Boundary3D as a wireframe box which will automatically adjust whenever the Boundary3D updates.  

### Flying a camera to an entity

Use a [CameraFlight](http://xeogl.org/docs/classes/CameraFlight.html) to fly 
the [Camera](http://xeogl.org/docs/classes/Camera.html) to look at entities:

````javascript
var cameraFlight = new xeogl.CameraFlight(); 
cameraFlight.flyTo(gear53);
````

This CameraFlight is flying the default scene Camera to the World-space boundary of our gear Entity. CameraFlights are 
pretty flexible - they can be set up to control different Cameras, as well as fly to anything that provides an AABB or OBB.
 
See a live example of this [here](http://xeogl.org/examples/#boundaries_flyToBoundary).
 
### Iterating over components
A Model holds its components in a [Collection](http://xeogl.org/docs/classes/Collection.html), 
which conveniently lets us process them as a set, to iterate over them, get their collective boundary etc.

Iterate over the Collection like so:
````javascript
var components = gearbox.collection.components;
for (var id in components) {
  if (components.hasOwnProperty(id)) {
      var component = components[id];
      if (component.isType("xeogl.Entity")) {
         gearboxModel.log("Entity found: " + component.id);
      }
  }
}
````
or with the convenient iterator method:
````javascript
gearbox.collection.iterate(function(component) {
    if (component.isType("xeogl.Entity")) {
        this.log("Entity found: " + component.id);
    }
});
````

* Note how we can check the type of each Component using its ````isType```` method, which returns ````true```` if the 
Component is the given type, or extends the given type.  
* It's a bad idea to add or remove Components to and from a Model's Collection; the Model 
is responsible for the life cycles of the components within its Collection.

### Finding components by type

A Collection also organizes its components by type:
````javascript
var entities = gearbox.collection.types["xeogl.Entity"];
for (var id in entities) {
  if (entities.hasOwnProperty(id)) {
      var entity = entities[id];  
      this.log("Entity found: " + entity.id);
  }
}
```` 

## Querying a Model's Boundary
Use a [CollectionBoundary](http://xeogl.org/docs/classes/CollectionBoundary.html) to get the collective World-space 
AABB and OBB of the Entities within our Model's Collection:
 
````javascript
var collectionBoundary = new xeogl.CollectionBoundary({
    collection: gearbox.collection
});

var worldBoundary = collectionBoundary.worldBoundary;

var aabb = worldBoundary.aabb; // Object containing min/max for each axis 
var obb = worldBoundary.obb; // Flattened array of eight box vertices 
var center = worldBoundary.center; // 3D point
````
The Boundary3D provided by the CollectionBoundary will automatically update whenever the Model is loaded from a 
different source, or whenever the Geometry or Transforms on the Model's Entities are updated. 

As shown earlier, subscribe to updates on the Boundary3D like so:
  
````javascript
worldBoundary.on("updated", function() {     
     var aabb = this.aabb;
     var obb = this.obb;
      
     // ...etc.
});
````

### Visualizing a model's boundary    

To visualize the World-space boundary of our Model, simply create an Entity with 
a [BoundaryGeometry](http://xeogl.org/docs/classes/BoundaryGeometry.html) that's attached to our CollectionBoundary's 
Boundary3D:

````javascript
  new xeogl.Entity({
    geometry: new xeogl.BoundaryGeometry({
        boundary: collectionBoundary.worldBoundary
    }),
    material: new xeogl.PhongMaterial({
        diffuse: [1,0,0],
        lineWidth: 2
    })
});
````
     
That Entity will render the Boundary3D as a wireframe box which will automatically adjust whenever the Model is destroyed, reloaded 
or its Entities are moved.

## Attaching Transforms to Models
Like Entities, we can attach Models to the leaves of [Transform](http://xeogl.org/docs/classes/Transform.html) hierarchies, 
to scale, rotate and translate them within the World coordinate system. 

### Attaching via constructor

We can attach Transform hierarchies to a Model via the Model's constructor: 

````javascript
// Creating a transform hierarchy within the Model constructor
var buggy = new xeogl.Model({
    src: "models/gltf/buggy/buggy.gltf",

    transform: new xeogl.Translate({ 
        xyz: [-35, 0, 0],
        parent: new xeogl.Rotate({ // Rotate about center
            id: "buggyRotation",
            xyz: [0, 1, 0],
            angle: 0,
            parent: new xeogl.Scale({ // Scale just for kicks
                xyz: [0.5, 0.5, 0.5]
            })
        })
    })
});
````

See a live example of this [here](http://xeogl.org/examples/#importing_gltf_techniques_configTransform).

### Attaching after loading

Alternatively, following xeogl's emphasis on complete scene mutability, existing Model instances can be dynamically 
attached to [Transform](http://xeogl.org/docs/classes/Transform.html) hierarchies at any time: 

````javascript
// Attaching an existing Model to a transform hierarchy
buggy.transform = new xeogl.Translate({ 
    xyz: [-35, 0, 0],

    parent: new xeogl.Rotate({ // Rotate about center
        id: "buggyRotation",
        xyz: [0, 1, 0],
        angle: 0,
        parent: new xeogl.Scale({ // Scale just for kicks
            xyz: [0.5, 0.5, 0.5]
        })
    })
});
````

See a live example of this [here](http://xeogl.org/examples/#importing_gltf_techniques_attachTransform).

### Animating transforms

Now we can animate the [Transforms](http://xeogl.org/docs/classes/Transform.html) in the usual way:

````javascript
// Walk up the Model's transform hierarchy to the Rotate
// and update its angle 
buggy.transform.parent.angle++;

// Walk up the Model's transform hierarchy to the Translate 
// and update its translation vector
buggy.transform.parent.parent.xyz = [2,0,0];

// Alternatively, since we put IDs on the transform components, 
// we can find them by ID
var buggyRotation = gearbox.scene.components["buggyRotation"];
buggyRotation.angle = 45;

````

## Optimizing glTF for Performance

TODO: Notes on how best to structure glTF for fast rendering on xeogl

# Acknowledgements

* The [Khronos Group](https://github.com/KhronosGroup) and the [glTF](https://github.com/KhronosGroup/glTF) project 
* [Patrick Cozzi](https://github.com/pjcozzi) and [Tony Parisi](https://github.com/tparisi) for the glTF specification and accompanying tutorials
* Tony Parisi for the [glTF parsers](https://github.com/KhronosGroup/glTF/tree/master/loaders) on which xeogl's parser is based   












