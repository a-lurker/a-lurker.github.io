The following is courtesy of Rex Beckett on the Vera forum and has been edited throughout. See this link for the original information:

https://community.ezlo.com/t/conditional-scene-execution-some-examples/178331/1

--------------------------------------------------------------------------------
# Conditional Scene Execution: Some Examples
RexBeckett:

One of the most frequent questions on this forum is How do I stop my scene running when… This has been asked and answered for many different types of condition and diligent searching will often find you a solution. To help newcomers to Vera, I am posting a few of the most common scenarios in this thread.

The mechanism for preventing a scene running is simple: You insert Luup code into the scene that returns false if you want the scene blocked or true if you want to allow it to run. You can insert the code in a Trigger’s Luup event to allow or block only that trigger. You can also insert the code into the scene’s main LUUP tab where it will allow or block all triggers and manual operation. You can use a combination of both types for more complex scenarios. UI7 does not currently allow code to be attached to individual triggers so only the main Luup Code tab may be used.

--------------------------------------------------------------------------------
# Examples list:
RexBeckett:

[Day or Night Conditions](Day-or-Night-Conditions.md)

[Z-Wave and Virtual Switches](z-wave-and-virtual-switches.md)

[Time Period](time-period.md)

[Temperature Range](temperature-range.md)

[Humidity Range](humidity-range.md)

[Light Level](
light-level.md)

[VariableContainer](variable-container.md)

[Day Range](day-range.md)

[Date Range](date-range.md)

[Multiple Conditions](multiple-conditions.md)

[Using Functions](using-functions.md)

[Multiple Triggers](multiple-triggers.md)

[Time Window](time-window.md)

[Service IDs, Variables and Actions](service-ids-variables-and-actions.md)

[Delayed Actions](delayed-actions.md)

[Delayed Actions - Passing a Serialized Table](delayed-actions-passing-a-serialized-table.md)

[Debugging Lua Code - kwikLog](debugging-lua-code-kwiklog.md)

[Run Scene when a Variable Changes](run-scene-when-a-variable-changes.md)

[Finding the Correct Service ID]()

[Testing Lua Code - LuaTest]()

--------------------------------------------------------------------------------
# Day or Night conditions
RexBeckett:

To allow a scene to run only at night:

```Lua
return luup.is_night()
```

To prevent a scene running at night:

```Lua
return not luup.is_night()
```

Or a more generic form:

```Lua
local allow = true -- true runs scene at night, false blocks it return ((luup.is_night()) == allow)
```

Many of us use the DayTime (Day or Night) plugin as an alternative to the luup.is_night() function. It has a few advantages: You can configure offsets from sunrise and sunset to control your definition of daytime; You can manually set it to Day or Night to test your scenes; It gives you an indicator on your dashboard to show its current state.

Generic DayTime:

```Lua
local dID = 23 -- Device ID of your DayTime plugin
local allow = true -- true runs scene during daytime, false runs it at night
local status = luup.variable_get("urn:rts-services-com:serviceId:DayTime","Status",dID)
return ((status == "1") == allow)
```

--------------------------------------------------------------------------------
# Z-Wave and Virtual Switches
RexBeckett:

We often want to allow or block scenes depending on the state of a Z-Wave switch (E.g. a light is on) or a VirtualSwitch (e.g. Home/Away). The VirtualSwitch (VS) plugin is very useful as a means of controlling scenes. You can create several different VS devices to signify various states of your home. E.g. Home/Away, OnVacation, HaveGuests, etc. VS devices can be set manually through the UI, by scenes or with other plugins.

Testing the state of a switch is essentially the same whether it is real or virtual but the serviceID must match the type of switch being tested.

Z-Wave Switch:

```Lua
local dID = 66 -- Device ID of your Z-Wave Switch
local allow = true -- true runs scene if switch is on, false blocks it
local status = luup.variable_get("urn:upnp-org:serviceId:SwitchPower1","Status",dID) 
return ((status == "1") == allow)
```

Virtual Switch:

```Lua
local dID = 32 -- Device ID of your VirtualSwitch
local allow = true -- true runs scene if switch is on, false blocks it
local status = luup.variable_get("urn:upnp-org:serviceId:VSwitch1","Status",dID) 
return ((status == "1") == allow)
```

MultiSwitch:

```Lua
local dID = 94 -- Device ID of your MultiSwitch local button = 1 -- MultiSwitch button number (1-8) 
local allow = true -- true runs scene if switch is on, false blocks it
local status = luup.variable_get("urn:dcineco-com:serviceId:MSwitch1","Status"..button,dID)
return ((status == "1") == allow)
```

--------------------------------------------------------------------------------
# Time Period
RexBeckett:

We frequently want to control the time periods during which a scene may run. You would probably prefer that your bedroom light did not come on in the middle of the night when you are clocked by the motion-sensor on your way to the bathroom. ;D

Here is a generic routine that can be set to allow or block a scene in the period between two times. The start and end times can be set within the same day or either side of midnight. Both times must be in 24-hour form and entered as HH:MM. As with previous generic examples, the variable allow determines whether to allow or block the scene during the specified period.

Generic Time Period:

```Lua
local pStart = "22:30" -- Start of time period
local pEnd = "06:15" -- End of time period local allow = true -- true runs scene during period, false blocks it
local hS, mS = string.match(pStart,"(%d+)%:(%d+)")
local mStart = (hS * 60) + mS 
local hE, mE = string.match(pEnd,"(%d+)%:(%d+)") 
local mEnd = (hE * 60) + mE local tNow = os.date("*t")
local mNow = (tNow.hour * 60) + tNow.min
if mEnd >= mStart then
    return (((mNow >= mStart) and (mNow <= mEnd)) == allow)
else
    return (((mNow >= mStart) or (mNow <= mEnd)) == allow)
end
```

With a small modification, we can take the start time as sunset with a +/- offset. Strictly speaking, this will use the time of the next sunset so will give a slightly different result before and after sunset. The difference is about two minutes so should not be a major problem.

Sunset to End Time:

```Lua
local pStart = 0 -- Start of time period, minutes offset from sunset 
local pEnd = "06:15" -- End of time period
local allow = true -- true runs scene during period, false blocks it
local mStart = math.floor( (luup.sunset() % 86400) / 60 ) + pStart
local hE, mE = string.match(pEnd,"(%d+)%:(%d+)") 
local mEnd = (hE * 60) + mE local tNow = os.date("*t")
local mNow = (tNow.hour * 60) + tNow.min
if mEnd >= mStart then
   return (((mNow >= mStart) and (mNow <= mEnd)) == allow)
else
   return (((mNow >= mStart) or (mNow <= mEnd)) == allow) end
```

We can also use sunrise as one of our times. This version has a specified start time and the end time is sunrise with a +/- minutes offset.

Start Time to Sunrise:

```Lua
local pStart = "22:30" -- Start of time period
local pEnd = 0 -- End of time period, minutes offset from sunrise
local allow = true -- true runs scene during period, false blocks it local hS, mS = string.match(pStart,"(%d+)%:(%d+)")
local mStart = (hS * 60) + mS 
local mEnd = math.floor( (luup.sunrise() % 86400) / 60 ) + pEnd
local tNow = os.date("*t") local mNow = (tNow.hour * 60) + tNow.min 
if mEnd >= mStart then
   return (((mNow >= mStart) and (mNow <= mEnd)) == allow)
else
   return (((mNow >= mStart) or (mNow <= mEnd)) == allow)
end
```

--------------------------------------------------------------------------------
# Temperature Range
RexBeckett:

This code will enable you to set a range of temperatures within which your scene will, or will not, be run. Set tLow and tHigh to define the range. As with previous generic examples, the variable allow determines whether to allow or block the scene when the current temperature is within the specified range.

Generic Temperature Range:

```Lua
local dID = 55 -- Device ID of your thermostatic/temperature sensor
local tLow = 18 -- Lowest temperature of range
local tHigh = 22 -- Highest temperature of range
local allow = true -- true runs scene when in range, false blocks it
local tCurrent = tonumber((luup.variable_get("urn:upnp-org:serviceId:TemperatureSensor1","CurrentTemperature",dID))) 
return (((tCurrent >= tLow) and (tCurrent <= tHigh)) == allow)
```

--------------------------------------------------------------------------------
# Humidity Range
RexBeckett:

This code will enable you to set a range of humidity within which your scene will, or will not, be run. Set hLow and hHigh to define the range. As with previous generic examples, the variable allow determines whether to allow or block the scene when the current humidity level is within the specified range.

Generic Humidity Range:

```Lua
local dID = 31 -- Device ID of your humidity sensor
local hLow = 50 -- Lowest humidity of range local hHigh = 80 -- Highest humidity of range
local allow = true -- true runs scene when in range, false blocks it
local hCurrent = tonumber((luup.variable_get("urn:micasaverde-com:serviceId:HumiditySensor1","CurrentLevel",dID)))
return (((hCurrent >= hLow) and (hCurrent <= hHigh)) == allow)
```

--------------------------------------------------------------------------------
# Light Level
RexBeckett:

This code will enable you to set a range of light level within which your scene will, or will not, be run. Set lLow and lHigh to define the range. As with previous generic examples, the variable allow determines whether to allow or block the scene when the current light level is within the specified range.

Generic Light Level Range:

```Lua
local dID = 30 -- Device ID of your light sensor
local lLow = 0 -- Lowest level of range
local lHigh = 20 -- Highest level of range
local allow = true -- true runs scene when in range, false blocks it
local lCurrent = tonumber((luup.variable_get("urn:micasaverde-com:serviceId:LightSensor1","CurrentLevel",dID)))
return (((lCurrent >= lLow) and (lCurrent <= lHigh)) == allow)
```

--------------------------------------------------------------------------------
# VariableContainer
RexBeckett:

The VariableContainer (VC) plugin provides a convenient way in which values may be entered and changed without requiring a Vera restart. It also provides a form of scratchpad that can be used to convey values from one scene or plugin to another.

This code will enable you to set a range of values in a VC variable within which your scene will, or will not, be run. Set vLow and vHigh to define the range and vNo to define the VC variable number (1-5) in which the value is stored. As with previous generic examples, the variable allow determines whether to allow or block the scene when the VC variable value is within the specified range.

Generic VirtualContainer Value Range:
```Lua
local dID = 76 -- Device ID of your VariableContainer
local vNo = 5 -- Variable number (1-5) to test local vLow = 100 -- Lowest value of range
local vHigh = 199 -- Highest value of range
local allow = true -- true runs scene when in range, false blocks it
local sVC = luup.variable_get("urn:upnp-org:serviceId:VContainer1","Variable" .. vNo,dID)
local vVC = tonumber(sVC) or 0 
return (((vVC >= vLow) and (vVC <= vHigh)) == allow)
```

Another great use for VC is to allow us to change the limits in our scene Luup without restarting Vera. The following code tests for the current time to be within the period set by two VC variables. These variables must contain valid times in HH:MM form. Other than taking the Start and Stop times from VC variables, this code works the same as the earlier Time Range example.

Generic Time Range from VC Variables:

```Lua
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
local mEnd = (hE * 60) + mE local tNow = os.date("*t")
local mNow = (tNow.hour * 60) + tNow.min
if mEnd >= mStart then
    return (((mNow >= mStart) and (mNow <= mEnd)) == allow)
else
    return (((mNow >= mStart) or (mNow <= mEnd)) == allow) end
```

--------------------------------------------------------------------------------
# Day Range
RexBeckett:

This code will enable you to set a range of days during which your scene will, or will not, be run. Set dFirst and dLast to define the range. These are day numbers (1-7) where 1 is Sunday. As with previous generic examples, the variable allow determines whether to allow or block the scene when the current day is within the specified range.

Generic Day Range:

```Lua
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

--------------------------------------------------------------------------------
# Date Range
RexBeckett:

This code will enable you to set a range of dates during which your scene will, or will not, be run. Set mdStart and mdEnd to define the range. These are dates in the form of MM/DD (in deference to our US members). The included period may span the change of year if required. As with previous generic examples, the variable allow determines whether to allow or block the scene when the current date is within the specified range.

Generic Date Range:

```Lua
local mdStart = "12/01" -- Start of period (MM/DD)
local mdEnd = "12/31" -- End of period (MM/DD)
local allow = true -- true runs scene during period, false blocks it
local smS, sdS = string.match(mdStart,"(%d+)%/(%d+)")
local smE, sdE = string.match(mdEnd,"(%d+)%/(%d+)") local mS = tonumber(smS)
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


```Lua
local isDay = not luup.is_night()

local dID = 55 -- Device ID of your thermostatic/temperature sensorlocal tLow = 18 – Lowest temperature of range

local tHigh = 22 -- Highest temperature of range

local allow = true -- true runs scene when in range, false blocks it

local tCurrent = tonumber((luup.variable_get("urn:upnp-org:serviceId:TemperatureSensor1","CurrentTemperature",dID)))

return (((tCurrent >= tLow) and (tCurrent <= tHigh)) == allow) and isDay
```

--------------------------------------------------------------------------------
# Multiple Conditions
RexBeckett:

You can combine multiple conditions in scene or trigger Luup by converting each piece of code into a Lua function. Then you can combine the results of each test in a single return statement using and/or operators.

Converting to a function is easy:

```Lua
local function functionName() <lines of code> end
```

Combining the results is also simple:

```Lua
return function1() and function2()
```

Example to allow a scene to run between 08:00 and 22:30 at weekends:

```Lua
local function checkTime()
   local pStart = "08:00" – Start of time period
   local pEnd = "22:30" – End of time period
   local allow = true – true runs scene during period, false blocks it
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
   local dFirst = 7 – Start day of period (1-7) Sunday = 1
   local dLast = 1 – End day of period (1-7) Sunday = 1
   local allow = true – true runs scene during period, false blocks it
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

--------------------------------------------------------------------------------
# Using Functions
RexBeckett:

I showed in Multiple Conditions 1 how a code chunk could be converted to a function. With small adjustments, the function call can pass the parameters to be used for the test. Now a single piece of code can be used to test several devices. The function code can even be added to your Startup Lua so it may be called from any scene. In the following examples, I changed the original lines that set the parameters to comments (with --) as a reminder of what each parameter means.

Function to check if a temperature is within a range

```Lua
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

Function to check the state of a Z-Wave switch

```Lua
function checkSwitch(dID, allow) local dID = 66
-- Device ID of your Z-Wave Switch 
local allow = true
-- true runs scene if switch is on, false blocks it

local status = luup.variable_get("urn:upnp-org:serviceId:SwitchPower1","Status",dID) 
return ((status == "1") == allow) end
```

We can use these two functions to implement a more-complex set of conditions. This example, suggested by @parkerc, checks the temperature in three different rooms. If any of the temperatures are out of range, checkTemp(device,min,max,true) returns false. In this case we check if the switch controlling our heating is off and, if so, allow the scene to run to turn it on.

```Lua
if checkTemp(123,18,22,true) and checkTemp(124,18,22,true) and checkTemp(125,18,22,true)
then return false

-- All temperatures are in range, don't run scene else return checkSwitch(101,false) -- Run scene if switch is off end
```

Adding functions to Startup Lua

Select the APPS tab, click on Develop Apps and select Edit Startup Lua. Add your functions to the end of the existing code and click GO. Now click on Vera’s Reload button to restart the luup engine. Your functions should now be available to the luup in any scene.

--------------------------------------------------------------------------------
# Multiple Triggers
RexBeckett:

Sometimes we want to have several different events or schedules result in essentially the same action but with different settings. An example would be setting different temperature setpoints or dimmer levels depending and time schedules or any of the conditions in the previous examples. The obvious way to do this is with several scenes but there is another way. We can use a global variable to transport our required settings from Trigger Luup event to the main scene LUUP.

This example is for a scene that sets a Thermostat setpoint based on one of three triggers.

Trigger 1 Luup event - Set to 20 during daytime, do nothing at night.

```Lua
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

Trigger 2 Luup event - Set to 20 during daytime or 10 at night

```Lua
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

Trigger 3 Luup event - Set to 18

setTemp = 18 return true

Main scene LUUP - Version 1

```Lua
local dID = 55 if (setTemp ~= nil) then
   luup.call_action("urn:upnp-org:serviceId:TemperatureSetpoint1_Heat","SetCurrentSetpoint",{NewCurrentSetpoint=setTemp},dID)
   setTemp = nil
end
```

Main scene LUUP - Version 2

```Lua
local dID = 55 if (setTemp == nil) then
   setTemp = 20
end 
luup.call_action("urn:upnp-org:serviceId:TemperatureSetpoint1_Heat","SetCurrentSetpoint",{NewCurrentSetpoint=setTemp},dID)
setTemp = nil
```

Version 1 of the Main scene LUUP will do nothing if run from the UI because, if a trigger did not occur, setTemp will be nil. Version 2, if run from the UI, will set the temperature setpoint to 20.
16 days later

--------------------------------------------------------------------------------
# Time Window
RexBeckett:

There were two requests for this type of scene condition in the last 24 hours so it must be worth posting here. The objective is to allow or prevent a scene executing when triggers occur in a short time period (window).

All these examples save a timestamp each time they are run using a global variable tLastOnD99. On each run, the saved timestamp is subtracted from the current time and compared to the specified time window (twSecs). To allow this code to be used for more than one triggering device, the 99 in tLastOnD99 should be replaced with the device ID of the triggering device. This is simply to avoid undesired side effects so any other name could be used instead.

As with previous examples, the code may be placed in either a Trigger’s Luup event to allow/prevent that trigger or on the scene’s LUUP tab where it will affect all triggers and manual operation.

Allow the scene to run when a trigger occurs within twSecs of the last one (e.g. on/off/on):

```Lua
local twSecs = 5 -- Number of seconds in time window
local tNow = os.time()
local tLastOn = tLastOnD99 or 0 tLastOnD99 = tNow
return ((tNow - tLastOn) <= twSecs)
```

Allow the scene to run provided a trigger occurs at least twSecs after the last one:

```Lua
local twSecs = 5 -- Number of seconds in time window
local tNow = os.time()
local tLastOn = tLastOnD99 or 0 tLastOnD99 = tNow
return ((tNow - tLastOn) >= twSecs)
```

Generic version:
```Lua
local twSecs = 5 -- Number of seconds in time window
local allow = true -- true runs scene during time window, false blocks it
local tNow = os.time()
local tLastOn = tLastOnD99 or 0 tLastOnD99 = tNow
return (((tNow - tLastOn) <= twSecs) == allow)
```

--------------------------------------------------------------------------------
# Service IDs, Variables and Actions
RexBeckett:

One of the most frequent problems we see in requests for help with Lua code, is errors in Service IDs and variable or parameter names in luup function calls. These errors can be tricky to find. Even one character in the wrong case will prevent the call working correctly: A luup.variable_get(…) will return a nil instead of the expected value; A luup.call_action(…) may do absolutely nothing.

Listed below are example luup calls for the most common device types. All examples assume that local variable dID contains the ID of the device being addressed. e.g. local dID = 123 Alternatively, replace dID in the calls with the required device ID number. Examples are given for reading a device variable using luup.variable_get(…) and for initiating an action with luup.call_action(…). It is also possible to directly set device variables but, in many cases, this will have little or no apparent effect on the device. Generally it is better to use luup.call_action(…) rather than directly setting device variables.

The examples are also in the attached pdf file. If you drop it on your desktop, you can quickly open it to copy and paste the calls into your Lua. If the device you want to address is not listed, you can find the Service ID by hovering the mouse cursor over the name of the variable on the device’s Advanced tab.

On/Off Switch
Set Target, read Status. "0" = Off, "1" = On.

```Lua
local status = luup.variable_get("urn:upnp-org:serviceId:SwitchPower1", "Status", dID)

luup.call_action("urn:upnp-org:serviceId:SwitchPower1", "SetTarget", {newTargetValue = "1"}, dID)
```

Virtual Switch
Set Target, read Status. "0" = Off, "1" = On.

```Lua
local status = luup.variable_get("urn:upnp-org:serviceId:VSwitch1", "Status", dID)

luup.call_action("urn:upnp-org:serviceId:VSwitch1", "SetTarget", {newTargetValue = "1"}, dID)
```

Dimmable Light
Set LoadLevelTarget, read LoadLevelStatus. "0" = Off, "100" = Full.

```Lua
local level = luup.variable_get("urn:upnp-org:serviceId:Dimming1", "LoadLevelStatus", dID)

luup.call_action("urn:upnp-org:serviceId:Dimming1", "SetLoadLevelTarget", {newLoadlevelTarget = "50"}, dID)
```

Thermostat
Set ModeTarget, read ModeStatus. "Off", "HeatOn", "CoolOn", "AutoChangeOver".

```Lua
local mode = luup.variable_get("urn:upnp-org:serviceId:HVAC_UserOperatingMode1", "ModeStatus", dID)

luup.call_action("urn:upnp-org:serviceId:HVAC_UserOperatingMode1", "SetModeTarget", {NewModeTarget = "Off"}, dID)
```

Set and read CurrentSetpoint. (Degrees)

```Lua
local setpoint = luup.variable_get("urn:upnp-org:serviceId:TemperatureSetpoint1_Heat", "CurrentSetpoint", dID)

luup.call_action("urn:upnp-org:serviceId:TemperatureSetpoint1_Heat", "SetCurrentSetpoint", {NewCurrentSetpoint = "25"}, dID)

local setpoint = luup.variable_get("urn:upnp-org:serviceId:TemperatureSetpoint1_Cool", "CurrentSetpoint", dID)

luup.call_action("urn:upnp-org:serviceId:TemperatureSetpoint1_Cool", "SetCurrentSetpoint", {NewCurrentSetpoint = "30"}, dID)
```

Temperature Sensor

```Lua
local temp = luup.variable_get("urn:upnp-org:serviceId:TemperatureSensor1", "CurrentTemperature", dID)
```

Generic Sensor

```Lua
local level = luup.variable_get("urn:micasaverde-com:serviceId:GenericSensor1", "CurrentLevel", dID)
```

Light Sensor

```Lua
local level = luup.variable_get("urn:micasaverde-com:serviceId:LightSensor1", "CurrentLevel", dID)
```

Humidity Sensor

```Lua
local level = luup.variable_get("urn:micasaverde-com:serviceId:HumiditySensor1", "CurrentLevel", dID)
```

Security Sensor

```Lua

local tripped = luup.variable_get("urn:micasaverde-com:serviceId:SecuritySensor1", "Tripped", dID)

local armed = luup.variable_get("urn:micasaverde-com:serviceId:SecuritySensor1", "Armed", dID)

local lasttrip = luup.variable_get("urn:micasaverde-com:serviceId:SecuritySensor1", "LastTrip", dID)

luup.call_action("urn:micasaverde-com:serviceId:SecuritySensor1", "SetArmed", {newArmedValue = "1"}, dID)

```

Window Covering
Use Dimmable Light Status variable for current position. Down = 0, Up = 100.

```Lua
local level = luup.variable_get("urn:upnp-org:serviceId:Dimming1", "LoadLevelStatus", dID)

luup.call_action("urn:upnp-org:serviceId:WindowCovering1", "Up", {}, dID)

luup.call_action("urn:upnp-org:serviceId:WindowCovering1", "Down", {}, dID)

luup.call_action("urn:upnp-org:serviceId:WindowCovering1", "Stop", {}, dID)
```

Variable Container
Set and read VariableN where N is 1 to 5.

```Lua
local vcvar1 = luup.variable_get("urn:upnp-org:serviceId:VContainer1", "Variable1", dID)

luup.variable_set("urn:upnp-org:serviceId:VContainer1", "Variable1",newvalue, dID)

luup.variable_set("urn:upnp-org:serviceId:VContainer1", "Variable5","newtext", dID)
```

DayTime plugin
Set and read Status. "0" = Night, "1" = Day.

```Lua
local itsday = luup.variable_get("urn:rts-services-com:serviceId:DayTime", "Status", dID)

luup.variable_set("urn:rts-services-com:serviceId:DayTime", "Status", "1", dID)
```

--------------------------------------------------------------------------------
# Delayed Actions
RexBeckett:

Another favourite forum question concerns using delays in scene Lua. It is tempting to do this with luup.sleep(milliseconds) but this can lead to problems. While luup.sleep(…) is executing, other Vera events can be blocked. A peak of activity at this point can cause a Vera restart. This is one possible cause of unexplained restarts.

Luup provides a more-robust way of achieving delays with luup.call_delay("callfunction",secs,"parmstring"). This will call the function named callfunction after a delay of secs seconds and pass it the optional parameter parmstring. The called function must be global (i.e. not local). After the luup.call_delay(…), the main code continues to execute and may terminate. Multiple luup.call_delay(…) calls may be made to the same or different called functions.

For example, the following code turns on a light and then turns it off after ten seconds. You could, of course, do this simply with scene action delays but it’s a stepping stone.

```Lua
luup.call_action("urn:upnp-org:serviceId:SwitchPower1","SetTarget",{ newTargetValue=1 },66)
luup.call_delay("delayOff",10)
-- Main code terminates at this point

-- Function to be called after delay
function delayOff()
   luup.call_action("urn:upnp-org:serviceId:SwitchPower1","SetTarget",{ newTargetValue=0},66)
end
```

Because the called function is global, it does not have access to any of the local variables in the main part of your code. You could make your variables global but be careful how you name them to avoid side effects with other code. A cleaner approach is to pass them via the optional parameter. This parameter is a string so you will need to convert numeric data for some purposes.

In this example, the device ID of the switch is passed as the parameter.

```Lua
local dID = 66
luup.call_action("urn:upnp-org:serviceId:SwitchPower1","SetTarget",{ newTargetValue=1 },dID)
luup.call_delay("delayOff",10,dID)

function delayOff(dev)
   local devno = tonumber(dev)
   luup.call_action("urn:upnp-org:serviceId:SwitchPower1","SetTarget",{ newTargetValue=0},devno)
end
```

The called function can also issue a luup.call_delay(…) to itself. This allows timed sequences to be programmed. The following example will dim a light by 10% every two seconds until it reaches zero.

```Lua
local dID = 99
luup.call_delay("delayDim",2,dID)

function delayDim(dev)
   local devno = tonumber(dev)
   local lls = tonumber((luup.variable_get("urn:upnp-org:serviceId:Dimming1", "LoadLevelStatus", devno)))
   local newlls = lls - 10
   if newlls < 0 then newlls = 0 end
   luup.call_action("urn:upnp-org:serviceId:Dimming1", "SetLoadLevelTarget", {newLoadlevelTarget = newlls}, devno)
   if newlls > 0 then luup.call_delay("delayDim",2,dev) end
end
```

