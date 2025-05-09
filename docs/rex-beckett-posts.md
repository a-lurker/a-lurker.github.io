# Conditional Scene Execution: Some Examples

The following is courtesy of Rex Beckett on the Vera forum and has been edited throughout. See this link for the original information:

https://community.ezlo.com/t/conditional-scene-execution-some-examples/178331/1

RexBeckett:

One of the most frequent questions is: How do I stop my scene running when… This has been asked and answered for many different types of condition and diligent searching will often find you a solution. Here are some of the most common scenarios.

The mechanism for preventing a scene running is simple: You insert Luup code into the scene that returns false if you want the scene blocked or true if you want to allow it to run. You can insert the code in a Trigger’s Luup event to allow or block only that trigger. You can also insert the code into the scene’s main LUUP tab where it will allow or block all triggers and manual operation. You can use a combination of both types for more complex scenarios. UI7 does not currently allow code to be attached to individual triggers so only the main Luup Code tab may be used.

Examples follow:

## Day or Night conditions
RexBeckett:

To allow a scene to run only at night:

```lua
return luup.is_night()
```

To prevent a scene running at night:

```lua
return not luup.is_night()
```

Or a more generic form:

```lua
local allow = true -- true runs scene at night, false blocks it
return ((luup.is_night()) == allow)
```

Many of us use the DayTime (Day or Night) plugin as an alternative to the luup.is_night() function. It has a few advantages: You can configure offsets from sunrise and sunset to control your definition of daytime; You can manually set it to Day or Night to test your scenes; It gives you an indicator on your dashboard to show its current state.

Generic DayTime:

```lua
local dID = 23 -- Device ID of your DayTime plugin
local allow = true -- true runs scene during daytime, false runs it at night
local status = luup.variable_get("urn:rts-services-com:serviceId:DayTime","Status",dID)
return ((status == "1") == allow)
```

## Z-Wave and Virtual Switches
RexBeckett:

We often want to allow or block scenes depending on the state of a Z-Wave switch (E.g. a light is on) or a VirtualSwitch (e.g. Home/Away). The VirtualSwitch (VS) plugin is very useful as a means of controlling scenes. You can create several different VS devices to signify various states of your home. E.g. Home/Away, OnVacation, HaveGuests, etc. VS devices can be set manually through the UI, by scenes or with other plugins.

Testing the state of a switch is essentially the same whether it is real or virtual but the serviceID must match the type of switch being tested.

Z-Wave Switch:

```lua
local dID = 66 -- Device ID of your Z-Wave Switch
local allow = true -- true runs scene if switch is on, false blocks it
local status = luup.variable_get("urn:upnp-org:serviceId:SwitchPower1","Status",dID)
return ((status == "1") == allow)
```

Virtual Switch:

```lua
local dID = 32 -- Device ID of your VirtualSwitch
local allow = true -- true runs scene if switch is on, false blocks it
local status = luup.variable_get("urn:upnp-org:serviceId:VSwitch1","Status",dID)
return ((status == "1") == allow)
```

MultiSwitch:

```lua
local dID = 94 -- Device ID of your MultiSwitch
local button = 1 -- MultiSwitch button number (1-8)
local allow = true -- true runs scene if switch is on, false blocks it
local status = luup.variable_get("urn:dcineco-com:serviceId:MSwitch1","Status"..button,dID)
return ((status == "1") == allow)
```

## Time Period
RexBeckett:

We frequently want to control the time periods during which a scene may run. You would probably prefer that your bedroom light did not come on in the middle of the night when you are clocked by the motion-sensor on your way to the bathroom. ;D

Here is a generic routine that can be set to allow or block a scene in the period between two times. The start and end times can be set within the same day or either side of midnight. Both times must be in 24-hour form and entered as HH:MM. As with previous generic examples, the variable allow determines whether to allow or block the scene during the specified period.

Generic Time Period:

```lua
local pStart = "22:30" -- Start of time period
local pEnd = "06:15" -- End of time period
local allow = true -- true runs scene during period, false blocks it
local hS, mS = string.match(pStart,"(%d+)%:(%d+)")
local mStart = (hS * 60) + mS
local hE, mE = string.match(pEnd,"(%d+)%:(%d+)")
local mEnd = (hE * 60) + mE
local tNow = os.date("*t")
local mNow = (tNow.hour * 60) + tNow.min
if mEnd >= mStart then
    return (((mNow >= mStart) and (mNow <= mEnd)) == allow)
else
    return (((mNow >= mStart) or (mNow <= mEnd)) == allow)
end
```

With a small modification, we can take the start time as sunset with a +/- offset. Strictly speaking, this will use the time of the next sunset so will give a slightly different result before and after sunset. The difference is about two minutes so should not be a major problem.

Sunset to End Time:

```lua
local pStart = 0 -- Start of time period, minutes offset from sunset
local pEnd = "06:15" -- End of time period
local allow = true -- true runs scene during period, false blocks it
local mStart = math.floor( (luup.sunset() % 86400) / 60 ) + pStart
local hE, mE = string.match(pEnd,"(%d+)%:(%d+)")
local mEnd = (hE * 60) + mE
local tNow = os.date("*t")
local mNow = (tNow.hour * 60) + tNow.min
if mEnd >= mStart then
   return (((mNow >= mStart) and (mNow <= mEnd)) == allow)
else
   return (((mNow >= mStart) or (mNow <= mEnd)) == allow) end
```

