# Canvas Picking Entities
 
The most basic type of picking involves finding the closest entity at the given canvas coordinates. This is equivalent to firing a ray through the canvas, down the negative Z-axis, to find the closest intersecting entity. However, this type of  picking only finds the entity and does not return any information about the ray intersection.


To start with, let's create a [Model](http://xeoengine.org/docs/classes/Model.html) component that loads a glTF model of a gearbox. We'll use this scene in all our examples.

````javascript 
var gearbox = new XEO.Model({ src: "models/gltf/gearbox/gearbox_assy.gltf" });  

// Our gearbox is not centered at the World-space origin, 
// so we need to move the camera to arrange it in view var view = gearbox.scene.camera.view; 
view.eye = [184.21, 10.54, -7.03]; 
view.look = [159.20, 17.02, 3.21]; 
view.up = [-0.15, 0.97, 0.13]; 
````

  Now let's attempt to pick the closest entity at the given canvas coordinates:  

````javascript 
var hit = gearbox.scene.pick({  canvasPos: [500,400] });
````

 If we hit an entity, then xeoEngine will return a hit result containing a reference to the entity: 

````javascript 
if (hit) { 
   var entity = hit.entity;  
}
````

Internally, xeoEngine performs the following steps for this type of picking:  

1. User picks at given canvas coordinates. 
2. Do a render pass to a hidden frame buffer, rendering each entity with a unique colour. Each colour is the RBGA-encoded index of the entity's position within xeoEngine's internal display list. 
3. Read the colour from the framebuffer at the canvas coordinates, map the colour back to the entity. 