Only one parameter string can be passed to the called function. If you need to pass more than one item, you will need to encode them into the string. Simple numeric items can be passed as a comma-separated list. The next example uses this technique to pass three numeric values. The resulting code will either dim a light down to zero or up to 100% - depending on its level when the scene is run.

```Lua
local dID = 99
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

luup.call_action("urn:upnp-org:serviceId:Dimming1", "SetLoadLevelTarget", {newLoadlevelTarget = newlls}, devno)
if newlls ~= target then luup.call_delay("delayDim",2,parms) end
end
```

Parameters may also be passed as a serialized table. See Passing a Serialized Table 1.

I hope some of these examples will help to point the way to a solution for your particular requirements. If not, I recommend taking a look at the Program Logic Event Generator (PLEG) plugin.

--------------------------------------------------------------------------------
# Delayed Actions - Passing a Serialized Table
RexBeckett:

As explained in an earlier post, the optional parameter passed by luup.call_delay(…) is a single string. If you need to pass several parameters to the called function, a Lua table can be serialized into that string and recreated when the called function executes.

The general-purpose serialize function in the following example will convert a table containing numeric, string, boolean or nested table elements into a single string. The table can be recreated using the Lua loadstring statement.

This example performs the same function as the previous one but a serialized table is used to pass the three numeric values. The values are initially set in the table dparms which is then serialized and sent as the parameter in luup.call_delay(…). The called function uses loadstring to recreate the table as xparms and can then access the individual variables using xparms. notation. The resulting code will either dim a light down to zero or up to 100% - depending on its level when the scene is run.

```Lua

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

--------------------------------------------------------------------------------
# Debugging Lua Code - kwikLog
RexBeckett:

Debugging Lua code in scenes can be very frustrating. When something doesn’t work as you expected, you need to be able to see which parts of your code are being executed and, often, the value of key variables. Vera provides luup.log(…) to help with this but it is not always easy to find your log entries in LuaUPnP.log with all the other activity there.

kwikLog is a simple function that logs your messages, with a time stamp, to a standalone file. The file may be viewed from a browser by entering /kwikLog.txt. A browser page refresh will show you the latest entries.

You can use kwikLog in several ways:

- Add it to your scene Lua (above where you call it from your code)
- Add it before code you run in Test Luup code (Lua)
- Add it to Startup Lua so that you can use it from any scene

If you add it to Startup Lua (APPS → Develop Apps → Edit Startup Lua), remove the word local from the first line.

```Lua
local function kwikLog(message, clear) local socket = require("socket")
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

```Lua
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

```Lua
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

--------------------------------------------------------------------------------
# Running a scene when a variable changes
RexBeckett:

There have been several recent posts on this subject so I think it is worth including here. The issue is that normal scene triggers that include conditions, like temperature is lower than 18, will fire when the temperature drops below 18 but will not fire again unless it rises to or above 18 and then drops below it again.

If you use this type of trigger for a scene that includes Lua code for additional conditions - say a time period - it will not work very well. If the trigger fires outside of the allowed time period, you may not get another trigger during the period when you want to take some action.

One solution is to have your scene triggered by a periodic schedule. Then you can use Lua code from the examples in this thread to decide if you want the actions to be executed.

A better technique is to watch the variable in question and have your scene run when it changes. As above, you can then use Lua code to decide if you want to execute some action(s). Vera provides a function for this very purpose. Here is how you can use it:

```Lua
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

```Lua
– Table entries { "ServiceID", "VariableName", DeviceNo, SceneNo }
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
– Wait for 30 seconds after restart then run setWatch

luup.call_delay("setWatch",30)
```

The last part of this code causes the setWatch function to be run 30 seconds after Vera restarts. This is to allow all devices to have completed their initialization.

--------------------------------------------------------------------------------
# Finding the Correct Service ID
RexBeckett:

First adventures into using Lua code frequently fail due to the use of incorrect Service IDs in luup.variable_get(…), luup.variable_set(…) and luup.call_action(…) calls. A common error is to use the Device-type instead of the Service ID. Tip: A Service ID string will contain the word serviceId and will not contain the words schemas or device.

I listed some of the common ones in Service IDs, Variables and Actions 1. It is also possible to see the Service ID for a device variable by hovering your mouse cursor over the variable name on the device’s Advanced tab. This is OK if you have a good visual memory. :wink:

One way to see the Service ID in a copy/paste-able form is to enter this command in your browser - replacing with the IP address of your Vera (without the <>):

```Lua
http://<veraip>:3480/data_request?id=status&output_format=xml
```

This will list every variable for all of your devices so you will need to scroll down the list until you find the one you want. You can also get the information for a particular device by using this - replacing 123 with the device number:

```Lua
http://<veraip>:3480/data_request?id=status&output_format=xml&DeviceNum=123
```

To see all the actions available for your devices, enter this:

```Lua
http://<veraip>:3480/data_request?id=invoke
```

or, for a single device:

```Lua
http://<veraip>:3480/data_request?id=invoke&DeviceNum=123
```

Note that not all devices will actually implement all the available actions. The list is a super-set covering all devices of the given Device-type. Theoretically, actions marked with an asterisk (*) should be implemented but this is not always the case. Clicking on an entry in the invoke list will fire the action so be careful!

If you would prefer a less-cluttered list, you could use LuaTest which has buttons to display Variables, Values, Actions and individual device Status.

--------------------------------------------------------------------------------
# Testing Lua Code - LuaTest
RexBeckett:

LuaTest is Lua code designed to  faciltate debugging your own code. See the forum discussion here:

>[LuaTest - A Tool for Testing Scene Lua Code](https://community.ezlo.com/t/luatest-a-tool-for-testing-scene-lua-code/180205)

The documentation is here:

>[LuaTest: Documentation](LuaTest-Documentation)

and the code itself here:

>[LuaTest: code](LuaTest-code)

with some updates here:

>[LuaTest: code 2](cgmartin/RBLuaTest.lua)

@@@@@@@@@@@@@@@@@@@@

--------------------------------------------------------------------------------
kiethr

Not a code expert by any means, but i could use a little help. When a door is triggered a light will come on for 5 minutes during certain times of the day (done by LUUP code) What I want to do is add code to only run the scene if the light being triggered is off. Its kind of a pain if the light was on when the scene ran and the light goes off 5 minutes later when someone else is still working in the room. So here is the code i have been playing with. The first two lines is what i added to the code that was working. the device ID is 32 which is a GE/JASCO single pole wall switch. While we are at it I would like to change the local start time to sunset and leave the end time as something I can configure. Thanks

local switchOnOff = luup.variable_get("urn:upnp-org:serviceId:SwitchPower1", "Status", 32)
if (switchOnOff == "0") then

local startTime = "18:00"
local endTime = "23:59"

local hour = tonumber( startTime:sub( startTime:find("%d+") ) )
local minute = tonumber(startTime:sub(-2))

if hour and minute then
startTime = hour * 100 + minute
else
luup.log("ERROR: invalid start time")
return false
end

hour = tonumber( endTime:sub( endTime:find("%d+") ) )
minute = tonumber(endTime:sub(-2))

if hour and minute then
endTime = hour * 100 + minute
else
luup.log("ERROR: invalid end time")
return false
end

local currentTime = os.date("*t")
currentTime = currentTime.hour * 100 + currentTime.min

luup.log("startTime = " … startTime … "; currentTime = " … currentTime … "; endTime = " … endTime)

if startTime <= endTime then
– Both the start time and the end time are in the same day:
– if the current time is in the given interval, run the scene.
if startTime <= currentTime and currentTime <= endTime then
return true
end
else
– The start time is before midnight, and the end time is after midnight:
– if the current time is not outside the given interval, run the scene.
if not (endTime < currentTime and currentTime < startTime) then
return true
end
end

return false
parkerc

--------------------------------------------------------------------------------
Developers

Hi @kiethr

What are you using to check if someone is still in the room working?

I have a similar set up for my kitchen light but using PLEG, but rather than use the light switch as the trigger I have the EZMotion 3in1 sensor working as an occupancy sensor, so when it’s tripped (between a certain time) it comes on and then turns off after x mins when the sensor reports there’s no longer any motion/occupancy

--------------------------------------------------------------------------------
RexBeckett

Try this, @kiethr:

```Lua
local switchOnOff = luup.variable_get("urn:upnp-org:serviceId:SwitchPower1", "Status", 32)
if (switchOnOff == "1") then return false end

local endTime = "23:59"

local ssTime = luup.sunset() % 86400

local hour = math.floor(ssTime / 3600)
local minute = math.floor((ssTime % 3600) / 60)

local startTime = hour * 100 + minute

hour = tonumber( endTime:sub( endTime:find("%d+") ) )
minute = tonumber(endTime:sub(-2))

if hour and minute then
endTime = hour * 100 + minute
else
luup.log("ERROR: invalid end time")
return false
end

local currentTime = os.date("*t")
currentTime = currentTime.hour * 100 + currentTime.min

luup.log("startTime = " … startTime … "; currentTime = " … currentTime … "; endTime = " … endTime)

if startTime <= endTime then
– Both the start time and the end time are in the same day:
– if the current time is in the given interval, run the scene.
if startTime <= currentTime and currentTime <= endTime then
return true
end
else
– The start time is before midnight, and the end time is after midnight:
– if the current time is not outside the given interval, run the scene.
if not (endTime < currentTime and currentTime < startTime) then
return true
end
end

return false
```

--------------------------------------------------------------------------------
kiethr

Just the status of the light switch, I do not have a motion sensor…yet.

Thank you I will give that a try and report back!
parkerc

--------------------------------------------------------------------------------
Developers

You’ve got me thinking about ways of checking if someone is still in the room.

If the ‘work’ you are referring to involves the use of a networked PC in that room, then you could also look to add the Ping Sensor as a method for telling you if someone is there and working.

If you have a music player in there too. E.g like a Sonos, you could look to see if it is being used (status as Playing) as another way of checking occupancy.

--------------------------------------------------------------------------------
kiethr

No actually i was painting the hallway. The kids kept coming and going and that darn light would shut off after 5 minutes every time they left :slight_smile:

--------------------------------------------------------------------------------
kiethr

@RexBeckett

First thank you for attempting to help me!

The code you supplied did change what time the trigger started to run, so that part is working just fine. However the timer still runs when the light is on. I’m not really sure where the problem is within the code.

Sent from my iPad using Tapatalk HD
parkerc

--------------------------------------------------------------------------------
Developers

    No actually i was painting the hallway.

:slight_smile: Considering the way the Internet of Things (IoT) is going, it’s probably only a matter of time before you’ll have an IP address for our paint brushes 8)

--------------------------------------------------------------------------------
RexBeckett

    The code you supplied did change what time the trigger started to run, so that part is working just fine. However the timer still runs when the light is on. I'm not really sure where the problem is within the code.

When you say [i]the timer still runs[/i], do you mean that the scene actions are still being executed even if the light is on? Do you have an instant action to turn the light on and a delayed one to turn it off?

Where did you place the Lua code - in the scene Luup tab or in the trigger Luup event? Is there just one trigger for the scene? Will you re-post your current code in case a typo crept in?

--------------------------------------------------------------------------------
kiethr

I created the scene in the following way: I have Aeon labs hidden door sensor on the front door. When the sensor is triggered (from sunset until 2359) it triggers the hallway light (controlled by a GE/JASCO wall switch) to come on. Then I set a delay to turn the hallway light off 5 minutes later. I entered the code contained below in the scene under the LUUP tab.

local minute = math.floor((ssTime % 3600) / 60)

local startTime = hour * 100 + minute

hour = tonumber( endTime:sub( endTime:find("%d+") ) )
minute = tonumber(endTime:sub(-2))

if hour and minute then
endTime = hour * 100 + minute
else
luup.log("ERROR: invalid end time")
return false
end

local currentTime = os.date("*t")
currentTime = currentTime.hour * 100 + currentTime.min

luup.log("startTime = " … startTime … "; currentTime = " … currentTime … "; endTime = " … endTime)

if startTime <= endTime then
– Both the start time and the end time are in the same day:
– if the current time is in the given interval, run the scene.
if startTime <= currentTime and currentTime <= endTime then
return true
end
else
– The start time is before midnight, and the end time is after midnight:
– if the current time is not outside the given interval, run the scene.
if not (endTime < currentTime and currentTime < startTime) then
return true
end
end

return false

--------------------------------------------------------------------------------
RexBeckett

@kiethr, that isn’t the code I posted. The first part is missing. When you change the code, make sure you click the Save lua button at the bottom before clicking Confirm changes.

--------------------------------------------------------------------------------
kiethr

@RexBeckett

Thanks, I would be lying if I said that has never happened to me before. The SAVE button is off my laptop screen and I do forget to click that sucker every now and then!! I’ll throw the code in and give it a try when i get home.

Also i stumbled across your PLEG Basics manual, its good reading. Thanks for putting it together!

Sent from my iPad

--------------------------------------------------------------------------------
kiethr

Hello @RexBeckett

I did add the updated code to my veralite. The light that would normally trigger will not trigger. If the light that is being triggered is already on when triggered it doesn’t go off after the delay in the scene is reached.

Any thoughts?

Sent from my iPad using Tapatalk HD



--------------------------------------------------------------------------------
RexBeckett

    It a straight up GE/JASCO single pole on/off zwave switch

Will you copy the code from your scene’s Luup tab and post it?

--------------------------------------------------------------------------------
RexBeckett

@keithr, try this code and post the section from LuaUPnP.log where you test it.

```Lua
local switchOnOff = luup.variable_get("urn:upnp-org:serviceId:SwitchPower1", "Status", 32)
if (switchOnOff == "1") then
luup.log("Scene Debug: Light is already on.")
return false
end
luup.log("Scene Debug: Light is off.")
local endTime = "23:59"

local ssTime = luup.sunset() % 86400

local hour = math.floor(ssTime / 3600)
local minute = math.floor((ssTime % 3600) / 60)

local startTime = hour * 100 + minute

hour = tonumber( endTime:sub( endTime:find("%d+") ) )
minute = tonumber(endTime:sub(-2))

if hour and minute then
endTime = hour * 100 + minute
else
luup.log("ERROR: invalid end time")
return false
end

local currentTime = os.date("*t")
currentTime = currentTime.hour * 100 + currentTime.min

luup.log("Scene Debug: startTime = " … startTime … "; currentTime = " … currentTime … "; endTime = " … endTime)

if startTime <= endTime then
– Both the start time and the end time are in the same day:
– if the current time is in the given interval, run the scene.
if startTime <= currentTime and currentTime <= endTime then
return true
end
else
– The start time is before midnight, and the end time is after midnight:
– if the current time is not outside the given interval, run the scene.
if not (endTime < currentTime and currentTime < startTime) then
return true
end
end
luup.log("Scene Debug: Skipping actions.")
return false
```

See Logs Explained.

--------------------------------------------------------------------------------
kiethr

@RexBeckett’

Attached is the log, I think it captured it correctly. The first part of the log is with the triggered light off when the door sensor is tripped. The second part is with the triggered light on when the door sensor is tripped.

Thank you again for taking time to help me with this.

--------------------------------------------------------------------------------
RexBeckett

@kiethr, the code is working correctly but Vera thinks sunset in your parts is at 23:24…

Check SETUP → Location → Timezone and Currentcity

--------------------------------------------------------------------------------
kiethr

its just like I set it up, time zone is correct, city is correct. Screen shot attached

--------------------------------------------------------------------------------
kiethr

I changed the time zone to mexico, when my veralite restarted i confirmed it was showing the correct time in mexico. I then reset it back to where it should be, reloaded the code you supplied and we will see what happens this evening.

@RexBeckett If you have any other suggestions please let me know.

--------------------------------------------------------------------------------
RexBeckett

@kiethr, paste the following code into APPS → Develop Apps → Test Luup code (Lua) then click GO. Check the log and see if it is all correct:

luup.log("Timezone: " .. luup.timezone .. " hours") luup.log("City: " .. luup.city) luup.log("Latitude: " .. luup.latitude) luup.log("Longitude: " .. luup.longitude) local srtime = os.date("%c",luup.sunrise()) luup.log("Sunrise: " .. srtime) local sstime = os.date("%c",luup.sunset()) luup.log("Sunset: " .. sstime)

If your Timezone, City, Latitude and Longitude are all correct but Sunrise and Sunset are still wrong, I have no idea what’s broken - raise an MCV Support ticket.

--------------------------------------------------------------------------------
kiethr

Sure did I even entered the lat and Lon into Google and it pointed to the city I live in


--------------------------------------------------------------------------------
kiethr

One last thought before i submit a help ticket. I have the day/night app installed and i use it to turn on and off outdoor lights. It works fine. Doesn’t the sunset/sunrise information that app uses come from the same place the code here gets its information?

Sent from my iPad using Tapatalk HD

--------------------------------------------------------------------------------
RexBeckett

    One last thought before i submit a help ticket. I have the day/night app installed and i use it to turn on and off outdoor lights. It works fine. Doesn’t the sunset/sunrise information that app uses come from the same place the code here gets its information?

Day/Night uses Vera’s luup.call_timer(…) function to get callbacks for sunrise and sunset. The times should be the same as those returned by luup.sunrise() and luup.sunset() but, on your system, are apparently not.

--------------------------------------------------------------------------------
kiethr

Okay, thank you. I did submit the ticket and will report back when I get a response.

--------------------------------------------------------------------------------
rmohsen

Here is my scenario

I have a smart switch that turn on the living room lights if a motion is detected and turn it off after 1 minute. But whenever I’m watching tv and someone walks in the light goes on. I want to convert the smart switch to a PLEG action ( which I can easily do ) but I want this scene to only run if XBMC is not playing. I have the XBMC addon installed. Any help with that ?

--------------------------------------------------------------------------------
RexBeckett

[quote="Ramiii, post:57, topic:178331"]Here is my scenario

I have a smart switch that turn on the living room lights if a motion is detected and turn it off after 1 minute. But whenever I’m watching tv and someone walks in the light goes on. I want to convert the smart switch to a PLEG action ( which I can easily do ) but I want this scene to only run if XBMC is not playing. I have the XBMC addon installed. Any help with that ?[/quote]
@Ramiii, would you mind posting this question on the Program Logic Plugins board? Thanks.

--------------------------------------------------------------------------------
rmohsen

@RexBeckett no problem. But I don’t want this to be done using PLEG. I mean the condition. I’ll just use PLEG for the lights and Luup code for the condition , that’s why I posted here

--------------------------------------------------------------------------------
RexBeckett

    @RexBeckett no problem. But I don’t want this to be done using PLEG. I mean the condition. I’ll just use PLEG for the lights and Luup code for the condition , that’s why I posted here

Well you got me confused. Smartswitch, PLEG, XMBC…

If your problem is with determining if XMBC is playing, I cannot help you as I don’t use it. You may have more success posting your question on the XMBC plugin thread so another user can assist you.

--------------------------------------------------------------------------------
RexBeckett

    Trying to adapt this for a multiswitch. Any thoughts?

You need to change the Service ID to the one used by MultiSwitch:

local dID = 175 -- Device ID of your MultiSwitch local allow = true -- true runs scene if switch is on, false blocks it local status = luup.variable_get("urn:dcineco-com:serviceId:MSwitch1","Status2",175) return ((status2 == "1") == allow)

--------------------------------------------------------------------------------
kiethr

@RexBeckett

Here is what tech support had to say:

I checked your unit and it seems that the timezone for your location is setup properly. Please note that the timezone you are able to see in the logs is in UTC format, what?s why you are seeing there 23:24, but after applying your timezone, which is GMT-4, it is 19:24.
If you know for sure that the luup code is written properly, you can also try to use luup.is_night() instead of luup.sunset() and let me know how it goes.

so i am going to change that part of the code and see what happens.

Sent from my iPad using Tapatalk HD

--------------------------------------------------------------------------------
Minnies

[quote="RexBeckett, post:68, topic:178331"]

    Trying to adapt this for a multiswitch. Any thoughts?

You need to change the Service ID to the one used by MultiSwitch:

local dID = 175 -- Device ID of your MultiSwitch local allow = true -- true runs scene if switch is on, false blocks it local status = luup.variable_get("urn:dcineco-com:serviceId:MSwitch1","Status2",175) return ((status2 == "1") == allow) [/quote]

Thanks but still no success when trying it in the test Luup code area.

Here is the data from the switch file:

Name: MultiSwitch
Device_type: urn:schemas-dcineco-com:device:MSwitch175:1
device_file: D_MSwitch175.xml
id: 175
Variables:
Status1
Status2
Status3
Status4
Status5
Status6
Status7
Status8

--------------------------------------------------------------------------------
RexBeckett

I missed spotting your typo in the last line. You are reading the value of Status2 into variable status but using variable status2 in the return statement which does not exist so will return nil. Try this:

local dID = 175 -- Device ID of your MultiSwitch local allow = true -- true runs scene if switch is on, false blocks it local status = luup.variable_get("urn:dcineco-com:serviceId:MSwitch1","Status2",dID) return ((status == "1") == allow)

Edit: I have added the code for MultiSwitch to the examples for Z-Wave and Virtual switches.
16 days later

--------------------------------------------------------------------------------
quinn

I am in the process of committing to using Vera3 for a $2M+ house. I have been uncertain whether to proceed being doubtful whether certain functionality was possible because of Mios’s poor, disorganized and inconsistent documentation, until I found RexBeckett. His examples are clear and sufficiently detailed to answer most of my concerns. After reading RexBeckett’s work, which skillfully and completely illuminates LUUP plugins even for a na?ve user, LUUP itself seems to be a fairly straight forward and easily mastered language, mostly a C derivative I would say.

For example, in the case of the Kicklog function he explains "Enter /kwikLog.txt in a browser window to see if your location data is correct. Replace with the IP address of your Vera (without the <>)". Notice the careful disambiguating of what "" is in his syntax. Mios should really be paying Rex.

None of this is to suggest that online support is not good. They are responsive and generally knowledgeable, but one question at a time to them is a very resource consuming approach–on both sides.

Incidentally, I wrote my first program in 1963, I know half a dozen programming languages, including the infamous SAS macro language. The first program you write in any language is always the toughest. That is why one or two careful and complete examples in the main documentation would have been most helpful, but a single reference to RexBeckett’s forum page would have been much more.

--------------------------------------------------------------------------------
RexBeckett

Many thanks for the feedback @quinn. I’m glad this thread has helped you to get started with Lua.

--------------------------------------------------------------------------------
ilikelife

Hi @RexBeckett,
Since you pointed me here, I hope it’s ok to ask questions here about delayed actions. If not, please just point me in the right direction.
I think I understand that a function is global, so I need to either define variables within the function, or be very careful with naming global vars. I also read somewhere it’s not wise to question your code, but I’m missing something in the following:

```Lua
local dID = 66
luup.call_action("urn:upnp-org:serviceId:SwitchPower1","SetTarget",{ newTargetValue=1 },dID)
luup.call_delay("delayOff",10,dID)

function delayOff(dev)
local devno = tonumber(dev)
luup.call_action("urn:upnp-org:serviceId:SwitchPower1","SetTarget",{ newTargetValue=0},devno)
end
```

What is (dev) ? The 66 started as dID, and within the function, it’s devno, but I’m lost with dev. I hope I caught you in a typo, but I’m not betting on it.

I’ve written a solution for keeping a light on for some period past the "no motion" signal, if it’s re-tripped during the delay period that seems to work well. Right now I’m using the brute force method (mostly not using variables) but am passing the DELAY as the single variable until I figure out your encoding them into a string or table.

To turn the light on, I have an empty scene with a trigger on sensor 45 TRIPPED and this in the scene LUUP

```Lua
local SENSOR = 45 – The door/window sensor device number
local Garage = 16 – the Garage Light device number
local SES_SID = "urn:micasaverde-com:serviceId:SecuritySensor1"
local SWITCH_SID = "urn:upnp-org:serviceId:SwitchPower1"

local status = luup.variable_get(SWITCH_SID,"Status",Garage)

if (status == "0") then -- Don't keep turning it on if it's already on.
luup.call_action(SWITCH_SID,"SetTarget",{newTargetValue="1" },Garage)

end
```

Then to turn it off, another empty scene with a trigger for sensor 45 NOT TRIPPED, and this in the scene LUUP.

```Lua
local SENSOR = 45 – The door/window sensor device number
local Garage = 16 – the Garage Light device number
local DELAY = 15 – Seconds for testing with a door sensor
local SES_SID = "urn:micasaverde-com:serviceId:SecuritySensor1"
local SWP1_SID = "urn:upnp-org:serviceId:SwitchPower1"

luup.call_delay( "checkLastTrip", DELAY, DELAY)

