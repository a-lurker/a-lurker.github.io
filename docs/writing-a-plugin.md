# Quick start
First thing to note is that Vera uses it own subset of the variously seen xml tags. Many tags are just plain ignored and hence don't work as expected.

Additionally the json code used to describe the UI layouts is hard to navigate and it can be tricky to get the outcome one may like. In short; it's confusing and awkward.

Coding errors generally result from non matching xml tags between the various xml files, when it's expected that they do match. Ditto for the json file.

A plugin typically consists of five files (but can be three, four or six) with a hierarchy roughly as follows:

|File name|Prefix|Note|
|---|---|---|
|D_SimplePlugin1.xml|D_ = Description file|`required`|
|D_SimplePlugin1.json|D_ = Description file for UI|optional but generally in use|
|J_SimplePlugin1.js|J_ = Javascript file for UI|optional but not seen overly often|
|S_SimplePlugin1.xml|S_ = Service file|`required` but unorthodox methods can see it being left out|
|I_SimplePlugin1.xml|I_ = Implementation file|`required`. Some setups include the Lua code in the implementation file. Don't do this - it's the road to hell!|
|L_SimplePlugin1.lua|L_ = Lua file|`required`|

The file names are prefixed by a capital letter (as listed in the table above) followed by an underscore. By convention a "1" is placed after the plugin name and before the period. Words begin with uppercase and no separators are used.

The files are placed in the `../cmh-ludl/` directory.

### D_SimplePlugin1.xml
The `Description xml` file contains information about the plugin device itself.
- Plugin name
- a reference to the Description json file for the UI
- a reference to the Service file
- a reference to the Implementation file
- etc

### D_SimplePlugin1.json
The `Description json` file contains the layout information for the device in the user interface. This file is optional but is typically in place.

### J_SimplePlugin1.js
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
The `Lua` file contains the actual Lua code that makes the plugin function.

# xml tags: D_SimplePlugin1.xml

|XML tag|Description|Note|
|---|---|---|
|device_type|urn:schemas-dali-org:device:DaliPlanet:1|Must be of the form shown here.
|json_file|Name of the json file used to create the UI eg D_SimplePlugin1.json|Only required if a UI is needed for the device|
|friendly_name|A string that appears in various spots|Can be whatever - not critical|
|manufacturer|A string that appears in various spots|Can be whatever - not critical|
|modelName|A string that appears in various spots|Can be whatever - not critical|
|category_num|A number: What category is the device in? eg 11 is "General I/O"|Inconsistently implemented|
|subcategory_num|See category_num|Optional and inconsistently implemented|
|handle_children|If this device has multiple children then add the handle_children tag and set it to "1", else leave out||
|service_list|Essential: references the service file|See also tags `service`, `serviceType`, `serviceId` & `SCPDURL`|
|impl_file|Name of the implementation file eg I_SimplePlugin1.xml|Essential|
|local_udn|Unique identifier|Optional (if even used?)|
|modelNumber|String|Optional (if even used?)|

# xml tags: S_SimplePlugin1.xml

|XML tag|Description|Note|
|---|---|---|
|serviceStateTable|Table of state variable descriptions|See stateVariable tag below|
|stateVariable|Description of a variable used by the plugin|Set `sendEvents="no"` or leave out. Events don't work. See also tags `stateVariable`, `name`, `dataType`|
|actionList|List of actions used by the plugin|See action tag below|
|action|Description of an action used by the plugin|See also tags `name`, `argumentList`|
|argument|Description of the action's arguments|See also tags `name`, `relatedStateVariable`|

# xml tags: I_SimplePlugin1.xml

|XML tag|Description|Note|
|---|---|---|
|files|Reference to the Lua code to run|eg L_SimplePlugin1.lua|
|settings|List of settings. Deprecated|eg see protocol tag below|
|protocol|Various serial protocols|Deprecated: use the Lua socket library instead|
|startup|The name of the function that initialises the plugin in the Lua code|eg `luaStartUp`|
|actionList|List of actions used by the plugin|See action tag below|
|action|Description of an action used by the plugin|See also tags `serviceId`, `name`, `run`|

# Example D_SimplePlugin1.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<root xmlns="urn:schemas-upnp-org:device-1-0">
    <specVersion>
      <major>1</major>
      <minor>0</minor>
    </specVersion>

    <device>
        <deviceType>urn:schemas-dali-org:device:DaliPlanet:1</deviceType>
        <friendlyName>DALI Planet</friendlyName>
        
Note the reference here to the D_SimplePlugin1.json file:

        <staticJson>D_SimplePlugin1.json</staticJson>
        <modelDescription>LED light controller</modelDescription>
        <modelName>Super LED</modelName>
        <modelNumber>1.0</modelNumber>
        <Category_Num>11</Category_Num>
        <handleChildren>1</handleChildren>

        
Note the reference here to the S_SimplePlugin1.xml file:

        <serviceList>
            <service>
                <serviceType>urn:schemas-dali-org:service:Dali:1</serviceType>
                <serviceId>urn:dali-org:serviceId:Dali1</serviceId>
                <SCPDURL>S_SimplePlugin1.xml</SCPDURL>
            </service>
        </serviceList>

        
Note the reference here to the I_SimplePlugin1.xml file:

        <implementationList>
            <implementationFile>I_SimplePlugin1.xml</implementationFile>
        </implementationList>

    </device>
