# Importing .3DS

A **Nintendo3DSGeometry** is a {{#crossLink "Geometry"}}{{/crossLink}} that's loaded from a
     <a href="https://en.wikipedia.org/wiki/Nintendo_3DS" target = "_other">Nintendo 3DS</a> file.

## Examples

## Usage

````javascript
var entity = new XEO.Entity({

    geometry: new XEO.Nintendo3DSGeometry({
        src: "models/3ds/lexus.3ds"
    }),

    material: new XEO.PhongMaterial({
        diffuseMap: new XEO.Texture({
            src: "models/3ds/lexus.jpg"
        }),
        specular: [0, 0, 0]
    }),

    // We need to rotate this particular .3DS model
    transform: new XEO.Rotate({
        xyz: [1,0,0],
        angle: -90,
        parent: new XEO.Rotate({
            xyz: [0,1,0],
            angle: 90
        })
    })
});

// When the Nintendo3DSGeometry has loaded,
// fly the camera to fit the entity in view

var cameraFlight = new XEO.CameraFlight();

entity.geometry.on("loaded", function () {
    cameraFlight.flyTo({
        aabb: entity.worldBoundary.aabb
    });
});
````