– Turn off the light only when it’s not tripped and the DELAY time is expired.
function checkLastTrip()
local DelayTime = tonumber(DELAY)
local tripped = luup.variable_get( "urn:micasaverde-com:serviceId:SecuritySensor1", "Tripped", 45) or "0"
local lastTrip = luup.variable_get( "urn:micasaverde-com:serviceId:SecuritySensor1", "LastTrip", 45) or os.time()
if (tripped == "0" and (os.time() - lastTrip >= DelayTime)) then
luup.call_action("urn:upnp-org:serviceId:SwitchPower1","SetTarget",{newTargetValue="0" },16)
end
end
```

It works, and doesn’t seem to tie up the system with loops that don’t end, but it just doesn’t seem good form to use empty scenes, but I can’t make it work any other way. Any suggestions or warnings or improvements that you can offer would be much appreciated.

Mike

--------------------------------------------------------------------------------
RexBeckett

    I also read somewhere it's not wise to question your code, but I'm missing something in the following:

    What is (dev) ? The 66 started as dID, and within the function, it's devno, but I'm lost with dev. I hope I caught you in a typo, but I'm not betting on it...

You're being very cautious, @ilikelife, but I don't bite. ;D

The third parameter in luup.call_delay(callback, delay, parameter) is a string that is passed to the callback function when the delay expires. I simply use this parameter to pass the device ID that should be turned off. I called the argument dev in the definition of the callback function delayOff. Remember that it must be a string so I converted the string in dev to a number in devno. See: no magic, no typos…

it just doesn't seem good form to use empty scenes

Empty scenes don't really cost anything but if it bothers you, use a single scene and put your code on individual triggers.

    It works...

I'm surprised if it continues to work reliably because there are a few problems with it:

Your global checkLastTrip function is referring to the local variable DELAY. This may not exist if it has been garbage-collected by the time the callback happens. Either define it inside checkLastTrip or pass it as the string parameter. When you pass a string through luup.call_delay(…), the callback function needs to define the receiving arguments as in my example.

The values returned by luup.variable_get(…) are strings. To avoid problems, use tonumber(…) to convert them before using them in calculations or comparisons

--------------------------------------------------------------------------------
ilikelife

@RexBeckett

You probably don’t remember, but in another thread you were walking someone through solving his problem, and he questioned your code. Your response was was a feigned indignant reply something like "Now you’re questioning MY code?" I enjoyed that exchange, but was not seriously worried


luup.call_delay("delayOff",10,dID)

    function delayOff(dev)

So the device number (66) declared as dID gets passed as the 3rd variable. But does it then get named "dev" because it’s in the one variable position in the function named?

I see the conversion to numeric devno, that gets used in the CALL_ACTION line.

I was attempting to both use and pass the value in DELAY with my

    luup.call_delay( "checkLastTrip", DELAY, DELAY)
    
But think I missed bringing it in.

    function checkLastTrip()

Should have been

    function checkLastTrip(NameGoesHere)

Followed by

    local DelayTime = tonumber(NameGoesHere)

Is that correct?

And, yes, you are correct that it blew up shortly after I posted.

Do I need to keep my function names unique (checkLastTripSensor45) ? And if the functions are unique, do I have a problem with the local tripped and lastTripped within the functions if when I have this delay running on different sensors at once?

Maybe that’s why everyone else uses plugins, huh?

--------------------------------------------------------------------------------
RexBeckett

    You probably don't remember...
    

I had already surmised that this was the reason for your comment. Chris and I have many exchanges...

    So the device number (66) declared as dID gets passed as the 3rd variable. But does it then get named "dev" because it's in the one variable position in the function named?

Exactly.

    Should have been Quote function checkLastTrip(NameGoesHere) Followed by Quote local DelayTime = tonumber(NameGoesHere) Is that correct?

Precisely.

    Do I need to keep my function names unique (checkLastTripSensor45) ? And if the functions are unique, do I have a problem with the local tripped and lastTripped within the functions if when I have this delay running on different sensors at once?

As all global functions and variables in your scene code share the same environment along with Startup Lua, you can get contention. Using unique names for global items will avoid this. It is OK to have multiple copies of the same (identical) function defined. The last one loaded gets to live. Problems occur when functions share the same name but are not identical...

Variables defined as local within a function get created when it is called. Each caller gets its own copy of the variables.

    Maybe that's why everyone else uses plugins, huh?

Plugins are great if there is one that does exactly what you want and you can afford the memory load. Plugins like PLEG can handle a massive amount of logic in one instance, of course. PLEG can now do just about anything a scene can do but I still like to use a few lines of Lua for simple tasks.

--------------------------------------------------------------------------------
ilikelife

@RexBeckett

Thank you so much for taking the time and having the patience to help me with this. My first (new) question is: other than bothering you each time I run into problems, is there a good reference book that covers the Vera/LUUP part of this? I’ve got the Programming in Lua book (3rd edition) from Lua.org and it helps. (It might help even more if I would spend some time studying it, instead of plowing forward with my live system, but this is more fun.) ;D I’ve used the Wiki pages, and this forum is very helpful, but is there a reference to the device and action commands in LUUP that are not covered in Lua? BTW, am I correct in saying the "LUUP" part is specific and proprietary to Vera / Mi Casa Verde?

The "new and improved" code seems to work without contention. It may be overkill, but I’m renaming everything, because I’m still not clear on what’s global and local. I’m only really using DELAY46, but I left the other variables for my reference.

```Lua
local Motion46 = 46 – The kitchen motion sensor device number
local KitchenLights = 10 – the Kitchen Lights device number
local DELAY46 = 1200 – Seconds = 20 minutes
local SES_SID = "urn:micasaverde-com:serviceId:SecuritySensor1"
local SWP1_SID = "urn:upnp-org:serviceId:SwitchPower1"

luup.call_delay( "checkLastTrip46", DELAY46, DELAY46)

– Turn off the light only when it’s not tripped and the DELAY time is expired.
function checkLastTrip46(DELAY46)
local DelayTime46 = tonumber(DELAY46)
local tripped = luup.variable_get( "urn:micasaverde-com:serviceId:SecuritySensor1", "Tripped", 46) or "0"
local lastTrip = luup.variable_get( "urn:micasaverde-com:serviceId:SecuritySensor1", "LastTrip", 46) or os.time()
if (tripped == "0" and (os.time() - lastTrip >= DelayTime46)) then
luup.call_action("urn:upnp-org:serviceId:SwitchPower1","SetTarget",{newTargetValue="0" },10)
end
end
```

The last one loaded gets to live.

Say I have the kitchen and garage lights on Schlage RS200 motion sensors with a fixed 4 minute "On time". I want the kitchen lights to stay on for 20 minutes past the last detected motion. With the garage lights, 6 minutes seems about right.
If both scenes call checkLastTrip but 1 has a DelayTime of 20 minutes, and the other of 6, what happens when the garage scene fires while the kitchen has been running for a few minutes?

Again, thanks for helping me. I feel dense with this sometimes, but it keeps me out of the pool hall.

Mike

--------------------------------------------------------------------------------
RexBeckett

    ...is there a reference to the device and action commands in LUUP that are not covered in Lua?

The only reference that I know is [url=http://wiki.micasaverde.com/index.php/Luup_Lua_extensions]Luup Extensions[/url]. It isn't big on explanation so you must be prepared to experiment a little. LuaTest is your friend. ;)

    BTW, am I correct in saying the "LUUP" part is specific and proprietary to Vera / Mi Casa Verde?

Yes. luup is a library of functions and variables that is predefined in Vera's Lua environment. It does not exist in normal (non-Vera) Lua environments. This is why there is no detailed documentation out there.

    I'm still not clear on what's global and local.

You might understand it a little more if you read the first couple of chapters of your book. ::) I'll give you a clue, though: if the definition does not start with [b]local[/b], it isn't! This also means that if you don't define a variable before you stick something in it, it is intrinsically global... You really need to understand variable [i]scope[/i] before this makes sense. Hit that book. :D

    If both scenes call checkLastTrip but 1 has a DelayTime of 20 minutes, and the other of 6, what happens when the garage scene fires while the kitchen has been running for a few minutes?

Right now your [b]checkLastTrip[/b] function is hard-coded for a specific light so you would not want to call it from both scenes. You can have a single function that works for any device provided you pass the device number as part of the string argument. I show a couple of ways to do this in [url=http://forum.micasaverde.com/index.php/topic,18679.msg154756.html#msg154756]Delayed Actions[/url]. If you write a general-purpose function, you could place it in [b]Startup Lua[/b] and call it from any scene.

--------------------------------------------------------------------------------
ilikelife

Hit the books, huh? :cry: Like I haven’t heard that before. But just one more before homework. I get that my checkLastTrip is hard coded now. I saw your instructions on passing multiple vars, and I’ll try that soon. What I’m asking is, IF I pass device, delay and other variables to a function named checkLastTrip, there could be multiple instances of that function with different (local) vars like tripped, lastTripped and DelayTime. When you said

    The last one loaded gets to live.

Does that mean my other light stays on?

Right after typing that, I think I heard someone saying

    You really need to understand variable scope before this makes sense. Hit that book. 

I’ll post again in a week or two. :). Thanks again.

--------------------------------------------------------------------------------
RexBeckett

When I said

    The last one loaded gets to live.

I was referring to the case when several scenes contain a global function with the same name but different code. The last scene to run overwrites the function definition. Any pending delayed callbacks can end-up using the wrong version. It’s bad - don’t do it. :wink:

When a function uses local variables, each caller gets a fresh set on the stack so is isolated from other users. The combination of local variables and passed parameters allows general-purpose functions to be shared.

    I'll post again in a week or two. :).

You don't have to read the whole book in one go. You will find the first twenty pages enlightening and, after fifty pages, it will all make much more sense.

--------------------------------------------------------------------------------
ilikelife

Well, this makes me believe I’m on the right track. The checkLastTrip function can be used with different combinations of sensors, switches and delay (if I learn to pass variables) with just a few lines of code. This seems like it will minimize system resources used, and let me "tweak" as needed.

I was only planning on a couple of chapters when I said a week or two. Ok, I’m not always that slow but life tends to interfere with my hobbies at times. 8)

--------------------------------------------------------------------------------
Cslagle

RexBeckett, thank you for your work on this. Due to your efforts I am jumping into the Lua language with both feet. I’m looking forward to understanding and exploring the whole home automation with Vera capabilities better. I really ( and truly) appreciate your writing in complete and coherent sentences. So many posters don’t do that. In other words, it’s all your fault. When she complains, I’ll give my wife your number. Of course I don’t have your number but she doesn’t know that. Thanks again.
RexBeckett
Beta
May '14

    When she complains, I'll give my wife your number.

I’m only an Engineer. I do alright with electronics, mechanics and software but I can’t fix wives. I’m still waiting for somebody to write a manual. ;D

--------------------------------------------------------------------------------
Donna

This is EXACTLY what I need, BUT I have NO IDEA how to IMPLEMENT your instructions. I see I am supposed to put in the lump code but I don’t know how or where to do that. PLEASE put up step by step instructions: Go to the dashboard, click on Automation, etc.

Thank you so much!!!

--------------------------------------------------------------------------------
RexBeckett

[quote="Donna, post:85, topic:178331"]This is EXACTLY what I need, BUT I have NO IDEA how to IMPLEMENT your instructions. I see I am supposed to put in the lump code but I don’t know how or where to do that. PLEASE put up step by step instructions: Go to the dashboard, click on Automation, etc.

Thank you so much!!![/quote]

Here is a set of instructions for Creating Scenes. You should copy/paste the Lua code onto the scene’s LUUP tab. Click Save Lua, then Confirm changes to save the scene. You will need to Save/Reload Vera before your new or edited scenes can be used.

--------------------------------------------------------------------------------
henryiii

I tried the virtual switch. From you post, I took it as:

    I open an new scene.
    go to the LUUP tab
    paste the code copied from your virtual switch post
    change the ID to the one I want in the scene
    Save
    test switch
    Operation that should occur. When I activate the scene once, if the device is off, it should turn on. If the device is on, it should turn off. Its an on/off virtual switch. That is what I assumed when I attempted to create this scene. I found the device ID by going to the advance tab, selecting the drop box with the list of all the devices and chose the number to the left of the name of the device.

I have successfully created a schedule for an outlet. It works properly. I’ve also created a scene that turns all devices off.

I’m missing steps or something.

Thank you.

--------------------------------------------------------------------------------
lmet

@RexBeckett
Hi,
I arrived a little late just to congratulate you for this excellent work. I think it has or will help many of us.

--------------------------------------------------------------------------------
cw-kid

Hi

I would like to NOT run a scene if my lamps are already on.

I have a lights on scene that runs at sunset, but if I have already turned on those lights manually I don’t want this scene to run.

I’ve looked at the code examples but not really sure how to do this?

Many thanks

--------------------------------------------------------------------------------
Grwebster

Thanks for all your work on this Rexbeckett! This is exactly what I was looking for. Let’s me make more complex scenes without getting buried in code!

--------------------------------------------------------------------------------
MarkAgain

I have two questions. The first question is, I built an interface that allows me to set values into several Variable Containers with my Pronto remote. I would like to be able to something similar with the Day Or Night plugin. Is it possible to set a day and night offset into the Day Or Night Plugin with Lua? I can set the plugin status with:
luup.call_action("urn:rts-services-com:serviceId:DayTime", SetTarget {newTargetValue="0"}, dID).
I could not find a way to enter either the day or night offset.

If I can’t get the code to accomplish the question above, I would like to get suggestions to accomplish the following. I have a time offset stored in a variable container. I have several scenes that I want to run before or after sunset using on the value stored in a variable container for the offset. Any help would be appreciated. :wink:

Thank you, Mark

--------------------------------------------------------------------------------
RexBeckett

[quote="cw-kid, post:89, topic:178331"]Hi

I would like to NOT run a scene if my lamps are already on.

I have a lights on scene that runs at sunset, but if I have already turned on those lights manually I don’t want this scene to run.

I’ve looked at the code examples but not really sure how to do this?

Many thanks[/quote]

Add to your scene that runs at sunset Lua code that checks the state of the lamps. The appropriate example is that for a Z-Wave switch 1.

--------------------------------------------------------------------------------
RexBeckett

[quote="MarkAgain, post:91, topic:178331"]I have two questions. The first question is, I built an interface that allows me to set values into several Variable Containers with my Pronto remote. I would like to be able to something similar with the Day Or Night plugin. Is it possible to set a day and night offset into the Day Or Night Plugin with Lua? I can set the plugin status with:
luup.call_action("urn:rts-services-com:serviceId:DayTime", SetTarget {newTargetValue="0"}, dID).
I could not find a way to enter either the day or night offset.

If I can’t get the code to accomplish the question above, I would like to get suggestions to accomplish the following. I have a time offset stored in a variable container. I have several scenes that I want to run before or after sunset using on the value stored in a variable container for the offset. Any help would be appreciated. :wink:

Thank you, Mark[/quote]

The DayTime plugin does not support actions for changing the offsets from Lua.

The most reliable alternative would be to have a scene that runs, say, every five minutes. In the scene’s Luup, get the time of the next sunset using luup.sunset() and compare that with the current time (os.time()) and the value of your saved offset. Be aware that after sunset, the time returned by luup.sunset() will be for the next day so you will need to deal with that. You will also need to bypass the code once it has detected sunset until the following day.

I suggest the periodic scene because it will survive a Vera restart without the need for special restart processing.

--------------------------------------------------------------------------------
MarkAgain

Welcome back Rex! Thank you for the reply! ;D

I setup several scenes using the Virtual Clock plugin for the trigger. Took a little while to figure how to set the end time.
Needed to get the current clock start time, calc the seconds to end time, set the duration (seconds) and then re-save the start time.

Thanks again, Mark

--------------------------------------------------------------------------------
RexBeckett

Virtual Clock is a good solution. It does a good job of managing restarts. As you have found out, you need to send the SetAlarmTime action as the last step in order to restart the timer with the new times.
2 months later

--------------------------------------------------------------------------------
conchordian

[quote="RexBeckett, post:3, topic:178331"]local pStart = -15 -- Start of time period, minutes offset from sunset local pEnd = "23:30" -- End of time period local allow = true -- true runs scene during period, false blocks it local mStart = math.floor( (luup.sunset() % 86400) / 60 ) + pStart local hE, mE = string.match(pEnd,"(%d+)%:(%d+)") local mEnd = (hE * 60) + mE local tNow = os.date("*t") local mNow = (tNow.hour * 60) + tNow.min if mEnd >= mStart then return (((mNow >= mStart) and (mNow <= mEnd)) == allow) else return (((mNow >= mStart) or (mNow <= mEnd)) == allow) end[/quote]

I’m trying to get this to work from 15mins before sunset to 11:30PM the same night, but the scene seems to run all the time. I’ve looked at the kwikLog output and everything looks correct for my timezone, etc.
Any ideas what’s going on, please?

Cheers

50 10/09/14 14:32:10.447 luup_log:0: Scene Debug: mStart= 457;mNow= 872; mEnd = 1410 <0x31df3680>

The example doesn’t seem to work, but this seems to work, so far…

```Lua
local currentTime = os.date("%H%M") + 0
local sunset = os.date("%H%M",luup.sunset()) + 0
local endTime = "2330" + 0
Offset = "-15" + 0
sunsetOffset = sunset + Offset

luup.log("Scene Debug: sunset = " … sunset … "; sunsetOffset = " … sunsetOffset … ";currentTime = " … currentTime … "; endTime = " … endTime)

if ((currentTime >= sunsetOffset) and (currentTime <= endTime))

then return true
else return false

end
```

50 10/09/14 14:36:44.057 luup_log:0: Scene Debug: sunset = 1752; sunsetOffset = 1737;currentTime = 1436; endTime = 2330 <0x31df3680>

--------------------------------------------------------------------------------
tiwas

I hope this hasn’t been answered, but I think this might be the solution to my problem…

I have a qubino dimmer. I1 (the physical one) is connected to my dimmable lights in the kitchen. I2 and I3 should be used to turn my ceiling lights on/off - however, I2 and I3 are only "motion detectors" so switching them on trips the dector while turning them off "un-trips" them. This would be easy enough to use, BUT…if I turn them on with the switch, then turn them off through a scene then the physical switch is in the wrong position. Also, as all my other switches are monostable (I’ve fitted a return spring) it would fit my setup a lot better if these could be monostable as well.

Which brings me to my question - is it possible to use luup to turn them on if they’re off or off if they’re on? I can make two scenes with the same trigger (I2 detector triggered/turned on), but I want one of them to not execute based on the status of the lights.

If this is possible I would greatly appreciate it if I could get some help figuring this out.

Cheers!

--------------------------------------------------------------------------------
conchordian

    Which brings me to my question - is it possible to use luup to turn them on if they’re off or off if they’re on? I can make two scenes with the same trigger (I2 detector triggered/turned on), but I want one of them to not execute based on the status of the lights.

http://forum.micasaverde.com/index.php/topic,8630.msg82895.html#msg82895 1 with GUI or

http://forum.micasaverde.com/index.php/topic,11349.msg81403.html#msg81403 with LUUP might help

--------------------------------------------------------------------------------
tiwas

    [quote="tiwas, post:97, topic:178331"]Which brings me to my question - is it possible to use luup to turn them on if they’re off or off if they’re on? I can make two scenes with the same trigger (I2 detector triggered/turned on), but I want one of them to not execute based on the status of the lights.

http://forum.micasaverde.com/index.php/topic,8630.msg82895.html#msg82895 with GUI or

http://forum.micasaverde.com/index.php/topic,11349.msg81403.html#msg81403 1 with LUUP might help[/quote]
Thanks - that helped a lot! :slight_smile: Now I just need to learn LUUP so I can set my wall switch to step up the lights in 20% increments until it’s 100% - when I turn it off completely and allow a new step up :slight_smile:

Cheers

--------------------------------------------------------------------------------
RexBeckett

[quote="tiwas, post:97, topic:178331"]I hope this hasn’t been answered, but I think this might be the solution to my problem…

I have a qubino dimmer. I1 (the physical one) is connected to my dimmable lights in the kitchen. I2 and I3 should be used to turn my ceiling lights on/off - however, I2 and I3 are only "motion detectors" so switching them on trips the dector while turning them off "un-trips" them. This would be easy enough to use, BUT…if I turn them on with the switch, then turn them off through a scene then the physical switch is in the wrong position. Also, as all my other switches are monostable (I’ve fitted a return spring) it would fit my setup a lot better if these could be monostable as well.

Which brings me to my question - is it possible to use luup to turn them on if they’re off or off if they’re on? I can make two scenes with the same trigger (I2 detector triggered/turned on), but I want one of them to not execute based on the status of the lights.

If this is possible I would greatly appreciate it if I could get some help figuring this out.

Cheers![/quote]

The following luup code provides a toggle action. If you trigger the scene when your pseudo-motion-sensor trips, it should do what you want.

local dID = 123 -- Device ID of your Z-Wave Switch local status = luup.variable_get("urn:upnp-org:serviceId:SwitchPower1","Status",dID) if status == "1" then luup.call_action("urn:upnp-org:serviceId:SwitchPower1","SetTarget",{newTargetValue=0},dID) else luup.call_action("urn:upnp-org:serviceId:SwitchPower1","SetTarget",{newTargetValue=1},dID) end

--------------------------------------------------------------------------------
B3rt

Why does this code not work?

I want the status of button 7 which is set as name ‘O: ON’

local switch1 = luup.variable_get("urn:dcineco-com:serviceId:MSwitch1","Status7",80)

if (tonumber(switch1) == 1)
  return true
end
return false

D_MSwitch80.xml
id: 80
urn:schemas-dcineco-com:device:MSwitch80:1
button 7 is a radiobutton with 8

When i test this code in test luup code it errors.

--------------------------------------------------------------------------------
RexBeckett

[quote="B3rt, post:101, topic:178331"]Why does this code not work?

I want the status of button 7 which is set as name ‘O: ON’

local switch1 = luup.variable_get("urn:dcineco-com:serviceId:MSwitch1","Status7",80)

if (tonumber(switch1) == 1)
  return true
end
return false

D_MSwitch80.xml
id: 80
urn:schemas-dcineco-com:device:MSwitch80:1
button 7 is a radiobutton with 8

When i test this code in test luup code it errors.[/quote]

You need a then in the if statement: if (tonumber(switch1) == 1) then return true end

You could also simplify the code to:

local switch1 = luup.variable_get("urn:dcineco-com:serviceId:MSwitch1","Status7",80) return (tonumber(switch1) == 1)

--------------------------------------------------------------------------------
theslimone

[quote="Ramiii, post:57, topic:178331"]Here is my scenario

I have a smart switch that turn on the living room lights if a motion is detected and turn it off after 1 minute. But whenever I’m watching tv and someone walks in the light goes on. I want to convert the smart switch to a PLEG action ( which I can easily do ) but I want this scene to only run if XBMC is not playing. I have the XBMC addon installed. Any help with that ?[/quote]

Hey There, I have this function attached to a trigger to do just that (Check if XBMC is playing , return false if so)

[sub] local function videocheck()
local lul_tmp = luup.variable_get("urn:upnp-org:serviceId:XBMCState1","PlayerStatus",9)
if (lul_tmp == "Video_start") then
return false
else
return true
end
end[/sub]

This should work great for you. Just change your device ID to match.

--------------------------------------------------------------------------------
Caffreyboy

If I wish to run a scene when the switch is off, rather than on, how do I modify this code?

function checkSwitch(dID, allow)
–local dID = 66 – Device ID of your Z-Wave Switch
–local allow = true – true runs scene if switch is on, false blocks it
local status = luup.variable_get("urn:upnp-org:serviceId:SwitchPower1","Status",dID)
return ((status == "1") == allow)
end

--------------------------------------------------------------------------------
RexBeckett

[quote="Caffreyboy, post:104, topic:178331"]If I wish to run a scene when the switch is off, rather than on, how do I modify this code?

function checkSwitch(dID, allow)
–local dID = 66 – Device ID of your Z-Wave Switch
–local allow = true – true runs scene if switch is on, false blocks it
local status = luup.variable_get("urn:upnp-org:serviceId:SwitchPower1","Status",dID)
return ((status == "1") == allow)
end[/quote]

Just call the function checkSwitch(…) with the second argument (allow) as false. It will return true if the switch is off.

--------------------------------------------------------------------------------
Caffreyboy

Have I done this correctly?

function checkSwitch(dID, allow)
–local dID = 41 – Device ID of your Z-Wave Switch
–local allow = false – true runs scene if switch is on, false blocks it
local status = luup.variable_get("urn:upnp-org:serviceId:SwitchPower1","Status",dID)
return ((status == "1") == allow)
end

--------------------------------------------------------------------------------
RexBeckett

[quote="Caffreyboy, post:106, topic:178331"]Have I done this correctly?

function checkSwitch(dID, allow)
–local dID = 41 – Device ID of your Z-Wave Switch
–local allow = false – true runs scene if switch is on, false blocks it
local status = luup.variable_get("urn:upnp-org:serviceId:SwitchPower1","Status",dID)
return ((status == "1") == allow)
end[/quote]

Your code is that of a function - to be called from elsewhere. The parameters dID and allow are provided by arguments in the call. The original code was not a function and the parameters were specified locally. The local variables have been commented out so that they have no effect.

If you want to keep checkSwitch as a function, you should call it like this:

```Lua
function checkSwitch(dID, allow)
– dID is the Device ID of your Z-Wave Switch
– allow is true to run the scene if switch is on, false inverts it.
local status = luup.variable_get("urn:upnp-org:serviceId:SwitchPower1","Status",dID)
return ((status == "1") == allow)
end

