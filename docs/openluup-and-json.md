# openLuup and json

openLuup has a decode json parser loading hierarchy depending on what is available on the host device.

1. [rapidjson](http://miloyip.github.io/rapidjson/) -- very rapid
2. [cjson](https://www.kyne.com.au/%7Emark/software/lua-cjson.php) -- at least 10x faster than the openLuup module
3. native openLuup json module -- fallback position

Note that the openLuup module is used for all encode calls, because it sorts table parameters and pretty prints its output.

It's highly recommended to have rapidjson or cjson (lua version) be made available on the host device.

There is potentially a huge benefit for plugins such as VeraBridge and ZWay. As the openLuup JSON module will use the C-library based cjson module or rapidjson for decoding.

Raspberry Pis typically have the cjson module installed already. Check with this command:

```bash
sudo apt-cache search lua-cjson
```

## Plugin developers
Developers can make use of the selected openluup parser like so:

```lua
local json = require "openLuup.json"
```

## Vera and json
Early Vera incantations had no json parser available at all. Later on, dkjson was included. So plugin developers used any old parser they could find (if lucky). Consequently you ended up with bizarre code such as this. Plus the huge risk of the different parsers being subtly incompatible.

```lua
-- If possible, get a JSON parser. If none available, returns nil. Note that typically UI5 may not have a parser available.
local function loadJsonModule()
    local jsonModules = {
        'rapidjson',            -- how many json libs are there?
        'cjson',                -- openLuup?
        'dkjson',               -- UI7 firmware
        'openLuup.json',        -- https://community.getvera.com/t/pure-lua-json-library-akb-json/185273
        'akb-json',             -- https://community.getvera.com/t/pure-lua-json-library-akb-json/185273
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
