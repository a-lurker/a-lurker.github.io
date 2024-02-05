# Luup Engine

## Functions

luup.function_name:

- attr_get
- attr_set
- call_action
- call_delay
- call_timer
- create_device
- device_message
- device_supports_service
- devices_by_service
- ip_set
- is_night
- is_ready
- job_watch
- log
- mac_set
- modelID
- register_handler
- reload
- set_failure
- sleep
- sunrise
- sunset
- task
- variable_get
- variable_set
- variable_watch


### attr_get
attr_get (attribute, device)

|Identifier|Type|Comments|
|---|---|---|
|Arguments:|||
|attribute|string|eg "id", "name", etc |
|device|string or integer|String is the device's udn, else it's the device number.|
|.|||
|Returns:|||
|attribute value|string||

Example:
```lua
local macAddress = luup.attr_get ('mac', 23)
```

### attr_set
attr_set (attribute, value, device, refresh_user_data)

|Identifier|Type|Comments|
|---|---|---|
|Arguments:|||
|attribute|string|eg id, name |
|value|string||
|device|string or integer|String is the device's udn, else it's the device number.|
|refresh_user_data|boolean|Increments DataVersion in the user_data.json file|
|.|||
|Returns:|||
|Nil|||

Example:
```lua
luup.attr_set ("mac", "2D:EE:91:87:46:A3", 23, false)
```

### call_action
call_action (service, device, arguments, device)

|Identifier|Type|Comments|
|---|---|---|
|Arguments:|||
|service|string|eg "urn:upnp-org:serviceId:SwitchPower1"|
|action|string|eg "SetTarget"|
|arguments|table|eg \{newTargetValue = level}|
|device|string or integer|String is the device's udn, else it's the device number.|
|.|||
|Returns:|||
|error|integer||
|error_msg|string||
|job|integer||
|arguments|table||

Example:
```lua
luup.call_action("urn:upnp-org:serviceId:SwitchPower1", "SetTarget", {"newLoadlevelTarget" = "1"}, 43)
```

### call_delay
call_delay (function_name, seconds, argument_for_function, thread)

|Identifier|Type|Comments|
|---|---|---|
|Arguments:|||
|function_name|string|eg lightOn|
|seconds|integer|eg 3600 is one hour|
|argument_for_function|string|Multiple arguments need to be serialiased|
|thread|boolean|deprecated - not required|
|.|||
|Returns:|||
|result|integer|0 on success|

Example:
```lua
luup.call_delay ("lightOn", "3600", "colorRed")
```

### call_timer
call_timer (function_name, type, time, days, argument_for_function)

|Identifier|Type|Comments|
|---|---|---|
|Arguments:|||
|function_name|string|eg lightOn|
|type|integer||
|time|string||
|days|string||
|argument_for_function|string|Multiple arguments need to be serialiased|
|.|||
|Returns:|||
|result|integer|0 on success|

|Type value|Timer|
|---|---|
|1|Interval|
|2|Day of week|
|3|Day of month|
|4|Absolute|


Example:
```lua
luup.call_timer ("flashLight", 1, "5m", ,"colorBlue")
```

### create_device
create_device ()
Do we really need to do this?

### device_message
Deprecated.

### device_supports_service
Deprecated.

### devices_by_service
Deprecated.

### ip_set
ip_set (value, device)

|Identifier|Type|Comments|
|---|---|---|
|Arguments:|||
|value|string| eg 192.168.1.1|
|device|string or integer|String is the device's udn, else it's the device number.|
|.|||
|Returns:|||
|Nil|||

Example:
```lua
luup.ip_set ("192.168.1.1", 49)
```

### is_night
is_night ()

|Identifier|Type|Comments|
|---|---|---|
|Arguments:|||
|nil|||
|.|||
|Returns:|||
|night_time|boolean||

Example:
```lua
local night_time = luup.is_night ()
```

### is_ready
Deprecated.

### job_watch
Deprecated.

### log
log ()

