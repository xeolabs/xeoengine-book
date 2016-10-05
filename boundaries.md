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