We can also use sunrise as one of our times. This version has a specified start time and the end time is sunrise with a +/- minutes offset.

Start Time to Sunrise:

```lua
local pStart = "22:30" -- Start of time period
local pEnd = 0 -- End of time period, minutes offset from sunrise
local allow = true -- true runs scene during period, false blocks it
local hS, mS = string.match(pStart,"(%d+)%:(%d+)")
local mStart = (hS * 60) + mS
local mEnd = math.floor( (luup.sunrise() % 86400) / 60 ) + pEnd
local tNow = os.date("*t")
local mNow = (tNow.hour * 60) + tNow.min
if mEnd >= mStart then
   return (((mNow >= mStart) and (mNow <= mEnd)) == allow)
else
   return (((mNow >= mStart) or (mNow <= mEnd)) == allow)
end
```

## Temperature Range
RexBeckett:

This code will enable you to set a range of temperatures within which your scene will, or will not, be run. Set tLow and tHigh to define the range. As with previous generic examples, the variable allow determines whether to allow or block the scene when the current temperature is within the specified range.

Generic Temperature Range:

```lua
local dID = 55 -- Device ID of your thermostatic/temperature sensor
local tLow = 18 -- Lowest temperature of range
local tHigh = 22 -- Highest temperature of range
local allow = true -- true runs scene when in range, false blocks it
local tCurrent = tonumber((luup.variable_get("urn:upnp-org:serviceId:TemperatureSensor1","CurrentTemperature",dID)))
return (((tCurrent >= tLow) and (tCurrent <= tHigh)) == allow)
```

## Humidity Range
RexBeckett:

This code will enable you to set a range of humidity within which your scene will, or will not, be run. Set hLow and hHigh to define the range. As with previous generic examples, the variable allow determines whether to allow or block the scene when the current humidity level is within the specified range.

Generic Humidity Range:

```lua
local dID = 31 -- Device ID of your humidity sensor
local hLow = 50 -- Lowest humidity of range
local hHigh = 80 -- Highest humidity of range
local allow = true -- true runs scene when in range, false blocks it
local hCurrent = tonumber((luup.variable_get("urn:micasaverde-com:serviceId:HumiditySensor1","CurrentLevel",dID)))
return (((hCurrent >= hLow) and (hCurrent <= hHigh)) == allow)
```

## Light Level
RexBeckett:

This code will enable you to set a range of light level within which your scene will, or will not, be run. Set lLow and lHigh to define the range. As with previous generic examples, the variable allow determines whether to allow or block the scene when the current light level is within the specified range.

Generic Light Level Range:

```lua
local dID = 30 -- Device ID of your light sensor
local lLow = 0 -- Lowest level of range
local lHigh = 20 -- Highest level of range
local allow = true -- true runs scene when in range, false blocks it
local lCurrent = tonumber((luup.variable_get("urn:micasaverde-com:serviceId:LightSensor1","CurrentLevel",dID)))
return (((lCurrent >= lLow) and (lCurrent <= lHigh)) == allow)
```

## VariableContainer
RexBeckett:

The VariableContainer (VC) plugin provides a convenient way in which values may be entered and changed without requiring a Vera restart. It also provides a form of scratchpad that can be used to convey values from one scene or plugin to another.

This code will enable you to set a range of values in a VC variable within which your scene will, or will not, be run. Set vLow and vHigh to define the range and vNo to define the VC variable number (1-5) in which the value is stored. As with previous generic examples, the variable allow determines whether to allow or block the scene when the VC variable value is within the specified range.

Generic VirtualContainer Value Range:
```lua
local dID = 76 -- Device ID of your VariableContainer
local vNo = 5 -- Variable number (1-5) to test
local vLow = 100 -- Lowest value of range
local vHigh = 199 -- Highest value of range
local allow = true -- true runs scene when in range, false blocks it
local sVC = luup.variable_get("urn:upnp-org:serviceId:VContainer1","Variable" .. vNo,dID)
local vVC = tonumber(sVC) or 0
return (((vVC >= vLow) and (vVC <= vHigh)) == allow)
```

Another great use for VC is to allow us to change the limits in our scene Luup without restarting Vera. The following code tests for the current time to be within the period set by two VC variables. These variables must contain valid times in HH:MM form. Other than taking the Start and Stop times from VC variables, this code works the same as the earlier Time Range example.

Generic Time Range from VC Variables:

```lua
local dID = 76 -- Device ID of your VariableContainer
local vStart = 4 -- Variable number (1-5) of Start time
local vEnd = 5 -- Variable number (1-5) of End time
local allow = true -- true runs scene when during time range, false blocks it
local pStart = luup.variable_get("urn:upnp-org:serviceId:VContainer1","Variable" .. vStart,dID) or ""
if #pStart == 0 then
    pStart = "00:00"
end
local pEnd = luup.variable_get("urn:upnp-org:serviceId:VContainer1","Variable" .. vEnd,dID) or ""
if #pEnd == 0 then
   pEnd = "00:00"
end
local hS, mS = string.match(pStart,"(%d+)%:(%d+)")
local mStart = (hS * 60) + mS
local hE, mE = string.match(pEnd,"(%d+)%:(%d+)")
local mEnd = (hE * 60) + mE
local tNow = os.date("*t")
local mNow = (tNow.hour * 60) + tNow.min
if mEnd >= mStart then
    return (((mNow >= mStart) and (mNow <= mEnd)) == allow)
else
    return (((mNow >= mStart) or (mNow <= mEnd)) == allow) end
```