|Identifier|Type|Comments|
|---|---|---|
|Arguments:|||
|log_this|string||
|log_level|integer|optional: default is 50|
|.|||
|Returns:|||
|nil|||

Levels:
1. Critical
2. Warning
50. Default

Example:
```lua
luup.log ("Another stuff up occured in function cannot_count")
```

### mac_set
mac_set (value, device)

|Identifier|Type|Comments|
|---|---|---|
|Arguments:|||
|value|string| eg 2D:EE:91:87:46:A3|
|device|string or integer|String is the device's udn, else it's the device number.|
|.|||
|Returns:|||
|Nil|||

Example:
```lua
luup.mac_set ("2D:EE:91:87:46:A3", 49)
```

### modelID
Deprecated. Refers to Vera models.

```lua
print(luup.modelID())
-- Result is "35" for Vera Edge and "0" for openLuup
```

### register_handler

Great function. It's very useful!

register_handler (callback_handler_function_name, request_id)

|Identifier|Type|Comments|
|---|---|---|
|Arguments:|||
|callback_handler_function_name|string|When the handler is hit this function is executed. The function will look at the passed in request and act accordingly.|
|request_id|string|URL?id=thisParameter|
|.|||
|Returns:|||
|Nil|||

The callback handler is passed three variables as follows:

|Identifier|Type|Comments|
|---|---|---|
|lul_request|string|This indicates what needs to be done with the associated URL parameters|
|lul_parameters|table|All the stuff left in the url string after filtering out the request and the "output_format". Can be empty.|
|lul_outputformat|string|output_format=xml, json, gpx, etc. All optional and you decide on the format abbreviation and act accordingly.|

**openLuup enhancement:**
Different proctocols can be recognised and acted on accordingly.

```lua
luup.register_handler ("myHandler", "tcp:1234")                  -- incoming TCP connection on port 1234
luup.register_handler ("myHandler", "udp:1234")                  -- incoming UDP -- " --
luup.register_handler ("myHandler", "mailto:me@openLuup.local")  -- incoming email for me@...
luup.register_handler ("myHandler", "mailto:me@openLuup.local")  -- incoming email for me@...
luup.register_handler ("myHandler", "mqtt:My/Topic/Name")        -- MQTT
```

MQTT: One additional argument:
- message number (integer)

The MQTT callback handler is passed three variables as follows:

|Identifier|Type|Comments|
|---|---|---|
|topic|string|MQTT topic|
|message|table|Topic json payload - needs to be decoded|
|messageNumber|integer|Matches the number in the registration of the handler|

Example:
```lua
local PLUGIN_URL_ID = callmyplugin

-- url = http://192.32.26.10/port_3480/data_request?id=callMyPlugin&fnc=getWebPage1&output_format=notNeededThisTime

function myPluginCallbackHandler (lul_request, lul_parameters, lul_outputformat)

    debug('request is: '..tostring(lul_request))
    for k,v in pairs(lul_parameters) do debug ('parameters are: '..tostring(k)..'='..tostring(v)) end
    debug('outputformat is: '..tostring(lul_outputformat))

    if not (lul_request:lower() == PLUGIN_URL_ID) then return end

    -- set the parameters key and value to lower case
    local lcParameters = {}
    for k, v in pairs(lul_parameters) do lcParameters[k:lower()] = v:lower() end

    -- output the intro web page?
    if not lcParameters.fnc then return htmlIntroPage() end -- no 'fnc' parameter so do the intro

    if (lcParameters.fnc == 'getwebpage1') then return getLog(lcParameters) end
    if (lcParameters.fnc == 'getwebpage2') then return getZWInfo() end
    if (lcParameters.fnc == 'getwebpage3') then return getZWDeviceInfo(lcParameters) end
    if (lcParameters.fnc == 'getwebpage4') then return getIP() end

    return 'Error', 'text/html'
end

    luup.register_handler ('myPluginCallbackHandler', callmyplugin)
```


### reload
reload ()

