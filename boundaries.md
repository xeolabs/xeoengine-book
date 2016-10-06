# Boundaries

### 3D Boundaries


```` javascript
var entity = new XEO.Entity({
    geometry: new XEO.BoxGeometry(),
    transform: new XEO.Translate({
        xyz: [-5, 0, 0]
    })
});
````

````javascript
var worldBoundary = entity.worldBoundary;

worldBoundary.on("updated", function() {
    var obb = worldBoundary.obb;
    var aabb = worldBoundary.aabb;
    var center = worldBoundary.center;

    //...
});
````

````javascript
var x = 0;
entity.scene.on("tick", function() {
    translate.xyz: [x, 0, 0];
    x += 0.5;
});
````
  
### 2D Boundaries


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

### Collection Boundaries

Sometimes we need to automatically track the collective World-space boundary of a group of [Entities](). The xeoEngine 
[CollectionBoundary]() component makes this easy and efficient. 

First, we create a Collection to hold our Entities, then attach a CollectionBoundary, get the CollectionBoundary's 
Boundary3D, and then at any time we can query the Boundary3D for its extents. We can also attach an "updated" listener 
to the Boundary3D, to get a notification whenever its extents change. 

![](http://xeoengine.org/assets/images/CollectionBoundary.png)

To show it this works, let's start by creating a [Collection]() to hold some [Entities]():

````javascript
var collection = new XEO.Collection();
````

Then we'll attach a [CollectionBoundary]() to our [Collection]():

````javascript
var collectionBoundary = new XEO.CollectionBoundary({    
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
var torus = new XEO.Entity({ 
    // Red torus    
    geometry: new XEO.TorusGeometry(),    
    material: new XEO.PhongMaterial({        
        diffuse: [1.0, 0.3, 0.3]    
    }),    
    transform: new XEO.Translate({        
        xyz: [-10, 0, 0]    
    })
});

var sphere = new XEO.Entity({ 
    // Green sphere    
    geometry: new XEO.SphereGeometry(),    
    material: new XEO.PhongMaterial({        
        diffuse: [0.3, 1.0, 0.3]    }),    
    transform: new XEO.Translate({        
        xyz: [0, 0, 0]    
    })
});

var box = new XEO.Entity({ 
    // Blue box    
    geometry: new XEO.BoxGeometry(),    
    material: new XEO.PhongMaterial({        
        diffuse: [0.3, 0.3, 1.0]    
    }),    
    transform: new XEO.Translate({       
        xyz: [10, 0, 0]    
    })
});

collection.add(torus);
collection.add(sphere);
collection.add(box);
````

#### Buffering Goodness
Note that, even though we added three [Entities]() to the [Collection](), our "updated" listener will only fire once. 
That's thanks to the way that xeoEngine buffers everything as tasks to it's internal FIFO task queue. The [Collection]() 
fires an "added" event for each Entity we add to it, which triggers the CollectionBoundary to schedule a rebuild task to 
the queue **if it hasn't scheduled the rebuild already**. In short, this buffering system means that three Entity 
additions to the [Collection]() triggers just one rebuild of the [CollectionBoundary](). 

