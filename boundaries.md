# Boundaries

Bounding boxes may be "axis-aligned," meaning that no matter what the orientation of the underlying shape, the sides of the bounding box are always parallel to the X/Y/Z places.  Or they may be "oriented," meaning that the box rotates with the underlying shape.  With axis-aligned bounding boxes (AABBs) the collision detection is extremely simple, but false-positives are more common.  With OBBs, false positives are less common since they fit more tightly to the underlying shapes, but the collision detection is a bit more complicated.

### World Space Scene Boundary



### World Space Entity Boundaries

First, let's create an [Entity]() and give it a dynamically-updated [Translate]() viewing transform to make it 
glide across the scene.  

```` javascript
var entity = new xeogl.Entity({
    geometry: new xeogl.BoxGeometry(),
    transform: new xeogl.Translate({
        xyz: [-5, 0, 0]
    })
});
````

````javascript
var x = 0;
entity.scene.on("tick", function() {
    translate.xyz: [x, 0, 0];
    x += 0.5;
});
````
  
Now we'll get the [Entity]()'s [WorldBoundary]() and subscribe to updates to its extents. 

````javascript
var worldBoundary = entity.worldBoundary;

worldBoundary.on("updated", function() {
    var obb = worldBoundary.obb;
    var aabb = worldBoundary.aabb;
    var center = worldBoundary.center;

    //...
});
````


### View Space Entity Boundaries

 ````javascript
var viewBoundary = entity.viewBoundary;
  
viewBoundary.on("updated", function() {
    var obb = viewBoundary.obb;
    var aabb = viewBoundary.aabb;
    var center = viewBoundary.center;
  
    //...
});
````

### Canvas Space Entity Boundaries

```` javascript
var canvasBoundary = entity.canvasBoundary;

canvasBoundary.on("updated", function() {
    var aabb = canvasBoundary.aabb;
    var center = canvasBoundary.center;

    //...
});
````
````javascript
var x = 0;
entity.scene.on("tick", function() {
    entity.transform.xyz: [x, 0, 0];
    x += 0.5;
});
````

### World Space Collection Boundaries

Sometimes we need to automatically track the collective World-space boundary of a group of [Entities](). The xeogl 
[CollectionBoundary]() component makes this easy and efficient. 

First, we create a Collection to hold our Entities, then attach a CollectionBoundary, get the CollectionBoundary's 
Boundary3D, and then at any time we can query the Boundary3D for its extents. We can also attach an "updated" listener 
to the Boundary3D, to get a notification whenever its extents change. 

![](http://xeogl.org/assets/images/CollectionBoundary.png)

To show it this works, let's start by creating a [Collection]() to hold some [Entities]():

````javascript
var collection = new xeogl.Collection();
````

Then we'll attach a [CollectionBoundary]() to our [Collection]():

````javascript
var collectionBoundary = new xeogl.CollectionBoundary({    
    collection: collection});
````

Next, we'll get the [Boundary3D]() from the [CollectionBoundary] and add a listener to it observe updates to its 
World-space extents:

````javascript
var worldBoundary = collectionBoundary.worldBoundary;
worldBoundary.on("updated", function() {    
    var obb = worldBoundary.obb;    
    var aabb = worldBoundary.aabb;    
    var center = worldBoundary.center;
});
````

Finally, we'll add three [Entities]() to our [Collection](), which will cause our [CollectionBoundary]()'s [Boundary3D]() 
to expand to fit them, which in turn fire our [Boundary3D]()'s "updated" listener.

````javascript
var torus = new xeogl.Entity({ 
    // Red torus    
    geometry: new xeogl.TorusGeometry(),    
    material: new xeogl.PhongMaterial({        
        diffuse: [1.0, 0.3, 0.3]    
    }),    
    transform: new xeogl.Translate({        
        xyz: [-10, 0, 0]    
    })
});

var sphere = new xeogl.Entity({ 
    // Green sphere    
    geometry: new xeogl.SphereGeometry(),    
    material: new xeogl.PhongMaterial({        
        diffuse: [0.3, 1.0, 0.3]    }),    
    transform: new xeogl.Translate({        
        xyz: [0, 0, 0]    
    })
});

var box = new xeogl.Entity({ 
    // Blue box    
    geometry: new xeogl.BoxGeometry(),    
    material: new xeogl.PhongMaterial({        
        diffuse: [0.3, 0.3, 1.0]    
    }),    
    transform: new xeogl.Translate({       
        xyz: [10, 0, 0]    
    })
});

collection.add(torus);
collection.add(sphere);
collection.add(box);
````

#### Buffering Goodness
Note that, even though we added three [Entities]() to the [Collection](), our "updated" listener will only fire once. 
That's thanks to the way that xeogl buffers everything as tasks to it's internal FIFO task queue. The [Collection]() 
fires an "added" event for each Entity we add to it, which triggers the CollectionBoundary to schedule a rebuild task to 
the queue **if it hasn't scheduled the rebuild already**. In short, this buffering system means that three Entity 
additions to the [Collection]() triggers just one rebuild of the [CollectionBoundary](). 