## Day Range
RexBeckett:

This code will enable you to set a range of days during which your scene will, or will not, be run. Set dFirst and dLast to define the range. These are day numbers (1-7) where 1 is Sunday. As with previous generic examples, the variable allow determines whether to allow or block the scene when the current day is within the specified range.

Generic Day Range:

```lua
local dFirst = 2 -- Start day of period (1-7) Sunday = 1
local dLast = 6 -- End day of period (1-7) Sunday = 1
local allow = true -- true runs scene during period, false blocks it
local tNow = os.date("*t")
local dNow = tNow.wday
if dLast >= dFirst then
    return (((dNow >= dFirst) and (dNow <= dLast)) == allow)
else
    return (((dNow >= dFirst) or (dNow <= dLast)) == allow)
end
```

## Date Range
RexBeckett:

This code will enable you to set a range of dates during which your scene will, or will not, be run. Set mdStart and mdEnd to define the range. These are dates in the form of MM/DD (in deference to our US members). The included period may span the change of year if required. As with previous generic examples, the variable allow determines whether to allow or block the scene when the current date is within the specified range.

Generic Date Range:

```lua
local mdStart = "12/01" -- Start of period (MM/DD)
local mdEnd = "12/31" -- End of period (MM/DD)
local allow = true -- true runs scene during period, false blocks it
local smS, sdS = string.match(mdStart,"(%d+)%/(%d+)")
local smE, sdE = string.match(mdEnd,"(%d+)%/(%d+)")
local mS = tonumber(smS)
local dS = tonumber(sdS)
local mE = tonumber(smE)
local dE = tonumber(sdE)
local tNow = os.date("*t")
local mN = tNow.month
local dN = tNow.day
if (mE > mS) or ((mE == mS) and (dE >= dS)) then
    return (((mN > mS) or ((mN == mS) and (dN >= dS))) and ((mN < mE) or ((mN == mE) and (dN <= dE))) == allow)
else return (((mN > mS) or ((mN == mS) and (dN >= dS))) or ((mN < mE) or ((mN == mE) and (dN <= dE))) == allow)
end
```

If the preceding examples don’t cover what you want to achieve:

Try the good old search facility. There’s a very good chance that someone else had a similar requirement and found a solution. Google search with site:community.ezlo.com may give you better results than the forum’s own search-engine.

You may need to combine more than one piece of code. This can be done by using a variable to hold the result of one test and and-ing or or-ing it with another. For example, to run a scene when a temperature is within a range but only during the day, try this:


```lua
local isDay = not luup.is_night()

local dID = 55 -- Device ID of your thermostatic/temperature sensor

local tLow = 18 - Lowest temperature of range
local tHigh = 22 -- Highest temperature of range

local allow = true -- true runs scene when in range, false blocks it

local tCurrent = tonumber((luup.variable_get("urn:upnp-org:serviceId:TemperatureSensor1","CurrentTemperature",dID)))

return (((tCurrent >= tLow) and (tCurrent <= tHigh)) == allow) and isDay
```

## Multiple Conditions
RexBeckett:

You can combine multiple conditions in scene or trigger Luup by converting each piece of code into a Lua function. Then you can combine the results of each test in a single return statement using and/or operators.

Converting to a function is easy:

```lua
local function functionName() <lines of code> end
```

Combining the results is also simple:

```lua
return function1() and function2()
```

Example to allow a scene to run between 08:00 and 22:30 at weekends:

```lua
local function checkTime()
   local pStart = "08:00" - Start of time period
   local pEnd = "22:30" - End of time period
   local allow = true - true runs scene during period, false blocks it
   local hS, mS = string.match(pStart,"(%d+)%:(%d+)")
   local mStart = (hS * 60) + mS
   local hE, mE = string.match(pEnd,"(%d+)%:(%d+)")
   local mEnd = (hE * 60) + mE
   local tNow = os.date("*t")
   local mNow = (tNow.hour * 60) + tNow.min
   if mEnd >= mStart then
      return (((mNow >= mStart) and (mNow <= mEnd)) == allow)
   else
      return (((mNow >= mStart) or (mNow <= mEnd)) == allow)
   end
end

local function checkDay()
   local dFirst = 7 - Start day of period (1-7) Sunday = 1
   local dLast = 1 - End day of period (1-7) Sunday = 1
   local allow = true - true runs scene during period, false blocks it
   local tNow = os.date("*t")
   local dNow = tNow.wday
   if dLast >= dFirst then
       return (((dNow >= dFirst) and (dNow <= dLast)) == allow)
   else
       return (((dNow >= dFirst) or (dNow <= dLast)) == allow)
   end
end

return checkTime() and checkDay()
```

## Using Functions
RexBeckett:

I showed in Multiple Conditions 1 how a code chunk could be converted to a function. With small adjustments, the function call can pass the parameters to be used for the test. Now a single piece of code can be used to test several devices. The function code can even be added to your Startup Lua so it may be called from any scene. In the following examples, I changed the original lines that set the parameters to comments (with --) as a reminder of what each parameter means.

Function to check if a temperature is within a range:

