# Loading glTF 

Load a glTF file by creating a [Model](http://xeoengine.org/docs/classes/Model.html) component:

````javascript
var gearbox = new XEO.Model({
   id: "gearbox",
   src: "models/gltf/gearbox/gearbox_assy.gltf"
});
````

We've created this particular Model within xeoEngine's default [Scene](http://xeoengine.org/docs/classes/Scene.html), 
since we didn't specify a Scene to its constructor. Internally, xeoEngine has lazy-created the default Scene at this point, 
along with its HTML canvas, which fills the available space in the page. We use the default Scene in most examples in 
order to keep the code simple. 
  
A Model prefixes its own ID to those of its components. Like all components, the Model's ID is optional, and defaults to 
the value of the Model's ````src```` property. Therefore, providing our own short Model ID keeps those component IDs 
short and easy to use.
  
## Subscribing to load completion 
The Model begins loading the glTF file immediately. To be notified when the Model has loaded, bind a callback (which 
fires immediately if the Model happens to be already loaded):
  
````javascript
gearbox.on("loaded", function() {
        // Model has loaded!
    });
````

## Switching to a different glTF file
To switch a Model to a different glTF file, simply update its ````src```` property:

````javascript
gearbox.src = "models/gltf/buggy/buggy.gltf"
````
Recall that almost everything in xeoEngine is dynamically editable. 











