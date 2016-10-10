# Importing glTF - Accessing Components

Once the Model has loaded, its Scene will contain various [Components](http://xeogl.org/docs/classes/Component.html) 
that represent the elements of the glTF file. We'll now access some of those components by ID, to query and update them 
programmatically.

Every Component has a reference to its Scene, so we'll get ours from the Model:

```javascript
var scene = gearbox.scene;
```

We can also get our Scene off the XEO namespace, since it happens to be the default Scene:

```javascript
var scene = xeogl.scene;
```

### Accessing glTF transforms

Let's reposition one of the Entities in our Model. We'll get the [Transform](http://xeogl.org/docs/classes/Transform.html) that 
positions our target Entity, in this case a gear. Then we'll update its matrix to translate it ten units along the negative Z-axis.

```javascript
var transform = 
    scene.components["gearbox#n274017_gear_53t-node.transform"];
transform.matrix = xeogl.math.translationMat4v([0,0,-10]);
```

Note the format of the Transform's ID:

`<Model ID>#<glTF node ID>.transform`

From left to right, the format contains the Model's ID, the ID of the glTF node that contains the transform, then 
"transform", to distinguish it from the IDs of any other components loaded from elements on the same glTF node.

### Accessing glTF materials

TODO

### Accessing glTF entities

Let's make our gear Entity invisible. This time we'll get the Entity itself, then update 
its [Visibility](http://xeogl.org/docs/classes/Visibility.html) component:

```javascript
var gear53 = scene.components["gearbox#n274017_gear_53.entity.0"];
gear53.visibility.visible = false;
```

Note the format of the Entity's ID: `<Model ID>#<glTF node ID>.entity.<glTF mesh index>`

A glTF scene node may contain multiple meshes, and for each of those ````xeogl```` will create an individual Entity. As 
before, the part before the hash is the ID of the Model, which is then followed by the ID of the glTF node, then "entity" 
to signify that this is an Entity ID, then finally an index to differentiate the Entity from those loaded from other 
meshes on the same glTF node.

When we load multiple Entities from a glTF node, then they will share the same Transform and Visibility components. This 
lets us update their transformation and visibility as a group, as if they were a composite entity that represents 
the glTF node.

### Querying glTF entity boundaries

We can query an Entity's boundary within the Local, World, View and Canvas coordinate systems, and also attach a callbacks \(TODO\)

**Local-space** is the 3D coordinate system that's local to the an Entity's [Geometry](http://xeogl.org/docs/classes/Geometry.html), ie. before modelling transforms are 
applied. Each Entity provides its Local boundary as a [Boundary3D](http://````xeogl````.org/docs/classes/Boundary3D.html), from 
which we can then get OBB and AABB representations:

```javascript
var localBoundary = gear53.localBoundary;

var aabb = localBoundary.aabb; // Object containing 3D min/max for each axis 
var obb = localBoundary.obb; // Flattened array of eight 3D box vertices 
var center = localBoundary.center; // 3D point
```

Local-space Boundary3Ds automatically update whenever their Entity's Geometries are edited. We can subscribe to 
notifications of their boundary updates like so:

```javascript
localBoundary.on("updated", function() {    
      var aabb = this.aabb;
      var obb = this.obb;
      var center = this.center;
      // ...etc.
  });
```

**World-space** is the 3D coordinate system after modelling transforms are applied, and **View-space** is the 3D 
coordinate system after both modelling and viewing transforms. Entity's also provide boundaries within those as 
Boundary3D's, which work in same way as the Local-space ones we just saw:

```javascript
var worldBoundary = gear53.worldBoundary;
//... etc

var viewBoundary = gear53.viewBoundary;
//... etc
```

World-space Boundary3Ds automatically update whenever their Entity's Geometries or modelling Transforms are updated. View-space 
Boundary3Ds automatically update whenever their Entity's Geometries, modelling Transforms or associated 
[Camera](http://````xeogl````.org/docs/classes/Camera.html)'s view transformations are updated. Subscribe to updates on those 
in the same way as shown above for Local-space Boundaries.

**Canvas-Space** is the 2D canvas coordinate system after modelling, viewing and projection transforms are applied. Each 
Entity provides its Canvas boundary as a [Boundary2D](http://````xeogl````.org/docs/classes/Boundary2D.html), which provides 
a 2D AABB representation, along with the center point.

```javascript
var canvasBoundary = gear53.canvasBoundary;

var aabb = canvasBoundary.aabb; // Object containing 2D min/max for each axis  
var center = canvasBoundary.center; // 2D point
```

Boundary2Ds automatically update their boundaries whenever their Entity's Geometry, modelling Transforms, or Cameras 
viewing or projection transforms are updated. As with Boundary3Ds, we can subscribe to their boundary updates like so:

```javascript
canvasBoundary.on("updated", function() {    
     var aabb = this.aabb;
     var center = this.center;
     // ...etc.
  });
```

### Visualizing glTF entity boundaries

To visualize the World-space [Boundary3D](http://````xeogl````.org/docs/classes/Boundary3D.html) of our 
Entity, simply create an Entity with a [BoundaryGeometry](http://````xeogl````.org/docs/classes/BoundaryGeometry.html) that's 
attached to our Entity's World-space Boundary3D:

```javascript
new ````xeogl````.Entity({
    geometry: new ````xeogl````.BoundaryGeometry({
        boundary: gear53.worldBoundary
    }),
    material: new ````xeogl````.PhongMaterial({
        diffuse: [1,0,0],
        lineWidth: 2
    })
});
```

That Entity will render the Boundary3D as a wireframe box which will automatically adjust whenever the Boundary3D updates.

### Flying cameras to glTF entities

Use a [CameraFlight](http://````xeogl````.org/docs/classes/CameraFlight.html) to fly 
the [Camera](http://````xeogl````.org/docs/classes/Camera.html) to look at entities:

```javascript
var cameraFlight = new ````xeogl````.CameraFlight(); 
cameraFlight.flyTo(gear53);
```

This CameraFlight is flying the default scene Camera to the World-space boundary of our gear Entity. CameraFlights are 
pretty flexible - they can be set up to control different Cameras, as well as fly to anything that provides an AABB or OBB.

See a live example of this [here](http://````xeogl````.org/examples/#boundaries_flyToBoundary).

### Iterating over glTF components

A Model holds its components in a [Collection](http://xeogl.org/docs/classes/Collection.html), 
which conveniently lets us process them as a set, to iterate over them, get their collective boundary etc.

Iterate over the Collection like so:

```javascript
var components = gearbox.collection.components;
for (var id in components) {
  if (components.hasOwnProperty(id)) {
      var component = components[id];
      if (component.isType("xeogl.Entity")) {
         gearboxModel.log("Entity found: " + component.id);
      }
  }
}
```

or with the convenient iterator method:

```javascript
gearbox.collection.iterate(function(component) {
    if (component.isType("xeogl.Entity")) {
        this.log("Entity found: " + component.id);
    }
});
```

* Note how we can check the type of each Component using its `isType` method, which returns `true` if the 
  Component is the given type, or extends the given type.  
* It's a bad idea to add or remove Components to and from a Model's Collection; the Model 
  is responsible for the life cycles of the components within its Collection.

### Finding glTF components by type

A Collection also organizes its components by type:

```javascript
var entities = gearbox.collection.types["xeogl.Entity"];
for (var id in entities) {
  if (entities.hasOwnProperty(id)) {
      var entity = entities[id];  
      this.log("Entity found: " + entity.id);
  }
}
```