|Identifier|Type|Comments|
|---|---|---|
|Arguments:|||
|Nil|||
|.|||
|Returns:|||
|Nil|||

Restarts the luup engine.

Example:
```lua
luup.reload ()
```

### set_failure
set_failure ()

|Identifier|Type|Comments|
|---|---|---|
|Arguments:|||
|value|integer|0 is OK, 1 is not OK|
|device|string or integer|String is the device's udn, else it's the device number.|
|.|||
|Returns:|||
|Nil|||

Example:
```lua
-- heart beat function indicates device at far end is off line. We can't continue.
luup.set_failure (1)
```

### sleep
sleep ()

|Identifier|Type|Comments|
|---|---|---|
|Arguments:|||
|value|integer|milliseconds|
|device|string or integer|String is the device's udn, else it's the device number.|
|.|||
|Returns:|||
|Nil|||

Example:
```lua
-- Don't sleep too long. It may cause a luup engine restart!
luup.sleep (25)
```

### sunrise
sunrise ()

|Identifier|Type|Comments|
|---|---|---|
|Arguments:|||
|nil|||
|.|||
|Returns:|||
|next_sunrise|integer|UNIX time stamp|

Example:
```lua
local next_sunrise = luup.sunrise ()
```

### sunset
sunset ()

|Identifier|Type|Comments|
|---|---|---|
|Arguments:|||
|nil|||
|.|||
|Returns:|||
|next_sunrise|integer|UNIX time stamp|

Example:
```lua
local next_sunset = luup.sunset ()
```

### task
task ()

### variable_get
variable_get (service, variable_name, device)

|Identifier|Type|Comments|
|---|---|---|
|Arguments:|||
|service|string|eg "urn:upnp-org:serviceId:SwitchPower1"|
|variable_name|string||
|device|string or integer|String is the device's udn, else it's the device number.|
|.|||
|Returns:|||
|variable_value|string||
|time|integer|UNIX time stamp|

Example:
```lua
local lightStatus, timeStamp = luup.variable_get ("urn:upnp-org:serviceId:SwitchPower1", "Status", 43)
```

**openLuup enhancement:**

Two additional arguments:
- start time as a UNIX timestamp
- start time as a UNIX timestamp

Returns a table, containing two nested tables, with the historical result as times and values. eg

```lua
local v2,t2 = luup.variable_get ("urn:upnp-org:serviceId:TemperatureSensor1","CurrentTemperature", 33, {os.time()-3600, os.time()})
print ("over the last hour", pretty {value = v2, times=t2})

-- over the last hour 	{
--   times = {1530023886,1530023926.5466,1530024526.9677,1530025127.1189,1530026327.7046,1530026929.0275,1530027486},
--   value = {31.1,31,31.2,31.3,31.2,31.3,31.3}
}
```

### variable_set
variable_set (service, variable_name, variable_value, device)

|Identifier|Type|Comments|
|---|---|---|
|Arguments:|||
|service|string|eg "urn:upnp-org:serviceId:SwitchPower1"|
|variable_name|string||
|variable_value|string||
|device|string or integer|String is the device's udn, else it's the device number.|
|startup|boolean|Deprecated|
|.|||
|Returns:|||
|nil|||

Example:
```lua
luup.variable_set ("urn:upnp-org:serviceId:SwitchPower1", "Status", 43)
```

### variable_watch
variable_watch (function_name, service, variable_name, device)

|Identifier|Type|Comments|
|---|---|---|
|Arguments:|||
|function_name|string|The function is called when the variable changes.|
|service|string|eg "urn:upnp-org:serviceId:SwitchPower1"|
|variable_name|string||
|device|string or integer|String is the device's udn, else it's the device number.|
|.|||
|Returns:|||
|nil|||

Example:
```lua
luup.variable_watch ("lightSwitchOperated", "urn:upnp-org:serviceId:SwitchPower1", "Status", 43)
```

## Numbers

### latitude
Used to calculate sunset, sunrise, etc. Can be altered in the start up code.
```lua
print(luup.latitude))
```

