# Geometry

A xeoEngine [Geometry](http://xeoengine.org/docs/classes/Geometry.html) component defines the shape of attached [Entites](http://xeoengine.org/docs/classes/Entity.html).

Like everything in xeoEngine, all properties on a Geometry are dynamically editable. When no shape is specified, a Geometry will be a 2x2x2 box by default. A [Scene](http://xeoengine.org/docs/classes/Scene.html) provides a 2x2x2 box for [Entites](http://xeoengine.org/docs/classes/Entity.html) default to when they are not configured with a Geometry. A Geometry provides its local-space boundary as a [Boundary3D](http://xeoengine.org/docs/classes/Boundary3D.html).

<img src="http://xeoengine.org/assets/images/Geometry.png"></img>

### Default Geometry Shape

 If you create a Geometry with no specified shape, it will default to a box-shaped triangle mesh with dimensions 2x2x2:

```` javascript 
var entity = new XEO.Entity({ 
    geometry: new XEO.Geometry() // 2x2x2 box
}); 
````

### Scene's Default Geometry

 If you create an [Entity](http://xeoengine.org/docs/classes/Entity.html) with no Geometry, it will inherit its [Scene](http://xeoengine.org/docs/classes/Scene.html)'s default {{#crossLink "Scene/geometry:property"}}{{/crossLink}}, which is a 2x2x2 triangle mesh box:

```` javascript 
var entity2 = new XEO.Entity(); 
````

### Sharing Geometries among Entities

 xeoEngine components can be shared among multiple [Entites](http://xeoengine.org/docs/classes/Entity.html). For components like Geometry and [Scene](http://xeoengine.org/docs/classes/Texture.html), this can provide significant memory and performance savings. To render the example below, xeoEngine will issue two draw WebGL calls, one for each [Scene](http://xeoengine.org/docs/classes/Entity.html), but will only need to bind the Geometry's arrays once on WebGL.

```` javascript  
var boxGeometry = new XEO.BoxGeometry();

new XEO.Entity({ 
    geometry: boxGeometry 
});

new XEO.Entity({ 
    geometry: boxGeometry, 
    transform: new XEO.Translate({ 
        xyz: [5, 0, 0 ]
    }) 
}); 
````

### Creating a custom Geometry

Let's create an [Entity](http://xeoengine.org/docs/classes/Entity.html) with a custom Geometry that's a quad-shaped triangle mesh:

 ```` javascript  
var quadGeometry = new XEO.Geometry({

    // Supported primitives are 'points', 'lines', 'line-loop', 
    // 'line-strip', 'triangles', 'triangle-strip' and 
    // 'triangle-fan'.
    primitive: "triangles",

    // Vertex positions  
    positions : [ 
        -1.0, -1.0, 1.0, // 0 
         1.0, -1.0, 1.0, // 1 
         1.0, 1.0, 1.0, // 2 
        -1.0, 1.0, 1.0 // 3 
    ],

    // Vertex colors  
    colors: [ 
         1.0, 1.0, 1.0, 1.0, // 0 
         1.0, 0.0, 0.0, 1.0, // 1 
         0.0, 1.0, 0.0, 1.0, // 2 
         0.0, 0.0, 1.0, 1.0 // 3 
    ],

    // Vertex normals  
    normals: [ 
         0, 0, 1, // 0 
         0, 0, 1, // 1 
         0, 0, 1, // 2 
         0, 0, 1 // 3 
    ],

    // UV coordinates  
    uv: [ 
          0, 0, // 0 
          1, 0, // 1 
          1, 1, // 2 
          1, 0  // 3 
    ],

     // Triangle indices  
     indices: [ 0, 1, 2, 0, 2, 3 ]
 });

 var quadEntity = new XEO.Entity({ 
     geometry: quadGeometry 
 });  
````

### Editing Geometry

 Recall that everything in xeoEngine is dynamically editable. Let's update the [indices](http://xeoengine.org/docs/classes/Geometry.html#property_indices) to reverse the direction of the triangles:

 ````javascript  
customGeometry.indices = [ 2, 1, 0, 3, 2, 0 ];  
````

 Now let's make it wireframe by changing its primitive type from ````triangles```` to ````lines````:

 ````javascript  
 quadGeometry.primitive = "lines";  
 ````

### Toggling back-faces on and off

 Now we'll attach a [Modes](http://xeoengine.org/docs/classes/Modes.html) to that last [Entity](http://xeoengine.org/docs/classes/Entity.html), so that we can show or hide its [Geometry's](http://xeoengine.org/docs/classes/Geometry.html) back-faces:

 ```` javascript  
 var modes = new XEO.Modes();

 quadEntity.modes = modes;

 // Hide backfaces
 modes.backfaces = false;  
 ````

### Setting front-face vertex winding

 The <a href="https://www.opengl.org/wiki/Face_Culling" target="other">vertex winding order</a> of each face determines whether it's a front-face or a back-face. By default, xeoEngine considers faces to be front-faces if they have a counter-clockwise winding order, but we can change that by setting the [Modes](http://xeoengine.org/docs/classes/Modes.html) [frontFaces](http://xeoengine.org/docs/classes/Modes.html#property_frontface) property:

 ````javascript  
 // Set the winding order for front-faces to clockwise  
 // Options are "ccw" for counter-clockwise or "cw" for clockwise
 modes.frontface = "cw";  
 ````

### Getting Geometry Boundaries

````javascript  
var localBoundary = quadGeometry.localBoundary;

localBoundary.on("updated", function() { 
    obb = localBoundary.obb;  
    aabb = localBoundary.aabb;  
    center = localBoundary.center;

    //...  
}); 
 ````

### Examples 
<ul> <li>[Simple triangle mesh](../../examples/#geometry_triangles)</li> <li>[Triangle mesh with diffuse texture](../../examples/#geometry_triangles_texture)</li> <li>[Triangle mesh with vertex colors](../../examples/#geometry_triangles_vertexColors)</li> <li>[Wireframe box](../../examples/#geometry_lines)</li> <li>[Dynamically modifying a TorusGeometry](../../examples/#geometry_modifying)</li> </ul>
