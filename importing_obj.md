# Importing OBJ

An **ObjGeometry** is a {{#crossLink "Geometry"}}{{/crossLink}} that's loaded from a
<a href="https://en.wikipedia.org/wiki/Wavefront_.obj_file" target = "_other">Wavefront .OBJ</a> file.

## Examples

## Usage

````javascript
var entity = new XEO.Entity({

    geometry: new XEO.OBJGeometry({
        src: "models/obj/raptor.obj"
    }),

    material: new XEO.PhongMaterial({
        diffuseMap: new XEO.Texture({
            src: "models/obj/raptor.jpg"
        })
    }),

    transform: new XEO.Rotate({
        xyz: [1, 0, 0],
        angle: 0,

        parent: new XEO.Translate({
            xyz: [10, 3, 10]
        })
    })
});

// When the OBJGeometry has loaded, fly the camera to fit the entity in view

var cameraFlight = new XEO.CameraFlight();

entity.geometry.on("loaded", function () {

        cameraFlight.flyTo({
            aabb: entity.worldBoundary.aabb
        });
    });
````