local result = checkSwitch(41, false)
… other conditions …
return result
```

If you just want a single condition in your scene, change the code to this:

local dID = 41 -- Device ID of your Z-Wave Switch local allow = false -- true runs scene if switch is on, false blocks it local status = luup.variable_get("urn:upnp-org:serviceId:SwitchPower1","Status",dID) return ((status == "1") == allow)

--------------------------------------------------------------------------------
Caffreyboy

Thank you for help.

If this unit is a dimmer switch, is this the correct code for a single condition?

local dID = 41 – Device ID of your Z-Wave Switch
local allow = false – true runs scene if switch is on, false blocks it
local status = luup.variable_get("urn:schemas-upnp-org:device:DimmableLight:1","Status",dID)
return ((status == "1") == allow)

--------------------------------------------------------------------------------
RichardTSchaefer

Wrong serviceID … you can always find the CORRECT service ID by going to the Advanced TAB of the device in question and letting the mouse hover over the variable name.

Since I told you how to find the correct ServiceID I will not post the correct change here!

--------------------------------------------------------------------------------
RexBeckett

    If this unit is a dimmer switch, is this the correct code for a single condition?

If you follow @RichardTSchaefer’s good suggestion and investigate your dimmer’s Advanced tab, you will discover that it contains both Status and LoadLevelStatus variables. They have different service IDs.

Status indicates whether the dimmer is on or off with values of "1" or "0" - just like a non-dimmable light switch. LoadLevelStatus shows the current dimming level from "0" to "100". You can use either one as a condition (with the correct service ID) as long as you test for appropriate values.

--------------------------------------------------------------------------------
Caffreyboy

Thank you! It finally dawned upon me. Much appreciated to you both.

--------------------------------------------------------------------------------
Dvbit

[quote="RexBeckett, post:84, topic:178331"]

    When she complains, I’ll give my wife your number.

I’m only an Engineer. I do alright with electronics, mechanics and software but I can’t fix wives. I’m still waiting for somebody to write a manual. ;D[/quote]

Eng too over here…
that book will never be written.
And when it will be written a wife will say it is wrong

--------------------------------------------------------------------------------
tmozer

So… Apparently it appears you can create scene scripts in Luup code. Is there any programming instructions around to get started with?

--------------------------------------------------------------------------------
RexBeckett

    So… Apparently it appears you can create scene scripts in Luup code. Is there any programming instructions around to get started with?

There are hundreds of examples on this forum. Search using Google for whatever you are trying to achieve. For general guidance, see the following:

Lua Reference Manual 2
Programming in Lua 2
Luup Extensions 2
Conditional Scene Execution 4

--------------------------------------------------------------------------------
sely
RexBeckett,
This thread is the greatest tool i have found in all the Vera documentation. Thank you. Can you help with this? Can i write 2 sets of luup.call_action in a if statement? these are 4 of the things i’ve tried. BTW when I only use 1 call my code works.

Try 1:
if (lul_snsr == "1") then
luup.call_action("urn:upnp-org:serviceId:SwitchPower1","SetTarget",{ newTargetValue="1" }, (18 and 39))

Try 2:
if (lul_snsr == "1") then
(luup.call_action("urn:upnp-org:serviceId:SwitchPower1","SetTarget",{ newTargetValue="1" },18)) and
(luup.call_action("urn:upnp-org:serviceId:SwitchPower1","SetTarget",{ newTargetValue="1" },39))

try 3:
if (lul_snsr == "1") then
(luup.call_action("urn:upnp-org:serviceId:SwitchPower1","SetTarget",{ newTargetValue="1" },18)),(luup.call_action("urn:upnp-org:serviceId:SwitchPower1","SetTarget",{ newTargetValue="1" },39))

Try 4:
if (lul_snsr == "1") then
luup.call_action("urn:upnp-org:serviceId:SwitchPower1","SetTarget",{ newTargetValue="1" },18)
luup.call_action("urn:upnp-org:serviceId:SwitchPower1","SetTarget",{ newTargetValue="1" },39)
else


--------------------------------------------------------------------------------
RexBeckett

Hi @sely. You are close with Try 4. I think what you want is:

if (lul_snsr == "1") then luup.call_action("urn:upnp-org:serviceId:SwitchPower1","SetTarget",{ newTargetValue="1" },18) luup.call_action("urn:upnp-org:serviceId:SwitchPower1","SetTarget",{ newTargetValue="1" },39) end

--------------------------------------------------------------------------------
ascari

Can I have help.

My Lua work all time, I need work only at night :frowning:

local device=23
local switchOnOff = luup.variable_get("urn:upnp-org:serviceId:SwitchPower1", "Status", device)
if ( luup.is_night() ) and (switchOnOff == "0")
then
luup.call_action("urn:upnp-org:serviceId:SwitchPower1","SetTarget",{ newTargetValue="1" },device)
luup.call_action("urn:upnp-org:serviceId:Dimming1", "SetLoadLevelTarget", {newLoadlevelTarget = "35"}, device)
end

--------------------------------------------------------------------------------
RexBeckett

    My Lua work all time, I need work only at night :(

The Lua actions should only run at night. Do you have any actions selected on the Devices or Advanced tabs of the scene? If so, they will run regardless. Either remove them or have your Lua code return false when you don’t want them to run.
ascari
Jan '15

RexBeckett Indeed I have an action in the other tab.

But now it never work.

If i check only switchOnOff it’s ok but with the and luup.is_night() not ok

--------------------------------------------------------------------------------
RexBeckett

[quote="ascari, post:119, topic:178331"]RexBeckett Indeed I have an action in the other tab.

But now it never work.

If i check only switchOnOff it’s ok but with the and luup.is_night() not ok[/quote]

It isn’t night where I am so I wouldn’t expect it to work. I see it is night in your timezone, though. Check your location settings in Vera are correct.
ascari
Jan '15

I think it’s ok now, but I need wait this night :slight_smile:

```Lua
local device=23
local switchOnOff = luup.variable_get("urn:upnp-org:serviceId:SwitchPower1", "Status", device)
if (luup.is_night()) then
if (switchOnOff == "0") then
luup.call_action("urn:upnp-org:serviceId:SwitchPower1","SetTarget",{ newTargetValue="1" },device)
luup.call_action("urn:upnp-org:serviceId:Dimming1", "SetLoadLevelTarget", {newLoadlevelTarget = "35"}, device)
return true
end
else
return false
end
```

--------------------------------------------------------------------------------
# Running a scene when a variable changes
RexBeckett

There have been several recent posts on this subject so I think it is worth including here. The issue is that normal scene triggers that include conditions, like temperature is lower than 18, will fire when the temperature drops below 18 but will not fire again unless it rises to or above 18 and then drops below it again.

If you use this type of trigger for a scene that includes Lua code for additional conditions - say a time period - it will not work very well. If the trigger fires outside of the allowed time period, you may not get another trigger during the period when you want to take some action.

One solution is to have your scene triggered by a periodic schedule. Then you can use Lua code from the examples in this thread to decide if you want the actions to be executed.

A better technique is to watch the variable in question and have your scene run when it changes. As above, you can then use Lua code to decide if you want to execute some action(s). Vera provides a function for this very purpose. Here is how you can use it:

```Lua
-- Set up variable-watch for device 123
luup.variable_watch("doChange123","urn:upnp-org:serviceId:TemperatureSensor1","CurrentTemperature",123)

– Process variable-watch callback for device 123. Run scene 12
function doChange123()
luup.call_action("urn:micasaverde-com:serviceId:HomeAutomationGateway1", "RunScene", {SceneNum = 12}, 0)
end
```

If this code is placed in Vera’s Startup Lua, it will set up a variable-watch for thermostat/thermometer device 123 current temperature. When this changes, scene number 12 will be executed.

To edit Startup Lua, select APPS → Develop Apps → Edit Startup Lua then paste the code on the end of any existing line and click GO. You will need to restart Vera before the changes take effect.

If you want to watch other variables, you can repeat the code for each one. You will need to change the name of the function in both the function and the luup.variable_watch(…) statements to keep them unique.

If you want to watch several variables, the following example shows how this can be done by using a table instead of multiple pieces of code. watchTable contains an entry for each variable that you want to watch. Each entry includes the ServiceId, variable name and device number that you want to watch and the scene number to be run if it changes. Each entry, other than the last one, should be terminated with a comma.

The function setWatch reads this table and sets up a watch for each entry. All the watches use the same callback function. This function, catchWatch, searches the table for a matching entry and, if it finds one, runs the specified scene.

```Lua
– Table entries { "ServiceID", "VariableName", DeviceNo, SceneNo }
watchTable = {
{"urn:upnp-org:serviceId:TemperatureSensor1","CurrentTemperature",123,11},
{"urn:micasaverde-com:serviceId:LightSensor1","CurrentLevel",124,12}
}
– Setup Variable Watch for each entry in watchTable

function setWatch()
for n,t in ipairs(watchTable) do
   luup.variable_watch("catchWatch",t[1],t[2],t[3])
end
end

– When a watched variable changes, call the specified scene
function catchWatch(lul_device, lul_service, lul_variable, lul_value_old, lul_value_new)
   for n,t in ipairs(watchTable) do
      if ( (t[3] == lul_device) and (t[1] == lul_service) and (t[2] == lul_variable) ) then
      luup.call_action("urn:micasaverde-com:serviceId:HomeAutomationGateway1", "RunScene", {SceneNum = t[4]}, 0)
      end
   end
end
– Wait for 30 seconds after restart then run setWatch

luup.call_delay("setWatch",30)
```

The last part of this code causes the setWatch function to be run 30 seconds after Vera restarts. This is to allow all devices to have completed their initialization.

--------------------------------------------------------------------------------
conchordian

Thanks very much, Rex! I’ve been trying to get scene control to work with my Fibaro dimmer modules, but without your example I was lost.

http://forum.micasaverde.com/index.php/topic,15855.msg200101.html#msg200101 7 is an example of how I used it.

--------------------------------------------------------------------------------
maja

This is GREAT!

Is there also a way to check the location of a particular cellular phone? For example, "if phone X is within geolocation Y but phone X is outside geolocation Y then do this"?

--------------------------------------------------------------------------------
RexBeckett

    Is there also a way to check the location of a particular cellular phone? For example, "if phone X is within geolocation Y but phone X is outside geolocation Y then do this"?

If you have phones running Android, you could use the Vera Proximity app to set MultiSwitch buttons according to their positions wrt the defined fences. I’m sure there is also a way to do this with IOS phones.

--------------------------------------------------------------------------------
lhjunk

[quote="RexBeckett, post:18, topic:178331"]There were two requests for this type of scene condition in the last 24 hours so it must be worth posting here. The objective is to allow or prevent a scene executing when triggers occur in a short time period (window).

All these examples save a timestamp each time they are run using a global variable tLastOnD99. On each run, the saved timestamp is subtracted from the current time and compared to the specified time window (twSecs). To allow this code to be used for more than one triggering device, the 99 in tLastOnD99 should be replaced with the device ID of the triggering device. This is simply to avoid undesired side effects so any other name could be used instead.

As with previous examples, the code may be placed in either a Trigger’s Luup event to allow/prevent that trigger or on the scene’s LUUP tab where it will affect all triggers and manual operation.

Allow the scene to run when a trigger occurs within twSecs of the last one (e.g. on/off/on):

local twSecs = 5 -- Number of seconds in time window local tNow = os.time() local tLastOn = tLastOnD99 or 0 tLastOnD99 = tNow return ((tNow - tLastOn) <= twSecs)

Allow the scene to run provided a trigger occurs at least twSecs after the last one:

local twSecs = 5 -- Number of seconds in time window local tNow = os.time() local tLastOn = tLastOnD99 or 0 tLastOnD99 = tNow return ((tNow - tLastOn) >= twSecs)

Generic version:

local twSecs = 5 -- Number of seconds in time window local allow = true -- true runs scene during time window, false blocks it local tNow = os.time() local tLastOn = tLastOnD99 or 0 tLastOnD99 = tNow return (((tNow - tLastOn) <= twSecs) == allow)[/quote]

@ Rex & @ Richard : - I have a single pole switch in my Sons room. A ceiling fan is wired , with only a single black wire.

If I install an Enerwave dual relay switch in the canopy, I would need to either install a new wall scene controller, $~45-159+ depending if I go leviton or not…, or I was thinking of how your code above could be tweaked to handle the following use cases:

    Turn on Light - simple - single wall switch triggers light relay
    Turn Light off, check - not needed, I have an auto off trigger for the bedrooms after 20 minutes
    leave the light on, but trigger the ceiling fan relay to have both on
    Turn light off, but leave fan on (summer nights)

Would this be better suited in PLEG, and if so, any guidance you can offer would be fantastic!

Thanks!

--------------------------------------------------------------------------------
RexBeckett

    Would this be better suited in PLEG, and if so, any guidance you can offer would be fantastic!

I think PLEG is a better solution for for this type of time-based logic. Signalling using on/off/on switching only works if the Z-Wave device supports instant-status reporting.

--------------------------------------------------------------------------------
lhjunk

Thanks Rex, yes, the Enerwave does support instant status, a very important feature , but unknown to me when I started this adventure and used to buy based on price :frowning:

I use the enerwave dual relay in my master, works great with instant status. I’ll play with PLEG and see what I can figure out. Thanks!

--------------------------------------------------------------------------------
RexBeckett
Beta
Mar '15

First adventures into using Lua code frequently fail due to the use of incorrect Service IDs in luup.variable_get(…), luup.variable_set(…) and luup.call_action(…) calls. A common error is to use the Device-type instead of the Service ID. Tip: A Service ID string will contain the word serviceId and will not contain the words schemas or device.

I listed some of the common ones in Service IDs, Variables and Actions 1. It is also possible to see the Service ID for a device variable by hovering your mouse cursor over the variable name on the device’s Advanced tab. This is OK if you have a good visual memory. :wink:

One way to see the Service ID in a copy/paste-able form is to enter this command in your browser - replacing with the IP address of your Vera (without the <>):

http://<veraip>:3480/data_request?id=status&output_format=xml

This will list every variable for all of your devices so you will need to scroll down the list until you find the one you want. You can also get the information for a particular device by using this - replacing 123 with the device number:

http://<veraip>:3480/data_request?id=status&output_format=xml&DeviceNum=123

To see all the actions available for your devices, enter this:

http://<veraip>:3480/data_request?id=invoke

or, for a single device:

http://<veraip>:3480/data_request?id=invoke&DeviceNum=123

Note that not all devices will actually implement all the available actions. The list is a super-set covering all devices of the given Device-type. Theoretically, actions marked with an asterisk (*) should be implemented but this is not always the case. Clicking on an entry in the invoke list will fire the action so be careful!

If you would prefer a less-cluttered list, you could use LuaTest which has buttons to display Variables, Values, Actions and individual device Status.

--------------------------------------------------------------------------------
aa6vh

This thread has touched on running a scene (see "Run Scene when a Variable Changes"), but I thought it would be beneficial to elaborate a little, plus give coding examples that can improve your LUA code, and make the code more self documenting.

Why would you want to run a separate scene? Suppose you want to lock your door. When locking the door, you might first want to turn on or off the porch light, or to perhaps check to see if the door is closed first. It is much easier to perform these actions in a separate scene rather than performing all of these actions in the first scene, especially if you want to lock the door from multiple scenes.

First copy and paste the following three lines of code in your LUA startup code section.

function RunScene(scnnum)
  luup.call_action("urn:micasaverde-com:serviceId:HomeAutomationGateway1","RunScene",{ SceneNum = scnnum },0)
end

This creates a global function accessible from all scenes, that when called will cause the specified scene to be scheduled to be run. You do need to determine the scene number of the scene you wish to run.

Coding tip: Specifying a scene number in a function call can be hard to understand when later re-reading the code. Also, it can be hard to locate all uses of that scene number should that scene ever have to be recreated (say if you have to completely reload Vera after a factory reset), as it may be getting a new scene number. The way to make it easier on yourself is to have a global variable that contains the scene number, specifying that global variable in the LUA startup code section. If the scene number ever changes for any reason, you will only have to change the number in one place.

scnLockDoor = "17"

Again, the above code statement needs to be in the LUA startup code section. The example statement just specifies that the Lock Door scene was given scene number 17 by Vera. Entering scnLockDoor will be much easier than remembering and entering "17".

Now suppose you have a scene that you want to lock the door if it is night. Your would simply enter in the LUA scene code:

if luup.is_night() then
  RunScene(scnLockDoor)
end

The above example shows how to run a scene, and also demonstrates how to encapsulate LUUP calls within a function. By having just one function, you can avoid typing errors in subsequent uses, and make the code much easier to follow.

You can do the same thing to turn on or off lights, or to dim lights, within LUA scene code. You will need the following global functions:

function DimLight(sDevice, ilevel)
  local idevice = tonumber(sDevice)
  luup.call_action("urn:upnp-org:serviceId:Dimming1","SetLoadLevelTarget",{ newLoadlevelTarget = ilevel },idevice)
end

function ToggleLight(sDevice, sNewState)
  local idevice = tonumber(sDevice)
  local I = "0"
  if sNewState == "on" then
    I = "1"
  end
  luup.call_action("urn:upnp-org:serviceId:SwitchPower1","SetTarget",{ newTargetValue = I },idevice)
end

Again, using global variables in place of device numbers will make the code easier to follow, and easier to change should that device number ever get changed. The following examples should be placed in the LUA startup code section, same as the above functions.

dPorchLight = 23
dEntryLight = 24
dDiningLight = 54

Now when LUA scene code needs to turn on the porch light, turn off the entry light, and dim the dining room light to say 75 percent:

ToggleLight(dPorchLight, "on")
ToggleLight(dEntryLight, "off")
DimLight(dDiningLight, 75)

--------------------------------------------------------------------------------
grrgold

[quote="RexBeckett, post:6, topic:178331"]This code will enable you to set a range of light level within which your scene will, or will not, be run. Set lLow and lHigh to define the range. As with previous generic examples, the variable allow determines whether to allow or block the scene when the current light level is within the specified range.

Generic Light Level Range:

local dID = 30 -- Device ID of your light sensor local lLow = 0 -- Lowest level of range local lHigh = 20 -- Highest level of range local allow = true -- true runs scene when in range, false blocks it local lCurrent = tonumber((luup.variable_get("urn:micasaverde-com:serviceId:LightSensor1","CurrentLevel",dID))) return (((lCurrent >= lLow) and (lCurrent <= lHigh)) == allow)

Coming next: VariableContainer[/quote]

I tried adding this code to a Fibaro 3-1 sensor. The code seems to work once but then the light sensor disappears from my device list (UI7 on a Vera Edge)… any ideas?

--------------------------------------------------------------------------------
RexBeckett

    I tried adding this code to a Fibaro 3-1 sensor. The code seems to work once but then the light sensor disappears from my device list (UI7 on a Vera Edge)... any ideas?

This sounds like the same UI7 bug reported by several people: Vera loses recent configuration changes. I think there is some issue with the way the data is being saved following any modification. I suggest you contact Vera Support.

--------------------------------------------------------------------------------
xgutterratx

[quote="RexBeckett, post:2, topic:178331"]We often want to allow or block scenes depending on the state of a Z-Wave switch (E.g. a light is on) or a VirtualSwitch (e.g. Home/Away). The VirtualSwitch (VS) plugin is very useful as a means of controlling scenes. You can create several different VS devices to signify various states of your home. E.g. Home/Away, OnVacation, HaveGuests, etc. VS devices can be set manually through the UI, by scenes or with other plugins.

Testing the state of a switch is essentially the same whether it is real or virtual but the serviceID must match the type of switch being tested.

Z-Wave Switch:

local dID = 66 -- Device ID of your Z-Wave Switch local allow = true -- true runs scene if switch is on, false blocks it local status = luup.variable_get("urn:upnp-org:serviceId:SwitchPower1","Status",dID) return ((status == "1") == allow)

Virtual Switch:

local dID = 32 -- Device ID of your VirtualSwitch local allow = true -- true runs scene if switch is on, false blocks it local status = luup.variable_get("urn:upnp-org:serviceId:VSwitch1","Status",dID) return ((status == "1") == allow)

MultiSwitch:

local dID = 94 -- Device ID of your MultiSwitch local button = 1 -- MultiSwitch button number (1-8) local allow = true -- true runs scene if switch is on, false blocks it local status = luup.variable_get("urn:dcineco-com:serviceId:MSwitch1","Status"..button,dID) return ((status == "1") == allow)[/quote]

I have a motion sensor that when armed and detects motion it turns on 3 different lights in my downstairs area.
I want the scene to not activate the scene if any light in my downstairs area is already on.

I have been able to get this to work if one light is on

local dID = 3 – Device ID of your Z-Wave Switch
local allow = true – true runs scene if switch is on, false blocks it
local status = luup.variable_get("urn:upnp-org:serviceId:SwitchPower1","Status",dID)
return ((status == "0") == allow)

but when i have tried every combination i can think of to add the extra lights it no longer works

any help would be appreciated

--------------------------------------------------------------------------------
RexBeckett

@xgutterratx, here is some code to make your life easier. It checks all the switches specified by device number in the table switchIDs and sets the variable anyon if any of them are on. This is then checked against the setting of allow to decide whether to run the scene or not.

```Lua
local function checkSwitch(deviceNo)
local status = luup.variable_get("urn:upnp-org:serviceId:SwitchPower1","Status",deviceNo)
return ((status or "0") == "1")
end

local switchIDs = { 3, 123, 321 }
local allow = false – true runs scene if any switch is on, false blocks it
local anyon = false
for i=1,table.maxn(switchIDs) do
anyon = anyon or checkSwitch(switchIDs[i])
end
return (anyon == allow)

```

Just set the correct switch device numbers in the switchIDs table separated by commas. Note that they are numbers not strings. You can have any number of entries in this table.

--------------------------------------------------------------------------------
xgutterratx

Thanks that worked a treat.

I have been thinking about it and i would like to change the scene a little a bit.
and was hoping you could help again.

i have a motion sensor in my downstairs entry area. So the sensor gets you when you come in the door or down the stairs.

at this point of time when activated and detects motion it turns on 5 lights in my down stairs area id 3, 4, 5, 6, 8
As long as id 3 ,4, 5,6, 8 is not on already then waits 10 mins and turns id3, 4, 5, 6, 8, off.

i have another light relay on my stairs but this also is in my upstairs living area which the light is normally on of a night time…

what i would also like to achieve is if stairs light id 14 is off still activate the scene but also turn ID14 on and off in 10 mins

but if id 14 is on still activate the scene but don’t turn id 14 off in 10 mins

hope you can help.

--------------------------------------------------------------------------------
RexBeckett

So, if I understand this right, you are switching lights 3, 4, 5, 6 & 8 on with normal scene actions and switching them back off using delayed scene actions. To allow light 14 to be handled differently, you would need to control it with separate action calls. The following code should achieve that:

```Lua
local function checkSwitch(deviceNo)
local status = luup.variable_get("urn:upnp-org:serviceId:SwitchPower1","Status",deviceNo)
return ((status or "0") == "1")
end

function delayOff(dev)
local devno = tonumber(dev)
luup.call_action("urn:upnp-org:serviceId:SwitchPower1","SetTarget",{ newTargetValue=0},devno)
end

local switchIDs = { 3, 4, 5, 6, 8 }
local allow = false – true runs scene if any switch is on, false blocks it
local anyon = false
for i=1,table.maxn(switchIDs) do
anyon = anyon or checkSwitch(switchIDs[i])
end
if not (checkSwitch(14) or anyon) then
luup.call_action("urn:upnp-org:serviceId:SwitchPower1","SetTarget",{newTargetValue=1},14)
luup.call_delay("delayOff",600,14)
end
return (anyon == allow)

