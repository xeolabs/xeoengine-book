# Importing .3DS

A **Nintendo3DSGeometry** is a Geometry that's loaded from a
     <a href="https://en.wikipedia.org/wiki/Nintendo_3DS" target = "_other">Nintendo 3DS</a> file.

## Examples

## Usage

````javascript
var entity = new xeogl.Entity({

    geometry: new xeogl.Nintendo3DSGeometry({
        src: "models/3ds/lexus.3ds"
    }),

    material: new xeogl.PhongMaterial({
        diffuseMap: new xeogl.Texture({
            src: "models/3ds/lexus.jpg"
        }),
        specular: [0, 0, 0]
    }),

    // We need to rotate this particular .3DS model
    transform: new xeogl.Rotate({
        xyz: [1,0,0],
        angle: -90,
        parent: new xeogl.Rotate({
            xyz: [0,1,0],
            angle: 90
        })
    })
});

// When the Nintendo3DSGeometry has loaded,
// fly the camera to fit the entity in view

var cameraFlight = new xeogl.CameraFlight();

entity.geometry.on("loaded", function () {
    cameraFlight.flyTo({
        aabb: entity.worldBoundary.aabb
    });
});
````