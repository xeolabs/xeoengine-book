# Picking

## Canvas Picking

#### Canvas Picking Entities

The most basic type of picking involves finding the closest entity at the given canvas coordinates. This is equivalent to firing a ray through the canvas, down the negative Z-axis, to find the closest intersecting entity. However, this type of  picking only finds the entity and does not return any information about the ray intersection.

To start with, let's create a [Model](http://xeogl.org/docs/classes/Model.html) component that loads a glTF model of a gearbox. We'll use this scene in all our examples.

```javascript
var gearbox = new xeogl.Model({ 
    src: "models/gltf/gearbox/gearbox_assy.gltf" 
});  

// Our gearbox is not centered at the 
// World-space origin, so we need to 
// move the camera to arrange it in view 

var view = gearbox.scene.camera.view; 
view.eye = [184.21, 10.54, -7.03]; 
view.look = [159.20, 17.02, 3.21]; 
view.up = [-0.15, 0.97, 0.13]; 
```

Now let's attempt to pick the closest entity at the given canvas coordinates:

```javascript
var hit = gearbox.scene.pick({  canvasPos: [500,400] });
```

If we hit an entity, then xeogl will return a hit result containing a reference to the entity:

```javascript
if (hit) { 
   var entity = hit.entity;  
}
```

Internally, xeogl performs the following steps for this type of picking:

1. User picks at given canvas coordinates. 
2. Do a render pass to a hidden frame buffer, rendering each entity with a unique colour. Each colour is the RBGA-encoded index of the entity's position within xeogl's internal display list. 
3. Read the colour from the framebuffer at the canvas coordinates, map the colour back to the entity. 

#### Canvas Picking Triangles

As with the previous example, this type of picking fires a ray through the canvas, down the negative Z-axis, to pick the first entity that intersects the ray. However, this time we'll get some geometric information about the intersection.
Reusing the scene that we created for the previous example, we'll now fire a ray through the canvas coordinates, this time  supplying a `pickSurface` flag, causing it to pick a 3D **position** on the surface of the entity:

```javascript
var hit = gearbox.scene.pick({      
    canvasPos: [500,400],     pickSurface: true, // <<--------- Indicates that we want to pick on surface });````

This time, in addition to the entity, the hit result will contain information about the position that we picked on the entity's surface:

````javascriptif (hit) { // Picked an Entity

 var entity = hit.entity; // Entity we picked var primitive = hit.primitive; // Type of primitive we picked, usually "triangles" var primIndex = hit.primIndex; // Triangle's first index within geometry indices var indices = hit.indices; // Triangle's vertex indices (a three element array) var localPos = hit.localPos; // Local-space position within the triangle var worldPos = hit.worldPos; // World-space position within the triangle var viewPos = hit.viewPos; // View-space position within the triangle var bary = hit.bary; // Barycentric coordinate within the triangle var normal = hit.normal; // Interpolated normal vector within the triangle var uv = hit.uv; // Interpolated UVs within the triangle}````

xeogl performs the following steps for this type of picking: 1. User ray-picks at given canvas coordinates.2. Do a render pass to a hidden frame buffer, rendering each entity with a unique colour. Each colour is the RBGA-encoded index of the entity within xeogl's internal display list.3. Read the colour from the framebuffer at the canvas coordinates, map the colour back to the entity. 4. Clear the framebuffer, and render a second render pass, this time rendering only the triangles of the picked entity, each with a unique color. Each colour is the RBGA-encoded index of the triangle within the entity's geometry.5. Read the colour from the framebuffer at the canvas coordinates, map the colour back to the triangle.6. Now that we have the entity and the triangle, make a ray in clip-space from the eye position that passes through the near projection plane, then unproject that ray to get a ray in the entity's local coordinate space.7. Find the intersection of the ray with the triangle in local space.8. Find the barycentric coordinates of the local-space intersection, then use those to interpolate within the triangle to find the normal vector and UV coordinates at that position.

For step (4) we lazy-compute geometry arrays to render individually-colored triangles for the entity. This does have a small performance hit the first time you pick the entity, but those arrays are retained for the entity and reused as you continue to pick it.


##Ray Casting

#### Ray Casting Entities

[![]({{ site.url }}/images/xeogl/worldRayPicking.gif)](http://xeogl.org/examples/#interaction_picking_raycasting_triangles) 
<br>[Click to run](http://xeogl.org/examples/#interaction_picking_raycasting_triangles) 

#### Ray Casting Triangles

World-space ray casting involves fire an arbitrarily-positioned ray through the scene in World-space, to pick 
 the first entity that intersects the ray. This is usually done when we want to interact with a 
  scene using a 3D stylus input device.<br><br>
[![]({{ site.url }}/images/xeogl/worldRayPicking.gif)](http://xeogl.org/examples/#interaction_picking_raycasting_triangles) 
<br>[Click to run](http://xeogl.org/examples/#interaction_picking_raycasting_triangles) 
<br><br>
Fire a ray through the scene in World-space, to pick a 3D position on the surface of the first entity hit by the ray:

````javascript
var hit = scene.pick({
    origin: [0,0,-5],   // Ray origin
    direction: [0,0,1], // Ray direction
    pickSurface: true   // <<--------- Indicates that we want to pick on surface
});

if (hit) { // Picked an Entity with the ray

    // Hit result contains the same properties as the previous example    

     var entity = hit.entity;        // Entity we picked
     var primitive = hit.primitive;  // Type of primitive we picked, usually "triangles"
     var primIndex = hit.primIndex;  // Triangle's first index within geometry indices
     var indices = hit.indices;      // Triangle's vertex indices (a three element array)
     var localPos = hit.localPos;    // Local-space position within the triangle
     var worldPos = hit.worldPos;    // World-space position within the triangle
     var viewPos = hit.viewPos;      // View-space position within the triangle
     var bary = hit.bary;            // Barycentric coordinate within the triangle
     var normal = hit.normal;        // Interpolated normal vector within the triangle
     var uv = hit.uv;                // Interpolated UVs within the triangle
}
```

xeogl performs the following steps for this type of picking:

1. User ray-picks at given canvas coordinates.
2. Do a render pass to a hidden frame buffer, rendering each entity with a unique colour. Each colour is the RBGA-encoded 
  index of the entity within xeogl's internal display list.
3. Read the colour from the framebuffer at the canvas coordinates, map the colour back to the entity. 
4. Clear the framebuffer, and render a second render pass, this time rendering only the triangles of the picked entity, 
  each with a unique color. Each colour is the RBGA-encoded index of the triangle within the entity's geometry.
5. Read the colour from the framebuffer at the canvas coordinates, map the colour back to the triangle.
6. Now that we have the entity and the triangle, make a ray in clip-space from the eye position that passes through 
  the near projection plane, then unproject that ray to get a ray in the entity's local coordinate space.
7. Find the intersection of the ray with the triangle in local space.
8. Find the barycentric coordinates of the local-space intersection, then use those to interpolate within the triangle 
  to find the normal vector and UV coordinates at that position.  