```

--------------------------------------------------------------------------------
xgutterratx

your awesome !t works great

I think i may be pushing my luck now but.

i have one last piece of the puzzle for this scene.

when my motion sensor id 9 is armed and detects motion it activates the above scene.

Lights turn on and then 10min delay and the lights turn off ( as per the above conditions)

the sensor is in a tripped state (detects motion ) for 1 min,

what i would like to happen, that doesn’t happen at the moment.
Is if after that 1 min of tripped if the sensor detects motion again that the 10 min delay is restarted before the lights are switched off.

I guess what i am saying is if the armed sensor doesn’t detect motion in 10 mins then complete the 2nd part of the scene

--------------------------------------------------------------------------------
RexBeckett

    what i would like to happen, that doesn't happen at the moment. Is if after that 1 min of tripped if the sensor detects motion again that the 10 min delay is restarted before the lights are switched off.

    I guess what i am saying is if the armed sensor doesn’t detect motion in 10 mins then complete the 2nd part of the scene

OK, you’ve now crossed into PLEG territory. PLEG is by far the easiest way to implement this type of logic. See PLEG Basics for an introduction. There are numerous examples on the PLEG board of motion-triggered lighting logic.

The problem with scenes is that, when the immediate actions are fired, the delayed actions are queued for execution and they cannot be stopped. You could achieve it by using separate scenes for on and off but it would be more complex than doing it with PLEG.

--------------------------------------------------------------------------------
seanmahrt

OK, I give. I’m working OK with Lua code, but I really want to add a conditional to a scene (time of day window). I see and understand the Lua code, but I’m not seeing where I stuff that code into the scene:

"You can also insert the code into the scene’s main LUUP tab where it will allow or block all triggers and manual operation"

I don’t find a main LUUP tab for the scene. I see where to run the LUUP code after the scene has triggered, and using that successfully. But where do I inhibit the firing of the scene based on LUA return values?

Sean

--------------------------------------------------------------------------------
RexBeckett

    I don't find a main LUUP tab for the scene. I see where to run the LUUP code after the scene has triggered, and using that successfully. But where do I inhibit the firing of the scene based on LUA return values?

In UI5, there is a LUUP tab in the scene editor where you can enter Lua code. In UI7, in Step 3 of the scene editor you can specify Lua code. In both cases, if the code returns false, the scene actions will not be executed.

In UI5 there is also the possibility to attach Lua code to each scene trigger using the Luup Event tab. If this code returns false, the trigger will be suppressed. This feature is not yet available in UI7.

--------------------------------------------------------------------------------
slief

Please forgive me for asking this question as I know it’s kind of been beaten to death.

I browsed through this thread and others and found myself more confused. I am a new Vera user running UI7 on a VeraEdge. I have a motion sensor on my porch that I put there to control the porch light. Ideally, I only want the porch light to be triggered via the sensor at night. I created a scene using the front porch motion senor as a trigger and set it to run when the sensor is armed and disarmed. It was set so that the light will turn on, then wait for 2 minutes then turn off. I set the scene up so that it only runs in Night Mode. That didn’t seem to work right as the scene didn’t execute properly in night mode or any mode for that matter. I then added the Day Night App and still couldn’t get it to trigger. I’ve tried manually switching from Home to Night with no luck there either. I know there is something I am missing. My understanding is that the Day Night app should toggle Vera between day and night or at least allow a scene to run based on Day or Night in the app. I did double check my location in settings and while I had the city and state correct, the longitude and latitude was off but still a California location. I just changed those numbers to reflect my current city so maybe that will help but I won’t know if Day or Night toggles until later this evening. I have 00:00:00 both for sunrise and sunset offsets in the Day Or Night device.

Thus far, the only way I can get the light to come on automatically from motion is to associate the motion detector to the porch light and omit the scene. In that configuration, the light will turn on for 3 minutes when motion is detected but it does it day or night.

Can somebody help me with this… I’m usually pretty good at this stuff but this one has me baffled.

What am I missing? Do I need to add some LUUP or PLEG code?
I read the thread below but that really through me for a "luup"… ;D
http://forum.micasaverde.com/index.php/topic,30314.0.html

--------------------------------------------------------------------------------
RexBeckett

As per the first post in this thread, you can add Lua/Luup code to a scene to limit when it can be run. The trigger, a motion sensor in your case, still attempts to run the scene but the Lua code can prevent the action(s) from happening.

You should be able to get the condition you want by adding this code to the scene’s Luup tab:

return luup.is_night()

The Day or Night plugin provides another way to indicate day or night. It does not directly change the mode of Vera but can be used by Lua code in a scene like this:

local dID = 23 -- Device ID of your DayTime plugin local allow = false -- true runs scene during daytime, false runs it at night local status = luup.variable_get("urn:rts-services-com:serviceId:DayTime","Status",dID) return ((status == "1") == allow)

Change dID from 23 to the device number of your Day or Night plugin.

The HouseModes provided in UI7 act as an additional condition for scenes. If you are using Lua code to implement conditions, you should configure the scene to run in all modes to avoid confusion.

--------------------------------------------------------------------------------
slief

[quote="RexBeckett, post:142, topic:178331"]As per the first post in this thread, you can add Lua/Luup code to a scene to limit when it can be run. The trigger, a motion sensor in your case, still attempts to run the scene but the Lua code can prevent the action(s) from happening.

You should be able to get the condition you want by adding this code to the scene’s Luup tab:

return luup.is_night()

The Day or Night plugin provides another way to indicate day or night. It does not directly change the mode of Vera but can be used by Lua code in a scene like this:

local dID = 23 -- Device ID of your DayTime plugin local allow = false -- true runs scene during daytime, false runs it at night local status = luup.variable_get("urn:rts-services-com:serviceId:DayTime","Status",dID) return ((status == "1") == allow)

Change dID from 23 to the device number of your Day or Night plugin.

The HouseModes provided in UI7 act as an additional condition for scenes. If you are using Lua code to implement conditions, you should configure the scene to run in all modes to avoid confusion.[/quote]

Thanks you very much for the insight and help. I don’t plan on manually toggling the modes from Home to Night. I want to depend on the Day and Night app to handle those kinds of changes within scenes. My Day and Night plugin is device ID 46.

Lets see if I am understanding what I need to do. Please correct me if I am wrong.

First change the scene to run in all modes since I won’t be messing with the modes anyway. Add both sets of codes to the lua code page of my scene so that it looks like this:

return luup.is_night()
local dID = 46
local allow = false
local status = luup.variable_get("urn:rts-services-com:serviceId:DayTime","Status",dID)
return ((status == "1") == allow)

And if I understand correctly, that would set the scene to only run or at least trigger the light at night and the App Day or Night will automatically change from day to night at sunset and switch back to day at sunrise since my offset in day or night is set to 00:00:00…

--------------------------------------------------------------------------------
aa6vh

The code choices that Rex gave you was an OR, not an AND. Either do one or the other, not both.

And I recommend using "return luup.is_night()", as it is easier (one line instead of four). Just be sure that your location is set to your actual location within the Vera interface.

(And if you want to get technical, nothing after the first "return" would get executed anyway.)

--------------------------------------------------------------------------------
slief

[quote="aa6vh, post:144, topic:178331"]The code choices that Rex gave you was an OR, not an AND. Either do one or the other, not both.

And I recommend using "return luup.is_night()", as it is easier (one line instead of four). Just be sure that your location is set to your actual location within the Vera interface.

(And if you want to get technical, nothing after the first "return" would get executed anyway.)[/quote]

Ahhhh… Thank you very much for the clarification… I did make sure my location was correct in my location settings and also revised my longitude and latitude to match my location. I will copy that code in and see how it works & report back. I assume I should disassociate the light with the motion sensor in the motion sensor device settings? Since the other option Rex mentioned appeared to be based on the use of the Day and Night app, will this work in conjunction with that app or will this work only when I toggle between Home and Night mode? I won’t be toggling between Home and Night modes if I don’t have to.

Thanks to both you and Rex!

--------------------------------------------------------------------------------
slief

[quote="aa6vh, post:144, topic:178331"]The code choices that Rex gave you was an OR, not an AND. Either do one or the other, not both.

And I recommend using "return luup.is_night()", as it is easier (one line instead of four). Just be sure that your location is set to your actual location within the Vera interface.[/quote]
I just checked Vera and Day Night switched to night in the devices page which is something I didn’t notice yesterday. I just checked the motion seansor and the front light came on so it seems to be working. Thanis again for the help.

Now if only I understood the coding. I’ve got a very extensive aquarium controller on my 650 gallon reef tank with almost a thousand lines of code that I programed myself. The tank is fully automated from water changes to top off for evaporation to lighting with ramp up and ramp down to simulate sunrise/sunset and so much more. For me that programming is very basic but I’ve been using that kind of code on aquarium controllers for the last 20 or so years… It’s a lot of IF this, IF that kind of code. This Vera lua and PLEG code seems quite a bit different. I guess I am going to have to really study the coding examples to get a better understanding for it because it’s really gibberish to me at this point.

Anyhow, thank you guys for the help. I am sure I’ll have more questions as I get deeper into my automation but this was a big help on what amounts to a very trivial process.

Thanks again!

Scott

--------------------------------------------------------------------------------
ctguess

    I just checked Vera and Day Night switched to night in the devices page which is something I didn't notice yesterday. I just checked the motion seansor and the front light came on so it seems to be working. Thanis again for the help.

    Now if only I understood the coding. I’ve got a very extensive aquarium controller on my 650 gallon reef tank with almost a thousand lines of code that I programed myself. The tank is fully automated from water changes to top off for evaporation to lighting with ramp up and ramp down to simulate sunrise/sunset and so much more. For me that programming is very basic but I’ve been using that kind of code on aquarium controllers for the last 20 or so years… It’s a lot of IF this, IF that kind of code. This Vera lua and PLEG code seems quite a bit different. I guess I am going to have to really study the coding examples to get a better understanding for it because it’s really gibberish to me at this point.

    Anyhow, thank you guys for the help. I am sure I’ll have more questions as I get deeper into my automation but this was a big help on what amounts to a very trivial process.

    Thanks again!

    Scott

Scott,

I could be reading into your last post incorrectly, but if I’m not, I wanted to try to clarify something for you. The "return luup.is_night()" is in no way related to the Day/Night plugin. You mentioned you saw the "switch" go to Night in the devices page and your scene worked correctly. That plugin switch, however, is not the reason your scene did what you expected.

The native luup.is_night() function and the DayNight plugin do virtually the same thing. The plugin gives you the ability to offset the sunset/sunrise time, however. It’s an easy way to say "do something 15 minutes before sunset everynight". The function luup.is_night() just uses your location’s exact sunset time as the criteria for night vs. day.

In other words, you could delete the DayNight plugin and the "return luup.is_night()" code would still work and your scene would behave properly…only turning the front light on at night when motion is detected.

Sorry if that was already clear and I misinterpreted your previous post.

--------------------------------------------------------------------------------
RichardTSchaefer

the lump.is _night function toggles a sunrise/sunset,

The day night plugin allows you to specify an offset from sunrise and sunset to suit your needs

It also allows you to manually toggle it to test/override your automation logic.

--------------------------------------------------------------------------------
slief

[quote="ctguess, post:147, topic:178331"]

    I just checked Vera and Day Night switched to night in the devices page which is something I didn’t notice yesterday. I just checked the motion seansor and the front light came on so it seems to be working. Thanis again for the help.

    Now if only I understood the coding. I’ve got a very extensive aquarium controller on my 650 gallon reef tank with almost a thousand lines of code that I programed myself. The tank is fully automated from water changes to top off for evaporation to lighting with ramp up and ramp down to simulate sunrise/sunset and so much more. For me that programming is very basic but I’ve been using that kind of code on aquarium controllers for the last 20 or so years… It’s a lot of IF this, IF that kind of code. This Vera lua and PLEG code seems quite a bit different. I guess I am going to have to really study the coding examples to get a better understanding for it because it’s really gibberish to me at this point.

    Anyhow, thank you guys for the help. I am sure I’ll have more questions as I get deeper into my automation but this was a big help on what amounts to a very trivial process.

    Thanks again!

    Scott

Scott,

I could be reading into your last post incorrectly, but if I’m not, I wanted to try to clarify something for you. The "return luup.is_night()" is in no way related to the Day/Night plugin. You mentioned you saw the "switch" go to Night in the devices page and your scene worked correctly. That plugin switch, however, is not the reason your scene did what you expected.

The native luup.is_night() function and the DayNight plugin do virtually the same thing. The plugin gives you the ability to offset the sunset/sunrise time, however. It’s an easy way to say "do something 15 minutes before sunset everynight". The function luup.is_night() just uses your location’s exact sunset time as the criteria for night vs. day.

In other words, you could delete the DayNight plugin and the "return luup.is_night()" code would still work and your scene would behave properly…only turning the front light on at night when motion is detected.

Sorry if that was already clear and I misinterpreted your previous post.[/quote]

[quote="RichardTSchaefer, post:148, topic:178331"]the lump.is _night function toggles a sunrise/sunset,

The day night plugin allows you to specify an offset from sunrise and sunset to suit your needs

It also allows you to manually toggle it to test/override your automation logic.[/quote]

Thanks for the clarification on that. That does shed some light. No pun intended. I may want to play with this using the alternate code in conjunction with Day Night so I can offset sunrise. It’s pretty light in So. Cal at sunrise which is about 7:40AM according to most charts and there is really no need for a light on after 6 AM. At least during the summer. As such, I can see how setting - 01:00:00 or even - 01:40:00 in Day Nights sunrise offset could be useful. Still the current luup code is a better alternative to the light coming on during the day like it was when I had the motion sensor associated with the front porch light.

Thanks again for all your help. It’s much appreciated. I’m very active on a number of forums related to my hobbies and it’s great to have a Vera forum to fall back on when I can’t figure out my own answers. It’s people you guys that make forums like this a great place for the less experienced. I’m sure you haven’t heard the last of me yet!

Thanks again!

--------------------------------------------------------------------------------
slief

I just noticed that my motion is still activating the front light. It seems as though the code isn’t recognizing that it’s still day time. I’ve verified my location in settings and even made sure my logitude and latitude are correct.

I currently have:
return luup.is_night() in my luup code for the scene associated with the motion detector.

I was under the impression that the above code would work regardless of what mode I am in. Did I misunderstand that?

--------------------------------------------------------------------------------
RexBeckett

[quote="slief, post:150, topic:178331"]I just noticed that my motion is still activating the front light. It seems as though the code isn’t recognizing that it’s still day time. I’ve verified my location in settings and even made sure my logitude and latitude are correct.

I currently have:
return luup.is_night() in my luup code for the scene associated with the motion detector.

I was under the impression that the above code would work regardless of what mode I am in. Did I misunderstand that?[/quote]

Did you remove the association that you had previously set between the motion sensor and light?

--------------------------------------------------------------------------------
slief

[quote="RexBeckett, post:151, topic:178331"][quote="slief, post:150, topic:178331"]I just noticed that my motion is still activating the front light. It seems as though the code isn’t recognizing that it’s still day time. I’ve verified my location in settings and even made sure my logitude and latitude are correct.

I currently have:
return luup.is_night() in my luup code for the scene associated with the motion detector.

I was under the impression that the above code would work regardless of what mode I am in. Did I misunderstand that?[/quote]

Did you remove the association that you had previously set between the motion sensor and light?[/quote]

Yes. I even went back and double and triple checked to make sure the association was removed. Per the advice earlier in the thread, I set the scene to run in all modes. That wouldn’t be the issue would it? Not sure if that applied to the code for Day Night or for this more simplistic approach.

--------------------------------------------------------------------------------
RexBeckett

    Per the advice earlier in the thread, I set the scene to run in all modes. That wouldn't be the issue would it? Not sure if that applied to the code for Day Night or for this more simplistic approach.

The Lua code should prevent the scene actions from firing between sunrise and sunset - regardless of the mode. Things to check:

[ul][li]Is the Lua code really in the scene? I have been known to forget to hit the Save Lua button.[/li]
[li]Did the code get any extraneous characters during copy/paste?[/li]
[li]Are there any other scenes that could be interfering?[/li][/ul]

It would be worth trying the code that uses the Day or Night plugin rather than luup.is_night(). One advantage is that you can test it by clicking the Day or Night buttons.

--------------------------------------------------------------------------------
slief

[quote="RexBeckett, post:153, topic:178331"]

    Per the advice earlier in the thread, I set the scene to run in all modes. That wouldn’t be the issue would it? Not sure if that applied to the code for Day Night or for this more simplistic approach.

The Lua code should prevent the scene actions from firing between sunrise and sunset - regardless of the mode. Things to check:

[ul][li]Is the Lua code really in the scene? I have been known to forget to hit the Save Lua button.[/li]
[li]Did the code get any extraneous characters during copy/paste?[/li]
[li]Are there any other scenes that could be interfering?[/li][/ul]

It would be worth trying the code that uses the Day or Night plugin rather than luup.is_night(). One advantage is that you can test it by clicking the Day or Night buttons.[/quote]
The Lua code is definitely in the scene. I just snapped this picture. I assume that I have the code correct. I did change the scene to only run at night so I can test that when I get home. Instead, I guess I will probably try the alternate code since I already have the Day and Night app installed.

Just to verify, this is the code I should use in Luup to work in conjunction with Day And Night? Device ID 46 is the ID of my motion sensor.

local dID = 46
local allow = false
local status = luup.variable_get("urn:rts-services-com:serviceId:DayTime","Status",dID)
return ((status == "1") == allow)

--------------------------------------------------------------------------------
ctguess

local dID needs to be the device # of the DayNight plugin. You are getting the status of that "switch" which will be either 0 or 1 then allowing or preventing the scene to run according to the value you set for allow=false (or allow=true)

--------------------------------------------------------------------------------
slief

    local dID needs to be the device # of the DayNight plugin. You are getting the status of that "switch" which will be either 0 or 1 then allowing or preventing the scene to run according to the value you set for allow=false (or allow=true)

Would this be right then?

local dID = 46
local allow = false
local status = luup.variable_get("urn:rts-services-com:serviceId:DayTime","Status",46)
return ((status == "1") == allow)

--------------------------------------------------------------------------------
RexBeckett

    Would this be right then?

No. That change had no effect. First find the device number of your Day or Night plugin. It is shown on the device’s Advanced tab. Replace 123 with that number in the code:

local dID = 123 local allow = false -- False allows the scene to run at night local status = luup.variable_get("urn:rts-services-com:serviceId:DayTime","Status",dID) return ((status == "1") == allow)

--------------------------------------------------------------------------------
slief

[quote="RexBeckett, post:157, topic:178331"]

    Would this be right then?

No. That change had no effect. First find the device number of your Day or Night plugin. It is shown on the device’s Advanced tab. Replace 123 with that number in the code:

local dID = 123 local allow = false -- False allows the scene to run at night local status = luup.variable_get("urn:rts-services-com:serviceId:DayTime","Status",dID) return ((status == "1") == allow)[/quote]

OK… I had it right the first time. I was mistaken when I said the ID of my motion was 46. That’s the ID of my Day and Night.

So this should work.
local dID = 46
local allow = false
local status = luup.variable_get("urn:rts-services-com:serviceId:DayTime","Status",dID)
return ((status == "1") == allow)

On another note, the motion may be working now that I changed to scene to only run in night mode. I just checked the motion sensor and my lights didn’t come on when I was in front of the door. I will have to try it a bit later and see what happens. Either way I will likely switch to the Day Night code as I like the flexibility of being able to set the sunrise/sunset off set. That does bring up a question though. If I want Day to start an hour after before sunrise, do I just add the following to the offset sunrise offset field? -01:00:00 or do I need a space between +/- and the time (- 01:00:00)?

--------------------------------------------------------------------------------
dannieboiz

I want my scene to NOT run between monday - Friday from 12:30pm to 4pm can someone tell me if this is correct?

all i do is add this to the LUUP section of the scenes?

    local function checkTime() local pStart = "12:30" -- Start of time period local pEnd = "16:00" -- End of time period local allow = false -- true runs scene during period, false blocks it local hS, mS = string.match(pStart,"(%d+)%:(%d+)") local mStart = (hS * 60) + mS local hE, mE = string.match(pEnd,"(%d+)%:(%d+)") local mEnd = (hE * 60) + mE local tNow = os.date("*t") local mNow = (tNow.hour * 60) + tNow.min if mEnd >= mStart then return (((mNow >= mStart) and (mNow <= mEnd)) == allow) else return (((mNow >= mStart) or (mNow <= mEnd)) == allow) end end

    local function checkDay()
    local dFirst = 2 – Start day of period (1-7) Sunday = 1
    local dLast = 6 – End day of period (1-7) Sunday = 1
    local allow = false – true runs scene during period, false blocks it
    local tNow = os.date("*t")
    local dNow = tNow.wday
    if dLast >= dFirst then
    return (((dNow >= dFirst) and (dNow <= dLast)) == allow)
    else
    return (((dNow >= dFirst) or (dNow <= dLast)) == allow)


--------------------------------------------------------------------------------
RexBeckett

    I want my scene to NOT run between monday - Friday from 12:30pm to 4pm can someone tell me if this is correct?

Not quite. You are missing a few lines. Try this:

```Lua
local function checkTime()
local pStart = "12:30" – Start of time period
local pEnd = "16:00" – End of time period
local allow = false – true runs scene during period, false blocks it
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
local dFirst = 2 – Start day of period (1-7) Sunday = 1
local dLast = 6 – End day of period (1-7) Sunday = 1
local allow = false – true runs scene during period, false blocks it
local tNow = os.date("*t")
local dNow = tNow.wday
if dLast >= dFirst then
return (((dNow >= dFirst) and (dNow <= dLast)) == allow)
else
return (((dNow >= dFirst) or (dNow <= dLast)) == allow)
end
end

return (checkDay() or checkTime()) – Allows scene unless Mon-Fri, 12:30-16:00

```

    all i do is add this to the LUUP section of the scenes?

That should do it.

--------------------------------------------------------------------------------
RichardTSchaefer

What a great example of why you should use PLEG!

The Logic is quite obscured by the various NEGATIVE aspects of the logic.
For those that know Demorgan’s law (And I am sure Rex knows of him … he’s a brit!) applied to logic you can easily spot the logical transform and understand the logic … For the rest of you … USE PLEG and keep the logic readable!

[hr]
Logically you want:
NOT (SpecifiedHours and SpecifiedDays)

Using Demorgan’s Law this can be Transformed to:
((NOT SpecifedHours) OR (NOT SpecifiedDays))

The way the LUA code is written … CheckTime returns NOT Specified Hours
CheckDay returns NOT Specified Days

That’s why the final answer is:
return (checkDay() or checkTime())

--------------------------------------------------------------------------------
RexBeckett

    For those that know Demorgan's law (And I am sure Rex knows of him ... he's a brit!) applied to logic you can easily spot the logical transform and understand the logic ... For the rest of you .... USE PLEG and keep the logic readable!

I find that understanding De Morgan’s laws is helpful for PLEG logic too. He was indeed a Brit (although apparently not a nice chap) but the rules of logic transcend international boundaries…

The laws as I was taught them are:

The negation of a conjunction is the disjunction of the negations.
The negation of a disjunction is the conjunction of the negations.

Do what now?

The way I remember them is:

not (A and B) is equivalent to (not A) or (not B).

not (A or B) is equivalent to (not A) and (not B).

--------------------------------------------------------------------------------
dannieboiz

[quote="RexBeckett, post:160, topic:178331"]

    I want my scene to NOT run between monday - Friday from 12:30pm to 4pm can someone tell me if this is correct?

Not quite. You are missing a few lines. Try this:

```Lua
local function checkTime()
local pStart = "12:30" – Start of time period
local pEnd = "16:00" – End of time period
local allow = false – true runs scene during period, false blocks it
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
local dFirst = 2 – Start day of period (1-7) Sunday = 1
local dLast = 6 – End day of period (1-7) Sunday = 1
local allow = false – true runs scene during period, false blocks it
local tNow = os.date("*t")
local dNow = tNow.wday
if dLast >= dFirst then
return (((dNow >= dFirst) and (dNow <= dLast)) == allow)
else
return (((dNow >= dFirst) or (dNow <= dLast)) == allow)
end
end

return (checkDay() or checkTime()) – Allows scene unless Mon-Fri, 12:30-16:00

```

    all i do is add this to the LUUP section of the scenes?

That should do it.[/quote]

That seems to have done the trick. Now I realized that I need these same scenes not to run to also NOT run every night between 11pm to 8am the next morning every day.

Is this possible?

--------------------------------------------------------------------------------
RichardTSchaefer

    That seems to have done the trick. Now I realized that I need these same scenes not to run to also NOT run every night between 11pm to 8am the next morning every day.

    Is this possible?

Having the Time and Date on different dates will break the Predicate Functions as defined …

Now you should definitely use PLEG!

--------------------------------------------------------------------------------
dannieboiz

[quote="RichardTSchaefer, post:164, topic:178331"]

    That seems to have done the trick. Now I realized that I need these same scenes not to run to also NOT run every night between 11pm to 8am the next morning every day.

    Is this possible?

Having the Time and Date on different dates will break the Predicate Functions as defined …

Now you should definitely use PLEG![/quote]

I’m going to give PLEG another shot. Last time around, I couldn’t understand a damn thing I was looking at.

--------------------------------------------------------------------------------
RexBeckett

    That seems to have done the trick. Now I realized that I need these same scenes not to run to also NOT run every night between 11pm to 8am the next morning every day.

    Is this possible?

Richard is correct - once you reach a certain level of complexity then PLEG is your best option. If you do want to stick with scenes and Lua, though, it can be done. Try this:

```Lua
local function checkTime()
local pStart = "12:30" – Start of time period
local pEnd = "16:00" – End of time period
local allow = false – true runs scene during period, false blocks it
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
local dFirst = 2 – Start day of period (1-7) Sunday = 1
local dLast = 6 – End day of period (1-7) Sunday = 1
local allow = false – true runs scene during period, false blocks it
local tNow = os.date("*t")
local dNow = tNow.wday
if dLast >= dFirst then
return (((dNow >= dFirst) and (dNow <= dLast)) == allow)
else
return (((dNow >= dFirst) or (dNow <= dLast)) == allow)
end
end

local function checkNight()
local pStart = "23:00" – Start of time period
local pEnd = "08:00" – End of time period
local allow = false – true runs scene during period, false blocks it
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

return ((checkDay() or checkTime()) and checkNight()) – Allows scene unless Mon-Fri, 12:30-16:00 or any day 23:00-08:00
```

--------------------------------------------------------------------------------
dannieboiz

thanks, let me try to add it to the scene and see what happens.

Is there a way to tell immediately that there’s an error or do I just want till those times and see if it works?

--------------------------------------------------------------------------------
RichardTSchaefer

With PLEG:

Input Schedules:
SkipSchedule1 On on M-F at 12:30 PM
Off on M-F at 4:00 PM

SkipSchedule2 On M-S at 11:00 PM
Off M-S at 8:00 AM

Input Triggers
Trigger1 What ever your existing scene trigger is

Condition:
OkToRun NOT (SkipSchedule 1 or SkipSchedule2)
ValidTrigger Trigger1 and OkToRun

Put your actions on the Condition ValidTrigger

--------------------------------------------------------------------------------
dannieboiz

I think i found a simplier solution. I’m using tasker on the tablet. I can just add a task to mute the volume at those time.

--------------------------------------------------------------------------------
Michael_N_Blackwell

[quote="RexBeckett, post:160, topic:178331"]

    I want my scene to NOT run between monday - Friday from 12:30pm to 4pm can someone tell me if this is correct?

Not quite. You are missing a few lines. Try this:

```Lua
local function checkTime()
local pStart = "12:30" – Start of time period
local pEnd = "16:00" – End of time period
local allow = false – true runs scene during period, false blocks it
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
local dFirst = 2 – Start day of period (1-7) Sunday = 1
local dLast = 6 – End day of period (1-7) Sunday = 1
local allow = false – true runs scene during period, false blocks it
local tNow = os.date("*t")
local dNow = tNow.wday
if dLast >= dFirst then
return (((dNow >= dFirst) and (dNow <= dLast)) == allow)
else
return (((dNow >= dFirst) or (dNow <= dLast)) == allow)
end
end

return (checkDay() or checkTime()) – Allows scene unless Mon-Fri, 12:30-16:00

