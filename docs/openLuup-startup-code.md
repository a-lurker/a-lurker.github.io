# openLuup startup code
The openLuup startup code is used to setup a variety of information that may be required by different programs.

## Defaults
These are always in the startup code and are pretty much self explanatory.

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

-- Any other startup processing may be inserted here...

luup.log "startup code completed"
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

## Suppress logging for luup.io.read in raw mode
Legacy plugins may use luup.io.read in "raw" mode. Every received byte results in a log file entry. You can suppress the logging using:
```lua
luup.attr_set ("openLuup.Logfile.Incoming", "false")
```
luup.io.xyz is deprecated.

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
Set the server port. The server port defaults to 2525 but it can be changed in the start up code:

```lua
luup.attr_set ("openLuup.SMTP.Port", 587) -- use port 587 instead of the 2525 default
```

More details about the SMPT server and using the mailboxes can be found [here](/openluup?id=openluup-and-email).

## Data historian
The act of inserting this line into the startup code, enables the archiving process, ie saving the data to disk. The data can then be utilised by Grafana and influx.

```lua
-- it's possible to place the data at any other favourite
-- spot, noting the path is relative to cmh-ludl/
-- this line will start the Historian
luup.attr_set ("openLuup.Historian.Directory","history/")
```

The Data historian automatically retains data for most numeric variables being used by openLuup. It decides what variables will be retained using a rule based system. The rules follow the [Grafana example](https://graphite-api.readthedocs.io/en/latest/api.html#graphing-metrics). The rules currently in force can be viewed in the openLuup Console at:

openLuup_IP_address:3480/console?page=rules

There may be occasions where a variable you want saved is not automatically saved, as it doesn't match any of the built in rules. In the start up code, you can add a new rule to force a match to suit eg:

```lua
do -- bespoke Historian archive rules
  local rules = require "openLuup.servertables" .archive_rules

  -- enable archiving of the selected numeric device variables
  rules[#rules+1] = {
      patterns = {"*.EKMmetering1.{ResettablekWh}"},
      retentions = "5m:28d"
    }

  rules[#rules+1] = {
      patterns = {"*.SMA_inverter1.{kWhToday}"},
      retentions = "5m:28d"
    }
end
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
The MQTT server port defaults to 1883 but it can be altered here. This line of code is optional.

```lua
-- choose any free port, you might not want to use this MQTT default
luup.attr_set ("openLuup.MQTT.Port", 1883)
```

## openLuup MQTT publishing
openLuup publishes all of its variables and device status. The latter as a json block. The publishing can be turned on and off with these attributes. They both default to false.

```lua
-- publish every variable update
luup.attr_set ("openLuup.MQTT.PublishVariableUpdates", true)

-- publish a single device status every N seconds (0 = never)
luup.attr_set ("openLuup.MQTT.PublishDeviceStatus", 2)
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
-- default 0.1 second
luup.attr_set ("openLuup.HTTP.SelectWait", 0.5)
```

You can also try halving or doubling the chunk length eg:
```lua
-- default 16000 bytes
luup.attr_set ("openLuup.HTTP.ChunkedLength", 8000)
```

## Add useful global variable or function definitions
You can add useful global variable or function definitions to your startup code. They then becomes available for use, within your Lua test code or Lua scene code.

For example: by including this in the startup:
```lua
json = require "openLuup.json"
```
It's then ready for use in the Lua test boxes or in your scene code:
```lua
local x = json.decode (y)
```

## Your own start up code
You can load your own code like so.
```lua
-- Any other startup processing may be inserted here...
myLua = require("L_MyLuaCodeOpenLuup")
```
L_MyLuaCodeOpenLuup.lua would look something like this:
```lua
local function f1()
    your_code
end

local function f2()
    more_code
end

return {
    f1 = f1,
    f2 = f2
}
```
Scene code would call the functions like so:
```lua
    myLua.f1()
```

