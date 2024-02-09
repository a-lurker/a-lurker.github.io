## Quick start
A plugin typically consists of five files with a hierarchy as follows:

|File name|Prefix|
|---|---|
|D_SimplePlugin1.xml|D_ = Description file|
|D_SimplePlugin1.json|D_ = Description file|
|S_SimplePlugin1.xml|S_ = Service file|
|I_SimplePlugin1.xml|I_ = Implementation file|
|L_SimplePlugin1.lua|L_ = Lua file|

The file names are prefixed by a capital letter (as listed in the table above) followed by an underscore. By convention a "1" is placed after the plugin name and before the period.

### D_SimplePlugin1.xml
The `Description xml` file contains information about the plugin device itself.
- Plugin name
- a reference to the Service file
- a reference to the Service file
- a reference to the Implementation file
- etc

### D_SimplePlugin1.json
The `Description json` file contains the layout information for the device in the user interface. This file is optional but is typically in place.

If the default UI functionality provided is not sufficient; you can use your own javascript file: `J_SimplePlugin1.js` to manipulate the UI.

### S_SimplePlugin1.xml
The `Service xml` file contains the information about services the plugin provides. Such as:
- plugin function names and associated parameters

### I_SimplePlugin1.xml
The `Implementation xml` file contains the information about how the services are implemented by the plugin. It provides
- a link to the Lua file
- the function to call to initialise the plugin
- what Lua code to call when an action is called.

### L_SimplePlugin1.lua
The `Lua xml` file contains the actual Lua code that makes the plugin function.

## Deep dive
@toggledbits has written [some info on plugins here](https://github.com/toggledbits/PluginTools).
