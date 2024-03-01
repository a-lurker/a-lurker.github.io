# Vera Bridge
The openLuup Vera Bridge can pretty much offload everything from Vera. Users with a more complex Z-Wave setup, may choose to leave it on Vera, in order to keep life simple. The typical outcome is a much more stable and flexible arrangement. There are plenty of Veras in the world and they can still be made good use, in particular as Z-Wave radios.


## Vera Bridge set up
When openLuup is installed so is the Vera Bridge plugin.

1. Set the remote Vera’s IP address and restart.

2. The VeraBridge plugin has an action, under the device Action tab, called GetVeraFiles. Just clicking on the action button, with no parameters, should download all the device, service, implementation files and icons to openLuup. It may take a couple of minutes. Effectively this transfers all the Vera plugins to openLuup. You can then delete them on Vera.

Note that GetVeraFiles only needs to be used once and that's during set up. Using it again will overwrite existing files of the same name, so if you’ve modified any of your device files, then make a backup first.

The bridge will also clone the room names in Vera to openLuup. If you don't want this then set the variable CloneRooms to 'false' and everything will be placed in one room. You can set this to true later on and use the GetRooms action to get the rooms. The reverse is not true. Setting the variable to false will not undo the rooms. Note that using CloneRooms can result in files being overwritten in openLuup.

## Install went badly
Delete the Vera Bridge plugin and restart the Luup Engine. You can then install the Vera Bridge again and retry.

## Day to day operation
The bridge can be connected to a Vera for as long as you want. The connection is extremely reliable.

## Vera move is complete
There may come a time when everything has been transferred to openLuup and a bridge to Vera is no longer required. Deleting the VeraBridge device will remove it and its child devices (if any). With the transfer being complete, then typically there would be no remaining child devices.

## Vera plugin compatibility with openLuup
Some plugins may not work:

- encrypted plugins simply won't run.
- ones that rely on the OpenWrt OS eg the Sonos UPnP Event Proxy
- particular to the Vera environment eg InfoViewer

## Room names and usage

### MiOS rooms
Devices which are in "No Room" remotely, are placed in the appropriately named "MiOS…" room in openLuup. This corresponds to the remote machine name ie it's serial number as printed on the label on the hardware. eg

Vera 1:  Room 'MiOS-12345678'

Vera 2:  Room 'MiOS-87654321'

### Room 101
We don't want to go here: It's your worst fear. Misc. stuff appears here when it appears to have no home.

## Multiple Veras
You can bridge more than one Vera. One bridge per Vera. Each Vera will have its own arbitrary 10,000 multiple range of device IDs assigned. openLuup devices are always in the 1 to 9,999 range.

An arbitrary example:

|Device|Range|
|---|---|
|openLuup|1 to 9,999|
|Vera 1| 10,000 to 19,999|
|Zigbee|20,000 to 29,999|
|Shellies|30,000 to 39,999|
|Vera 2|40,000 to 49,999|
|Etc||

## Vera Bridge actions

### GetVeraFiles
Copies all the required files and icons from remote vera.

It simply reads any .lzo (compressed) files in /etc/cmh-lu/ and /etc/cmh-ludl/ and copies them (uncompressed) to the files / folder. It also gets .png and .svg files from various folders (depending on Vera firmware version) and puts them into the icons/ folder.

In both cases it overwrites any existing files of the same name.

The action is really designed to get a base set of files to get you up and running. It shouldn't be needed thereafter.

### GetVeraScenes
GetVeraScenes action will copy, not just link, remote scenes. The action is not to be confused with the usual scene linking. The action makes new copies in the 100,000+ range to aid in logic transfer to openLuup.

## Vera Bridge variables
|Variable|Usage|
|---|---|
|AsyncPoll|This should be set to true to make the most of the opnLuup http async capabilities not found in Vera. Substantially improves response times.|
|BridgeScenes|.|
|CloneRooms|The CloneRooms variable should be set to true, to force all bridged devices to be in the same rooms, as on the remote Vera. Devices which are in "No Room" on the remote Vera, are placed in the appropriately named "MiOS..." room in openLuup, which corresponds to the remote machine name.|
|HouseModeMirror|See section further below.|
|LastUpdate|Shows most recent communication with the remote Vera.|
|Offset|The 10,000 multiple assigned to the Vera.|
|PK_AccessPoint|The serial number of the Vera as printed on the label on the hardware.|
|.||
|ExcludeDevices|A comma-separated list of devices to exclude from synchronisation by VeraBridge, takes precedence over IncludeDevices and ZWaveOnly. eg InfoViewer only works on Vera so you could exclude it in openLuup.|
|InludeDevices|A comma-separated list of devices to include; even if ZWaveOnly is set to true.|
|ZWaveOnly|If set to true, then only Z-Wave devices are considered by VeraBridge.|

## House Mode Mirror
This functionality assists in the interim period where some logic/scenes/plugins are still on Vera and yet others entirely on openLuup. Key variables which are controlled by the current openLuup logic may be 'mirrored' back onto devices in Vera and used as scene triggers or polled values by the logic there:

0. No mirroring.
1. Local mirrors remote machine: openLuup GETs the value from the REMOTE Vera.
2. Remote mirrors local machine: openLuup SETs the value on the REMOTE Vera.

You have to reload to make this change effective. Because of Vera’s built-in delay in changing HouseMode, mode 2 can take a while (typically 30 seconds or more) to kick in, but they should synchronise in the end.

Note that it's not easily possible to have this synchronisation work in both directions simultaneously because of a race condition that arises from the latency of Vera changing modes. You can, however, have openLuup mirror one bridged Vera's mode and have other bridged Veras which mirror that.

So this means that you can sync both Veras to the openLuup house mode, or indeed any of the two to the other one. But one machine has to be the 'master': you can’t have all responding to any machine which has a mode change.

## More Information
You an read a bit more about the Vera Bridge [here](openluup?id=more-about-verabridge).