```lua
function checkTemp(dID, tLow, tHigh, allow) --dID = 55
-- Device ID of your thermostatic/temperature sensor
local tLow = 18
-- Lowest temperature of range
local tHigh = 22
-- Highest temperature of range
local allow = true
-- true runs scene when in range, false blocks it

local tCurrent = tonumber((luup.variable_get( "urn:upnp-org:serviceId:TemperatureSensor1","CurrentTemperature",dID)))
return (((tCurrent >= tLow) and (tCurrent <= tHigh)) == allow)
end
```

Function to check the state of a Z-Wave switch:

```lua
function checkSwitch(dID, allow)
local dID = 66 -- Device ID of your Z-Wave Switch
local allow = true -- true runs scene if switch is on, false blocks it

local status = luup.variable_get("urn:upnp-org:serviceId:SwitchPower1","Status",dID)
return ((status == "1") == allow) end
```

We can use these two functions to implement a more-complex set of conditions. This example, suggested by @parkerc, checks the temperature in three different rooms. If any of the temperatures are out of range, checkTemp(device,min,max,true) returns false. In this case we check if the switch controlling our heating is off and, if so, allow the scene to run to turn it on:

```lua
if checkTemp(123,18,22,true) and checkTemp(124,18,22,true) and checkTemp(125,18,22,true)
then return false

-- All temperatures are in range, don't run scene else return checkSwitch(101,false) -- Run scene if switch is off end
```

Adding functions to Startup Lua:

Select the Apps tab, click on Develop Apps and select Edit Startup Lua. Add your functions to the end of the existing code and click GO. Now click on Vera’s Reload button to restart the luup engine. Your functions should now be available to the luup in any scene.

## Multiple Triggers
RexBeckett:

Sometimes we want to have several different events or schedules result in essentially the same action but with different settings. An example would be setting different temperature setpoints or dimmer levels depending and time schedules or any of the conditions in the previous examples. The obvious way to do this is with several scenes but there is another way. We can use a global variable to transport our required settings from Trigger Luup event to the main scene LUUP.

This example is for a scene that sets a Thermostat setpoint based on one of three triggers.

Trigger 1 Luup event - Set to 20 during daytime, do nothing at night:

```lua
local function checkDayTime()
   local dID = 23 -- Device ID of your DayTime plugin
   local allow = true -- true runs scene during daytime, false runs it at night
   local status = luup.variable_get("urn:rts-services-com:serviceId:DayTime","Status",dID)
   return ((status == "1") == allow)
end

if checkDayTime() then
   setTemp = 20
   return true
else
   return false
end
```

Trigger 2 Luup event - Set to 20 during daytime or 10 at night:

```lua
local function checkDayTime()
   local dID = 23 -- Device ID of your DayTime plugin
   local allow = true -- true runs scene during daytime, false runs it at night
   local status = luup.variable_get("urn:rts-services-com:serviceId:DayTime","Status",dID)
   return ((status == "1") == allow)
end

if checkDayTime() then
   setTemp = 20
else
   setTemp = 10
end

return true
```

Trigger 3 Luup event - Set to 18:

setTemp = 18 return true
Main scene LUUP - Version 1:

```lua
local dID = 55 if (setTemp ~= nil) then
   luup.call_action("urn:upnp-org:serviceId:TemperatureSetpoint1_Heat","SetCurrentSetpoint",{NewCurrentSetpoint=setTemp},dID)
   setTemp = nil
end
```

Main scene LUUP - Version 2:

```lua
local dID = 55 if (setTemp == nil) then
   setTemp = 20
end

luup.call_action("urn:upnp-org:serviceId:TemperatureSetpoint1_Heat","SetCurrentSetpoint",{NewCurrentSetpoint=setTemp},dID)
setTemp = nil
```

Version 1 of the Main scene LUUP will do nothing if run from the UI because, if a trigger did not occur, setTemp will be nil. Version 2, if run from the UI, will set the temperature setpoint to 20.
16 days later

## Time Window
RexBeckett:

There were two requests for this type of scene condition in the last 24 hours so it must be worth posting here. The objective is to allow or prevent a scene executing when triggers occur in a short time period (window).

All these examples save a timestamp each time they are run using a global variable tLastOnD99. On each run, the saved timestamp is subtracted from the current time and compared to the specified time window (twSecs). To allow this code to be used for more than one triggering device, the 99 in tLastOnD99 should be replaced with the device ID of the triggering device. This is simply to avoid undesired side effects so any other name could be used instead.

As with previous examples, the code may be placed in either a Trigger’s Luup event to allow/prevent that trigger or on the scene’s LUUP tab where it will affect all triggers and manual operation.

Allow the scene to run when a trigger occurs within twSecs of the last one (e.g. on/off/on):

```lua
local twSecs = 5 -- Number of seconds in time window
local tNow = os.time()
local tLastOn = tLastOnD99 or 0
tLastOnD99 = tNow
return ((tNow - tLastOn) <= twSecs)
```

Allow the scene to run provided a trigger occurs at least twSecs after the last one:

```lua
local twSecs = 5 -- Number of seconds in time window
local tNow = os.time()
local tLastOn = tLastOnD99 or 0
tLastOnD99 = tNow
return ((tNow - tLastOn) >= twSecs)
```

Generic version:
```lua
local twSecs = 5 -- Number of seconds in time window
local allow = true -- true runs scene during time window, false blocks it
local tNow = os.time()
local tLastOn = tLastOnD99 or 0
tLastOnD99 = tNow
return (((tNow - tLastOn) <= twSecs) == allow)
```