### longitude
Used to calculate sunset, sunrise, etc. Can be altered in the start up code.
```lua
print(luup.longitude))
```

### pk_accesspoint
Deprecated. Vera only. Used for cloud connections. Equals 88800000 or 87654321 in openLuup.

### version_branch
See below.

### version_major
See below.

### version_minor
```lua
print(luup.version_branch.."."..luup.version_major.."."..luup.version_minor)

-- shows "1.7.0"; same as print(luup.version)
```
## Strings

### city
Can be altered in the start up code. We know where you live.

### event_server
Deprecated. Vera only. Used for cloud connections.

### event_server_backup
Deprecated. Vera only. Used for cloud connections.

### hw_key
Deprecated. Vera only. Used for cloud connections.

### ra_server
Deprecated. Vera only. Used for cloud connections.

### ra_server_backup
Deprecated. Vera only. Used for cloud connections.

### timezone
UTC offset including DST.

### version
```lua
print(luup.version)
-- shows "1.7.0"
```


## luup.chdev

### append
luup.chdev.append (
device,
ptr,
id,
description,
device_type,
device_file_name,
implementation_file_name,
parameters,
embedded,
invisible
)

Existing children:
- if mentioned - are updated. You can change their info.
- if not mentioned - are deleted.

Any new children are added (the typical scenario).

|Identifier|Type|Comments|
|---|---|---|
|Arguments:|||
|device|string|eg id of this parent|
|ptr|C code reference|eg pointer to all the children located by luup.chdev.start()|
|id|string|The "altid" attribute. You decide what it's called.|
|description|string|A name for the child. You decide what it's called.|
|device_type|string|eg 'urn:schemas-upnp-org:device:BinaryLight:1'|
|device_filename|string|eg 'D_BinaryLight1.xml'|
|implementation_file_name|string| Use empty string, if implementation file is mentioned in the device file|
|parameters|string|eg Set up child variable defaults. Easier to set to empty string and set up vars in child code.|
|embedded|boolean|eg If true children cannot be split between rooms|
|invisible|boolean|optional - makes child invisible in the UI|
|.|||
|Returns:|||
|nil|||


Example:
```lua

local childDevices = luup.chdev.start(THIS_LUL_DEVICE)

luup.chdev.append(
    THIS_LUL_DEVICE,
    childDevices,
    'Group'..groupAddressStr,        -- altid
    "C-Bus Group "..groupAddressStr, -- name
    veraDevice,                      -- device type
    veraFile,                        -- device filename
    '',                              -- implementation filename
    '',                              -- parameters
    false)                           -- embedded

```

### start
luup.chdev.start (device)

Get a reference to the children info. Child data store may be empty or it may contain valid children.

|Identifier|Type|Comments|
|---|---|---|
|Arguments:|||
|device|string|eg id of this parent. String or integer. String is the device's udn, else it's the device number.|
|.|||
|Returns:|||
|ptr|C code reference|eg pointer to all the children located by this function|


Example:
```lua
local childDevices = luup.chdev.start(THIS_LUL_DEVICE)
```

### sync
luup.chdev.sync (device)

Tell the Luup engine that all the children have been set up. The luup engine will restart. If you code is problematic, you are now in danger of a loop of continual restarts.

|Identifier|Type|Comments|
|---|---|---|
|Arguments:|||
|device|string|eg id of this parent. String or integer. String is the device's udn, else it's the device number.|
|ptr|C code reference|eg pointer to all the children located by the start function|
|.|||
|Returns:|||
|nil|||

Example:
```lua
-- childDevices value comes from executing luup.chdev.start(THIS_LUL_DEVICE)
luup.chdev.sync(THIS_LUL_DEVICE, childDevices)
```

Refer to `luup.chdev.append` where most of these parameters are set up.

## luup.devices
Table of devices by indexed by device id.

