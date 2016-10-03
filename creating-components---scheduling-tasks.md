# Creating Components - Scheduling Tasks



```javascript
// Define the ResizableTorus component class

XEO.ResizableTorus = XEO.Component.extend({

    type: "XEO.ResizableTorus",

    _init: function (cfg) {

        this._torus = new XEO.Entity({
            geometry: new XEO.Geometry(),
            material: new XEO.PhongMaterial({
                diffuse: [0.5, 0.5, 1.0]
            })
        });

        this.radius = cfg.radius
    },

    // This executes the task that this ResizableTorus schedules to the xeoEngine task queue,
    // in this case to (re)build its torus-shaped geometry. Don't worry about the contents
    // of this function. It's just something computationally intensive that needs to be scheduled
    // to execute asynchronously via the task queue so that it doesn't disrupt the smoothness of
    // xeoEngine's "game loop".

    _update: function () {

        // Build torus geometry from _radius

        var tube = this._radius / 2;
        var radialSegments = 20;
        var tubeSegments = 20;
        var arc = Math.PI * 2;
        var positions = [];
        var normals = [];
        var uvs = [];
        var indices = [];
        var u, v, centerX, centerY, centerZ = 0, x, y, z, vec, i, j;
        for (j = 0; j <= radialSegments; j++) {
            for (i = 0; i <= tubeSegments; i++) {
                u = i / tubeSegments * arc; v = j / radialSegments * Math.PI * 2;
                centerX = this._radius * Math.cos(u); centerY = this._radius * Math.sin(u);
                x = (this._radius + tube * Math.cos(v) ) * Math.cos(u);
                y = (this._radius + tube * Math.cos(v) ) * Math.sin(u);
                z = tube * Math.sin(v);
                positions.push(x); positions.push(y); positions.push(z);
                uvs.push(1 - (i / tubeSegments)); uvs.push(1 - (j / radialSegments));
                vec = XEO.math.normalizeVec3(XEO.math.subVec3([x, y, z], [centerX, centerY, centerZ], []), []);
                normals.push(vec[0]); normals.push(vec[1]); normals.push(vec[2]);
            }
        }
        var a, b, c, d;
        for (j = 1; j <= radialSegments; j++) {
            for (i = 1; i <= tubeSegments; i++) {
                a = (tubeSegments+1)*j+i-1;
                b = (tubeSegments+1)*(j-1)+i-1;
                c = (tubeSegments+1)*(j-1)+i;
                d = (tubeSegments+1)*j+i;
                indices.push(a); indices.push(b); indices.push(c);
                indices.push(c); indices.push(d); indices.push(a);
            }
        }

        this._torus.geometry.positions = positions;
        this._torus.geometry.normals = normals;
        this._torus.geometry.uv = uvs;
        this._torus.geometry.indices = indices;
    },

    _props: {

        // The radius of this ResizableTorus
        radius: {
            set: function (radius) {
                this._radius = radius || 1;

                // Schedule our _update() method to the xeoEngine task queue,
                // to execute at the next available opportunity
                this._scheduleUpdate();
            },

            get: function () {
                return this._radius;
            }
        }
    }
});

// Instantiate a ResizableTorus (in the default XEO.Scene)
var resizableTorus = new XEO.ResizableTorus({
    radius: 1
});

// Periodically increase the radius of the ResizableTorus
setInterval(function () {
    resizableTorus.radius += 0.2;
}, 2000);
```

