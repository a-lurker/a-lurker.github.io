# Examining the Luup library

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

# Functions, luup.function_name:

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


## attr_get
attr_get (attribute, device)

|Identifier|Type|Comments|
|---|---|---|
|Arguments:|||
|attribute|string|eg id, name |
|device|string or integer|String is the device's udn, else it's the device number.|
|.|||
|Returns:|||
|attribute value|string||

Example:
```lua
local macAddress = luup.attr_get ('mac', 23)
```

## attr_set
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

## call_action
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

## call_delay
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
luup.call_delay ("lightOn", "3600", "color")
```

## call_timer
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
luup.call_timer ("flashLight", 1, "5m", ,"color")
```

## create_device
create_device ()
Do we really need to do this?

## device_message
Deprecated.

## device_supports_service
Deprecated.

## devices_by_service
Deprecated.

## ip_set
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

## is_night
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

## is_ready
Deprecated.

## job_watch
Deprecated.

## log
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

## mac_set
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

## modelID
Deprecated. Refers to Vera models.

```lua
print(luup.modelID())
-- Result is "35" for Vera Edge and "0" for openLuup
```

## register_handler
register_handler ()

We love this function. It is extremely useful!

## reload
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

## set_failure
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
-- heart beat function indicates device at far end is off line
luup.set_failure (1)
```

## sleep
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

## sunrise
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

## sunset
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

## task
task ()

## variable_get
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

## variable_set
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

## variable_watch
variable_watch (function_name, service, variable_name, device)

|Identifier|Type|Comments|
|---|---|---|
|Arguments:|||
|function_name|string|Called when the variable changes.|
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

# Numbers:
## latitude
Used to calculate sunset, sunrise, etc. Can be altered in the start up code.
```lua
print(luup.latitude))
```

## longitude
Used to calculate sunset, sunrise, etc. Can be altered in the start up code.
Used to calculate sunset, sunrise, etc. Can be altered in the start up code.
```lua
print(luup.longitude))
```

## pk_accesspoint
Deprecated. Vera only. Used for cloud connections. Equals 88800000 or 87654321 in openLuup.

## version_branch
See below.

## version_major
See below.

## version_minor
```lua
print(luup.version_branch.."."..luup.version_major.."."..luup.version_minor)

-- shows "1.7.0"; same as print(luup.version)
```
# Strings:
## city
Can be altered in the start up code. We know where you live.

## event_server
Deprecated. Vera only. Used for cloud connections.

## event_server_backup
Deprecated. Vera only. Used for cloud connections.

## hw_key
Deprecated. Vera only. Used for cloud connections.

## ra_server
Deprecated. Vera only. Used for cloud connections.

## ra_server_backup
Deprecated. Vera only. Used for cloud connections.

## timezone
UTC offset including DST.

## version
```lua
print(luup.version)
-- shows "1.7.0"
```
