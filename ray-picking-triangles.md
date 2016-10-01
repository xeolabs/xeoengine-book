# Ray-Picking Triangles

World-space ray casting involves fire an arbitrarily-positioned ray through the scene in World-space, to pick 
 the first entity that intersects the ray. This is usually done when we want to interact with a 
  scene using a 3D stylus input device.<br><br>
[![]({{ site.url }}/images/xeoengine/worldRayPicking.gif)](http://xeoengine.org/examples/#interaction_picking_raycasting_triangles) 
<br>[Click to run](http://xeoengine.org/examples/#interaction_picking_raycasting_triangles) 
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
````

xeoEngine performs the following steps for this type of picking: 
 
1. User ray-picks at given canvas coordinates.
2. Do a render pass to a hidden frame buffer, rendering each entity with a unique colour. Each colour is the RBGA-encoded 
index of the entity within xeoEngine's internal display list.
3. Read the colour from the framebuffer at the canvas coordinates, map the colour back to the entity. 
4. Clear the framebuffer, and render a second render pass, this time rendering only the triangles of the picked entity, 
each with a unique color. Each colour is the RBGA-encoded index of the triangle within the entity's geometry.
5. Read the colour from the framebuffer at the canvas coordinates, map the colour back to the triangle.
6. Now that we have the entity and the triangle, make a ray in clip-space from the eye position that passes through 
the near projection plane, then unproject that ray to get a ray in the entity's local coordinate space.
7. Find the intersection of the ray with the triangle in local space.
8. Find the barycentric coordinates of the local-space intersection, then use those to interpolate within the triangle 
to find the normal vector and UV coordinates at that position.  

<br>