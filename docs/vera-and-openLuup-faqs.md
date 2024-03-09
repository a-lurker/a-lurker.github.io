# FAQs
Click the arrows for more info on each item.

## Vera and openLuup
<details>
<summary>What version of Lua does openLuup and Vera use?</summary>

openLuup and Vera use Lua version 5.1 Here is the [manual](https://www.lua.org/manual/5.1/).
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

## openLuup
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

## Vera
<details>
<summary>How to migrate Z-Wave from a Vera to a USB stick?</summary>

@Rafele has a detailed [explanation here](https://github.com/rafale77/Z-Way).

The method described at that link, only allows for transfers from 500 series based Veras to 500 series USB sticks. To migrate a 300 series Vera  (eg a Vera 3), you have to back up the Z-Wave data, to the Micasaverde cloud and restore it to a 500 series Vera (eg a Vera Edge).

</details>