</root>
```

# Example D_SimplePlugin1.json
Describe the UI layout for the device. Some sections left out here for clarity. Here is a label and a button for the UI:

When the button is pushed an action URL is sent from the browser to the Luup engine. Note the service ID and the action parameters in the URL:
```http
http://openLuupIPaddress?id=action&output_format=json&DeviceNum=15&serviceId=urn:dali-org:serviceId:Dali1&action=FadeUpDown&addressFadeUpDown
```

```json
Make an entry box - start with a label stating what it is and how to use it.

            "Control": [
                {
                    "ControlType": "label",
                    "Label": {
                        "lang_tag": "Enter_DALI_device_address",
                        "text": "Enter DALI device address --->"
                    },
                    "Display": {
                        "Top": 24,
                        "Left": 20,
                        "Width": 150,
                        "Height": 20
                    }
                },

Here's the actual entry box - the entered value "ID" will be called "ourAddress". You call it anything or even just number it. The value is just used to reference the value later on.

                {
                    "ControlType": "input",
                    "ID": "ourAddress",
                    "Display": {
                        "Top": 20,
                        "Left": 200,
                        "Width": 150,
                        "Height": 20
                    }
                },

Here is a button that will send a fade up/down command to DALI address entered in the entry box above  Note the use of "ID":

                {
                    "ControlType": "button",
                    "Label": {
                        "lang_tag": "Fade_Up_Down",
                        "text": "Fade Up Down"
                    },
                    "Display": {
                        "Top": 80,
                        "Left": 20,
                        "Width": 150,
                        "Height": 20
                    },
                    "Command": {
                        "Service": "urn:dali-org:serviceId:Dali1",
                        "Action": "FadeUpDown",
                        "Parameters": [
                            {
                                "Name": "addressFadeUpDown",
                                "ID": "ourAddress"
                            }
                        ]
                    }
                }]
```

# Example S_SimplePlugin1.xml

Note that the `dataType` tag may typically contain `string`, `number`, `float`, `boolean`, `ui4`, etc. However the Vera Luup engine treats them all as `string`, so what's entered is irrelevant. Just use:
```xml
<dataType>string</dataType>
```
You may see these tags as part of the `stateVariable` declaration. They are ignored, so leave out:
```xml
<sendEventsAttribute>
<shortCode>
<direction>
<defaultValue>
<allowedValueList>
<allowedValue>
```

Regarding `<stateVariable sendEvents="no">`: `sendEvents="no"` is for UPnP and is not used by LuaUPnP. Leave it out and just use `<stateVariable>`.

Likewise the `specVersion` tag and its contents is often seen but not used.

The state variables are declared in the S_*1.xml. State variable names can be anything you like but must match in the actions. Actions are then described after the state variables and match the previously declared state variables.
```xml
<?xml version="1.0"?>
<scpd xmlns="urn:schemas-upnp-org:service-1-0">
    <specVersion>
        <major>1</major>
        <minor>0</minor>
    </specVersion>

    <serviceStateTable>
        <stateVariable>
        <name>A_ARG_TYPE_AddressFadeUpDown</name>
        <dataType>string</dataType>
        </stateVariable>
    </serviceStateTable>

    <actionList>
        <action>
            <name>FadeUpDown</name>
            <argumentList>
                <argument>
                    <name>AddressFadeUpDown</name>
                    <relatedStateVariable>A_ARG_TYPE_AddressFadeUpDown</relatedStateVariable>
                </argument>
            </argumentList>
        </action>
    </actionList>
</scpd>
```

# Example I_SimplePlugin1.xml

Actions are implemented as described in the I_*1.xml files
```xml
<?xml version="1.0"?>
<implementation>
    <specVersion>
        <major>1</major>
        <minor>0</minor>
    </specVersion>
        
Note the reference here to the L_SimplePlugin1.lua file:

    <files>L_SimplePlugin1.lua</files>

Note the reference here to the luaStartUp function seen in L_SimplePlugin1.lua file:

    <startup>luaStartUp</startup>

    <actionList>
        <action>
        <serviceId>urn:dali-org:serviceId:Dali1</serviceId>
        <name>FadeUpDown</name>

Note the reference here to the fadeUpDown function seen in L_SimplePlugin1.lua file:

        <run>fadeUpDown(lul_settings.addressFadeUpDown)</run>
        </action>

    </actionList>
</implementation>
```

# Example L_SimplePlugin1.lua

```lua
-- LED light controller. Originated by a-lurker, 15 Jan 2025, Version 0.51. 

local m_fadeUp = nil

-- The Luup engine implements the action as coded ihere in this L_*1.lua file. Function must be global not local.
function fadeUpDown(DALIaddress)
    debug('fading up/down')
    DALIaddress = tonumber(DALIaddress)

    if (m_fadeUp) then
        local ok1 = executeDALIcommand('DirectArcCommand', DALIaddress, DALI_MAX_LEVEL)
    else -- fade down
        local ok2 = executeDALIcommand('DirectArcCommand', DALIaddress, 0)
    end
    m_fadeUp = not m_fadeUp
end

-- The Luup engine implements the startup as coded here in this L_*1.lua file. 
function luaStartUp(lul_device)
   -- do the start up
    m_fadeUp = true
end
```

## Deep dive
@toggledbits has written [some info on plugins here](https://github.com/toggledbits/PluginTools).

## Testing code
The console has three code test boxes at:
Home -> lua_test, lua_test2, lua_test3

Large slabs of code can be tested here and it's even possible, but perhaps not particularly desirable, to develop plugins using these test boxes.

You can pretty print variables as needed eg:

```lua
local times = luup.openLuup.cpu_table()
print (pretty(times))

times = luup.openLuup.wall_table()
print (pretty(times))

local test = {33,44,55}
print (luup.openLuup.pretty(test))
```

The above makes use of:
```lua
luup.openLuup.pretty()
```
which is accessible throughout the openLuup code base.

