# usd_schema_bootstrap
Files to simplify USD schema building. 
Few manual things still to do to have a correct schema built with its python binding.

## Create a new schema
Inside an empty directory past the content of this repository to get something like:
```
myNewSchema/
  CMakeLists.txt
  src/
    CMakeLists.txt
    __init__.py
    plugInfo.json
    plugInfo.root.json
    module.cpp
    moduleDeps.cpp
    schema.usda
```

Inside `myNewSchema/CMakeLists.txt` file, edit `USDPLUGIN_NAME` value according to yours (we used `myNewSchema` for this example). If needed change the install directory, by default, install dir will be inside the build dir.
Edit the `myNewSchema/src/schema.usda` to define your new prims and API.
Add or replace existing values inside `myNewSchema/src/module.cpp` for each prim inside your schema add a line like:
```
# TF_WRAP(libraryPrefix + primName)
TF_WRAP(schemaFooBasePrim);
TF_WRAP(schemaFooSpePrim);
TF_WRAP(schemaFooBarAPI);
```
Replace existing values inside `myNewSchema/src/moduleDeps.cpp` by your `libraryName`:
```
TfScriptModuleLoader::GetInstance().RegisterLibrary(TfToken("myNewSchema"), TfToken("myNewSchema"), reqs);
```
Run `usdGenSchema myNewSchema/src/schema.usda myNewSchema/src`, the last argument is the destination directory it must be the same as our other files. 

### Let's build and install the plugin schema! 
Create a build dir inside `myNewSchema/` and execute the following commands, change the value of the variable `USD_BUILD_DIR` according to yours:
```
cd ./build
cmake .. -DUSD_BUILD_DIR="/your/usd/build-21.11" && make install
```
If everything is correct, inside the `install`, `plugin` and `python` dirs should be populated with your various libs and correctly set up.

> **_NOTE:_**  The installation is designed to be used for a unique `plugin` and `python` directory for all your plugins (a design used by USD or Houdini).

### Set environment variables
Three variables need to be set to allow USD execution to load all your plugins. 
```
export PXR_PLUGIN_PATH=$PXR_PLUGIN_PATH:myNewSchema/build/install/plugin
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:myNewSchema/build/install/plugin # set PATH instead for windows
export PYTHONPATH=$PYTHONPATH:myNewSchema/build/install/python
```

### Test your plugin inside python
```python
from pxr import Usd
import myNewSchema

stage = Usd.Stage.CreateNew('/tmp/test.usda')
spe = myNewSchema.SpePrim.Define(stage, '/spefoo')
stage.Save()
```