|Identifier|Type|Comments|
|---|---|---|
|Arguments:|||
|device_id|string or integer|String is the device's udn, else it's the device number.|
|.|||
|Returns:|||
|room_num|integer|The device is in this room|
|device_type|string||
|category_num|integer||
|subcategory_num|integer||
|device_num_parent|integer|If this is a child, then this its parent's id.|
|ip|string|ip address if relevant|
|mac|string|mac address if relevant|
|user|string|For authentication at ip address|
|pass|string|For authentication at ip address|
|id|string|AltID as seen in the UI - used to identify children|
|embedded|boolean|If child - can't be detacjhed from its parent|
|hidden|boolean|If true device is not shown in the UI|
|invisible|boolean|If true device is completely isolated from the user|
|description|string|Users descrption seen in the UI|
|udn|string|udn of the device, can be used instead of device id|

```lua
local ip_address = luup.devices[THIS_LUL_DEVICE].ip)
```

## luup.inet

### luup.inet.wget

wget(url, time_out, username, password)

|Identifier|Type|Comments|
|---|---|---|
|Arguments:|||
|url|string||
|time_out|string||
|username|string||
|password|string||
|.|||
|Returns:|||
|status_code||0 is success|
|msg|||
|http_status_code|||

**openLuup enhancement:**
luup.inet.wget() now supports Basic and Digest authentication,

Username/Password parameters may be defined in either the wget() parameter list or embedded in the URL.

