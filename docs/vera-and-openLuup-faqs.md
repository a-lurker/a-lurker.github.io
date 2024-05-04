# FAQs
Click the arrow heads for more info on each item.

## Vera and openLuup
<details>
<summary>Where's the forum?</summary>

The developer and users can generally be found at the [Smarthome Community Forum](https://smarthome.community/).
</details>

<details>
<summary>What version of Lua does openLuup and Vera use?</summary>

openLuup and Vera use Lua version 5.1 Here is the [manual](https://www.lua.org/manual/5.1/).

Out of interest: Lua is not an abbreviation of any particular phrase. It means moon in the Portuguese language. Lua was first developed in the Portuguese speaking country of Brazil.
</details>

<details>
<summary>How to force a Luup Engine reload?</summary>

1. Using AltUI: Tab 'Misc', select 'Reload Luup Engine'

2. Using Vera UI7: Settings ➔ Z-Wave Settings ➔ Advanced Tab: select 'Reload Engine'

3. URL call:

```html
http://openLuup_IP_address:3480/data_request?id=reload

```
4. Lua code
```lua
luup.reload()
```
</details>

<details>
<summary>How to make use of a scene controllers button push?</summary>

The trick with all these types of devices is to set a trigger on the LastSceneTime variable and then read the value of the sl_SceneActivated variable, to get which button was pressed.

So for example Hue light controller buttons return these values in sl_SceneActivated (note values not verified):

|Push type|Value|
|---|---|
|on|3|
|dim_up_hold|9|
|dim_up|8|
|dim_dwn_hold|14|
|dim_dwn|13|
|off|18|

</details>

<details>
<summary>Common application home page addresses eg Shelly, Grafana, etc</summary>

|App|URL|
|---|---|
|AltUI|http://openLuup_IP_address:3480/data_request?id=lr_ALTUI_Handler&command=home|
|openLuup console|http://openLuup_IP_address:3480/console?page=home|
|BroadLink AP|192.168.10.1|
|Grafana|http://Grafana_IP_address:3000|
|Shelly AP|http://192.168.33.1|
|Zigbee2MQTT|http://Zigbee2MQTT_IP_address::8080/#/|
|Z-Wave JS UI|http://Z-Wave_JS_UI_IP_address::8091|
</details>

## openLuup
<details>
<summary>Does openLuup require an internet connection?</summary>

openLuup does not need an internet connection. AltUI requires an internet connection to download java script libraries. However you can copy these cloud components to a local SSD. Plugins that use internet resources eg say a weather plugin or a Hue hub, will obviously need an internet connection to function.
</details>

<details>
<summary>Is openLuup a subscription service?</summary>

Absolutely not. It is open source. Users are encouraged to make a donation to cancer research. Please consider [donating](https://www.justgiving.com/DataYours/). The money goes to "Cancer Research UK".
</details>

<details>
<summary>Does openLuup auto update?</summary>

The user has full control on whether it updates automatically or not.
</details>

<details>
<summary>How to stop / start openLuup?</summary>

Assuming you are using systemd and enabled has already been run:

```bash
sudo systemctl start openluup
sudo systemctl stop openluup
```
</details>

<details>
<summary>How to enable / disable at boot with openLuup?</summary>

Assuming you are using systemd:

```bash
sudo systemctl enable openluup
sudo systemctl disable openluup
```
</details>

## Shelly
<details>
<summary>The Shelly web page at 192.168.33.1 is not accessible?</summary>

Typing the IP address into most browsers will automatically add the https:// prefix on hitting the enter key. The web page is only accessible using the http:// prefix.
</details>

## Zigbee
<details>
<summary>I deleted a Zigbee device in openLuup and now I can't recreate it.</summary>

openLuup uses the information in the payload of the 'zigbee2mqtt/bridge/devices' topic to automatically create any missing or newly paired devices. However this topic is only issued by (pairing a new device or) by unpairing and pairing an existing device. The zigbee2mqtt app can also be manually restarted to get the topic to be sent. Both methods are somewhat inconvenient.

The easiest alternative is to issue a 'zigbee2mqtt/bridge/request/restart' topic with an empty payload from say MQTT Explorer or from one of the openLuup test code test boxes. The deleted device will be recreated in openLuup. Remember to refresh the UI.

```lua
local mqtt = luup.openLuup.mqtt
mqtt.publish ("zigbee2mqtt/bridge/request/restart", "")
```

Even easier is to go the Zigbee2MQTT web page at: http://Zigbee2MQTT_IP_address:8080/#/settings/tools and push the "Restart Zigbee2MQTT" button.
</details>

## Vera
<details>
<summary>How to migrate Z-Wave from a Vera to a USB stick?</summary>

@Rafele has a detailed [explanation here](https://github.com/rafale77/Z-Way).

The method described at that link, only allows for transfers from 500 series based Veras to 500 series USB sticks. To migrate a 300 series Vera  (eg a Vera 3), you have to back up the Z-Wave data, to the Micasaverde cloud and restore it to a 500 series Vera (eg a Vera Edge).
</details>

