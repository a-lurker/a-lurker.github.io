# openLuup startup code
The openLuup startup code is used to setup a variety of information that may be required by different programs.

## Defaults
These are always in the startup code and are pretty self explanatory.

```lua
-- You can personalise the installation by changing these attributes,
-- which are persistent and may be removed from the Startup after a reload.
local attr = luup.attr_set

-- Geographical location
attr ("City_description", "Greenwich")
attr ("Country_description", "England")
attr ("Region_description", "Europe")
attr ("latitude", "51.4934")
attr ("longitude", "0.0098")

-- other parameters
attr ("TemperatureFormat", "C")
attr ("PK_AccessPoint", "88800000")  -- required by Vera only
attr ("currency", "£")
attr ("date_format", "dd/mm/yy")
attr ("model", "Apple MAC")
attr ("timeFormat", "24hr")

luup.log "startup code completed"

-- Any other startup processing may be inserted here...
```

## Change the log file defaults
Tailor the log file.

```lua
-- path to log file: default is LuaUPnP.log
luup.attr_set ("openLuup.Logfile.Name", "logs/openLuup.log")

-- lines per page: default is 2002
luup.attr_set ("openLuup.Logfile.Lines", 1000)

-- number of pages retained: default is 5
luup.attr_set ("openLuup.Logfile.Versions", 3)
```

## User data update period
The user_data.json file holds the openLuup configuration and is updated regularly in order to retain your changes. It's also saved on openLuup shutdown.
```lua
-- checkpoint every 60 minutes
luup.attr_set ("openLuup.UserData.Checkpoint", 60)
```

## Consolidate scene code
In Vera, each scene contains code to run, when the scene is triggered. With lots of scenes this become unmanageable as you end up with heaps of little code snippets hidden in the various scenes.

SCENE_Prolog and SCENE_Epilog are functions that you put in your start up code or just have these functions call a preloaded of library of you own.

The scene id is passed, in each case. Just use the scene id to switch to the code for that scene.

The final result is all you scene code in one place.

```lua
luup.attr_set ("openLuup.Scenes.Prolog", "SCENE_Prolog")
luup.attr_set ("openLuup.Scenes.Epilog", "SCENE_Epilog")
```

## Email server port
Set the server port.

```lua
luup.attr_set ("openLuup.SMTP.Port", 1234) -- use port 1234 instead
```

## Data historian
The act of inserting this line into the startup code, enables the archiving process, ie saving the data to disk. The data can then be utilised by Grafana and influx.

```lua
-- it's possible to place the data at any other favourite
-- spot, noting the path is relative to cmh-ludl/
luup.attr_set ("openLuup.Historian.Directory","history/")
```

## Using Graphite or Influx databases?
Set the server addresses.
```lua
luup.attr_set ("openLuup.Historian.Graphite_UDP", "127.0.0.1:2003")
luup.attr_set ("openLuup.Databases.Influx", "172.16.42.129:8089")
```

## openLuup shutdown code
Need to let some other device openLuup is shutting down. Write some code to send it a message and place it here.

```lua
-- add some special shutdown code if needed
luup.attr_set ("ShutdownCode", "-- Lua code goes here")
```

## MQTT server credentials
Login credentials default to none but you can add some here.

```lua
luup.attr_set ("openLuup.MQTT.Username", "---username---")
luup.attr_set ("openLuup.MQTT.Password", "---password---")
```

## MQTT server port
The MQTT server port defaults to 1883 but it can be altered here.

```lua
luup.attr_set ("openLuup.MQTT.Port", 1883)    -- choose any free port, you might not want to use this MQTT default
```

## openLuup MQTT publishing
openLuup publishes all of its variables and device status. The latter as a json block. The publishing can be turned on and off with these attributes. They both default to false.

```lua
luup.attr_set ("openLuup.MQTT.PublishVariableUpdates", true) -- publish every variable update
luup.attr_set ("openLuup.MQTT.PublishDeviceStatus", 2)       -- publish a single device status every N seconds (0 = never)
```

## UDP ➔ MQTT bridge
The bridge is always running and you can set the listening port here.

```lua
luup.attr_set ("openLuup.MQTT.Bridge_UDP", 2883) -- UDP bridge PORT number
```
## Tasmota MQTT
The prefix and topics for the Tasmota devices can be set. The default values are as shown below.

```lua
luup.attr_set ("openLuup.Tasmota.Prefix", "tele, tasmota/tele, stat")
luup.attr_set ("openLuup.Tasmota.Topic", "SENSOR, STATE, RESULT, LWT")
```

## MQTT debug
Turn the debugging on.

```lua
luup.attr_set ("openLuup.MQTT.DEBUG", true)
```

## openLuup remote access
While accessing openLuup via wireguard or similar, over world wide distances: You may get timeouts. Fine tune this value to allow the internet to keep up. Time is in fractions of a second.

```lua
luup.attr_set ("openLuup.HTTP.SelectWait", 0.5)
```

## Your own start up code
You can load your own code like so.

```lua
-- Any other startup processing may be inserted here...
myLua = require("L_MyLuaCodeOpenLuup")
```

