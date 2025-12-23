# Miscellaneous code examples

## Useful links: validators
[JSONLint](http://jsonlint.com/)

[HTML Validator](http://validator.w3.org/)

[XML Validator](http://www.w3schools.com/xml/xml_validator.asp)

[JavaScript Lint](http://www.javascriptlint.com/online_lint.php)

[CSS Validation](http://jigsaw.w3.org/css-validator/)

## Useful links: Lua
[Lua code playground](https://onecompiler.com/lua)

[Lua 5.1 Manual](http://www.lua.org/manual/5.1/)

[Lua Wiki](http://lua-users.org/wiki/LuaDirectory)

[Lua for windows](https://github.com/rjpcomputing/luaforwindows)

[ZeroBrane Studio for Vera](https://studio.zerobrane.com/vera.html)

[Lua with Eclipse](http://www.eclipse.org/koneki/ldt/)

## Are we running under openLuup or the Vera OS?
```lua
local function isOpenLuup()
    -- if we're using openLuup; variable openLuup will be a table, otherwise nil
    local openLuup = luup.attr_get('openLuup')
    return (openLuup ~= nil)
end

print("Running under openLuup is " .. tostring(isOpenLuup()))
```

## Using IDs & SIDs
Defining IDs & SIDs makes your code clearer and life easier:
```lua
local ID = {
    LIGHT_ENTRANCE_AREA         = 45,
    LIGHT_FRONT_GARDEN          = 33,
    LIGHT_FRONT_PORCH           = 15,
    LIGHT_FRONT_SPOT            = 21,
    LIGHT_HUE_ENTRANCE_AREA     = 19,
    LIGHT_HUE_ENTRANCE_CORRIDOR = 30,
    LIGHT_STAIRS                = 12
}

local SID = {
    AV_TRANSPORT     = "urn:upnp-org:serviceId:AVTransport",
    BLIND            = "urn:upnp-org:serviceId:WindowCovering1",
    DIMMER_SWITCH    = "urn:upnp-org:serviceId:Dimming1",
    DISCRETE_ON_OFF  = "urn:micasaverde-com:serviceId:DiscretePower1",
    ENERGY_METER     = "urn:micasaverde-com:serviceId:EnergyMetering1",
    HUE_COLOR        = "urn:micasaverde-com:serviceId:Color1",
    OPEN_LUUP        = "openLuup",
    RGBW_STRIP       = "urn:upnp-org:serviceId:RGBController1",
    SCENE_ACTIVATED  = "urn:micasaverde-com:serviceId:SceneController1",
    SOLAR            = "solar",
    SWITCH           = "urn:upnp-org:serviceId:SwitchPower1",
    TEMP_SENSOR      = "urn:upnp-org:serviceId:TemperatureSensor1"
}

local function setSwitches(switches, level)
    for _,v in ipairs(switches) do luup.call_action(SID.SWITCH, "SetTarget", {newTargetValue=level}, v) end
end

local function setDimmers(dimmers, level)
    for _,v in ipairs(dimmers) do luup.call_action(SID.DIMMER_SWITCH,"SetLoadLevelTarget", {newLoadlevelTarget = level}, v) end
end

local function sunset_sunrise(isSunset)
    local theSwitches = {
        ID.LIGHT_ENTRANCE_AREA,
        ID.LIGHT_FRONT_GARDEN.
        ID.LIGHT_FRONT_PORCH,
        ID.LIGHT_FRONT_SPOT,
        ID.LIGHT_STAIRS
    }

    local theDimmers = {
        ID.LIGHT_HUE_ENTRANCE_AREA,
        ID.LIGHT_HUE_ENTRANCE_CORRIDOR
    }

    if isSunset then
       setSwitches(theSwitches, 1)
       setDimmers (theDimmers, 10)
    else -- sunrise
       setSwitches(theSwitches, 0)
       setDimmers (theDimmers,  0)
    end
end
```

## Serialise a table
This is useful for passing multiple parameters in luup.call_delay():
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

## Encode a URL
```lua
local function urlEncode(str)
    if (str) then
        str = string.gsub (str, "\n", "\r\n")
        str = string.gsub (str, "([^%w %-%_%.%~])",
            function (c) return string.format ("%%%02X", string.byte(c)) end)
        str = string.gsub (str, " ", "+")
    end
    return str
end

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

## Telegram
You need a bot and chat ID from Telegram.
```lua
local function telegram(msg)
    local botID  = 'insert_bot_ID_here'
    local chatID = 'chat_ID_here'
    msg = urlEncode(msg)

    if (msg and (msg ~= '')) then
        luup.inet.wget('https://api.telegram.org/bot'..botID..'/sendEmail?chat_id='..chatID..'&text='..msg,5)
    else
        luup.log('Telegram message string is nil or blank',50)
    end
end

telegram('The house alarm has just been triggered!')
```

## Send an email
Adjust these values to suit your email set up:
- SENT_FROM   
- EMAIL_SERVER
- EMAIL_PORT
- USER
- PW

And call the function as seen in the code below:   sendEmail (sentTo, subjectLine, message).

With these values set as needed:
- sentTo
- subjectLine
- message

```lua
-- https://stackoverflow.com/questions/29312494/sending-email-using-luasocket-smtp-and-ssl
-- Michal Kottman, 2011, public domain. With some modifications as of April 2025.

local socket = require 'socket'
local smtp   = require 'socket.smtp'
local ssl    = require 'ssl'

smtp.TIMEOUT = 1

local SENT_FROM    = 'me@example.com'
local EMAIL_SERVER = 'mail.example.com'
local EMAIL_PORT   = '465'  -- others also used: 993, 995, etc

local USER = 'user_name'
local PW   = 'password'

local function sendEmail(sendTo, subjectLine, body)
    local function sslCreate()
        local sock = socket.tcp()
        return setmetatable({
            connect = function(_, host, port)
                local r, e = sock:connect(host, port)
                if not r then return r, e end
                
                -- various options for the protocol here eg: tlsv1, tlsv1_2, sslv3, etc
                sock = ssl.wrap(sock, {mode='client', protocol='tlsv1_2'})
                return sock:dohandshake()
            end
        }, {
            __index = function(t,n)
                return function(_, ...)
                    return sock[n](sock, ...)
                end
            end
        })
    end

    local msg = {
        headers = {
            from    = SENT_FROM,  -- becomes the 'reply to' address
            to      = sendTo,
            subject = subjectLine
        },
        body = body
    }

    local ok, err = smtp.send {
        -- from & rcpt email addresses should open/close with <>
        from = '<'..SENT_FROM..'>',

        -- can have multiple recipients in this table.
        -- default behavior is similar to BCC. Alter headers for To & CC
        rcpt = {'<'..sendTo..'>'},

        server   = EMAIL_SERVER,
        port     = EMAIL_PORT,
        user     = USER,
        password = PW,

        source = smtp.message(msg),
        
        -- you can comment this line out if SSL is not required
        create = sslCreate
    }
    if not ok then
        luup.log('Error - mail send failed: '..tostring(err),50)
    end
end

local sendTo      = 'someone@somedomain.com'
local subjectLine = 'Message dated ' .. os.date('%c')
local message     = 'Plugin says hello.\r\n'

sendEmail (sendTo, subjectLine, message)
```
