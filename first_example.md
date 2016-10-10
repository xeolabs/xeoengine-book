# First Example

In this first example we'll create the spinning torus shown in the screenshot below. To show how scenes are dynamically editable, we'll create it incrementally: first we'll create the basic torus entity, then set it spinning, and then add texture. 
        
<a href="http://xeogl.org/examples/#materials_fresnel_specular"><img src="http://xeogl.org/assets/images/torus.png"></a>
      
## Creating the Scene

First, include [xeogl.min.js](https://github.com/xeolabs/xeogl/tree/master/build) in your HTML page:

````html
<script src="xeogl.min.js"/>
````

Next, create the 3D scene as an entity-component graph, as shown in the diagram below. Note how a
[````Scene````](http://xeogl.org/docs/classes/Scene.html) is basically a container 
of [Components](http://xeogl.org/docs/classes/Component.html) that are tied together 
by [Entities](http://xeogl.org/docs/classes/Entity.html).

````javascript
var scene = new xeogl.Scene();

var material = new xeogl.PhongMaterial(scene, {
    diffuse: [ 0.6, 0.6, 0.7 ]
});

var geometry = new xeogl.TorusGeometry(scene);

var entity = new xeogl.Entity(scene, {
    material: material,
    geometry: geometry
});
````
<img src="http://xeogl.org/assets/images/conceptScene.png">

#### Defaults

xeogl provides defaults for pretty much everything, which means that we only need to create things wherever we need 
to override those defaults. For our [Entity](http://xeogl.org/docs/classes/Entity.html), we provided our 
own [PhongMaterial](http://xeogl.org/docs/classes/PhongMaterial.html) and [Geometry](http://xeogl.org/docs/classes/Geometry.html) components, leaving
the Entity to fall back on the [````Scene````](http://xeogl.org/docs/classes/Scene.html)'s default flyweight instances for all 
the other components it needs (eg. [Camera](http://xeogl.org/docs/classes/Camera.html), 
[Lights](http://xeogl.org/docs/classes/Lights.html) etc).

## Editing the Scene

You can edit **everything** within your [````Scene````]() dynamically, which is awesome for live coding. Create and destroy components, link or unlink them to each other, update their properties, and so on. Let's add a diffuse [Texture](http://xeogl.org/docs/classes/Texture.html) map to our [PhongMaterial](http://xeogl.org/docs/classes/PhongMaterial.html), which will immediately appear on our torus:

````javascript
material.diffuseMap = new xeogl.Texture(scene, {
    src: "textures/uvGrid2.jpg"
});
````

## Animating the Scene

Animate [Scenes]() by updating properties on their components. Almost everything in xeogl fires change events that you can subscribe to, which is quite handy for scripting.

````javascript
material.on("diffuse", function(value) {
    console.log("Material diffuse is now: " + value);
});

// This is going to fire our change listener above
material.diffuse = [0.9, 0.9, 0.6];
````
Likewise, you can update properties on any of the Scene's default flyweight components, such as the 
default [Camera](http://xeogl.org/docs/classes/Camera.html), which we'll orbit a little bit on each frame:
````javascript
scene.on("tick", function () {
    var view = scene.camera.view;
    view.rotateEyeY(0.6);
    view.rotateEyeX(0.3);
});
````