```

    all i do is add this to the LUUP section of the scenes?

That should do it.[/quote]

@RexBeckett, I assuming that the Mon-Fri condition is handle else where (not luup) within the scene in the "Trigger" portion?

--------------------------------------------------------------------------------
RexBeckett

    @RexBeckett, I assuming that the Mon-Fri condition is handle else where (not luup) within the scene in the "Trigger" portion?

The Mon-Fri condition is handled by the function checkDay(). The statement:

return (checkDay() or checkTime())     -- Allows scene unless Mon-Fri, 12:30-16:00

combines the results of checkDay() and checkTime(). This uses or rather than and because the two functions are being used in an inverted fashion.

--------------------------------------------------------------------------------
Michael_N_Blackwell

@RexBeckett, I guess it helps if one scrolls down to expose the rest of the luup code ??? thanxs

--------------------------------------------------------------------------------
Michael_N_Blackwell

@RichardTSchaefer, I’ve noticed through out several of the VERA threads that you’re pushing PLEG instead of using lua code. I’ve installed the PLEG plug-in to discover that I have to pay for it "After 30 days, unlicensed users are allowed a total of 3 PLEG and/or PLTS devices each with a max of 5 inputs and 5 conditions.
You can obtain a license that will allow you to create 4 PLEG/PLTS devices. (You can obtain as many licenses as you need). A license is $5.50+tax" I do not mind paying what it. I have a problem that I’m paying $5.50 for only 4 PLEG and/or PLTS devices, if I were to look at how many devices (+30) and wanted to create scenes with PLEG for each I would end up paying $41.25 which starts to become the most expensive piece of VERA software that I had to purchase to date. Why did you decide to limit PLEG devices to 4? one would have expected denominations of 5 or 10, with 10 devices being the preferred. It sounds like, one should use @RexBecket luup solutions to the maximum and for those items with conditions that are too complex then use PLEG sparingly to conserve PLEG devices. Don’t get me wrong I think PLEG is "the" solution that VERA should have had all along to make it a better Home Automation product, i just have a problem with your pricing schema. Mike

--------------------------------------------------------------------------------
RichardTSchaefer

Mike,it.s rare for people to need more than one or two licenses. A single licensed PLEG can handle unlimited inputs and conditions. Practically the conditions should not exceed 40 per PLEG device.
One condition is like a scene in Vera.

--------------------------------------------------------------------------------
Michael_N_Blackwell

Richard,
As I’m starting to use PLEG in "anger" I’m quickly realize that there are more dimensions than a standard VERA scene model. Question with potentially having 40 plus conditions, conceptually how does one separate the various conditions/scenes specifically whilst troubleshooting?

How did you arrive at the 4 PLEG licensing quota ? Mike

--------------------------------------------------------------------------------
RichardTSchaefer

I organize my PLEG by functionality.
One for motion/lighting, another for security/occupancy, another for environment control …

You really can"t add a lot of PLEG/PLTS devices (or any plugin for that matter) since Vera is memory constrained.

The STATUS report is very useful to debug your logic.
A naming convention also helps in understanding your logic when come back to look at it 6 months later.
I typically leave the first letter of the default names. (a letter that indicates a trigger, property, schedule, or condition.)

My pricing is VALUE based.
There is a free level that can support many simple problems. As you use more … You pay more.

Please do not use this plugin if it makes you angry. Life is to short for both of us.

--------------------------------------------------------------------------------
Michael_N_Blackwell

@RichardTSchaefer,
you made me chuckle for a Monday!. The term is an Angloise term which is implies to use whole heartily with zeal (sic) and not to be confused with the literal sense of being angry. ;D

Please do not use this plugin if it makes you angry. Life is to short for both of us.

--------------------------------------------------------------------------------
rtaxerxes

Hi,

Firstly, thanks for this great thread, I’ve referred to it many many times.

I’m using the ‘Run scene between times’ code to switch on my light if the PIR sensor is activated, to 40% before 22h00 and a 10% after 22h00. I have 2 scenes, the one is using the code to activate from sunset to 22:00, the other is from 22:00 to sunrise.

I have run this sensor and light combination before using luup.is.night and it works fine, starting from the correct sunset time and ending at the correct sunrise time. But with this new code, it seems to be ignoring the timezone and working off GMT.

I ran this code in Luatest:

print("Timezone: " .. luup.timezone .. " hours")
print("City: " .. luup.city)
print("Latitude: " .. luup.latitude)
print("Longitude: " .. luup.longitude)
local srtime = os.date("%c",luup.sunrise())
print("Sunrise: " .. srtime)
local sstime = os.date("%c",luup.sunset())
print("Sunset: " .. sstime)

and got this result:

Print output
Timezone: 2 hours
City: Cape Town
Latitude: -33.917
Longitude: 18.417
Sunrise: Thu Jul 9 07:51:03 2015
Sunset: Thu Jul 9 17:52:26 2015

So it seems location and time is correct.

This is the code from the scene that is supposed to allow the lights to come on from sunset to 22:00.

local pStart = 0         -- Start of time period, minutes offset from sunset
local pEnd = "22:00"     -- End of time period
local allow = true       -- true runs scene during period, false blocks it
local mStart = math.floor( (luup.sunset() % 86400) / 60) + pStart
local hE, mE = string.match(pEnd,"(%d+)%:(%d+)")
local mEnd = (hE * 60) + mE
local tNow = os.date("*t")
local mNow = (tNow.hour * 60) + tNow.min
if mEnd >= mStart then
     return (((mNow >= mStart) and (mNow <= mEnd)) == allow)
else 
     return (((mNow >= mStart) or (mNow <= mEnd)) == allow)
end

The other scene also seems to stop working 2 hours before sunrise. This is the code in that scene:

local pStart = "22:00"   -- Start of time period
local pEnd = 0           -- End of time period, minutes offset from sunrise
local allow = true       -- true runs scene during period, false blocks it
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

Any ideas? I checked what luup.sunset put out and did the math manually from this section of code "(luup.sunset() % 86400) / 60)" and it came to 15.8, which would make me think it is saying sunset is at 15h50ish.

--------------------------------------------------------------------------------
RichardTSchaefer

You need one more check … you need to make sure the current time of your Vera is correct.
NOTE: The Browser UI shows the time of the computer running the browser … not the time of Vera.

You can check the log file and see if it’s correct … note LOG file can be slightly delayed (seconds)

See:
http://Your.Vera.IP.Address/cgi-bin/cmh/log.sh?Device=LuaUPnP

--------------------------------------------------------------------------------
rtaxerxes

Thanks. I have just checked in the log, and the time is correct.

Last night I tried changing the offset in your code to add 120mins, and it seemed to work fine. So it does seem to be off by the timezone somehow.

--------------------------------------------------------------------------------
dgold21

I’ve been unable to get the lua code for conditional scenes to work for me (I know I’m typing something in wrong, the code fails when I test it). Would it be easier for me to just use PLEG? My needs aren’t complex, I just want to prevent a scene from running if the garage door status is closed at a particular time each day.

--------------------------------------------------------------------------------
RichardTSchaefer

It is quite easy to do this in PLEG.

--------------------------------------------------------------------------------
grrgold

    It is quite easy to do this in PLEG.

Absolutely Right!!

thanks again for all your help Ricahrd

--------------------------------------------------------------------------------
tlazay

This is a great thread. I’m hoping someone can post the code on how to accomplish this scene with two condition triggers with an AND condition.

I want the motion detector to only trigger an outdoor lighting scene when a motion detector activates only if (a) it is night and (b) the switch for one of the lights is off.

I have this working with each of the two independent triggers below for the scene, but haven’t figured out how to make a complex trigger that requires an AND for both conditions for the scene to execute.

Only trigger at night using the DayTime plugin:

local dID = 136 -- Device ID of your DayTime plugin local allow = false -- true runs scene during daytime, false runs it at night local status = luup.variable_get("urn:rts-services-com:serviceId:DayTime","Status",dID) return ((status == "1") == allow)

Only trigger if the specified switch is OFF:

local dID = 94 -- Device ID of your Z-Wave Switch local allow = false -- true runs scene if switch is on, false blocks it local status = luup.variable_get("urn:upnp-org:serviceId:SwitchPower1","Status",dID) return ((status == "1") == allow)
--------------------------------------------------------------------------------
smeghead

[quote="tlazay, post:184, topic:178331"]This is a great thread. I’m hoping someone can post the code on how to accomplish this scene with two condition triggers with an AND condition.

I want the motion detector to only trigger an outdoor lighting scene when a motion detector activates only if (a) it is night and (b) the switch for one of the lights is off.

I have this working with each of the two independent triggers below for the scene, but haven’t figured out how to make a complex trigger that requires an AND for both conditions for the scene to execute.

Only trigger at night using the DayTime plugin:

local dID = 136 -- Device ID of your DayTime plugin local allow = false -- true runs scene during daytime, false runs it at night local status = luup.variable_get("urn:rts-services-com:serviceId:DayTime","Status",dID) return ((status == "1") == allow)

Only trigger if the specified switch is OFF:

local dID = 94 -- Device ID of your Z-Wave Switch local allow = false -- true runs scene if switch is on, false blocks it local status = luup.variable_get("urn:upnp-org:serviceId:SwitchPower1","Status",dID) return ((status == "1") == allow)[/quote]

You can simply use "and" as part of the return statement and joint them together (you can do several)

local AdID = 136 -- Device ID of your DayTime plugin local Aallow = false -- true runs scene during daytime, false runs it at night local Astatus = luup.variable_get("urn:rts-services-com:serviceId:DayTime","Status",AdID) local BdID = 94 -- Device ID of your Z-Wave Switch local Ballow = false -- true runs scene if switch is on, false blocks it local Bstatus = luup.variable_get("urn:upnp-org:serviceId:SwitchPower1","Status",BdID) return ((Astatus == "1") == Aallow) and ((Bstatus == "1") == Ballow)

I hope this makes sense!

--------------------------------------------------------------------------------
marcbea

Does anyone have an idea how to have 1 scene do 2 different actions depending on which trigger is true?

Here is what I want to do.

[ul][li]Turn on outside lights at sunset[/li]
[li]Turn off outside lights at midnight[/li][/ul]

I had the scene setup with toggle state on triggers but that only works in nobody touches the light switches and changes the state it is suppose to before the next trigger happens. I want to ensure if one or both of the lights are on when sunset comes around that they are not toggled to turn off but checked and left on or the odd one out turned on. And the same goes for midnight turn them off if the are on and not on if for some reason they are off.

I know this can be done with 2 scenes but trying my hand at trying to clean up all the scenes I have an make them smarter.

Any help in this area would be great.

--------------------------------------------------------------------------------
RichardTSchaefer

You would have to this in LUA code attached to the Scene … to decide which state the light should be set.

i.e.
In the sunrise trigger set a varible
my_light_state = 0

In the sunset trigger set the variable
my_light_state = 1

In the Scene LUA call the Action to set the light level

my_light_device = 11
SwPwrSvs = "urn:upnp-org:serviceId:SwitchPower1"
luup.call_action(SwPwrSvs, "SetTarget", {newTargetValue = my_light_state}, my_light_device)
[hr]
OR Use PLEG … this can done with ONE Input Schedule and TWO Conditions
Input Schedule:
Night On at Sunset EveryDay , Off at Sunrise EveryDay

Conitions:
LightOn Night
LightOff Not Night

Attach Actions to LinghOn and LightOff

[hr]
If you want to minimize the # of conditions you can have 1 condition
LightChange Night or Not Night
Set the repeats … this will always be true … but will trigger the action when it changes.
Set the action to turn on the light …
Then goto the advanced tab for the light action and change the target value from
1
to
{(Night ? 1 : 0)}

This last case more accurately models my LUA example

--------------------------------------------------------------------------------
Michael_N_Blackwell

Richard, PLEG question, I’ve used the example below from the "PLEG Basic" file, I’ve noticed that it works well when first tripped by motion but doesn’t seem to stay on when additional motion is introduced, it simply executes the AutoOff when the timer expires. what am I missing, do I need to set the AutoOn to repeat?

additionally if I turn on the lights manually it still executes the AutoOff is there a way to either detect manual vs commanded turn on/off from a switch device, or is there another way to keep the AutoOff from executing? Mike

```Lua

Triggers
LightOn Light is turned on
Motion Motion Sensor is tripped
ItsNight DayTime indicates night

Schedules
Timer On: Self-ReTrigger Off: Interval 10:00

Conditions
AutoOn !LightOn and ItsNight and Motion and (!LightOn; Motion > 30)
KeepOn (LightOn and Motion and (LightOn; Motion)) or AutoOn
AutoOff LightOn and !Timer and (LightOn; !Timer) and (!LightOn; LightOn > 10)

Actions
AutoOn Turn Light on
KeepOn PLEG StartTimer timerName=Timer
AutoOff Turn Light off
```

--------------------------------------------------------------------------------
glith

Hello,
I hope someone can help me with this…
I would like to trigger a scene when for example the "Day and Night" plugin transitions from Day to Night.
I do not want the scene to be triggered at Night only when it is switching to Night.

The purpose for that is to have my lights turn on(if not already on) when the night comes. However, I want to manually be able to switch the lights on and off during Night-time. And if I turn off the lights during the Night I do not want the scene to trigger again(Lights off + Night).
The same during the day. Maybe it is Day-time but very dark and I want to turn the lights on. Then I do not want my scene to trigger and turn off the lights(Lights on + Day).

So I only want to trigger the scene at the exact transition between Day and Night.

I hope you understand and that someone have a solution or hints.

Thanks.

--------------------------------------------------------------------------------
aa6vh

No special coding is needed for this.

Simply bring up your scene, create a schedule for running that scene. Select the Time of day option, then select Sunset (or Sunrise).

I do not have UI7, so the actual labels may differ.

--------------------------------------------------------------------------------
glith

Thank you! That was easy enough. =)
9 days later

--------------------------------------------------------------------------------
Frosth

[quote="RexBeckett, post:4, topic:178331"]This code will enable you to set a range of temperatures within which your scene will, or will not, be run. Set tLow and tHigh to define the range. As with previous generic examples, the variable allow determines whether to allow or block the scene when the current temperature is within the specified range.

Generic Temperature Range:

local dID = 55 -- Device ID of your thermostatic/temperature sensor local tLow = 18 -- Lowest temperature of range local tHigh = 22 -- Highest temperature of range local allow = true -- true runs scene when in range, false blocks it local tCurrent = tonumber((luup.variable_get("urn:upnp-org:serviceId:TemperatureSensor1","CurrentTemperature",dID))) return (((tCurrent >= tLow) and (tCurrent <= tHigh)) == allow)

Coming next: Humidity Range[/quote]

Thank you for a great thread. I have just one question. I’m trying to use this piece of code on my Vera Edge, but seem to get stuck when using temperatures below zero. How can I achieve for example lowest range at -15 and highest range at 0?
14 days later

--------------------------------------------------------------------------------
triwave

[quote="smeghead, post:185, topic:178331"][quote="tlazay, post:184, topic:178331"]This is a great thread. I’m hoping someone can post the code on how to accomplish this scene with two condition triggers with an AND condition.

I want the motion detector to only trigger an outdoor lighting scene when a motion detector activates only if (a) it is night and (b) the switch for one of the lights is off.

I have this working with each of the two independent triggers below for the scene, but haven’t figured out how to make a complex trigger that requires an AND for both conditions for the scene to execute.

Only trigger at night using the DayTime plugin:

local dID = 136 -- Device ID of your DayTime plugin local allow = false -- true runs scene during daytime, false runs it at night local status = luup.variable_get("urn:rts-services-com:serviceId:DayTime","Status",dID) return ((status == "1") == allow)

Only trigger if the specified switch is OFF:

local dID = 94 -- Device ID of your Z-Wave Switch local allow = false -- true runs scene if switch is on, false blocks it local status = luup.variable_get("urn:upnp-org:serviceId:SwitchPower1","Status",dID) return ((status == "1") == allow)[/quote]

You can simply use "and" as part of the return statement and joint them together (you can do several)

local AdID = 136 -- Device ID of your DayTime plugin local Aallow = false -- true runs scene during daytime, false runs it at night local Astatus = luup.variable_get("urn:rts-services-com:serviceId:DayTime","Status",AdID) local BdID = 94 -- Device ID of your Z-Wave Switch local Ballow = false -- true runs scene if switch is on, false blocks it local Bstatus = luup.variable_get("urn:upnp-org:serviceId:SwitchPower1","Status",BdID) return ((Astatus == "1") == Aallow) and ((Bstatus == "1") == Ballow)

I hope this makes sense![/quote]

I was trying to figure out the same thing using the luup.is_night() function. I’m doing something wrong because the scene triggers during the day - I imagine it’s in my understanding of what the function returns as I’m getting confused between "allow" , true/false and 0/1 status …

Can somebody point me towards the right direction for running a scene to turn on a light, triggered by motion, if it’s nighttime and it’s in an OFF state? If light is already on OR if it’s daylight I don’t want to run the scene…

This is what I tried where I got no code errors but not the expected behavior:

local dID = 5 – Device ID of your Z-Wave Switch
local allow = false – true runs scene if switch is on, false blocks it
local status = luup.variable_get("urn:upnp-org:serviceId:SwitchPower1","Status",dID)
local night = luup.is_night()
return ((status == "1") == allow) and ((night == "1") == allow)

Thanks

--------------------------------------------------------------------------------
specks

Only trigger if the specified switch is OFF:

local dID = 94 -- Device ID of your Z-Wave Switch local allow = false -- true runs scene if switch is on, false blocks it local status = luup.variable_get("urn:upnp-org:serviceId:SwitchPower1","Status",dID) return ((status == "1") == allow)
I’ve used this on my my vertical blinds switch and it works fine when I manually test it. But when the scene is triggered by my Mini Mote it doesn’t run. When I then edit the scene and remove the luup code, the Mini Mote still won’t trigger the scene anymore. I then restore a previous backup, without the luup code, and it runs fine. I’m using UI7. Any thoughts?
1 month later

--------------------------------------------------------------------------------
MSW

UI7. Trying to create a scene which triggers an immediate action and a delayed action. The delayed action should execute conditionally (e.g. only if its after 4 PM) and the immediate action should execute every time the scene is triggered. Can this be done in UI7?

Said differently, I want to put a condition on a delayed action, not on the trigger and not on the scene.

Thanks!

--------------------------------------------------------------------------------
RHINESEL

[quote="MSW, post:196, topic:178331"]UI7. Trying to create a scene which triggers an immediate action and a delayed action. The delayed action should execute conditionally (e.g. only if its after 4 PM) and the immediate action should execute every time the scene is triggered. Can this be done in UI7?

Said differently, I want to put a condition on a delayed action, not on the trigger and not on the scene.

Thanks![/quote]

I think this may have to be two conditions with the second one needing the 1st condition to be true.

Condition 1… action immediate
Condition 2: cCondition1 AND sScheduleAfter1600… action delayed.

--------------------------------------------------------------------------------
MSW

[quote="RHINESEL, post:197, topic:178331"][quote="MSW, post:196, topic:178331"]UI7. Trying to create a scene which triggers an immediate action and a delayed action. The delayed action should execute conditionally (e.g. only if its after 4 PM) and the immediate action should execute every time the scene is triggered. Can this be done in UI7?

Said differently, I want to put a condition on a delayed action, not on the trigger and not on the scene.

Thanks![/quote]

I think this may have to be two conditions with the second one needing the 1st condition to be true.

Condition 1… action immediate
Condition 2: cCondition1 AND sScheduleAfter1600… action delayed.[/quote]
It is two conditions, my question was how to specify two conditions like that in UI7. I only see a way to specify a condition on a scene or on a device based trigger.

--------------------------------------------------------------------------------
RHINESEL

[quote="MSW, post:198, topic:178331"][quote="RHINESEL, post:197, topic:178331"][quote="MSW, post:196, topic:178331"]UI7. Trying to create a scene which triggers an immediate action and a delayed action. The delayed action should execute conditionally (e.g. only if its after 4 PM) and the immediate action should execute every time the scene is triggered. Can this be done in UI7?

Said differently, I want to put a condition on a delayed action, not on the trigger and not on the scene.

Thanks![/quote]

I think this may have to be two conditions with the second one needing the 1st condition to be true.

Condition 1… action immediate
Condition 2: cCondition1 AND sScheduleAfter1600… action delayed.[/quote]
It is two conditions, my question was how to specify two conditions like that in UI7. I only see a way to specify a condition on a scene or on a device based trigger.[/quote]

Oops… sorry. Thought this was regarding PLEG. Had me scratching my head.

--------------------------------------------------------------------------------
MSW

[quote="RHINESEL, post:199, topic:178331"][quote="MSW, post:198, topic:178331"][quote="RHINESEL, post:197, topic:178331"][quote="MSW, post:196, topic:178331"]UI7. Trying to create a scene which triggers an immediate action and a delayed action. The delayed action should execute conditionally (e.g. only if its after 4 PM) and the immediate action should execute every time the scene is triggered. Can this be done in UI7?

Said differently, I want to put a condition on a delayed action, not on the trigger and not on the scene.

Thanks![/quote]

I think this may have to be two conditions with the second one needing the 1st condition to be true.

Condition 1… action immediate
Condition 2: cCondition1 AND sScheduleAfter1600… action delayed.[/quote]
It is two conditions, my question was how to specify two conditions like that in UI7. I only see a way to specify a condition on a scene or on a device based trigger.[/quote]

Oops… sorry. Thought this was regarding PLEG. Had me scratching my head.[/quote]

Now that that’s sorted - can anyone advise how to do this with UI7 and Lua?

--------------------------------------------------------------------------------
renato

[quote="tlazay, post:184, topic:178331"]This is a great thread. I’m hoping someone can post the code on how to accomplish this scene with two condition triggers with an AND condition.

I want the motion detector to only trigger an outdoor lighting scene when a motion detector activates only if (a) it is night and (b) the switch for one of the lights is off.

I have this working with each of the two independent triggers below for the scene, but haven’t figured out how to make a complex trigger that requires an AND for both conditions for the scene to execute.

Only trigger at night using the DayTime plugin:

local dID = 136 -- Device ID of your DayTime plugin local allow = false -- true runs scene during daytime, false runs it at night local status = luup.variable_get("urn:rts-services-com:serviceId:DayTime","Status",dID) return ((status == "1") == allow)

Only trigger if the specified switch is OFF:

local dID = 94 -- Device ID of your Z-Wave Switch local allow = false -- true runs scene if switch is on, false blocks it local status = luup.variable_get("urn:upnp-org:serviceId:SwitchPower1","Status",dID) return ((status == "1") == allow)[/quote]

Is "Daytime" plugin the same as "Day or Night" app?

--------------------------------------------------------------------------------
mcalistair

its this one:

    appstore: MiOS Apps
    developers page: Day Or Night - Vera Plugin


--------------------------------------------------------------------------------
RichardTSchaefer

    Is "Daytime" plugin the same as "Day or Night" app?

YES!

--------------------------------------------------------------------------------
renato

[quote="RichardTSchaefer, post:203, topic:178331"]

    Is "Daytime" plugin the same as "Day or Night" app?

YES![/quote]

thanks for your prompt response Richard & mcalistair. i will try to copy the LUA codes that were posted here. i have a scene: when i opened the garage door at exactly 11:10 pm, the armed motion detector is triggered and turns on ALL the lights (inside and outside the house), and after 2 minutes they will turn off. couple of times my wife was doing the laundry in the basement and of course the light in the basement is already on. after the 2-minute delay, it turned off and you can guess what!!! she was in total darkness and screaming (i am in trouble LOL). i also want the lights to turn on when an armed sensor is triggered but only those that are off so that the light in the room where the wife is currently occupying (with lights in that room already on of cousre) will be excluded from the scene. so after the 2 minute delay, the excluded light in that particular room will not turn off. this was so simple in UI5 whereby the there was an option that after the 2-minute delay it can "revert back to previous settings". this option is no longer available in UI7.

--------------------------------------------------------------------------------
renato

[quote="smeghead, post:185, topic:178331"][quote="tlazay, post:184, topic:178331"]This is a great thread. I’m hoping someone can post the code on how to accomplish this scene with two condition triggers with an AND condition.

I want the motion detector to only trigger an outdoor lighting scene when a motion detector activates only if (a) it is night and (b) the switch for one of the lights is off.

I have this working with each of the two independent triggers below for the scene, but haven’t figured out how to make a complex trigger that requires an AND for both conditions for the scene to execute.

Only trigger at night using the DayTime plugin:

local dID = 136 -- Device ID of your DayTime plugin local allow = false -- true runs scene during daytime, false runs it at night local status = luup.variable_get("urn:rts-services-com:serviceId:DayTime","Status",dID) return ((status == "1") == allow)

Only trigger if the specified switch is OFF:

local dID = 94 -- Device ID of your Z-Wave Switch local allow = false -- true runs scene if switch is on, false blocks it local status = luup.variable_get("urn:upnp-org:serviceId:SwitchPower1","Status",dID) return ((status == "1") == allow)[/quote]

You can simply use "and" as part of the return statement and joint them together (you can do several)

local AdID = 136 -- Device ID of your DayTime plugin local Aallow = false -- true runs scene during daytime, false runs it at night local Astatus = luup.variable_get("urn:rts-services-com:serviceId:DayTime","Status",AdID) local BdID = 94 -- Device ID of your Z-Wave Switch local Ballow = false -- true runs scene if switch is on, false blocks it local Bstatus = luup.variable_get("urn:upnp-org:serviceId:SwitchPower1","Status",BdID) return ((Astatus == "1") == Aallow) and ((Bstatus == "1") == Ballow)

I hope this makes sense![/quote]
Can someone help me please. I am testing the below code during the day. with only 2 lights BdID = 90 and CdID = 91. I tested it while both lights were off; they both turned on and turned off after 1 minute when a window sensor is triggered. Now I manually turned on BdID light and triggered the window sensor again. The CdID light turned on alright. But then both lights turned off after the delay of 1 minute. The BdID light should remain on isn’t it? In the last line I changed "and" to "or" ((Cstatus) =="1" =-= Callow because when I use "and", the scene does not run at all. any help will be appreciated.

local AdID = 119 – Device ID of your DayTime plugin
local Aallow = true – true runs scene during daytime, false runs it at night
local Astatus = luup.variable_get("urn:rts-services-com:serviceId:DayTime","Status",AdID)
local BdID = 90 – Device ID of your Z-Wave Switch
local Ballow = false – true runs scene if switch is on, false blocks it
local Bstatus = luup.variable_get("urn:upnp-org:serviceId:SwitchPower1","Status",BdID)
local CdID = 91 – Device ID of your Z-Wave Switch
local Callow = false – true runs scene if switch is on, false blocks it
local Cstatus = luup.variable_get("urn:upnp-org:serviceId:SwitchPower1","Status",CdID)
return ((Astatus == "1") == Aallow) and ((Bstatus == "1") == Ballow) or ((Cstatus) == "1") == Callow)

--------------------------------------------------------------------------------
r0okey

Guys,
If you can help with below problem I would appreciate it.

I know i’m not doing this correct but I would like to combine the two following LUP codes. Essentially what I would like to accomplish is if the Day or Night module is reporting Night and multiple light switches are in OFF state allow the scene to run. below is what I have and would need to figure out out to combine the two. Both codes will work individually.
Thanks in advance.

local dID = 166 – Device ID of your DayTime plugin
local allow = false – true runs scene during daytime, false runs it at night
local status = luup.variable_get("urn:rts-services-com:serviceId:DayTime","Status",dID)
return ((status == "1") == allow)local function checkSwitch(deviceNo)
local status = luup.variable_get("urn:upnp-org:serviceId:SwitchPower1","Status",deviceNo)
return ((status or "0") == "1")
end

local switchIDs = { 38, 40, 41 }
local allow = false – true runs scene if any switch is on, false blocks it
local anyon = false
for i=1,table.maxn(switchIDs) do
anyon = anyon or checkSwitch(switchIDs[i])
end
return (anyon == allow)

--------------------------------------------------------------------------------
virginian

I’m planning to add several door and window sensors to my vera-2 setup. Is there a simple way to activate all these sensors (and deactivate as well) when I’m away by running just one scene?

I have one sensor I can experiment with:

--------------------------------------------------------------------------------
TeTTiweTTi

[quote="RexBeckett, post:1, topic:178331"]One of the most frequent questions on this forum is How do I stop my scene running when… This has been asked and answered for many different types of condition and diligent searching will often find you a solution. To help newcomers to Vera, I am posting a few of the most common scenarios in this thread.

The mechanism for preventing a scene running is simple: You insert Luup code into the scene that returns false if you want the scene blocked or true if you want to allow it to run. You can insert the code in a Trigger’s Luup event to allow or block only that trigger. You can also insert the code into the scene’s main LUUP tab where it will allow or block all triggers and manual operation. You can use a combination of both types for more complex scenarios. UI7 does not currently allow code to be attached to individual triggers so only the main Luup Code tab may be used.[/quote]

Thank you for this introduction and code snippets. A very good starting point for every luup beginner.

--------------------------------------------------------------------------------
Pietor

[quote="virginian, post:210, topic:178331"][quote="Pietor, post:209, topic:178331"][quote="virginian, post:207, topic:178331"]I’m planning to add several door and window sensors to my vera-2 setup. Is there a simple way to activate all these sensors (and deactivate as well) when I’m away by running just one scene?

I have one sensor I can experiment with:[/quote]

I can see you are on UI5, I only have seen UI7, but in UI7 you can just enable / arm these devices when in away mode. You don’t need a scene for that. Not sure if this is possible in UI5 though. Otherwise create a scene that triggers when away and insert luup code for each device to arm them.

luup.variable_set("urn:micasaverde-com:serviceId:MotionSensor1", "Armed", "1", 12)

Same for entering the home mode.

luup.variable_set("urn:micasaverde-com:serviceId:MotionSensor1", "Armed", "0", 12)

Pietor,

Thank you very much.

Should I use device # (12) or device ID (18) in the LOOP code? I executed statement for the home mode but device still shows "ARM" on the dashboard.[/quote]
Unfortunately I’m not on UI5, so can’t tell, but you should use the actual device ID. If it doesn’t work, you could try luup.call_action / SetArmed or newArmedValue. Which device from everspring are you using?

--------------------------------------------------------------------------------
RichardTSchaefer

    Should I use device # (12) or device ID (18) in the LOOP code? 

You should use 12!

The Altid is used internally, in this case it is the Z-Wave node #,

--------------------------------------------------------------------------------
virginian

[quote="Pietor, post:211, topic:178331"][quote="virginian, post:210, topic:178331"][quote="Pietor, post:209, topic:178331"][quote="virginian, post:207, topic:178331"]I’m planning to add several door and window sensors to my vera-2 setup. Is there a simple way to activate all these sensors (and deactivate as well) when I’m away by running just one scene?

I have one sensor I can experiment with:[/quote]

I can see you are on UI5, I only have seen UI7, but in UI7 you can just enable / arm these devices when in away mode. You don’t need a scene for that. Not sure if this is possible in UI5 though. Otherwise create a scene that triggers when away and insert luup code for each device to arm them.

luup.variable_set("urn:micasaverde-com:serviceId:MotionSensor1", "Armed", "1", 12)

Same for entering the home mode.

luup.variable_set("urn:micasaverde-com:serviceId:MotionSensor1", "Armed", "0", 12)

Pietor,

Thank you very much.

Should I use device # (12) or device ID (18) in the LOOP code? I executed statement for the home mode but device still shows "ARM" on the dashboard.[/quote]
Unfortunately I’m not on UI5, so can’t tell, but you should use the actual device ID. If it doesn’t work, you could try luup.call_action / SetArmed or newArmedValue. Which device from everspring are you using?[/quote]

I’m using Everspring Z-Wave Door/Window Sensor
amazon.com
Amazon.com : Everspring Z-Wave Door/Window Sensor : Tools Products : Electronics

Amazon.com : Everspring Z-Wave Door/Window Sensor : Tools Products : Electronics

So far scene returns "delivery failed".

LUUP code:

luup.variable_set("urn:micasaverde-com:serviceId:MotionSensor1", "Arm", "0", 12)

--------------------------------------------------------------------------------
virginian

[quote="RichardTSchaefer, post:212, topic:178331"]

    Should I use device # (12) or device ID (18) in the LOOP code? 

You should use 12!

The Altid is used internally, in this case it is the Z-Wave node #,[/quote]

RichardTSchaefer,

Thank you.

--------------------------------------------------------------------------------
Batiatus

Wow, lots of pages to read and make my brain explode! I’ve very new to LUUP coding (as in no idea) but I’m looking to do something very minor, I hope. In a home theater there are 2 sets of lights, 1 set around 3 sides of the room controlled by 1 switch and 1 set that highlights the screen. Right now there are 4 scenes I’d like to have slow dimming added to. For 3 it’s about lowering the light levels, each light to different amounts. The last is for bringing the lights back up. I’ve tried to follow some of the code posted regarding this but get lost as I don’t know anything at this moment and so how the definitions work sends my head in circles. Is it copy/paste the code from this post or is there more to it than that?

Thanks!

--------------------------------------------------------------------------------
RichardTSchaefer

Doing automation does require some level of scripting … and understanding the scripting language.

I wrote PLEG to help people stay out of the low level syntax details … i.e. luup.variable_get , what’s a service ID ? whats a device ID …
You use menu entries to find data you want to attach to … and give it a meaningful name to you (These are PLEG inputs).

You still need to know about boolean operators to combine input variables together (Conditions in PLEG).

Actions are pretty similar to Scene in Vera. But there are a lot of advanced features for the advanced user.
13 days later

--------------------------------------------------------------------------------
ryanoc75

[quote="Frosth, post:193, topic:178331"][quote="RexBeckett, post:4, topic:178331"]This code will enable you to set a range of temperatures within which your scene will, or will not, be run. Set tLow and tHigh to define the range. As with previous generic examples, the variable allow determines whether to allow or block the scene when the current temperature is within the specified range.

Generic Temperature Range:

local dID = 55 -- Device ID of your thermostatic/temperature sensor local tLow = 18 -- Lowest temperature of range local tHigh = 22 -- Highest temperature of range local allow = true -- true runs scene when in range, false blocks it local tCurrent = tonumber((luup.variable_get("urn:upnp-org:serviceId:TemperatureSensor1","CurrentTemperature",dID))) return (((tCurrent >= tLow) and (tCurrent <= tHigh)) == allow)

Coming next: Humidity Range[/quote]

Thank you for a great thread. I have just one question. I’m trying to use this piece of code on my Vera Edge, but seem to get stuck when using temperatures below zero. How can I achieve for example lowest range at -15 and highest range at 0?[/quote]

Just thought I would bump this question as it seems to have been lost in the thread but I too am having issues with entering temps below zero. I am using F so it will let me input 34 degrees however when I try to input a temp around 14 degrees it causes luup to fail in the PLEG. Yes, unfortunately we Canadians have to deal with such ungodly temperatures. We have an ice eater that protects our boat port and I have it running at night when power costs are cheapest, however when the temps get really cold I would like it to turn on during the day, however PLEG is not playing nice with the temps below zero. This might be a bug for Rex?

--------------------------------------------------------------------------------
asggold

After reading through a number of these examples I am hoping someone might be able to help. I have been trying to write a simple scene that does the following:

When I set my alarm (or house mode, whichever is easier) to Night, Away, or Vacation I want to receive a notification/alert if my Garage Door is open 5 minutes after setting the alarm.

This seems like a pretty simple request but I can’t seem to get anything to work. I sent an email asking for help with this to Vera Support and they sent me code (twice) that was not even close. Their code actually turned off the Alarm if the Garage Door was open (not very helpful ;D)

So, if anyone might be able to help I would greatly appreciate it.

Thanks!

--------------------------------------------------------------------------------
RichardTSchaefer

If the garage door is open and is an alarm zone, most alarms will not allow you to ARM the alarm in the first place.

Anything can be done in LUA … but you may find it easier to use PLEG …

--------------------------------------------------------------------------------
asggold

[quote="RichardTSchaefer, post:219, topic:178331"]If the garage door is open and is an alarm zone, most alarms will not allow you to ARM the alarm in the first place.

Anything can be done in LUA … but you may find it easier to use PLEG …[/quote]

The way the Garage Door is set up is a separate GDC1 device. Basically an ON/OFF switch. When the GDC1 shows ON the Garage is OPEN, OFF is CLOSED. The Alarm connection to the Vera unit is separate.

I can set the trigger on the scenario to when the Alarm panel 1 is armed. I can also set who gets the notification is what is being checked for is ?true?. I can even put a 5 min delay to look for a second trigger. The only part I CAN?T seem to figure out (which I think the Luup code must be necessary for) is how look for the GDC1 as ?On? (again, to trigger the notification).

The problem all seems to be related to the options it gives you for the GDC1 triggers. All of the options are around energy consumption, with the exception of ?when the device is turned on/off?. The problem with this is that this is looking for the ACTION of something turning on/off, not looking if the GDC1 is already on/off.

So, bottom line is finding out if there is any way to trigger the ?looking for GDC1 on/off? versus action of GDC1 being turned on/off. If so, it should be an easy build in the scenario.

--------------------------------------------------------------------------------
trouty00

Hi Rex,
need a little bit of help with probably what is hopefully a simple task, just want to check , day, time and vswitch status is on.

Have added the three functions in to startup.lua and then run the final command (bottom of code) in the luup code of the scene, my understanding is that with the final command in the luup code it will block any action defined unless all return true

```Lua
–checktime
function checkTime(pStart,pEnd)
–local pStart = "08:00" – Start of time period
–local pEnd = "22:30" – End of time period
local allow = true – true runs scene during period, false blocks it
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

