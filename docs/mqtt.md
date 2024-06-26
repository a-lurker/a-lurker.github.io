# MQTT
openLuup has built in support for MQTT.

New to MQTT? Recommend making use of [mqtt explorer](http://mqtt-explorer.com/).

## MQTT initialisation
MQTT config such as username, passwords, etc are all set up in the [openLuup startup code](openLuup-startup-code.md).

## MQTT server
This server provides Quality of Service: QoS 0: ie at most once delivery, only. That is (according to the specification): "The message is delivered according to the capabilities of the underlying network. No response is sent by the receiver and no retry is performed by the sender. The message arrives at the receiver either once or not at all.

Plus and hash wild cards are supported.

QoS 0 is perfectly adequate for almost any home automation application operating over an internal LAN. Messages are hugely unlikely to get lost somewhere.

From openLuup Lua Startup, Scene Lua, or plugins, you can directly publish and subscribe.

To publish:

```lua
   local mqtt = luup.openLuup.mqtt
   mqtt.publish ("My/Topic/Name", "My Application message")
```

Publish a multi-line message. In this case, set the value of register xy, using a "Z-Wave JS" MQTT message:

```lua
  local topic   = "zwave/_CLIENTS/ZWAVE_GATEWAY-zwave-js-ui/api/writeValue/set"
  local message = [[
     {
       "args": [
         {
           "nodeId": 4,
           "commandClass": 38,
           "endpoint": 0,
           "property": xy
         },
         value_to_set_at_xy
       ]
     }
  ]]
  mqtt.publish (topic, message)
```

To subscribe, you use a standard Luup request handler:

```lua
function MyMQTThandler (topic, message)
   -- your handler code here
end

luup.register_handler ("MyMQTThandler", "mqtt:My/Topic/Name")
```

You can register any number of handlers to different topics, or a single handler to many topics. Currently only one wildcard topic is allowed, '#', which subscribes you to all user messages.

## Zigbee
openLuup can subscribe to topics published by [zigbee2mqtt](https://www.zigbee2mqtt.io/). zigbee2mqtt supports hundreds of [devices](https://www.zigbee2mqtt.io/supported-devices/).

openLuup automatically recognises many Zigbee devices and will create native openLuup devices to make use of them. Where the device is unknown, a generic device is produced and you can write startup or scene code to act on any variables the device publishes.

Got a `Hue hub`? Set up zigbee2mqtt and you won't need it!

Newly discovered Zigbee devices are automatically placed in the "Zigbee room".

## Tasmota
Example [Tasmota](https://tasmota.github.io/docs/) setup [here](https://smarthome.community/topic/506/openluup-tasmota-mqtt-bridge/47).

## Shelly
Shellies are high performance low cost WiFi devices. They can be set up to publish MQTT messages that openLuup can make use of. Newly discovered Shelly devices are automatically placed in the "Shellies room".

Devices available in openLuup:

Shelly Gen 1:

|Device name|Function|Device model|
|---|---|---|
|Shelly 1L|Relay|SHSW-1|
|Shelly 1PM|Relay with power meter|SHSW-PM|
|Shelly ix3|3 by AC input sensors|SHIX3-1|
|Shelly 2.5|2 relays with power measurement|SHSW-25|
|Shelly Plug-S|Power plug with power measurement|SHPLG-S|
|Shelly Plug|Power plug |SHPLG2-1|
|Shelly HT|Temperature & humidity sensor|SHHT-1|
|Shelly Dimmer 2|Dimmer - neutral not required|SHDM-2|
|Shelly RGBW2|12V or 24V RGBW LED driver|SHRGBW2|

Shelly Plus:

|Device name|Function|
|---|---|
|Shellyplusi4|4-digital inputs|
|Shellyplus2pm|2 relays with power measurement|
|Shellyplusht|Temperature & humidity sensor with display|
|Shellyplusuni|Solid state relays with potential free contacts, pulse counter, analog and 2 digital i/ps|

Shelly Pro - all DIN rail mountable:

|Device name|Function|
|---|---|
|Shellypro3em|Can use for three or single phase. Monitors voltage, power & energy.|

Not on the list above? A generic device is created. Newly discovered Shelly devices are automatically placed in the "Shellies room". In the MQTT settings; the "Generic status update over MQTT" checkbox does not need to be checked (as per the manufacturer's default).

List of [Shellies](https://www.openhab.org/addons/bindings/shelly/#supported-devices)

## Shelly like commands ➔ openLuup
External apps can publish MQTT commands that openLuup will respond to. They are similar to the Shelly MQTT commands:

```http
    Use device Id or name for NNN
    openLuup_IP_address:3480/relay/NNN?turn=[on/off/toggle]

    Use scene Id or name for NNN
    openLuup_IP_address:3480/scene/NNN?
```

## UDP ➔ MQTT bridge
The openLuup MQTT QoS 0 server incorporates a UDP ➔ MQTT bridge, which is uni-directional.

This means that any machine which can send a UDP datagram (and, frankly, that should be everything) can publish to subscribers of the openLuup MQTT server without the need for any MQTT client library. The use of UDP means that there is very little overhead to sending the data, and that there is absolutely no possibility of blocking the host process. The UDP 'fire and forget' concept also meshes very well with the MQTT QoS 0 service level.

The UDP datagram format for publishing is:

```text
TopicName/=/ApplicationMessage
```
By way of example, here's a tiny bit of Lua Startup code, which has been tested on a Vera. The code sets up a device variable watch for all variables, in specific services, in all devices.

```lua
local socket = require "socket"
local sock = socket.udp()
sock: setpeername("172.16.42.121", "2883")

-- send to openLuup IP and port
local pk = (luup.attr_get "PK_AccessPoint")
pk = "Vera-" .. pk: match "%d+"

function test_watch (d,s,v,o,n)
   local msg = table.concat ({pk,'update',d,s,v,'=',n}, '/')
   sock: send (msg)
end

local services = {
   -- list any services you want to watch here
   "urn:micasaverde-com:serviceId:EnergyMetering1",
   "urn:upnp-org:serviceId:SwitchPower1",
 }

for _,sid in ipairs(services) do
  luup.variable_watch ("test_watch", sid)
end
```

When one of the watched variables changes, a UDP datagram is sent to the UDP -> MQTT bridge and converted into a PUBLISH. Example datagram text:

```text
Vera-PK_AccessPoint/update/dev_number/serviceId/var_name/=/value
```

For example:
```text
    topic: Vera-35104005/update/203/urn:micasaverde-com:serviceId:EnergyMetering1/KWH
    message: 18900.9060
```
Here's an MQTT Explorer view the result:

IMAGE HERE Screenshot 2021-03-17 at 16.52.34.png

The updates are effectively instantaneous. There are plenty of basic devices, which could benefit from this transport mechanism. eg Arduino PCBs.

## MQTT virtual sensors
It is sometimes the case you have a device that sends lots of information but you are only interested in some of the info. You don't need a plugin that creates a plethora of child devices, many of which you will never make use of and clog up up your user interface.

In this case you can make use of the [Virtual Sensor](https://github.com/toggledbits/VirtualSensor) plugin. The plugin allows to you dial up what sort sensor it is eg temperature, switch state, etc and it will create a virtusl sensor to suit your needs.

## MQTT virtual devices
If there is no support for your device, you can create a virtual device. See the [Virtual Devices](https://github.com/dbochicchio/vera-VirtualDevices?tab=readme-ov-file#mqtt-support-version-30) plugin.

## Example MQTT handler
This code handles some arbitrary MQTT message:

This code makes use of the [Switchboard plugin](https://github.com/toggledbits/Switchboard-Vera).

MQTT server port:
```lua
   -- set the MQTT server port the in startup code
   luup.attr_set("openLuup.MQTT.Port", 1883)

   -- optional debugging
   luup.attr_set("openLuup.MQTT.DEBUG", true)
```
MQTT handler:
```lua
-- MQTT handler in the openLuup startup code
-- function needs to be global
function MQTThandler (topic, message, messageNumber)
    log('topic = '..topic)

    -- need to set up your json decoder reference here
    local info = m_json.decode(message)

    if (messageNumber == 1) then
        local device_temperature = info.device_temperature
        local illuminance        = info.illuminance
        local occupancy          = info.occupancy

        log('device_temperature = '..tostring(device_temperature))
        log('illuminance = '       ..tostring(illuminance))
        log('occupancy = '         ..tostring(occupancy))

        -- making use of the virtual device plugin
        -- need to set up your IDs here
        if (occupancy) then
            luup.call_action(SID.SWITCH, "SetTarget", {newTargetValue="1"}, ID.VIRTUAL_SW_MOTION)
            luup.call_action(SID.SWITCH, "SetTarget", {newTargetValue="1"}, ID.VIRTUAL_LIGHT_1)
        else
            luup.call_action(SID.SWITCH, "SetTarget", {newTargetValue="0"}, ID.VIRTUAL_SW_MOTION)
            luup.call_action(SID.SWITCH, "SetTarget", {newTargetValue="0"}, ID.VIRTUAL_LIGHT_1)
        end
    end
end
```

Register the MQTT handler:
```lua
   -- register the MQTT handler in the openLuup startup code
   local topic = 'mqtt:zigbee2mqtt/0x54ef4220004eaa56'   -- motion detector
   luup.register_handler ('MQTThandler', topic, 1)
```

## Monitoring and control of openLuup via MQTT
Monitoring and control of devices via MQTT, represents an intention to move away from the need for any UPnP Vera-style requests or polling.

### Request the status of an openluup variable
The MQTT server subscribes to the topic openLuup/query. Any MQTT "thing", can use this get an initial value from openLuup, as needed.

The "thing" would publish a message with the format: devNo.serviceId.variable eg:

```text
    topic: openLuup/query
    message: 2.openLuup.Memory_Mb
```

openLuup will immediately reply with the status of the requested variable:

```text
    topic: openLuup/update/2/openLuup/Memory_Mb
    message: 8.7 (or whatever the value is)
```

Notice the formatting difference between the message and the returned topic ('.' instead of '/'). This is intentional because it is in line with openLuup's existing dev.srv.var notation and reminding you that it's not a part of the query topic, but it goes in the message This avoids the need for openLuup to use a wildcard subscription.

### openluup status publishing
Using the built-in MQTT QoS 0 server, two new sets of PUBLISH topics may be sent by openLuup:

- topics named openLuup/update/device_no/short_service_id/var_name (where short_service_id is the useful part of the serviceId – ie. the last alphanumeric bit.) These are sent as the named variable changes value (ie. not if it is set to the same value.) The message text is simply the new string content of the variable.

- a single topic named openLuup/status is sent every few seconds (user definable) cycling in a round-robin way through all of the devices. The JSON-formatted text message contains the current status of all the variables in that particular device. This, then, is the 'push' replacement for long-polling with a /data_request?id=status HTTP request.

### openluup device control
Simple Shelly-style HTTP requests can be used to control switches and lights. These will be extended to dimmers and other controls in due course. TBA

```text
openLuup_IP:3480/relay/NNN?turn=[on/off/toggle] allows device Id or name for NNN

openLuup_IP:3480/scene/NNN? allows scene Id or name for NNN
```

### JSON format of device status report
openLuup.MQTT.PublishDeviceStatus:

The JSON format of a single device status report, which is very compact, is that of nested tables indexed by strings: device number / short ServiceId / variable name; such that flattening the table appropriately would give the MQTT topic used in openLuup/update messages (aside from that prefix.) For example, for openLuup (device #2):

```json
{"2":{
    "HaDevice1":{
      "CommFailure":"0",
      "CommFailureTime":"0"
    },
    "altui1":{
      "DisplayLine1":"8Mb, 0.1%cpu, 0.1days",
      "DisplayLine2":"[Home]"
    },
    "openLuup":{
      "CpuLoad":"0.1",
      "HouseMode":"1",
      "Memory_Mb":"7.6",
      "StartTime":"2021-03-11T14:53:02",
      "Uptime_Days":"0.06",
      "Version":"v21.3.11",
      "Vnumber":"210311"
    }
  }}
```

With a round-robin interval of 2 seconds, a moderately-sized setup of 150 devices would be cycled through in five minutes, providing a sanity check that no transient device variable updates have been missed.

## Lots of MQTT possibilities

### Other MQTT devices that could be integrated into openLuup:

- List of [awesome MQTT resources](https://github.com/akbooer/awesome-mqtt?tab=readme-ov-file#awesome-mqtt)
- Paradox alarm using the [PAI - Paradox Alarm Interface](https://github.com/ParadoxAlarmInterface/pai)
- Zwave via MQTT. See [Z-Wave JS UI](https://zwave-js.github.io/node-zwave-js/#/README)
- [Sonos 2 mqtt](https://sonos2mqtt.svrooij.io/) via MQTT.
- We like MQTT - got any more good ideas? Let us know on the [smarthome forum](https://smarthome.community/).
