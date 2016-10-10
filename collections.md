# Collections

A Collection is a general-purpose group of Components.

 ````javascript
 var material = new xeogl.PhongMaterial({
     id: "myMaterial",
     diffuse: [0.5, 0.5, 0.0]
 });

 var geometry = new xeogl.BoxGeometry();

 var entity = new xeogl.Entity({
    id: "myEntity",
    material: material,
    geometry: geometry
 });

 // Collection initialized with three components
 var collection1 = new xeogl.Collection({ 
     components: [
         "myMaterial",
         geometry,
         myEntity
     ]
 });
 ````



 ````javascript
 var collection2 = new xeogl.Collection();

 collection2.add([  // Add two components
    geometry,
    "xeogl.Entity",
 ]);
 ````

 ### Accessing Components in a Collection

 Iterate over the components in a Collection using the convenience iterator:

 ````javascript
 collection1.iterate(function(component) {
     if (component.isType("xeogl.Entity")) {
         this.log("Found the Entity: " + component.id);
     }
     //..
 });
 ````

 A Collection also registers its components by type:

 ````javascript
 var entities = collection1.types["xeogl.Entity"];
 var theEntity = entities["myEntity"];
 ````

### Removing Components from a Collection

 We can remove components from a Collection by instance, ID or type:

````javascript
// Remove one component by ID
collection1.remove("myMaterial"); 

// Remove two components by instance
collection1.remove([geometry, myEntity]); 

// Remove all Geometries
collection2.remove("xeogl.Geometry"); 
````

### Getting the 3D Boundary of a Collection

A [CollectionBoundary](http://xeogl.org/docs/classes/CollectionBoundary.html) provides a [Boundary3D](http://xeogl.org/docs/classes/Boundary3D.html) that
 dynamically fits to the collective World-space boundary of all the Components in a [Collection](http://xeogl.org/docs/classes/Collection.html).

 ````javascript
 var collectionBoundary = new xeogl.CollectionBoundary({
    collection: collection1
 });

 var worldBoundary = collectionBoundary.worldBoundary;
 ````
 The [Boundary3D](http://xeogl.org/docs/classes/Boundary3D.html) will automatically update whenever we add, remove or update any Components that have World-space boundaries. We can subscribe to updates on it like so:

 ````javascript
 worldBoundary.on("updated", function() {
     obb = worldBoundary.obb;
     aabb = worldBoundary.aabb;
     center = worldBoundary.center;
     //...
 });
 ````

 Now, if we now re-insert our {{#crossLink "Entity"}}{{/crossLink}} into to our Collection,
 the {{#crossLink "Boundary3D"}}{{/crossLink}} will fire our update handler.

 ````javascript
 collection1.add(myEntity);
 ````