## Service IDs, Variables and Actions

RexBeckett:

One of the most frequent problems we see in requests for help with Lua code, is errors in Service IDs and variable or parameter names in luup function calls. These errors can be tricky to find. Even one character in the wrong case will prevent the call working correctly: A luup.variable_get(…) will return a nil instead of the expected value; A luup.call_action(…) may do absolutely nothing.

Listed below are example luup calls for the most common device types. All examples assume that local variable dID contains the ID of the device being addressed. e.g. local dID = 123 Alternatively, replace dID in the calls with the required device ID number. Examples are given for reading a device variable using luup.variable_get(…) and for initiating an action with luup.call_action(…). It is also possible to directly set device variables but, in many cases, this will have little or no apparent effect on the device. Generally it is better to use luup.call_action(…) rather than directly setting device variables.

If the device you want to address is not listed, you can find the Service ID by hovering the mouse cursor over the name of the variable on the device’s Advanced tab.

On/Off Switch
Set Target, read Status. "0" = Off, "1" = On:

```lua
local status = luup.variable_get("urn:upnp-org:serviceId:SwitchPower1", "Status", dID)

luup.call_action("urn:upnp-org:serviceId:SwitchPower1", "SetTarget", {newTargetValue = "1"}, dID)
```

Virtual Switch
Set Target, read Status. "0" = Off, "1" = On:

```lua
local status = luup.variable_get("urn:upnp-org:serviceId:VSwitch1", "Status", dID)

luup.call_action("urn:upnp-org:serviceId:VSwitch1", "SetTarget", {newTargetValue = "1"}, dID)
```

Dimmable Light
Set LoadLevelTarget, read LoadLevelStatus. "0" = Off, "100" = Full:

```lua
local level = luup.variable_get("urn:upnp-org:serviceId:Dimming1", "LoadLevelStatus", dID)

luup.call_action("urn:upnp-org:serviceId:Dimming1", "SetLoadLevelTarget", {newLoadlevelTarget = "50"}, dID)
```

Thermostat
Set ModeTarget, read ModeStatus. "Off", "HeatOn", "CoolOn", "AutoChangeOver":

```lua
local mode = luup.variable_get("urn:upnp-org:serviceId:HVAC_UserOperatingMode1", "ModeStatus", dID)

luup.call_action("urn:upnp-org:serviceId:HVAC_UserOperatingMode1", "SetModeTarget", {NewModeTarget = "Off"}, dID)
```

Set and read CurrentSetpoint. (Degrees):

```lua
local setpoint = luup.variable_get("urn:upnp-org:serviceId:TemperatureSetpoint1_Heat", "CurrentSetpoint", dID)

luup.call_action("urn:upnp-org:serviceId:TemperatureSetpoint1_Heat", "SetCurrentSetpoint", {NewCurrentSetpoint = "25"}, dID)

local setpoint = luup.variable_get("urn:upnp-org:serviceId:TemperatureSetpoint1_Cool", "CurrentSetpoint", dID)

luup.call_action("urn:upnp-org:serviceId:TemperatureSetpoint1_Cool", "SetCurrentSetpoint", {NewCurrentSetpoint = "30"}, dID)
```

Temperature Sensor:

```lua
local temp = luup.variable_get("urn:upnp-org:serviceId:TemperatureSensor1", "CurrentTemperature", dID)
```

Generic Sensor:

```lua
local level = luup.variable_get("urn:micasaverde-com:serviceId:GenericSensor1", "CurrentLevel", dID)
```

Light Sensor:

```lua
local level = luup.variable_get("urn:micasaverde-com:serviceId:LightSensor1", "CurrentLevel", dID)
```

Humidity Sensor:

```lua
local level = luup.variable_get("urn:micasaverde-com:serviceId:HumiditySensor1", "CurrentLevel", dID)
```

Security Sensor:

```lua
local tripped = luup.variable_get("urn:micasaverde-com:serviceId:SecuritySensor1", "Tripped", dID)

local armed = luup.variable_get("urn:micasaverde-com:serviceId:SecuritySensor1", "Armed", dID)

local lasttrip = luup.variable_get("urn:micasaverde-com:serviceId:SecuritySensor1", "LastTrip", dID)

luup.call_action("urn:micasaverde-com:serviceId:SecuritySensor1", "SetArmed", {newArmedValue = "1"}, dID)
```

Window Covering
Use Dimmable Light Status variable for current position. Down = 0, Up = 100:

```lua
local level = luup.variable_get("urn:upnp-org:serviceId:Dimming1", "LoadLevelStatus", dID)

luup.call_action("urn:upnp-org:serviceId:WindowCovering1", "Up", {}, dID)

luup.call_action("urn:upnp-org:serviceId:WindowCovering1", "Down", {}, dID)

luup.call_action("urn:upnp-org:serviceId:WindowCovering1", "Stop", {}, dID)
```

Variable Container
Set and read VariableN where N is 1 to 5:

```lua
local vcvar1 = luup.variable_get("urn:upnp-org:serviceId:VContainer1", "Variable1", dID)

luup.variable_set("urn:upnp-org:serviceId:VContainer1", "Variable1",newvalue, dID)

luup.variable_set("urn:upnp-org:serviceId:VContainer1", "Variable5","newtext", dID)
```

