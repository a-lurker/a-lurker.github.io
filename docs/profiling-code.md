# Profiling code

## Console
The console provides a page that profiles the installed plugins.

```html
http://openLuup_IP_address:3480/console?page=plugins
```

It makes uses of these calls:
```lua
local times = luup.openLuup.cpu_table()
print (pretty(times))

times = luup.openLuup.wall_table()
print (pretty(times))
```

## Socket gettime()
The socket.gettime() function is capable of high accuracy timing:
```lua
local socket = require('socket')
local m_startTime = 0
local m_endTime = 0

-- insert the code to time here eg
m_startTime = socket.gettime()
whatsTakingSoLong()
m_endTime = socket.gettime()

local elapsedTime = (m_endTime - m_startTime)*1000
print(string.format('Elapsed time: %.2f msec', elapsedTime))
```

## Profiler example
Here's an example how to use the profiler code laid out further below. Just run it in the Lua test box(s).
```lua
local profile = require "miniprofiler"
profile "on"       -- start profiling

-- run whatever code you want here
-- we'll do this as an example (openLuup only)
local x, t = luup.variable_get ("openLuup", "Memory_Mb", 2, {0, os.time()})

profile "off"      -- turn off profiling
print (profile)    -- print the result

profile "clear"    -- clear the results to start again
```

## Profiler code
Save the following code block as 'miniprofiler.lua' and copy it to '/etc/cmh-ludl'

```lua
-- miniprofiler.lua

-- simple profiler for function calls
-- @akbooer  June 2020

--Use:
--  local profile = require "miniprofiler"
--  profile "on"       -- start profiling
--  profile "off"      -- stop  profiling
--  profile "clear"    -- reset counts
--  print (profile)    -- show  results

local calls = 0
local funcs = {}
local hook = debug.sethook

local function profile(event)
  if event == "call" then
    calls = calls + 1
    local i = debug.getinfo (2, 'n')
    local n = table.concat {(i.name or '?'), '/', i.namewhat}
    funcs[n] = (funcs[n] or 0) + 1
  end
end

local function report()
  local fcts = {}
  for n,v in pairs (funcs) do fcts[#fcts+1] = {n,v} end       -- make name index
  table.sort (fcts, function (a,b) return b[2] < a[2] end)    -- alphabetize
  local result = "%12s   %-8s   %s"
  local out = {result: format (calls, "TOTAL","Function calls")}
  local split = "^([^/]+)/(.*)"
  for _, x in ipairs(fcts) do
    local f, w = x[1]: match (split)
    out[#out+1] = result: format (x[2], w, f)
  end
  out[#out+1] = ''
  return table.concat (out, '\n')
end

local function call (_, x)
  local f = {
    on    = function () hook (profile, 'c') end,
    off   = function () hook () end,
    clear = function () for x in pairs(funcs) do funcs[x] = nil end calls = 0 end}
  do (f[x] or f.clear) () end
end

return setmetatable (funcs, {__call = call, __tostring = report})

```