–checkday
function checkDay(dFirst,dLast)
–local dFirst = 7 – Start day of period (1-7) Sunday = 1
–local dLast = 1 – End day of period (1-7) Sunday = 1
local allow = true – true runs scene during period, false blocks it
local tNow = os.date("*t")
local dNow = tNow.wday
if dLast >= dFirst then
return (((dNow >= dFirst) and (dNow <= dLast)) == allow)
else
return (((dNow >= dFirst) or (dNow <= dLast)) == allow)
end
end
–checkDay(dFirst,dLast)
checkDay(2,6)

function checkVSwitch(dID)
–local dID = 32 – Device ID of your VirtualSwitch
local allow = true – true runs scene if switch is on, false blocks it
local status = luup.variable_get("urn:upnp-org:serviceId:VSwitch1","Status",dID)
return ((status == "1") == allow)
end

–checkVSwitch(246)

–malweather scene
return checkTime("06:00","07:20") or checkDay(7,1) or checkVSwitch(246)
```

--------------------------------------------------------------------------------
Leigh

So, just picking this up, like most people! and using the example on the first page for running lights on/off between sunrise and sunset only, i thought i could set pStart and pEnd both to ‘0’ to define sunset to sunrise but i just get an error at the top of UI7 but when i thought about it i couldn’t work out how Vera would establish which was sunrise/sunset etc. Any guidance appreciated while im also trying to understand PLEG (which is now staring to make a little more sense, slowly…)

local pStart = 0         -- Start of time period, minutes offset from sunset
local pEnd = 0     -- End of time period
local allow = true       -- true runs scene during period, false blocks it
local mStart = math.floor( (luup.sunset() % 86400) / 60 ) + pStart
local hE, mE = string.match(pEnd,"(%d+)%:(%d+)")
local mEnd = (hE * 60) + mE
local tNow = os.date("*t")
local mNow = (tNow.hour * 60) + tNow.min
if mEnd >= mStart then
     return (((mNow >= mStart) and (mNow <= mEnd)) == allow)
else 
     return (((mNow >= mStart) or (mNow <= mEnd)) == allow)
end

EDIT:

So after looking at the start and stop codes in a bit more detail i have put the below together to try and get it to work only between sunset and sunrise the following day, im not getting any errors come up but would be grateful if anyone has any comments:

local pStart = 0         -- Start of time period, minutes offset from sunset
local pEnd = 0     -- End of time period
local allow = true       -- true runs scene during period, false blocks it
local mStart = math.floor( (luup.sunset() % 86400) / 60 ) + pStart
local mEnd = math.floor( (luup.sunrise() % 86400) / 60 ) + pEnd
local tNow = os.date("*t")
local mNow = (tNow.hour * 60) + tNow.min
if mEnd >= mStart then
     return (((mNow >= mStart) and (mNow <= mEnd)) == allow)
else 
     return (((mNow >= mStart) or (mNow <= mEnd)) == allow)
end

--------------------------------------------------------------------------------
BillC

[quote="RexBeckett, post:134, topic:178331"]@xgutterratx, here is some code to make your life easier. It checks all the switches specified by device number in the table switchIDs and sets the variable anyon if any of them are on. This is then checked against the setting of allow to decide whether to run the scene or not.

```Lua
local function checkSwitch(deviceNo)
local status = luup.variable_get("urn:upnp-org:serviceId:SwitchPower1","Status",deviceNo)
return ((status or "0") == "1")
end

local switchIDs = { 3, 123, 321 }
local allow = false – true runs scene if any switch is on, false blocks it
local anyon = false
for i=1,table.maxn(switchIDs) do
anyon = anyon or checkSwitch(switchIDs[i])
end
return (anyon == allow)

```

Just set the correct switch device numbers in the switchIDs table separated by commas. Note that they are numbers not strings. You can have any number of entries in this table.[/quote]

If I substitute my list of devices - local switchIDs = { 5,12,21,49,63 }, This fails, in fact if I change any of the device ID’s it fails. If I copy and paste this exactly into the code checker, it passes, but every time I put my devices in, it fails.
my entire code snippet is this

local function checkSwitch(deviceNo)
local status = luup.variable_get("urn:upnp-org:serviceId:SwitchPower1","Status",deviceNo)
return ((status or "0") == "1")
end

local switchIDs = {5,12,21,49,63 }
local allow = false – true runs scene if any switch is on, false blocks it
local anyon = false
for i=1,table.maxn(switchIDs) do
anyon = anyon or checkSwitch(switchIDs)
end
return (anyon == allow)
Can anyone see what I am doing wrong?

thanks

Bill

--------------------------------------------------------------------------------
ilikelife

@BillC,

I noticed you’re missing the counter ’ i ’ in the line below
The letter ishould be enclosed in [ ] but the post wants to turn that to italics.

anyon = anyon or checkSwitch(switchIDs[i])

I have not tried to run it, but that would cause a problem.

Hope it helps

--------------------------------------------------------------------------------
BillC

thanks, actually my code had that i, I just had to delete it when it italicized part of my post. I haven’t made many posts and it took me a while to figure out why part was italicized and some wasn’t.
Its just weird how if I copy the exact snippet from the original post, it works fine, but changing just one of the device numbers makes it fail

Bill

--------------------------------------------------------------------------------
jimim

[quote="RexBeckett, post:3, topic:178331"]We frequently want to control the time periods during which a scene may run. You would probably prefer that your bedroom light did not come on in the middle of the night when you are clocked by the motion-sensor on your way to the bathroom. ;D

Here is a generic routine that can be set to allow or block a scene in the period between two times. The start and end times can be set within the same day or either side of midnight. Both times must be in 24-hour form and entered as HH:MM. As with previous generic examples, the variable allow determines whether to allow or block the scene during the specified period.

Generic Time Period:

local pStart = "22:30" -- Start of time period local pEnd = "06:15" -- End of time period local allow = true -- true runs scene during period, false blocks it local hS, mS = string.match(pStart,"(%d+)%:(%d+)") local mStart = (hS * 60) + mS local hE, mE = string.match(pEnd,"(%d+)%:(%d+)") local mEnd = (hE * 60) + mE local tNow = os.date("*t") local mNow = (tNow.hour * 60) + tNow.min if mEnd >= mStart then return (((mNow >= mStart) and (mNow <= mEnd)) == allow) else return (((mNow >= mStart) or (mNow <= mEnd)) == allow) end

With a small modification, we can take the start time as sunset with a +/- offset. Strictly speaking, this will use the time of the next sunset so will give a slightly different result before and after sunset. The difference is about two minutes so should not be a major problem.

Sunset to End Time:

local pStart = 0 -- Start of time period, minutes offset from sunset local pEnd = "06:15" -- End of time period local allow = true -- true runs scene during period, false blocks it local mStart = math.floor( (luup.sunset() % 86400) / 60 ) + pStart local hE, mE = string.match(pEnd,"(%d+)%:(%d+)") local mEnd = (hE * 60) + mE local tNow = os.date("*t") local mNow = (tNow.hour * 60) + tNow.min if mEnd >= mStart then return (((mNow >= mStart) and (mNow <= mEnd)) == allow) else return (((mNow >= mStart) or (mNow <= mEnd)) == allow) end

We can also use sunrise as one of our times. This version has a specified start time and the end time is sunrise with a +/- minutes offset.

Start Time to Sunrise:

local pStart = "22:30" -- Start of time period local pEnd = 0 -- End of time period, minutes offset from sunrise local allow = true -- true runs scene during period, false blocks it local hS, mS = string.match(pStart,"(%d+)%:(%d+)") local mStart = (hS * 60) + mS local mEnd = math.floor( (luup.sunrise() % 86400) / 60 ) + pEnd local tNow = os.date("*t") local mNow = (tNow.hour * 60) + tNow.min if mEnd >= mStart then return (((mNow >= mStart) and (mNow <= mEnd)) == allow) else return (((mNow >= mStart) or (mNow <= mEnd)) == allow) end[/quote]

so i want to have certain lights turn on when a door is open at my house. but i only want those light s to come on between 11 pm and 6 am. and then turn back off 10 mins later.

    do i put the LUUP script in the trigger or in just the LUUP section when I make the scene or both places? under the trigger i can put the LUUP. but i can also put it under the LUUP section for a new scene.

    I got it to work during the specified time. (i used a time during now to test). but how do i make that same scene go off 10 mins later then?

is this the code i use?

local pStart = "23:00" – Start of time period
local pEnd = "06:00" – End of time period
local allow = true – true runs scene during period, false blocks it
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

--------------------------------------------------------------------------------
lotsaplots

[quote="RexBeckett, post:3, topic:178331"]We frequently want to control the time periods during which a scene may run. You would probably prefer that your bedroom light did not come on in the middle of the night when you are clocked by the motion-sensor on your way to the bathroom. ;D

Here is a generic routine that can be set to allow or block a scene in the period between two times. The start and end times can be set within the same day or either side of midnight. Both times must be in 24-hour form and entered as HH:MM. As with previous generic examples, the variable allow determines whether to allow or block the scene during the specified period.

Generic Time Period:

local pStart = "22:30" -- Start of time period local pEnd = "06:15" -- End of time period local allow = true -- true runs scene during period, false blocks it local hS, mS = string.match(pStart,"(%d+)%:(%d+)") local mStart = (hS * 60) + mS local hE, mE = string.match(pEnd,"(%d+)%:(%d+)") local mEnd = (hE * 60) + mE local tNow = os.date("*t") local mNow = (tNow.hour * 60) + tNow.min if mEnd >= mStart then return (((mNow >= mStart) and (mNow <= mEnd)) == allow) else return (((mNow >= mStart) or (mNow <= mEnd)) == allow) end

With a small modification, we can take the start time as sunset with a +/- offset. Strictly speaking, this will use the time of the next sunset so will give a slightly different result before and after sunset. The difference is about two minutes so should not be a major problem.

Sunset to End Time:

local pStart = 0 -- Start of time period, minutes offset from sunset local pEnd = "06:15" -- End of time period local allow = true -- true runs scene during period, false blocks it local mStart = math.floor( (luup.sunset() % 86400) / 60 ) + pStart local hE, mE = string.match(pEnd,"(%d+)%:(%d+)") local mEnd = (hE * 60) + mE local tNow = os.date("*t") local mNow = (tNow.hour * 60) + tNow.min if mEnd >= mStart then return (((mNow >= mStart) and (mNow <= mEnd)) == allow) else return (((mNow >= mStart) or (mNow <= mEnd)) == allow) end

We can also use sunrise as one of our times. This version has a specified start time and the end time is sunrise with a +/- minutes offset.

Start Time to Sunrise:

local pStart = "22:30" -- Start of time period local pEnd = 0 -- End of time period, minutes offset from sunrise local allow = true -- true runs scene during period, false blocks it local hS, mS = string.match(pStart,"(%d+)%:(%d+)") local mStart = (hS * 60) + mS local mEnd = math.floor( (luup.sunrise() % 86400) / 60 ) + pEnd local tNow = os.date("*t") local mNow = (tNow.hour * 60) + tNow.min if mEnd >= mStart then return (((mNow >= mStart) and (mNow <= mEnd)) == allow) else return (((mNow >= mStart) or (mNow <= mEnd)) == allow) end[/quote]

Okay, but how do you combine the two codes? For instance, start time being sunset and end time being sunrise. Also, if you are using this line of code for a trigger, does it go in the trigger luup section?

--------------------------------------------------------------------------------
RichardTSchaefer

You can replace the blocks of code as functions:

function Block1()
// Put Block 1 Code here
end

function Block2()
// Put Block 2 Code here
end

Then use boolean logic as appropriate

return Block1() and Block2()
or
return Block(1) or Block2()

To program in LUA you need to understand the execution order … which is slightly different based on how the scene is started.

For triggers:

    If Trigger LUA exists, run it, if it returns FALSE then stop, otherwise (TRUE, or nothing returned) continue.
    If Scene LUA exists, run it, if it returns FALSE then stop, otherwise (TRUE, or nothing returned) continue.
    Run Actions (Immediate and delayed)

For Manual and Schedule started scenes, skip step 1.

The decision to use Trigger or Scene level LUA is determined by your problem specification.

--------------------------------------------------------------------------------
lotsaplots

So you can’t just say "Sunset to Sunrise" and spell out the code somehow? That seems odd.

--------------------------------------------------------------------------------
RichardTSchaefer

You have more expressive/readable logic if you use PLEG instead of LUA to cancel a scene execution.

--------------------------------------------------------------------------------
lotsaplots

I think this is what I was looking for, but no one was helping me answer:

local pStart = 0 – Start of time period, minutes offset from sunset
local pEnd = 0 – End of time period
local allow = true – true runs scene during period, false blocks it
local mStart = math.floor( (luup.sunset() % 86400) / 60 ) + pStart
local mEnd = math.floor( (luup.sunrise() % 86400) / 60 ) + pEnd
local tNow = os.date("*t")
local mNow = (tNow.hour * 60) + tNow.min
if mEnd >= mStart then
return (((mNow >= mStart) and (mNow <= mEnd)) == allow)
else
return (((mNow >= mStart) or (mNow <= mEnd)) == allow)
end

This code should indicate that my scene will ONLY work between the hours of Sunset to Sunrise. Correct?

--------------------------------------------------------------------------------

z-waver.ru

Hi,

My goal is too know when motion sensor detects no motion. There is a LastTrip parameter, but there is no LastNoTrip.
How to track a time when motion sensor become not tripped?

Any help is appreciated.

--------------------------------------------------------------------------------
RichardTSchaefer

You will have to Watch the appropriate device state changes and record the times yourself … That’s what PLEG does.

--------------------------------------------------------------------------------
rafael.brasilia

Is there anyway to stop the execution of scene after it has started?

For example, say I start a scene that will turn my lights off after a 10 minute delay. But after I start the scene I change my mind and I dont want the lights to be turned off anymore.
Is there a way to stop the scene at that point?

Thanks!

--------------------------------------------------------------------------------
conchordian

[quote="rafael.brasilia, post:235, topic:178331"]Is there anyway to stop the execution of scene after it has started?

For example, say I start a scene that will turn my lights off after a 10 minute delay. But after I start the scene I change my mind and I dont want the lights to be turned off anymore.
Is there a way to stop the scene at that point?[/quote]

Use Futzle’s great countdown plugin, which has a ‘cancel’ and ‘restart’.

--------------------------------------------------------------------------------
michelhamelin

[quote="RexBeckett, post:122, topic:178331"]There have been several recent posts on this subject so I think it is worth including here. The issue is that normal scene triggers that include conditions, like temperature is lower than 18, will fire when the temperature drops below 18 but will not fire again unless it rises to or above 18 and then drops below it again.

If you use this type of trigger for a scene that includes Lua code for additional conditions - say a time period - it will not work very well. If the trigger fires outside of the allowed time period, you may not get another trigger during the period when you want to take some action.

One solution is to have your scene triggered by a periodic schedule. Then you can use Lua code from the examples in this thread to decide if you want the actions to be executed.

A better technique is to watch the variable in question and have your scene run when it changes. As above, you can then use Lua code to decide if you want to execute some action(s). Vera provides a function for this very purpose. Here is how you can use it:

```Lua
-- Set up variable-watch for device 123
luup.variable_watch("doChange123","urn:upnp-org:serviceId:TemperatureSensor1","CurrentTemperature",123)

– Process variable-watch callback for device 123. Run scene 12
function doChange123()
luup.call_action("urn:micasaverde-com:serviceId:HomeAutomationGateway1", "RunScene", {SceneNum = 12}, 0)
end
```

If this code is placed in Vera’s Startup Lua, it will set up a variable-watch for thermostat/thermometer device 123 current temperature. When this changes, scene number 12 will be executed.

To edit Startup Lua, select APPS → Develop Apps → Edit Startup Lua then paste the code on the end of any existing line and click GO. You will need to restart Vera before the changes take effect.

If you want to watch other variables, you can repeat the code for each one. You will need to change the name of the function in both the function and the luup.variable_watch(…) statements to keep them unique.

If you want to watch several variables, the following example shows how this can be done by using a table instead of multiple pieces of code. watchTable contains an entry for each variable that you want to watch. Each entry includes the ServiceId, variable name and device number that you want to watch and the scene number to be run if it changes. Each entry, other than the last one, should be terminated with a comma.

The function setWatch reads this table and sets up a watch for each entry. All the watches use the same callback function. This function, catchWatch, searches the table for a matching entry and, if it finds one, runs the specified scene.

```Lua
–
– Table entries { "ServiceID", "VariableName", DeviceNo, SceneNo }
watchTable = {
{"urn:upnp-org:serviceId:TemperatureSensor1","CurrentTemperature",123,11},
{"urn:micasaverde-com:serviceId:LightSensor1","CurrentLevel",124,12}
}
– Setup Variable Watch for each entry in watchTable
function setWatch()
for n,t in ipairs(watchTable) do
luup.variable_watch("catchWatch",t[1],t[2],t[3])
end
end
– When a watched variable changes, call the specified scene
function catchWatch(lul_device, lul_service, lul_variable, lul_value_old, lul_value_new)
for n,t in ipairs(watchTable) do
if ( (t[3] == lul_device) and (t[1] == lul_service) and (t[2] == lul_variable) ) then
luup.call_action("urn:micasaverde-com:serviceId:HomeAutomationGateway1", "RunScene", {SceneNum = t[4]}, 0)
end
end
end
– Wait for 30 seconds after restart then run setWatch

luup.call_delay("setWatch",30)
```

The last part of this code causes the setWatch function to be run 30 seconds after Vera restarts. This is to allow all devices to have completed their initialization.[/quote]

What the 0 mean

```Lua
-- Set up variable-watch for device 123
luup.variable_watch("doChange123","urn:upnp-org:serviceId:TemperatureSensor1","CurrentTemperature",123)

