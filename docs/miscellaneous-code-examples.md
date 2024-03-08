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

## Get our local LAN IP address
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

## Escape json
```lua
-- escape text chars to suit json
-- http://www.ietf.org/rfc/rfc4627.txt
local function escJSONentities(s)
    s = s:gsub('\\', '\\u005c')
    s = s:gsub('\n', '\\u000a')
    s = s:gsub('"',  '\\u0022')
    s = s:gsub("'",  '\\u0027')
    s = s:gsub('/',  '\\u002f')
    s = s:gsub('\t', '\\u0009')
    return s
end
```

## Escape HTML 1
```lua
local function escHTMLentities(s)
    s = s:gsub('&', '&amp;')
    s = s:gsub('<', '&lt;')
    s = s:gsub('>', '&gt;')
    s = s:gsub('"', '&quot;')
    s = s:gsub("'", '&#039;')
    return s
end
```

## Escape HTML 2
```lua
local function escHTMLentities2(s)
    s = s:gsub('&', '&#x26;')
    s = s:gsub('<', '&#x3c;')
    s = s:gsub('>', '&#x3e;')
    s = s:gsub('"', '&#x22;')
    s = s:gsub("'", '&#x27;')
    return s
end
```

