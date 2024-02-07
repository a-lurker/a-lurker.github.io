# MQTT
openLuup has built in support for and via MQTT:

## Zigbee
openLuup can subscribe to topics published by [zigbee2mqtt](https://www.zigbee2mqtt.io/). zigbee2mqtt supports hundreds of [devices](https://www.zigbee2mqtt.io/supported-devices/).
   
openLuup automatically recognises many Zigbee devices and will create native openLuup devices to make use of them. Where the device is unknown, a generic device is produced and you can write startup or scene code to act on any variables the device publishes.

## Shelly
Shellies are high performance low cost WiFi devices. They can be set up to publish MQTT messages that openLuup can make use of.

Devices available in openLuup:
   
|Device name|Function|Device model|
|---|---|---|
|Shelly 1L|Relay|SHSW-1|
|Shelly 1PM|Relay with power meter|SHSW-PM|
|Shelly ix3|3 by AC input sensors|SHIX3-1|
|Shelly 2.5|2 relays with power measurement|SHSW-25|
|Shelly Plug-S|Power plug with power measurement|SHPLG-S|
|Shelly Plug|Power plug |SHPLG2-1|
|Shelly HT|temperature & humidity sensor|SHHT-1|
|Shelly Dimmer 2|Dimmer - neutral not required|SHDM-2|

List of [Shellies](https://www.openhab.org/addons/bindings/shelly/#supported-devices)

## Tasmota

## MQTT initialisation
MQTT config such as username, passwords, etc are all set up in the [openLuup startup code](openLuup_startup_code.md).

## MQTT virtual devices

Use a virtual device for a MQTT topic not supported. This code makes use of the [Switchboard plugin](https://github.com/toggledbits/Switchboard-Vera) and is just an example.

The [Virtual Devices](https://github.com/dbochicchio/vera-VirtualDevices) plugin would be a better approach as it handles MQTT directly.

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

## Possibilities

### Other MQTT devices that could be integrated into openLuup:

- Paradox alarm using the [PAI - Paradox Alarm Interface](https://github.com/ParadoxAlarmInterface/pai)
- Zwave via MQTT. See [Z-Wave JS UI](https://zwave-js.github.io/node-zwave-js/#/README)