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

print('Numbers:\n'..numbersInfo..'\n\nStrings:\n'..stringsInfo..'\n\nfunctions:\n'..functionsInfo)
print('\nTables:\n'..pretty(tables))

return true
```

Which produces:

```lua
Numbers:
latitude
longitude
pk_accesspoint
version_branch
version_major
version_minor

Strings:
city
event_server
event_server_backup
hw_key
ra_server
ra_server_backup
timezone
version

functions:
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

Tables:
{"chdev","devices","inet","io","ir","job","remotes","rooms","scenes"}
```

# Functions

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

|Identifier|Type|Description|
|---|---|---|
|Arguments:|||
|attribute|string|eg id, name |
|device|string or number||
|.|||
|Returns:|||
|attribute value|string||

Example:
```lua
local macAddress = attr_get ('mac', 23)
```

## attr_set
attr_set (attribute, value, device, refresh_user_data)

|Identifier|Type|Description|
|---|---|---|
|Arguments:|||
|attribute|string|eg id, name |
|value|string||
|device|string or number||
|refresh_user_data|boolean|Increments DataVersion in the user_data.json file|
|.|||
|Returns:|||
|Nil|||

Example:
```lua
attr_set ('mac', '2D:EE:91:87:46:A3', 23, false)
```

## call_action
call_action (service, device)

|Identifier|Type|Description|
|---|---|---|
|Arguments:|||
|service|string|eg "urn:upnp-org:serviceId:SwitchPower1"|
|action|string|eg "SetTarget"|
|arguments|table|eg \{newTargetValue = level}|
|device|string or number||
|.|||
|Returns:|||
|error|number||
|error_msg|string||
|job|number||
|arguments|table||

Example:
```lua
luup.call_action("urn:upnp-org:serviceId:SwitchPower1", "SetTarget", {"newLoadlevelTarget" = "1"}, 43)
```