## luup.io
All these functions are deprecated. They are a complete waste of time, as they can be unbelievably slow. Use [luasocket](https://lunarmodules.github.io/luasocket/) instead.

**Functions:**
- luup.io.intercept
- luup.io.is_connected
- luup.io.open
- luup.io.read
- luup.io.write

## luup.ir

### luup.ir.pronto_to_gc100

This function is deprecated. Not particularly useful unless you have a [gc100](https://www.globalcache.com/). There are plugins that do this and more.

|Identifier|Type|Comments|
|---|---|---|
|Arguments:|||
|pronto code|string||
|.|||
|Returns:|||
|gc100 IR code|string||

```lua
local gc100Code = luup.ir.pronto_to_gc100 (pronto_IR_code)
```

## luup.job

### luup.job.set

luup.job.set(job, setting, value)

|Identifier|Type|Comments|
|---|---|---|
|Arguments:|||
|job|C code reference|eg pointer to the job's internal C code.
|setting|string||
|value|string||
|.|||
|Returns:|||
|nil|||

### luup.job.setting

luup.job.setting(job, setting)

|Identifier|Type|Comments|
|---|---|---|
|Arguments:|||
|job|C code reference|eg pointer to the job's internal C code.
|setting|string||
|.|||
|Returns:|||
|value|string||

### luup.job.status

|Identifier|Type|Comments|
|---|---|---|
|Arguments:|||
|job_number|integer||
|device|string|String or integer. String is the device's udn, else it's the device number.|
|.|||
|Returns:|||
|job_status|integer|
|notes|string|

|job_status|meaning|
|---|---|
|-1|job is nil|
|0|waiting to start|
|1|in progress|
|2|error|
|3|aborted|
|4|done|
|5|??|
|6|??|
|7|??|
|8|??|


## luup.remotes[]

nil - not used

## luup.rooms[]
   strings = ordered list of room names

```lua
print ("The broken light is in room "..luup.rooms[66])
```

## luup.scenes[]
   tables = table of scenes indexed by id

|Identifier|Type|Comments|
|---|---|---|
|Arguments:|||
|scene_id|integer||
|.|||
|Returns:|||
|room_num|integer||
|description|string||
|hidden|boolean||

```lua
print ("Scene 20 is attached to room "..luup.scenes[20].room_num)
```

## openLuup enhancements
These functions have enhanced
- luup.variable_get
- luup.register_handler

In addition we have:

### Solar functionality

The openLuup plugin device has variables for solar:

- Right Ascension (RA)
- Declination (DEC)
- Altitude (ALT)
- Azimuth (AZ)

which are updated every two minutes.

These calculations have always been done by openLuup to support scene times of sunrise/sunset, but are available for use (negating the need for a separate plugin, eg. Heliotrope.)

There is a GetSolarCoords action which gives these values for any given time (defaults to now) and latitude/longitude (defaults to current location.)

```lua
luup.call_action ("solar", "GetSolarCoords", coords, 2)
```

The defaults can be overloaded:
```lua
-- default to current time & location set up in startup ode
local coords = {Epoch = '', Latitude = '', Longitude = ''}

-- default to current time and specify location
local coords = {Epoch = '', Longitude  = -0.0045417, Latitude = 51.4820845}
```

Example:
```lua
local openLuupID = 2 -- always the case

-- default to current time and specify location
local coords = {Epoch = '', Latitude = 51.4820845, Longitude = -0.0045417}

luup.call_action ("solar", "GetSolarCoords", coords, openLuupID)

-- read RA, DEC, ALT and AZ as needed
local RA  = luup.variable_get("solar", "RA",  openLuupID)
local DEC = luup.variable_get("solar", "DEC", openLuupID)
local ALT = luup.variable_get("solar", "ALT", openLuupID)
local AZ  = luup.variable_get("solar", "AZ",  openLuupID)

-- all in degrees
print (RA)
print (DEC)
print (ALT)
print (AZ)

```



## Examining the Luup engine

With some code along the lines of the following, we can see what the Luup Engine encapsulates:

```lua
local function getLuupInfo(lookAt)
   local numbers   = {}
   local strings   = {}
   local functions = {}
   local tables    = {}

   for k, v in pairs(lookAt) do
      if (type(v) == 'number') then
         table.insert(numbers,k)
      elseif (type(v) == 'string') then
         table.insert(strings,k)
      elseif (type(v) == 'function') then
         table.insert(functions,k)
      elseif (type(v) == 'table') then
         table.insert(tables,k)
      else
         print(k..'='..type(k))
      end
   end

   table.sort(numbers)
   table.sort(strings)
   table.sort(functions)
   table.sort(tables)

   local numbersInfo   = table.concat(numbers,'\n')
   local stringsInfo   = table.concat(strings,'\n')
   local functionsInfo = table.concat(functions,'\n')

   return numbersInfo,
          stringsInfo,
          functionsInfo,
          tables
end

local numbersInfo, stringsInfo, functionsInfo, tables = getLuupInfo(luup)

print('numbers:\n'..numbersInfo..'\n\nstrings:\n'..stringsInfo..'\n\nfunctions:\n'..functionsInfo)
print('\ntables:\n'..pretty(tables))

return true
```

Which produces:

numbers:
```text
latitude
longitude
pk_accesspoint
version_branch
version_major
version_minor
```

strings:
```text
city
event_server
event_server_backup
hw_key
ra_server
ra_server_backup
timezone
version
```

functions:
```text
attr_get
attr_set
call_action
call_delay
call_timer
create_device
device_message
device_supports_service
devices_by_service
ip_set
is_night
is_ready
job_watch
log
mac_set
modelID
register_handler
reload
set_failure
sleep
sunrise
sunset
task
variable_get
variable_set
variable_watch
```

tables:
```text
{"chdev",
"devices",
"inet",
"io",
"ir",
"job",
"remotes",
"rooms",
"scenes"}
```

## Code subsections
The table above indicate further sub groups. We can examine each one using the following example. See "luup.chdev" in this example.
```
local numbersInfo, stringsInfo, functionsInfo, tables = getLuupInfo(luup.chdev)lua
```

Doing so results in the following:
```lua
luup.chdev
functions:
   append
   start
   sync


luup.devices
   tables = table of device ids


luup.inet
functions:
   wget


luup.io
functions:
   intercept
   is_connected
   open
   read
   write


luup.ir
functions:
   pronto_to_gc100


luup.job
functions:
   set
   setting
   status


luup.remotes
   nil


luup.rooms
   strings = ordered list of room names indices


luup.scenes
   tables = table of scenes indexed by id
```
