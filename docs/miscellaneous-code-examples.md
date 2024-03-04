# Miscellaneous code examples

## Are we running under openLuup or the Vera OS?
```lua
-- if we're using openLuup this will get a table, otherwise nil
local function isOpenLuup()
    local openLuup = luup.attr_get('openLuup')
    return (openLuup ~= nil)
end

print("Running under openLuup is " .. tostring(isOpenLuup()))
```

## Serialise a table
```lua
-- write
-- use this in openLuup
local json = require "openLuup.json"

-- use this in Vera
-- local json = require "dkjson"

local t = {a=math.pi, b="test"}
local v = json.encode (t)
luup.variable_set (serviceId, variable, v, 42)

-- read
local x = luup.variable_get (serviceId, variable, 42)
local y = json.decode (x)

-- y should now have a copy of the original table t
print(pretty(y))
```

## Get our local IP address
```lua
local socket = require('socket')

local function getOurIPaddress()
    local SOME_RANDOM_IP   = '1.1.1.1'
    local SOME_RANDOM_PORT = '1'

    local udp = socket.udp()
    if (udp == nil) then print('Socket failure: socket lib missing?',50) return '' end
    udp:setpeername(SOME_RANDOM_IP, SOME_RANDOM_PORT)

    -- now we can get our LAN IP address
    local ipa,_,_ = udp:getsockname()
    udp:close()

    return ipa
end

print(getOurIPaddress())
```

## base64 encode/decode
```lua
local mime = require( "mime" )
local encodedString = mime.b64("Hello World")
print(mime.unb64(encodedString))
```