DayTime plugin
Set and read Status. "0" = Night, "1" = Day:

```lua
local itsday = luup.variable_get("urn:rts-services-com:serviceId:DayTime", "Status", dID)

luup.variable_set("urn:rts-services-com:serviceId:DayTime", "Status", "1", dID)
```

## Delayed Actions
RexBeckett:

Another favourite question concerns using delays in scene Lua. It is tempting to do this with luup.sleep(milliseconds) but this can lead to problems. While luup.sleep(…) is executing, other Vera events can be blocked. A peak of activity at this point can cause a Vera restart. This is one possible cause of unexplained restarts.

Luup provides a more-robust way of achieving delays with luup.call_delay("callfunction",secs,"parmstring"). This will call the function named callfunction after a delay of secs seconds and pass it the optional parameter parmstring. The called function must be global (i.e. not local). After the luup.call_delay(…), the main code continues to execute and may terminate. Multiple luup.call_delay(…) calls may be made to the same or different called functions.

For example, the following code turns on a light and then turns it off after ten seconds. You could, of course, do this simply with scene action delays but it’s a stepping stone:

```lua
-- Function to be called after delay
function delayOff()
   luup.call_action("urn:upnp-org:serviceId:SwitchPower1","SetTarget",{ newTargetValue=0},66)
end

luup.call_action("urn:upnp-org:serviceId:SwitchPower1","SetTarget",{ newTargetValue=1 },66)
luup.call_delay("delayOff",10)
-- Main code terminates at this point
```

Because the called function is global, it does not have access to any of the local variables in the main part of your code. You could make your variables global but be careful how you name them to avoid side effects with other code. A cleaner approach is to pass them via the optional parameter. This parameter is a string so you will need to convert numeric data for some purposes.

In this example, the device ID of the switch is passed as the parameter:

```lua
local dID = 66

function delayOff(dev)
   local devno = tonumber(dev)
   luup.call_action("urn:upnp-org:serviceId:SwitchPower1","SetTarget",{ newTargetValue=0},devno)
end

luup.call_action("urn:upnp-org:serviceId:SwitchPower1","SetTarget",{ newTargetValue=1 },dID)
luup.call_delay("delayOff",10,dID)
```

The called function can also issue a luup.call_delay(…) to itself. This allows timed sequences to be programmed. The following example will dim a light by 10% every two seconds until it reaches zero:

```lua
local dID = 99

function delayDim(dev)
   local devno = tonumber(dev)
   local lls = tonumber((luup.variable_get("urn:upnp-org:serviceId:Dimming1", "LoadLevelStatus", devno)))
   local newlls = lls - 10
   if newlls < 0 then newlls = 0 end
   luup.call_action("urn:upnp-org:serviceId:Dimming1", "SetLoadLevelTarget", {newLoadlevelTarget = newlls}, devno)
   if newlls > 0 then luup.call_delay("delayDim",2,dev) end
end

luup.call_delay("delayDim",2,dID)
```

Only one parameter string can be passed to the called function. If you need to pass more than one item, you will need to encode them into the string. Simple numeric items can be passed as a comma-separated list. The next example uses this technique to pass three numeric values. The resulting code will either dim a light down to zero or up to 100% - depending on its level when the scene is run:

```lua
local dID = 99

function delayDim(parms)
   local pdev,ptgt,pstp = string.match(parms,"(%d+),(%d+),([%-%d]+)")
   local devno = tonumber(pdev)
   local target = tonumber(ptgt)
   local step = tonumber(pstp)
   local lls = tonumber((luup.variable_get("urn:upnp-org:serviceId:Dimming1", "LoadLevelStatus", devno)))
   local newlls = lls + step
   if step > 0 then
      if newlls > target then newlls = target end
   else
      if newlls < target then newlls = target end
end

local dstep, dtarget
local level = tonumber((luup.variable_get("urn:upnp-org:serviceId:Dimming1", "LoadLevelStatus", dID)))
if level > 50 then
   dtarget = 0
   dstep = -10
else
   dtarget = 100
   dstep = 10
end

local prms = string.format("%d,%d,%d",dID,dtarget,dstep)
luup.call_delay("delayDim",2,prms)

luup.call_action("urn:upnp-org:serviceId:Dimming1", "SetLoadLevelTarget", {newLoadlevelTarget = newlls}, devno)
if newlls ~= target then luup.call_delay("delayDim",2,parms) end
end
```

Parameters may also be passed as a serialized table. See Passing a Serialized Table 1.

I hope some of these examples will help to point the way to a solution for your particular requirements.

## Delayed Actions - Passing a Serialized Table
RexBeckett:

As explained in an earlier post, the optional parameter passed by luup.call_delay(…) is a single string. If you need to pass several parameters to the called function, a Lua table can be serialized into that string and recreated when the called function executes.

The general-purpose serialize function in the following example will convert a table containing numeric, string, boolean or nested table elements into a single string. The table can be recreated using the Lua loadstring statement.

This example performs the same function as the previous one but a serialized table is used to pass the three numeric values. The values are initially set in the table dparms which is then serialized and sent as the parameter in luup.call_delay(…). The called function uses loadstring to recreate the table as xparms and can then access the individual variables using xparms. notation. The resulting code will either dim a light down to zero or up to 100% - depending on its level when the scene is run.

