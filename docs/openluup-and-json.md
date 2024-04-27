# openLuup and json

openLuup has a decode json parser loading hierarchy depending on what is available on the host device.

1. [rapidjson](http://miloyip.github.io/rapidjson/) -- very rapid
2. [cjson](https://www.kyne.com.au/%7Emark/software/lua-cjson.php) -- at least 10x faster than the openLuup module
3. native openLuup json module -- fallback position

Note that the openLuup module is used for all encode calls, because it sorts table parameters and pretty prints its output.

It's highly recommended to have rapidjson or cjson (lua version) be made available on the host device.

There is potentially a huge benefit for plugins such as VeraBridge, ZWay, AltUI and AltHue. As the openLuup JSON module will use the C-library based cjson module or rapidjson for decoding.

You can check the console startup log to see what json decoder is currently being used by openLuup at:

http://ip_address:3480/console?page=startup_log

1. rapidjson decoder:
```text
2024-04-20 15:01:54.828  openLuup.json::   version RapidJSON (0.7.1) + openLuup (2021.05.01)  @akbooer
```

2. cjson decoder:
```text
2024-02-18 14:14:18.568  openLuup.json::   version Cjson (2.1.0.10) + openLuup
```

3. Internal openLuup json decoder:
```text
2024-02-15 08:08:01.139  openLuup.json::  version 2021.05.01  @akbooer
```

## Installing rapidjson or cjson:

### To install rapidjson on a Raspberry Pi:

```bash
# Do updates ready for install
sudo apt update
sudo apt upgrade

# Get Lua Rocks package installer
sudo apt install luarocks

# Get the rapidjson parser/encoder for Lua
sudo luarocks install rapidjson

# Check the installation
luarocks show rapidjson

# Probably best to reboot
sudo reboot
```
If you get this error message you probably need to install cmake:

```bash
Error: Build error: 'cmake' program not found. Make sure CMake is installed and is available in your PATH (or you may want to edit the 'variables.CMAKE' value in file '/etc/luarocks/config-5.1.lua')
```

Installing cmake:

```bash
sudo apt install cmake
```

### To install cjson on a Raspberry Pi:

```bash
# Do updates ready for install
sudo apt update
sudo apt upgrade

# Get Lua Rocks package installer
sudo apt install luarocks

# Get the lua-cjson parser/encoder for Lua
sudo luarocks install lua-cjson

# Check the installation
luarocks show lua-cjson

# Probably best to reboot
sudo reboot
```

## Plugin developers
Developers can make use of the selected openluup parser like so:

```lua
local json = require "openLuup.json"
```

## Vera and json
More recent versions of Vera hardware provided `dkjson.lua` located at `/etc/cmh-ludl/`. openLuup installs this file, so plugins that use it can find it.

Early Vera incantations had no json parser available at all. Later on, as mentioned above , dkjson was included. So plugin developers used any old parser they could find (if lucky). Consequently you ended up with bizarre code such as this. Plus the huge risk of the different parsers being subtly incompatible - particuarly on handling "nil".

```lua
-- If possible, get a JSON parser. If none available, returns nil. Note that typically UI5 may not have a parser available.
local function loadJsonModule()
    local jsonModules = {
        'rapidjson',            -- how many json libs are there?
        'cjson',                -- openLuup?
        'dkjson',               -- UI7 firmware
        'openLuup.json',        -- https://community.ezlo.com/t/pure-lua-json-library-akb-json/185273
        'akb-json',             -- https://community.ezlo.com/t/pure-lua-json-library-akb-json/185273
        'json',                 -- OWServer plugin
        'json-dm2',             -- dataMine plugin
        'dropbox_json_parser',  -- dropbox plugin
        'hue_json',             -- hue plugin
        'L_ALTUIjson',          -- AltUI plugin
    }

    local ptr  = nil
    local json = nil
    for n = 1, #jsonModules do
        -- require does not load the module, if it's already loaded
        -- Vera has overloaded require to suit their requirements, so it works differently from openLuup
        -- openLuup:
        --    ok:     returns true or false indicating if the module was loaded successfully or not
        --    result: contains the ptr to the module or an error string showing the path(s) searched for the module
        -- Vera:
        --    ok:     returns true or false indicating the require function executed but require may have or may not have loaded the module
        --    result: contains the ptr to the module or an error string showing the path(s) searched for the module
        --    log:    log reports 'luup_require can't find xyz.json'
        local ok, result = pcall(require, jsonModules[n])
        ptr = package.loaded[jsonModules[n]]
        if (ptr) then
            json = ptr
            debug('Using: '..jsonModules[n])
            break
        end
    end
    if (not json) then debug('No JSON library found',50) return json end
    return json
end
```

## openLuup internal json encoder/decoder
The encoder/decoder used by openLuup is based on a library written by akbooer some time ago. The salient points re: that library follow:

There is already a large number of options for [JSON libraries](http://lua-users.org/wiki/JsonModules]http://lua-users.org/wiki/JsonModules) available to Lua users. None are perfect; only some are pure Lua; some are quite slow.
However @akbooer created another!

A few features:
- it has two basic function calls: `encode` and `decode`. They both return two parameters:
  1. required conversion or nil
  2. error message, if first return is nil.
- include it in your code with `json = require "akb-json"` and call it with `JSON=json.encode(LUA); LUA=json.decode(JSON);`
- numbers bigger than json.default.huge, which can be changed but defaults to 8.88e888, are treated as infinity.
- json.default.max_array_length = 1000, and can be changed. Noting there the code guards against things like `json.encode{[1e6]=1}`.
- supports unicode escapes and UTF-8 encoding of Basic Multilingual Plane (BMP) codepoints.
- returns an error, if given a self-referencing structure to encode.
- returns an error, if encoding a table with mixed numeric and string indices as JSON can't represent these.
- `json.version` returns the library version number.

Speed: comprehensive unit testing including about 100 different cases, half of which are valid conversions, and three more extensive files (timings on a 2.66 GHz Intel Core 2 Duo iMac, about 20x the speed of a VeraLite.


- 6 kB Netatmo device file - decode: 1.5 mS, encode: 4.5 mS
- 40 kB dataMine configuration file - decode: 17 mS, encode: 48 mS
- 650 kB Luup user_data file - decode: 220 mS, encode: 725 mS

It scales fairly linearly.

This module is also embedded in applications Netatmo, EventWatcher, DataYours, and pretty-prints its encoded JSON strings.

The library is expected to be in `/usr/lib/lua/`.
