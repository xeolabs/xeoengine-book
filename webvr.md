# ZSpace Effect

A **ZSpace** component makes its [````Scene````](http://xeogl.org/docs/classes/Scene.html) viewable with a zSpace viewer.

* Plug-and-play: just create a ZSpace component within your xeogl [````Scene````](http://xeogl.org/docs/classes/Scene.html) to make it viewable with a ZSpace display.
* Activate or disable the ZSpace component at any time to switch between zSpace mode and normal mono viewing mode.
* Requires WebGL2 and WebVR support, which you'll have if you're running on a zSpace viewer.
* Attaches to a [Camera](http://xeogl.org/docs/classes/Camera.html), defaults to its [````Scene````](http://xeogl.org/docs/classes/Scene.html)'s default
 [````Scene````](http://xeogl.org/docs/classes/Scene.html#property_camera) if none is specified.
* Don't attach different view or projection transform components to the [Camera](http://xeogl.org/docs/classes/Camera.html) while the ZSpace component is active.
* You can however update the [Camera](http://xeogl.org/docs/classes/Camera.html)'s view transformation at any time, to move the
 viewpoint around.

 <img src="http://xeogl.org/assets/images/ZSpace.png"></img>

## Limitations

* Stylus tracking only works correctly when the browser window is maximized to fill the zSpace display. This is because
 stylus tracking requires that we know the location of the window within the display, which is currently only possible
 when the window is maximized.

## Examples

 <ul>
 <li>[zSpace cube](../../examples/webvr_zspace_cube.html)</li>
 <li>[zSpace with random geometries](../../examples/webvr_zspace_geometries.html)</li>
 <li>[zSpace with glTF gearbox model](../../examples/webvr_zspace_gearbox.html)</li>
 <li>[zSpace with glTF gearbox model and entity explorer](../../examples/webvr_zspace_gearbox_explorer.html)</li>
 </ul>

## Usage

 In the following example we're going to set up a ZSpace-viewable scene with xeogl, defining the scene step-by-step to
 emphasize the plug-and-play design of xeogl's API.

#### 1. Create an entity

 First we'll create a simple torus-shaped [Entity](http://xeogl.org/docs/classes/Entity.html), which will be within xeogl's default
 [````Scene````](http://xeogl.org/docs/classes/Scene.html), since we're not defining the [````Scene````](http://xeogl.org/docs/classes/Scene.html) component
 explicitly. Our [Entity](http://xeogl.org/docs/classes/Entity.html) is also implicitly connected to the
 [````Scene````](http://xeogl.org/docs/classes/Scene.html)'s default [Camera](http://xeogl.org/docs/classes/Camera.html), since we didn't create
 a [Camera](http://xeogl.org/docs/classes/Camera.html) for it either.

 ````javascript
 var entity = new xeogl.Entity({
     geometry: new xeogl.TorusGeometry(),
     material: new xeogl.PhongMaterial({
        diffuseMap: new xeogl.Texture({
            src: "textures/diffuse/uvGrid2.jpg"
        })
     })
 });
 ````

#### 2. Enable mouse/keyboard camera interaction

 At this point we've got a textured torus floating in the middle of the canvas (which is also created automatically
 since we didn't specify one). Now we'll create a
 [CameraControl](http://xeogl.org/docs/classes/CameraControl.html), which allows us to move our viewpoint around with the mouse and
 keyboard. This component is also within xeogl's default [````Scene````](http://xeogl.org/docs/classes/Scene.html) and connected to the
 [````Scene````](http://xeogl.org/docs/classes/Scene.html)'s default [Camera](http://xeogl.org/docs/classes/Camera.html).

 ````javascript
 new CameraControl();
 ````

#### 3. Enable ZSpace viewing

Now we can orbit, pan and zoom around the torus with the mouse and keyboard. Let's view it on a ZSpace display by simply dropping a ZSpace component into our default [````Scene````](http://xeogl.org/docs/classes/Scene.html).

 ````javascript
 var zspace = new ZSpace();
 ````

The ZSpace component immediately activates, so at this point if we're running on a ZSpace device we'll have a stereo
 view of the torus, which we can view with the stereo glasses.

 At any point we can always disable the ZSpace effect to switch between normal WebGL mono viewing mode:

 ````javascript
 zspace.active = false; // Back to normal mono viewing..
 zspace.active = true; // ..and then back to ZSpace stereo mode.
 ````

## Detecting support

The **ZSpace** will fire a "supported" event once it has determined whether or not the browser
 supports a zSpace viewer:

````javascript
zspace.on("supported", function (supported) {

    if (!supported) {

        // Not a zSpace device
        // Log error on the xeogl.ZSpace component
        this.error("This computer is not a ZSpace viewer!"); 
    }
});
````

## Handling stylus input

Reading the current World-space position and direction of the stylus:


````javascript
var stylusPos = zspace.stylusPos;
var stylusDir = zspace.stylusDir;
````

Note that these properties only have meaningful values once the ZSpace has fired at least one "stylusMoved" event.

 Subscribing to stylus movement:

 ````javascript
 zspace.on("stylusMoved", function() {
     var stylusPos = zspace.stylusPos;
     var stylusDir = zspace.stylusDir;
     //...
 });
 ````

 Reading the current state of each stylus button:

 ````javascript
 var button0 = zspace.stylusButton0; // Boolean
 var button1 = zspace.stylusButton1;
 var button2 = zspace.stylusButton2;
 ````

 Subscribing to change of state of each stylus button:

 ````javascript
 zspace.on("stylusButton0", function(value) { // Boolean value
     this.log("stylusButton0 = " + value);
 });

 zspace.on("stylusButton1", function(value) {
     this.log("stylusButton1 = " + value);
 });

 zspace.on("stylusButton2", function(value) {
     this.log("stylusButton2 = " + value);
 });
 ````

 Picking an [Entity](http://xeogl.org/docs/classes/Entity.html) with the stylus when button 0 is pressed:

 ````javascript
 zspace.on("stylusButton0", function() {

    var hit = zspace.scene.pick({
        pickSurface: true,
        origin: zspace.stylusPos,
        direction: zspace.stylusDir
    });

    if (hit) { // Picked an Entity

        var entity = hit.entity;

        // Other properties on the hit result:

        var primitive = hit.primitive; // Type of primitive that was picked, usually "triangles"
        var primIndex = hit.primIndex; // Position of triangle's first index in the picked Entity's Geometry's indices array
        var indices = hit.indices; // UInt32Array containing the triangle's vertex indices
        var localPos = hit.localPos; // Float32Array containing the picked Local-space position within the triangle
        var worldPos = hit.worldPos; // Float32Array containing the picked World-space position within the triangle
        var viewPos = hit.viewPos; // Float32Array containing the picked View-space position within the triangle
        var bary = hit.bary; // Float32Array containing the picked barycentric position within the triangle
        var normal = hit.normal; // Float32Array containing the interpolated normal vector at the picked position on the triangle
        var uv = hit.uv; // Float32Array containing the interpolated UV coordinates at the picked position on the triangle

        //...
    }
 });
 ````