```lua
local function serialize(val,name)
local tmp = ""
if name then
   tmp = tmp … name … "="
end
if type(val) == "table" then
   tmp = tmp … "{"
   for k, v in pairs(val) do
      if type(k) == "number" then
         tmp = tmp … serialize(v) … ","
      else
         tmp = tmp … serialize(v, k) … ","
      end
   end  -- for
   tmp = tmp … "}"

elseif type(val) == "number" then
   tmp = tmp … tostring(val)

elseif type(val) == "string" then
   tmp = tmp … string.format("%q", val)

elseif type(val) == "boolean" then
   tmp = tmp … (val and "true" or "false")

else
   tmp = tmp … ""[inserializeable datatype:" … type(val) … "]""
end

return tmp
end -- function

local dID = 99
local dparms = {}
dparms.dev = dID
local level = tonumber((luup.variable_get("urn:upnp-org:serviceId:Dimming1", "LoadLevelStatus", dID)))
if level > 50 then
   dparms.target = 0
   dparms.step = -10
else
   dparms.target = 100
   dparms.step = 10
end

luup.call_delay("delayDim",2,serialize(dparms))

function delayDim(parms)
   assert(loadstring("xparms="…parms))()
   local lls = tonumber((luup.variable_get("urn:upnp-org:serviceId:Dimming1", "LoadLevelStatus", xparms.dev)))
   local newlls = lls + xparms.step
   if xparms.step > 0 then
      if newlls > xparms.target then
         newlls = xparms.target
      end
   else
      if newlls < xparms.target then
         newlls = xparms.target
      end
   end
   luup.call_action("urn:upnp-org:serviceId:Dimming1", "SetLoadLevelTarget", {newLoadlevelTarget = newlls}, xparms.dev)

   if newlls ~= xparms.target then luup.call_delay("delayDim",2,parms) end
end
```

## Debugging Lua Code - kwikLog
RexBeckett:

Debugging Lua code in scenes can be very frustrating. When something doesn’t work as you expected, you need to be able to see which parts of your code are being executed and, often, the value of key variables. Vera provides luup.log(…) to help with this but it is not always easy to find your log entries in LuaUPnP.log with all the other activity there.

kwikLog is a simple function that logs your messages, with a time stamp, to a standalone file. The file may be viewed from a browser by entering /kwikLog.txt. A browser page refresh will show you the latest entries.

You can use kwikLog in several ways:

- Add it to your scene Lua (above where you call it from your code)
- Add it before code you run in Test Luup code (Lua)
- Add it to Startup Lua so that you can use it from any scene

If you add it to Startup Lua (APPS → Develop Apps → Edit Startup Lua), remove the word local from the first line.

```lua
local function kwikLog(message, clear)
local socket = require("socket")
local time = socket.gettime() or os.time()
local tms = string.format(".%03d ",math.floor (1000 * (time % 1)))
local stamp = os.date("%d %b %Y %T",math.floor(time)) .. tms
local mode = "a+" if clear then mode = "w+" end
local file = io.open("/www/kwikLog.txt", mode)
file:write(stamp .. (message or "") .. "\n")
file:close()
end
```

kwikLog has two arguments: The string you wish to log and, optionally, true if you want to clear the log file. If the second argument is missing or false, log messages are appended to the existing file. You can place calls to kwikLog anywhere in your code so you can see what is happening. For example:

```lua
kwikLog("SuperScene started",true)
local status = luup.variable_get("urn:upnp-org:serviceId:SwitchPower1","Status",123)
if status == "1" then
   kwikLog("Light is already on!")
end
local level = luup.variable_get("urn:upnp-org:serviceId:Dimming1","LoadLevelStatus",123) kwikLog("Dim level is " .. level)
if (status == "0") and (tonumber(level) > 0) then
   kwikLog("This can't happen...")
end
```
etc.

Several users have recently had problems with Sunset and Sunrise triggers due to incorrect location information. As a quick demonstration of kwikLog, paste the following code into APPS → Develop Apps → Test Luup code (Lua) and click GO.

```lua
local function kwikLog(message, clear)
local socket = require("socket")
local time = socket.gettime() or os.time()
local tms = string.format(".%03d ",math.floor (1000 * (time % 1)))
local stamp = os.date("%d %b %Y %T",math.floor(time)) … tms
local mode = "a+"
if clear then mode = "w+" end
local file = io.open("/www/kwikLog.txt", mode)
file:write(stamp … (message or "") … "\n")
file:close()
end

kwikLog("Timezone: " … luup.timezone … " hours")
kwikLog("City: " … luup.city)
kwikLog("Latitude: " … luup.latitude … " degrees")
kwikLog("Longitude: " … luup.longitude … " degrees")
local srtime = os.date("%c",luup.sunrise())
kwikLog("Next Sunrise: " … srtime)
local sstime = os.date("%c",luup.sunset())
kwikLog("Next Sunset: " … sstime)
```

Enter /kwikLog.txt in a browser window to see if your location data is correct. Replace with the IP address of your Vera (without the <>).

## Running a scene when a variable changes
RexBeckett:

There have been several recent posts on this subject so I think it is worth including here. The issue is that normal scene triggers that include conditions, like temperature is lower than 18, will fire when the temperature drops below 18 but will not fire again unless it rises to or above 18 and then drops below it again.

If you use this type of trigger for a scene that includes Lua code for additional conditions - say a time period - it will not work very well. If the trigger fires outside of the allowed time period, you may not get another trigger during the period when you want to take some action.

