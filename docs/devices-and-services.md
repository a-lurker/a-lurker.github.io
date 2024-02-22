# Service ids
Service ids are just used to address individual collections of variables and functions / actions.

## Anatomy of a service id
The service id consist of four parts separated by colons. They are case sensitive. eg:

urn:upnp-org:serviceId:SwitchPower1

1. `urn` ➔ "uniform resource name" Never changes.

2. `upnp-org` ➔ Name space. Typically a URL with dots replaced by dashes but can be anything. It's meant to indicate what organisation is associated with the service id. Plugins will typically have one "invented" by the plugin developer.

3. `serviceId` ➔ Indicates this "resoure" is a "serviceId" and always stays the same.

4. `SwitchPower1` ➔ The actual service. It's name is generally indicative of what it's for. It always has "1" on the end. Part of the spec apparently.

## openLuup enhancement
In openLuup all the services listed below can be shortened by leaving out the first three elements like so:

urn:upnp-org:serviceId:SwitchPower1 ➔ SwitchPower1

## List of some available services
The following is a very short summary of some of the more utilised services. Each service has an associated device eg Temperature sensor device.

Many of these serviceIds will have additional variables and actions that are rarely used.

Plugins that create other devices will add their own services.

In the user interface (AltUI) you can place the cursor on any variable or action and the tool tip will show the associated serviceId.

### openLuup
|urn:akbooer-com:serviceId:openLuupBridge1||
|---|---|
|Variable|See plugin|
|Action|See plugin|

### Misc.
|urn:upnp-org:serviceId:AltAppStore1||
|---|---|
|Variable|See plugin|
|Action|See plugin|

|urn:upnp-org:serviceId:altui1||
|---|---|
|Variable|See plugin|
|Action|See plugin|

### upnp.org
|urn:upnp-org:serviceId:Dimming1||
|---|---|
|Variable|LoadLevelStatus: 0 to 100|
|Action|SetLoadLevelTarget, \{newLoadlevelTarget = level}, deviceId|

|urn:upnp-org:serviceId:SwitchPower1||
|---|---|
|Variable|Status: 0 or 1|
|Action|SetTarget, \{newTargetValue = level}, deviceId|

|urn:upnp-org:serviceId:TemperatureSensor1||
|---|---|
|Variable|CurrentTemperature, deviceId|
|Action|-|

### Vera
|urn:micasaverde-com:serviceId:EnergyMetering1||
|---|---|
|Variable|Watts: power being consumed|
|Variable|KWH: energy consumed|
|Action||

|urn:micasaverde-com:serviceId:GenericSensor1||
|---|---|
|Variable|?|
|Action|?|

|urn:micasaverde-com:serviceId:HaDevice1||
|---|---|
|Variable|BatteryLevel: 0 to 100 %|
|Action|-|

|urn:micasaverde-com:serviceId:HomeAutomationGateway1||
|---|---|
|Variable|-|
|Action|-|

|urn:micasaverde-com:serviceId:HumiditySensor1||
|---|---|
|Variable|CurrentLevel: 0 to 100 %|
|Action|-|

|urn:micasaverde-com:serviceId:LightSensor1||
|---|---|
|Variable|CurrentLevel: 0 to 100 Lux|
|Action|-|

|urn:micasaverde-com:serviceId:SceneController1||
|---|---|
|Variable||
|Action||

|urn:micasaverde-com:serviceId:SecuritySensor1||
|---|---|
|Variable|Armed: 0 or 1|
|Variable|Tripped: 0 or 1|
|Action||

|urn:micasaverde-com:serviceId:ZWaveDevice1||
|---|---|
|Variable|-|
|Action|-|