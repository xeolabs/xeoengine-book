# First Example

In this first tutorial we'll create the spinning torus shown in the screenshot below. To show how scenes are dynamically editable, we'll create it incrementally: first we'll create the basic torus entity, then set it spinning, and then add texture. 
        
<a href="http://xeoengine.org/examples/#materials_fresnel_specular"><img src="http://xeoengine.org/assets/images/torus.png" width="500px"></a>
      
## Creating the Scene

First, include [xeoengine.min.js](https://github.com/xeolabs/xeoengine/tree/master/build) in your HTML page:
````html
<script src="xeoengine.min.js"/>
````

Next, create the 3D scene as an entity-component graph, as shown in the diagram below. Note how a
[Scene](http://xeoengine.org/docs/classes/Scene.html) is basically a container 
of [Components](http://xeoengine.org/docs/classes/Component.html) that are tied together 
by [Entities](http://xeoengine.org/docs/classes/Entity.html).

````javascript
var scene = new XEO.Scene();

var material = new XEO.PhongMaterial(scene, {
    diffuse: [ 0.6, 0.6, 0.7 ]
});

var geometry = new XEO.TorusGeometry(scene);

var entity = new XEO.Entity(scene, {
    material: material,
    geometry: geometry
});
````
<img src="http://xeoengine.org/assets/images/conceptScene.png">

### Defaults

xeoEngine provides defaults for pretty much everything, which means that we only need to create things wherever we need 
to override those defaults. For our [Entity](http://xeoengine.org/docs/classes/Entity.html), we provided our 
own [PhongMaterial](http://xeoengine.org/docs/classes/PhongMaterial.html) and [Geometry](http://xeoengine.org/docs/classes/Geometry.html) components, leaving
the Entity to fall back on the [Scene](http://xeoengine.org/docs/classes/Scene.html)'s default flyweight instances for all 
the other components it needs (eg. [Camera](http://xeoengine.org/docs/classes/Camera.html), 
[Lights](http://xeoengine.org/docs/classes/Lights.html) etc).

## Animating the Scene

Animate [Scenes]() by updating properties on their components. Almost everything in xeoEngine fires change events that you can subscribe to, which is quite handy for scripting.

````javascript
material.on("diffuse", function(value) {
    console.log("Material diffuse is now: " + value);
});

// This is going to fire our change listener above
material.diffuse = [0.9, 0.9, 0.6];
````
Likewise, you can update properties on any of the Scene's default flyweight components, such as the 
default [Camera](http://xeoengine.org/docs/classes/Camera.html), which we'll orbit a little bit on each frame:
````javascript
scene.on("tick", function () {
    var view = scene.camera.view;
    view.rotateEyeY(0.6);
    view.rotateEyeX(0.3);
});
````

## Editing the Scene

You can edit everything within your [Scene]() dynamically, at runtime. Create and destroy components, link or unlink
them to each other, update their properties, and so on. Let's add a diffuse 
[Texture](http://xeoengine.org/docs/classes/Texture.html) map to our [PhongMaterial](http://xeoengine.org/docs/classes/PhongMaterial.html),
which will immediately appear on our torus:

````javascript
material.diffuseMap = new XEO.Texture(scene, {
    src: "textures/uvGrid2.jpg"
});
````