One solution is to have your scene triggered by a periodic schedule. Then you can use Lua code from the examples in this thread to decide if you want the actions to be executed.

A better technique is to watch the variable in question and have your scene run when it changes. As above, you can then use Lua code to decide if you want to execute some action(s). Vera provides a function for this very purpose. Here is how you can use it:

```lua
-- Set up variable-watch for device 123
luup.variable_watch("doChange123","urn:upnp-org:serviceId:TemperatureSensor1","CurrentTemperature",123)

-- Process variable-watch callback for device 123. Run scene 12
function doChange123()
   luup.call_action("urn:micasaverde-com:serviceId:HomeAutomationGateway1", "RunScene", {SceneNum = 12}, 0)
end
```

If this code is placed in Vera’s Startup Lua, it will set up a variable-watch for thermostat/thermometer device 123 current temperature. When this changes, scene number 12 will be executed.

To edit Startup Lua, select APPS → Develop Apps → Edit Startup Lua then paste the code on the end of any existing line and click GO. You will need to restart Vera before the changes take effect.

If you want to watch other variables, you can repeat the code for each one. You will need to change the name of the function in both the function and the luup.variable_watch(…) statements to keep them unique.

If you want to watch several variables, the following example shows how this can be done by using a table instead of multiple pieces of code. watchTable contains an entry for each variable that you want to watch. Each entry includes the ServiceId, variable name and device number that you want to watch and the scene number to be run if it changes. Each entry, other than the last one, should be terminated with a comma.

The function setWatch reads this table and sets up a watch for each entry. All the watches use the same callback function. This function, catchWatch, searches the table for a matching entry and, if it finds one, runs the specified scene.

```lua
-- Watch table parameters:  { "ServiceID", "VariableName", DeviceNo, SceneNo }
watchTable = {
{"urn:upnp-org:serviceId:TemperatureSensor1","CurrentTemperature",123,11},
{"urn:micasaverde-com:serviceId:LightSensor1","CurrentLevel",124,12}
}

-- Setup Variable Watch for each entry in watchTable
function setWatch()
   for n,t in ipairs(watchTable) do
      luup.variable_watch("catchWatch",t[1],t[2],t[3])
   end
end

-- When a watched variable changes, call the specified scene
function catchWatch(lul_device, lul_service, lul_variable, lul_value_old, lul_value_new)
   for n,t in ipairs(watchTable) do
      if ( (t[3] == lul_device) and (t[1] == lul_service) and (t[2] == lul_variable) ) then
         luup.call_action("urn:micasaverde-com:serviceId:HomeAutomationGateway1", "RunScene", {SceneNum = t[4]}, 0)
      end
   end
end

-- Wait for 30 seconds after restart then run setWatch
luup.call_delay("setWatch",30)
```

The last part of this code causes the setWatch function to be run 30 seconds after Vera restarts. This is to allow all devices to have completed their initialization.

## Finding the Correct Service ID
RexBeckett:

First adventures into using Lua code frequently fail due to the use of incorrect Service IDs in luup.variable_get(…), luup.variable_set(…) and luup.call_action(…) calls. A common error is to use the Device-type instead of the Service ID. Tip: A Service ID string will contain the word serviceId and will not contain the words schemas or device.

I listed some of the common ones in Service IDs, Variables and Actions 1. It is also possible to see the Service ID for a device variable by hovering your mouse cursor over the variable name on the device’s Advanced tab. This is OK if you have a good visual memory. :wink:

One way to see the Service ID in a copy/paste-able form is to enter this command in your browser - replacing with the IP address of your Vera (without the <>):

```html
http://<veraip>:3480/data_request?id=status&output_format=xml
```

This will list every variable for all of your devices so you will need to scroll down the list until you find the one you want. You can also get the information for a particular device by using this - replacing 123 with the device number:

```html
http://<veraip>:3480/data_request?id=status&output_format=xml&DeviceNum=123
```

To see all the actions available for your devices, enter this:

```html
http://<veraip>:3480/data_request?id=invoke
```

or, for a single device:

```html
http://<veraip>:3480/data_request?id=invoke&DeviceNum=123
```

Note that not all devices will actually implement all the available actions. The list is a super-set covering all devices of the given Device-type. Theoretically, actions marked with an asterisk (*) should be implemented but this is not always the case. Clicking on an entry in the invoke list will fire the action so be careful!

If you would prefer a less-cluttered list, you could use LuaTest which has buttons to display Variables, Values, Actions and individual device Status.

## Testing Lua Code - LuaTest
RexBeckett:

LuaTest is Lua code designed to  faciltate debugging your own code. See the forum discussion here:

[LuaTest - A Tool for Testing Scene Lua Code](https://community.ezlo.com/t/luatest-a-tool-for-testing-scene-lua-code/180205)

The User Guide is here:

[LuaTest: Documentation](https://drive.google.com/file/d/0BykZKwGsCBsATkUyaEs3QVo3ZEE/edit?resourcekey=0-BDsSTPkNJlp5jSc6Msr9QA)

and the code itself here:

[LuaTest: code](https://drive.google.com/file/d/1ve5sCq-NASQS3WUQT86VhVkXwbNrZsLt/view)

with some updates here:

[LuaTest: code 2](https://gist.github.com/cgmartin/d52409fe473e1d4fc5044c3d676d109c)
