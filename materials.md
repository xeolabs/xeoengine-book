# Materials

A [PhongMaterial](http://xeoengine.org/docs/classes/PhongMaterial.html) is a type of [Material](http://xeoengine.org/docs/classes/Material.html) that defines the surface appearance of [Entities](http://xeoengine.org/docs/classes/Entity.html) using the Phong lighting model.

A PhongMaterial has the following shading properties:

* ambient,
* diffuse,
* specular,
* emissive and 
* opacity 

```javascript
 var entity = new XEO.Entity({

    geometry: new XEO.TorusGeometry()

    material: new XEO.PhongMaterial({

        ambient: [0.3, 0.3, 0.3],
        diffuse: [0.5, 0.5, 0.0],   
        specular: [1, 1, 1],
        shininess: 80, 
        opacity: 1.0 
    })
});
```

### Textures

A PhongMaterial applies its shading properties unformly across a surface, but you can replace each of them with a [Texture](http://xeoengine.org/docs/classes/Texture.html) to apply it as a pattern.

In the following example, our PhongMaterial has a `diffuse` attribute, but also has a `diffuseMap` property set to a Texture. That Texture's pixel colors will then directly provide the diffuse color of each fragment across the Geometry surface, overriding the `diffuse` attribute.

```javascript
 var entity = new XEO.Entity({

    geometry: new XEO.TorusGeometry(),

    material: new XEO.PhongMaterial({

        ambient: [0.3, 0.3, 0.3],        
        specular: [1, 1, 1],        
        shininess: 80,         
        opacity: 1.0 

        diffuseMap: new XEO.Texture({
            src: "diffuseMap.jpg"
        })
    })
});

```

### Fresnel Terms

```javascript
 var entity = new XEO.Entity({

     material: new XEO.PhongMaterial({

        ambient: [0.3, 0.3, 0.3],                        
        specular: [1, 1, 1],                
        shininess: 80,                 
        opacity: 1.0 

        diffuseMap: new XEO.Texture({
            src: "diffuseMap.jpg"
        }),

        specularFresnel: new XEO.Fresnel({
            edgeColor: [1.0, 1.0, 1.0],
            centerColor: [0.0, 0.0, 0.0],
            power: 4,
            bias: 0.2
        })
     }),

     new XEO.TorusGeometry()
 });
```

### Geometry attributes

In addition to the surface appearance attributes, a PhongMaterial also defines a couple of attributes for Geometry appearance:

* lineWidth: \* pointSize: 

When the Entity's Geometry has a primitive set to "lines" or "points" then only the PhongMaterial's emissive, emissiveMap, opacity and opacityMap will actually be applied, since those primitive types cannot be shaded.

### Usage

In this example we have an Entity with a [Lights](http://xeoengine.org/docs/classes/Lights.html) that contains an [AmbientLight](http://xeoengine.org/docs/classes/AmbientLight.html) and a [DirLight](http://xeoengine.org/docs/classes/DirLight.html), a PhongMaterial which applies a Texture as a diffuse map and a specular Fresnel, and

a TorusGeometry.

Note that xeoEngine will ignore the PhongMaterial's diffuse property, since we assigned the Texture to the PhongMaterial's diffuseMap property. The Texture's pixel colors directly provide the diffuse color of each fragment across the Geometry surface.

```javascript
 var entity = new XEO.Entity({

    geometry: new XEO.TorusGeometry(),

    material: new XEO.PhongMaterial({
        ambient: [0.3, 0.3, 0.3],
        diffuse: [0.5, 0.5, 0.0],   
        diffuseMap: new XEO.Texture({
            src: "diffuseMap.jpg"
        }),
        specular: [1, 1, 1],
        specularFresnel: new XEO.Fresnel({
            leftColor: [1.0, 1.0, 1.0],
            rightColor: [0.0, 0.0, 0.0],
            power: 4
        }),
        shininess: 80, // Default
        opacity: 1.0 // Default
    })
});
```

