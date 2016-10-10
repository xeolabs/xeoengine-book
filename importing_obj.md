# Importing OBJ

An ObjGeometry is a Geometry that's loaded from a
<a href="https://en.wikipedia.org/wiki/Wavefront_.obj_file" target = "_other">Wavefront .OBJ</a> file.

## Examples

## Usage

````javascript
var entity = new xeogl.Entity({

    geometry: new xeogl.OBJGeometry({
        src: "models/obj/raptor.obj"
    }),

    material: new xeogl.PhongMaterial({
        diffuseMap: new xeogl.Texture({
            src: "models/obj/raptor.jpg"
        })
    }),

    transform: new xeogl.Rotate({
        xyz: [1, 0, 0],
        angle: 0,

        parent: new xeogl.Translate({
            xyz: [10, 3, 10]
        })
    })
});

// When the OBJGeometry has loaded, fly the camera to fit the entity in view

var cameraFlight = new xeogl.CameraFlight();

entity.geometry.on("loaded", function () {

        cameraFlight.flyTo({
            aabb: entity.worldBoundary.aabb
        });
    });
````