– Process variable-watch callback for device 123. Run scene 12
function doChange123()
luup.call_action("urn:micasaverde-com:serviceId:HomeAutomationGateway1", "RunScene", {SceneNum = 12}, 0)
end
```

--------------------------------------------------------------------------------
OldManse

This is great stuff and I want to try out the variable_watch function to run a scene, but how can I find the scene number of my scene to include in the call_action statement - it does not appear to be evident anywhere in UI7

--------------------------------------------------------------------------------
tomgru

i want to use the code suggested above to block a time from a scene running. I have several tablets that will come on and show the front door camera, based on motion (set as a trigger) from the front door camera.

My challenge is that we have a lot of spiders right now, and the tablets frequently wake up in the middle of the night with the camera view. One is next to my bed and driving my wife nuts :slight_smile:

so, i want to use the code to keep this from happening between 11pm and 6am.

My question, same as the person above (that didn’t get answered that i could see), is where to put the LUA code?

I already have code in the LUUP section that makes the tablets all switch to the video stream, and on the advanced tab have every tablet go back to the home page after a 2 minute delay.
As mentioned, on the triggers tab i have the motion sensor set to trigger the scene.

Do i put the code in the LUUP Event box in the Triggers section, or
Above the existing code in the LUUP tab?

THanks!

--------------------------------------------------------------------------------
OldManse

To answer my own question as how find the scene number, it is there next to the scene name but almost invisible as it’s very light grey colour, which is just fine - I just have to squint at the screen a bit harder.

--------------------------------------------------------------------------------
aa6vh

[quote="tomgru, post:239, topic:178331"]Do i put the code in the LUUP Event box in the Triggers section, or
Above the existing code in the LUUP tab?[/quote]

The LUA code associated with a trigger is executed before the main scene LUA code. Either code snippets can cancel running the scene. (Code associated with other triggers are not executed.)

If you place the time check code in the main LUA section, the scene can never run between the specified times. If you put it tin the trigger section, the scene will not run only when that specific trigger occurs, which would allow other triggers to continue to run the scene. Having the code in the trigger will also allow manual execution of the scene.

--------------------------------------------------------------------------------
tomgru

[quote="aa6vh, post:241, topic:178331"][quote="tomgru, post:239, topic:178331"]Do i put the code in the LUUP Event box in the Triggers section, or
Above the existing code in the LUUP tab?[/quote]

The LUA code associated with a trigger is executed before the main scene LUA code. Either code snippets can cancel running the scene. (Code associated with other triggers are not executed.)

If you place the time check code in the main LUA section, the scene can never run between the specified times. If you put it tin the trigger section, the scene will not run only when that specific trigger occurs, which would allow other triggers to continue to run the scene. Having the code in the trigger will also allow manual execution of the scene.[/quote]

Awesome explanation…thanks

--------------------------------------------------------------------------------
lotsaplots

What about a scene that can only run when you are in specific modes?

For instance: If motion sensor triggered in hallway, then turn on fireplace lights ONLY if Night Mode = true.

Not sure what the luup code would be for the condition of night mode returning true would be (also, where would I put this code… In the trigger, the action, or…??)

--------------------------------------------------------------------------------
lotsaplots

Never mind… that was a stupid question. This capability already exists without the need for coding.

--------------------------------------------------------------------------------
pbellouny

How would I separate multiple of these conditional scenes in one luup box?

Let’s say I had a humidity condition and a temperature condition, how do I separate the two pieces of code?

Thanks in advance.

--------------------------------------------------------------------------------
svanni

Great, great thread !!!
Thank you for the info.

I’m a Lua rookie, so please bear with me.
Can you provide an example of a script triggered by time of day ?
I know I can set this up thru the Schedule tab, but was wondering what the script would look like.

Thank you in advance.

--------------------------------------------------------------------------------
RichardTSchaefer

You can schedule a scene with a schedule and use LUA to cancel the scene actions using conditional LUA techniques

OR you can schedule a scene based on Device events (i.e. motion, temperature, etc) and then use conditional LUA techniques to cancel the scene if not in the correct time span.

People do not typically SCHEDULE the scene from LUA.

--------------------------------------------------------------------------------
jswim788

In the second post on this topic, there is an example of sunset to a specified time. But that didn’t work for me. I traced the problem down to this line:

local mStart = math.floor( (luup.sunset() % 86400) / 60 ) + pStart

The sunset value returned is in UTC/GMT, but the rest of the code is using local time. So I had to replace the above line with:

local tStart = os.date("*t", luup.sunset())
local mStart = (tStart.hour * 60) + tStart.min + pStart

Now it works fine. But I’m surprised I had to do this. Did anyone else notice this issue? I have the timezone and location set properly and I have power cycled the Vera since getting those set (to make sure it knows the correct time). It seems to me that the original code could only work if you were in GMT with no timezone offset.

Otherwise the example works great for me. Many thanks for all of the effort on the examples!

--------------------------------------------------------------------------------
RichardTSchaefer

Rex lives in the UK!

--------------------------------------------------------------------------------
conchordian
Oct '16

    In the second post on this topic, there is an example of sunset to a specified time. But that didn’t work for me. Did anyone else notice this issue?

Yes, http://forum.micasaverde.com/index.php/topic,27489.msg196160.html#msg196160 was how I did it.

--------------------------------------------------------------------------------
kwieto

    A better technique is to watch the variable in question and have your scene run when it changes. As above, you can then use Lua code to decide if you want to execute some action(s). Vera provides a function for this very purpose. (…) If this code is placed in Vera’s Startup Lua, it will set up a variable-watch for thermostat/thermometer device 123 current temperature. When this changes, scene number 12 will be executed.

I assume that the scene will be triggered the same way as I would press the "Run" button, so ignoring parameters defined by the triggers?
So to make it properly run I have to put these parameters into the lua code of the scene? (i.e. trigger defined as a "temperature below X")?

One addtitional question - This code will run the scene regardless if it is turned ON or OFF in the scenes section?
If yes, how to add additional condition, that if the scene is disabled, then it shouldn’t be run?

--------------------------------------------------------------------------------
rafale77

On this Latest 1.7.19 firmware, I am finding some weird things. Maybe someone can help.
I am trying to trigger a scene through a luup http command but have it executed only in house modes 2 and 4.
The scene is an announcement and a vera notification.

When I run this code:

local http = require("socket.http") local message = string.format("http://192.XX.XX.XX:5005/sayall/blah blah!") local mode = luup.attr_get "Mode" result, status = http.request(message) return (tonumber(mode)==2 or tonumber(mode)==4)

With the notification set in the scene editor, I get a notification regardless of the house mode and… the http request is also executed even though it returns a "false" (Verified in LuaTest) since I am in house mode 1.

When I run this

local http = require("socket.http") local message = string.format("http://192.xx.xx.xx:5005/sayall/blah blah") local mode = luup.attr_get "Mode" if (tonumber(mode)==2 or tonumber(mode)==4) then result, status = http.request(message) end

Then it returns a nil, The http request does not run but I still get the notification. (I saw some other people observing that the notification is not tied to the scene execution but to the trigger)
Now I remember seeing somewhere that there is a http put I can send to emit a notification within the scene lua instead of being sent by the trigger. Anyone would have it by any chance?

On a separate note, I also realized that the "this scene run in house mode" is completely ignored for lua codes. It only affects actions added by the editor. I have a scene with both and even though I am in the excluded house mode, it will send the notification, run the lua code but not execute the device action.

Edit:

I found a lot of unanswered posts asking for the Lua code to issue a mcv notification. I found a workaround. I created a scene which contains only the notification I want to issue and then execute that scene in the original scene via Lua… Not super elegant but it works. The notification is now conditional…

--------------------------------------------------------------------------------
petequintanilla

Im having trouble to setup a scene that used to run real well in UI5, Im now using Ui7 its a conditional scene for something to happen only during certain time of the day, but i can get it to work now.

the l?a im using is [code]local function checkTime()
local pStart = "16:00" – Start of time period
local pEnd = "23:29" – End of time period
local allow = true – true runs scene during period, false blocks it
local hS, mS = string.match(pStart,"(: )/(e )")
local mStart = (hS * 60) mS
local hE, mE = string.match(pEnd,"(v )S(H )")
local mEnd = (hE * 60) mE
local tNow = os.date("*t")
local mNow = (tNow.hour * 60) tNow.min
if mEnd >= mStart then
return (((mNow >= mStart) and (mNow <= mEnd)) == allow)
else
return (((mNow >= mStart) or (mNow <= mEnd)) == allow)
end
end

local function checkDay()
local dFirst = 2 – Start day of period (1-7) Sunday = 1
local dLast = 4 – End day of period (1-7) Sunday = 1
local allow = true – true runs scene during period, false blocks it
local tNow = os.date("*t")
local dNow = tNow.wday
if dLast >= dFirst then
return (((dNow >= dFirst) and (dNow <= dLast)) == allow)
else
return (((dNow >= dFirst) or (dNow <= dLast)) == allow)
end
end

return checkTime() and checkDay()[/code] could some one please advise?


--------------------------------------------------------------------------------
blairRosen

I am quite the newbie and appreciate the effort that went into your instructions. However, I am not sure how to determine the "Service ID" of my device. I have created a virtual switch named summer, and wish to run my heating scenes when it is "off" and my cooling when it is on. Is the service id the entry in the device parameters entitled "device_type"? in my case the device type of the virtual switch is urn:schemas-upnp-org:device:VSwitch:1 with a device id of 148. Would the correct syntax for Luup command to prevent running a scene if the summer switch is in the off state be:

Summer:
Code: [Select]
local dID = 148
local allow = false
local status = luup.variable_get("urn:schemas-upnp-org:device:VSwitch:1,"Status",dID)
return ((status == "1") == allow)

--------------------------------------------------------------------------------
jswim788

You can go to the advanced tab of the device and hover your mouse over the variable to get the service ID. (Yes, this is silly, and you can’t capture with a cut/paste this way. There must be a better way to do this.) Otherwise you can download the implementation file and look directly inside it. Note that the service ID is case sensitive. It’s really easy to make a mistake with it. I think you need: "urn:upnp-org:serviceId:VSwitch1"

--------------------------------------------------------------------------------
blairRosen

Thank you. Wow. They couldn’t make it less intuitive if they tried. You are right, under Variable, if I hover of over "UI7check" it says "urn:upnp-org:serviceId:VSwitch1". Another question if you don’t mind. Since I don’t want to have the scene run because I have set "local allow = false" what is the function of the line "return ((status == "1") == allow)"

In any event , based on your comments, would it be correct to say the LUUP code for not allowing the scene to run when the Summer Virtual Switch ID 148 is on would be:

Summer:
Code: [Select]
local dID = 148
local allow = false
local status = luup.variable_get("urn:upnp-org:serviceId:VSwitch1","Status",dID)
return ((status == "1") == allow)

Thanks again.

--------------------------------------------------------------------------------
phillid2

+1 to jswim788.

--------------------------------------------------------------------------------
blairRosen

Following Up: This LUUA code did not work:

Summer:
Code: [Select]
local dID = 148
local allow = false
local status = luup.variable_get("urn:upnp-org:serviceId:VSwitch1","Status",dID)
return ((status == "1") == allow)

I got the message "error in lua for scenes and events"

--------------------------------------------------------------------------------
jswim788

You are not including the "Summer: and Code" are you? Not needed and will cause a syntax error. And the "allow" stuff was to make it flexible, but it confuses more people than it helps. Finally, you need the return to say whether the scene runs or not. This is simpler:

local dID = 148 local status = luup.variable_get("urn:upnp-org:serviceId:VSwitch1","Status",dID) return (status == "0")
By the way, run this in the test code window of Vera. It should indicate a pass if your switch is off and a fail if the switch is on.

Change the above to check for "1" if you want your scene to run if the switch is on.

--------------------------------------------------------------------------------
blairRosen

You are a genius. When I removed the lines "Summer: and Code", both variations of your code post, the original and the simpler one in your last post, worked without error. Now hopefully everything will work as planned. I very much appreciate the time and effort you took to solve my problem. One suggestion to help us newbies. When posting code, perhaps the programmers should distinguish between the text necessary to run and that which is not. A great way would be by enclosing the executable lines between horizontal bars as in your last post. BTW this has convinced me that I ought to at least learn the elements of rudimentary programming for lua. Can you suggest an online tutorial?

--------------------------------------------------------------------------------
jswim788

This is a good starting point for Lua: Programming in Lua : 1 1

I’d never heard of Lua until I stumbled into it with Vera. I’m not sure where else it is used.

--------------------------------------------------------------------------------
aa6vh

    I’d never heard of Lua until I stumbled into it with Vera. I’m not sure where else it is used.

The Watchmaker app for Android smartwatches uses the LUA Language. (Makes it a lot simplier when programming the smartwatch to control Vera stuff.)
1 month later

--------------------------------------------------------------------------------
doinitsideways

[quote="jswim788, post:259, topic:178331"]You are not including the "Summer: and Code" are you? Not needed and will cause a syntax error. And the "allow" stuff was to make it flexible, but it confuses more people than it helps. Finally, you need the return to say whether the scene runs or not. This is simpler:

local dID = 148 local status = luup.variable_get("urn:upnp-org:serviceId:VSwitch1","Status",dID) return (status == "0")
By the way, run this in the test code window of Vera. It should indicate a pass if your switch is off and a fail if the switch is on.

Change the above to check for "1" if you want your scene to run if the switch is on.[/quote]

Hi there, im trying to get a scene to turn on a light when i turn my water feature on ONLY at night using the DayNight plugin.

Everytime i run the following through the test area in develop app it just says code sent succesfully, no pass or fail comment regardless of changing the status to 1 or 0

local dID = 11
local allow = false 
local status = luup.variable_get("urn:schemas-rts-services-com:device:DayTime1","Status",dID)
return ((status == "1") == allow)

Not sure what i am doing wrong. The dID is from the device ID tab for the day night plugin and the urn… is copied directly from the Advanced-Device Type field…

Cheers,
Josh

--------------------------------------------------------------------------------
jswim788

Are you sure about that URN? I may have a different version of the DayTime plugin, but mine is "urn:rts-services-com:serviceId:DayTime"

Did you hover the mouse over the "Status" variable? That’s the way to see the URN. (Or open up the plugin’s files.)

If you had the URN incorrect, it would result in the behavior you see. If this doesn’t work, I’d really recommend installing AltUI and using its debug window which is really helpful. Or you can use luup.log and check your log file. But my guess is the URN.

On a side note, if you only want to check for night, there is the built in "luup.is_night()". But I suspect you want more than that which is why you are using DayTime.

--------------------------------------------------------------------------------
RichardTSchaefer

@jswim788 is correct … you have a device URN and not a service URN … these are different things in Vera.

--------------------------------------------------------------------------------
joshhartley

Hi
I am trying to create a Scene for kids bedtime.
I have modifed the code to Dim the lights to 25% then every 30 seconds dim another 1% until it reaches 5%…
That is all working as expected but… I want to turn the lights completly off after 30 mins. What is the best way to achieve this? Should i add a delay in the scene? or a delay in the code?

```Lua

local dID = 66 – Device ID of Device to be Dimmed
luup.call_action("urn:upnp-org:serviceId:Dimming1", "SetLoadLevelTarget", {newLoadlevelTarget = "25"}, dID) – Set Device to 25% then Dim Slowly to 5%
luup.call_delay("delayDim",30,dID) – 30 Second Delay Between Dimming

function delayDim(dev)
local devno = tonumber(dev)
local lls = tonumber((luup.variable_get("urn:upnp-org:serviceId:Dimming1", "LoadLevelStatus", devno)))
local newlls = lls - 1 – 1% Increments
if newlls < 5 then newlls = 5 end – Dont Dim Lower Than 5%
luup.call_action("urn:upnp-org:serviceId:Dimming1", "SetLoadLevelTarget", {newLoadlevelTarget = newlls}, devno)
if newlls > 5 then luup.call_delay("delayDim",30,dev) end
end
```

--------------------------------------------------------------------------------
RichardTSchaefer

There is no benefit of one vs the other.
You already have LUA code … so it’s up to you.

NOTE: If Vera restarts during ANY of this … your timing logic will stop where its at.
PLEG is more robust in the context of Vera restarts.

--------------------------------------------------------------------------------
joshhartley

Thanks Richard

I usually would use PLEG as i do for 99% of my automation, But i was really struggling to work out a way to do this with PLEG?

Would a Vera restart affect the timing logic even if I put the delay in the luup code?

--------------------------------------------------------------------------------
joshhartley

Also i cant figure out how to stop the code running if the lights are dim back up to full brightness (above 25%),
It just keeps dimming until it eventually gets down to 5%.
I would like it that if i want to stop the Dimming Scene i can set the lights back to full brightness and the scene stops?\

Can anyone help me out here?

--------------------------------------------------------------------------------
RichardTSchaefer

In LUA you need to have the various events set state in global variables.
Those state variables can be accessed by the delayed code … when it wakes up to decide what to do next … i.e. stop, keep working, …
Of course certain events will start the timers … so you need to make sure you do not start a new timer if one is already running … more global state is needed to keep track of this.

You are not getting takers because this problem is beyond the scope of simple cut/paste LUA developers … it requires a good understanding of event driven code, and good design techniques to manage the global state. Note: It’s another level of complexity to make sure this logic would survive a restart.

--------------------------------------------------------------------------------
zedrally

[quote="RichardTSchaefer, post:267, topic:178331"]There is no benefit of one vs the other.
You already have LUA code … so it’s up to you.[/quote]
Richard,
What would this look like in PLEG?
The similarity to my awning problem is striking and could be the solution I was looking for.

--------------------------------------------------------------------------------
RichardTSchaefer

Input
LightLevel Current Light Level

Schedule
NextStepTimer On - Self Trigger, Off After 30 sec interval
OffTimer On - Self Trigger, Off after 30 minutes

Condition
Start What every you use to start this (i.e. some Multi-Switch turns on
KeepGoing (Start; !NextStepTimer) and !NextStepTimer and (LightLevel > 5) and (LightLevel <= 25)
AllOff (Start;OffTimer) and (LightLevel == 5)

Actions:
Start - Set light to 25 % and trigger NextStepTimer with an interval of 30 and trigger OffTimer with an interval of 30:00
KeepGoing Set Light to {(LightLevel - 1}} and trigger NextStepTimer with an interval of 30
AllOff Turn Off Light

NOTE: THis works even if Vera is reloaded!!!
With this code … if you manually change the light > 25 … this will disable the logic. You will have to retrigger the Start condition

--------------------------------------------------------------------------------
babgvant

[quote="RichardTSchaefer, post:229, topic:178331"]For triggers:

    If Trigger LUA exists, run it, if it returns FALSE then stop, otherwise (TRUE, or nothing returned) continue.
    If Scene LUA exists, run it, if it returns FALSE then stop, otherwise (TRUE, or nothing returned) continue.
    Run Actions (Immediate and delayed)

For Manual and Schedule started scenes, skip step 1.[/quote]

Is 2 the highlighted section

--------------------------------------------------------------------------------
babgvant

[quote="RichardTSchaefer, post:274, topic:178331"]Yes!

I think MCV left out the ability for LUA on triggers like they did in UI5 …
But the runtime still supports it. (i.e. Scenes migrated from UI5 to UI7 that used this feature still work).

I have not looked to see if AltUI provides scene definition with this ability.[/quote]

Thank you

--------------------------------------------------------------------------------
whited

Hi,
I am trying to execute my irrigation every other day at 3am. I am running UI5. I believe there are several ways to go about this.
I have first attempted in modifying your code and plugging it in to startup luup without success.

-- Schedule irrigation (Scene#94) every other day at 3am. local tStart = "03:00" -- Start watering local tDelay = 48 -- Delay until next time watered in hours local hS, mS = string.match(tStart,"(%d+):(%d+)") local mStart = (hS * 60) + mS local tNow = os.date("*t") local mNow = (tNow.hour * 60) + tNow.min local varD94 = 0 local lastHourD94 = 0 if (tNow.hour ~= lastHourD94) then varD94 = varD94 + 1 lastHourD94 = tNow.hour end if ((varD94 >= tDelay) and (mNow >= mStart)) then varD94 = 0 luup.call_action("urn:micasaverde-com:serviceId:HomeAutomationGateway1", "RunScene", { SceneNum= "94"}, 0) end
I then tried allowing the scene scheduled to run every day at 3am then added luup code within the scene to return false every other time. This attempt was successful when triggering the scene manually but when run through the schedule, it never gets executed. -- Bypass scene every other day if (everyOtherDayScene94 == 0) then everyOtherDayScene94 = 1 else everyOtherDayScene94 = 0 return false end

--------------------------------------------------------------------------------
RichardTSchaefer

Your assuming your Vera does not restart …
Every time it does … the next time it evaluates:
(everyOtherDayScene94 == 0)

This will be false … because everyOtherDayScene94 is undefined.
So it will skip … and if Vera restarts the next day … it will skip again.
If you really want schedule/timer reliability … you should use PLEG

--------------------------------------------------------------------------------
whited

Richard,
I agree with you 100% about vera restarting and having an undefined value.
What if I somehow used luup.variable_set?

I found this link addressing the memory problem that you and Rex discussed:
http://forum.micasaverde.com/index.php/topic,17791.0.html 1

I am still working on a solution.

--------------------------------------------------------------------------------
whited

Can a variable be saved in eePROM or to usb memory stick?

--------------------------------------------------------------------------------
whited

I believe I may have solved my problem. I like to keep my system with minimal apps so I found one I am currently using called MultiString.
It is a variable container. I changed my code to this:-- Bypass scene every other day local value = luup.variable_get("urn:upnp-org:serviceId:VContainer1", "Variable5", YourDeviceID) or "0" if (value == "0") then luup.variable_set("urn:upnp-org:serviceId:VContainer1", "Variable5", "1", YourDeviceID) else luup.variable_set("urn:upnp-org:serviceId:VContainer1", "Variable5", "0", YourDeviceID) return false endI have tested it by rebooting my Vera3 and the value in Variable5 doesn’t change, which is a good thing!

I will keep you informed if it doesn’t work after a few days. :o

--------------------------------------------------------------------------------
stefanradu5

Hello,
I am trying to make a scene for AC / HEATING that is blocked by two security sensors (Fibaro door window sensors).
When any of the windows are open …the scene must be blocked from running.
The scene’s trigger is a temperature sensor.
I inserted the Luup code in the Trigger’s "Luup Event" but i receive an error in Events and Scenes.
I am beginner and don’t fully understand the architecture of the code but I am still trying.
So, what I am doing wrong? If someone can help me, please.
Thank you.

local function windowbedroom()
local dID = 32
local allow = true 
local tripped = luup.variable_get("urn:micasaverde-com:serviceId:SecuritySensor1", "Tripped", dID)
end

local function windowlinvingroom()
local dID = 51
local allow = true 
local tripped = luup.variable_get("urn:micasaverde-com:serviceId:SecuritySensor1", "Tripped", dID)
end

return windowbedroom((status == "0") and windowlivingroom((status == "0") == allow)


--------------------------------------------------------------------------------
kwieto

If you embed the code within a function, you have to call it to make it work.
That’s the first thing.
Second is that you have the same local variables for both of your sensors (including "allow") which in turn will give you a result that only the most recent one will be considered. Then you use "status" variable in the return function, which is also not declared anywhere.

To make your code work I would do it as follow:

local tripped_bedroom = luup.variable_get("urn:micasaverde-com:serviceId:SecuritySensor1", "Tripped", 32)
local tripped_livingroom = luup.variable_get("urn:micasaverde-com:serviceId:SecuritySensor1", "Tripped", 51)
local allow = true

return (((tripped_bedroom == "0") and (tripped_livingroom == "0")) == allow)

I put device ID’s directly to the "variable_get command", as you need them only there and I don’t see a need to declare them as local variables

From logical point it works like that:
first you check the trip status of your devices and store it into local variables (data from device 32 is stored in "tripped_bedroom" and data from device 51 in "tripped_livingroom")
Then you make a check of both variables and run the scene only if both are not tripped.

--------------------------------------------------------------------------------
stefanradu5

Thank you for help and tutorial.

Now I make some logic about the syntax.
I have just inserted the code , no errors yet.

It should work if any of the windows are opened?.. or it will block the scene if only one sensor is tripped (window open).

--------------------------------------------------------------------------------
kwieto

The meaning of the logic: (tripped_bedroom == "0") and (tripped_livingroom == "0")
is that none of your sensor should be tripped to make scene run ("0" in device’s "Tripped" variable mens "untripped" while "1" is "tripped").

If you want to allow the scene to run if at least one of your sensors is untripped it should be:

(tripped_bedroom == "0") or (tripped_livingroom == "0")

For the condition that at least one of your devices is tripped (in this case the same result as code for "at least one untripped", but sometimes it makes difference) it should be:

(tripped_bedroom == "1") or (tripped_livingroom == "1")

Then the last case is the condition where you want to run your scene only if both devices are tripped:

(tripped_bedroom == "1") and (tripped_livingroom == "1")


--------------------------------------------------------------------------------
stefanradu5

The first case is the solutions.
When both sensors are "0" (both windows closed)

--------------------------------------------------------------------------------
CarsonY101

Hello all,

I’m really new to the luup coding. I would like to have certain lights turn on at sunset to sunrise when home mode is activated. I know this code will be needed:

I’m thinking it’ll include:

local pStart = 0 – Start of time period, minutes offset from sunset
local pEnd = 0 – End of time period, minutes offset from sunrise
local allow = true – true runs scene during period, false blocks it
local hS, mS = string.match(pStart,"(%d+)%:(%d+)")
local mStart = math.floor( (luup.sunset() % 86400) / 60 ) + pStart
local mEnd = math.floor( (luup.sunrise() % 86400) / 60 ) + pEnd
local tNow = os.date("*t")
local mNow = (tNow.hour * 60) + tNow.min
if mEnd >= mStart then
return (((mNow >= mStart) and (mNow <= mEnd)) == allow)
else
return (((mNow >= mStart) or (mNow <= mEnd)) == allow)
end

but where would I put if it matches then: luup.call_action("urn:micasaverde-com:serviceId:HomeAutomationGateway1","RunScene",{SceneNum="ADD_SCENE_NUMBER"},0)

and if it doesn’t then: luup.call_action("urn:micasaverde-com:serviceId:HomeAutomationGateway1","RunScene",{SceneNum="ADD_SCENE_NUMBER"},0)

Sorry for my ignorance and thank you for your time.

--------------------------------------------------------------------------------
stardobas

Sorry, I’ve a problem and need to solve;
I have vera plus UI7 and using fibaro FGMS-001;
I’ve set up a scene where when sensor detect motion turn light on; I need to play this scene only when room light is under 50 lux…
so, using you expample I add in a scene this lua script (into "add lua code " inside a scene after trigger)
this is a lua code copy/past from scene:

local dID = 94 – Device ID of your light sensor
local lLow = 0 – Lowest level of range
local lHigh = 50 – Highest level of range
local allow = true – true runs scene when in range, false blocks it
local lCurrent = tonumber((luup.variable_get("urn:micasaverde-com:serviceId:LightSensor1","CurrentLevel",dID)))
return (((lCurrent >= lLow) and (lCurrent <= lHigh)) == allow)

using this link: http://192.168.1.205:3480/data_request?id=status&output_format=xml

I obtain this:

this is a sensor I use…
How you can see the sensor light value is 60 for LUX…but the scene is played ignoring the lux value…

PLEASE HELP TO SOLVE

--------------------------------------------------------------------------------
jsauser11

Hi,
Trying to leverage some code found above to accomplish the following:
Scene trigger is Thermostat mode switches from "idle" to "cooling".
LUUP code to check multiple security sensors for the "tripped" state, and continue scene exicution if any of them are tripped (doors or windows open).
Code will then broadcast a message over Sonos to close the open doors or windows.
Here is what I have so far for the code:

local tripped_frontdoor = luup.variable_get("urn:micasaverde-com:serviceId:SecuritySensor1", "Tripped", 34)
local tripped_patiodoor = luup.variable_get("urn:micasaverde-com:serviceId:SecuritySensor1", "Tripped", 132)
local allow = true

return (((tripped_frontdoor == "1") or (tripped_patiodoor == "1")) == allow)

luup.call_action("urn:micasaverde-com:serviceId:Sonos1", "Say",
{Text="Air conditioning is turning on, please close all open doors and windows", Language="en"},
145)
But, get errors when I run it it the "test LUUP code" on my VeraPlus. Do I need to put this into a function for it to work?
Thanks for any help in coding this.

--------------------------------------------------------------------------------
rigpapa

You call_action follows an unconditional return, so will never be executed. Lua certainly won’t like that.

--------------------------------------------------------------------------------

