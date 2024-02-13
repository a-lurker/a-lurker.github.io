# Graphing data
openLuup provides a variety of ways to graph data.

## Data historian
The data historian monitors pretty much every openLuup variable and saves it to a cache. Optionally it saves the same data to disk. All graphing uses this data.

The data historian also uses the Graphite Finder standard as an interface to the WSAPI CGI graphite_cgi, which may then be accessed by dashboard applications or servers like Grafana.

Note: ONLY numeric variable values are supported by the historian.

The Historian maintains an in-memory cache of the 1024 most recent values for numeric variables by default. Note that an openLuup restart will empty the cache.

The act of inserting this line into the startup code, enables the archiving process, ie saving the data to disk. The data can then be utilised by Grafana and influx.

```lua
-- place this line in in your startup code
-- it's possible to place the data at any other favourite
-- spot, noting the path is relative to cmh-ludl/
luup.attr_set ("openLuup.Historian.Directory","history/")
```

Do a Luup engine restart. The Whisper files for every variable is then created in the history/ directory. Do another restart to get the archiving running.

- Different data types have different sample rates and retention policies. For example, temperature data is sampled every 20 minutes, initially, but reduced in a number of stages to once per day after a year, and battery levels are just sampled once per day.
- Default total on-disk archive duration is 10 years. Typical file sizes, one per variable (containing multiple resolution data) are about 300 Kbyte.

- Whisper file headers are cached and renaming/changing the file mid-stream will completely trash it. Stop openLuup first.

### Quick graph view
In the openLuup console go to "Home-->Historian-->Cache. Just click the little graph icon next to the variable of interest. Uses Google Charts, so an internet connection is required.

### Historian variables retained
If the on disk archiving has been enabled the Historian automatically starts archiving the majority of the openLuup variables.

In the Console --> Historian--> cache: The checkboxes will show what variables are being retained on disk. Note: these checkboxes are **READ ONLY**.

To see the variable archiving possibilities refer to the console:  Historian --> Rules page.

If the on disk archiving has been enabled the Data Historian Grafana Data Source API can be used.

Data may be easily visualised if you already have a Grafana installation.

A similar arrangement allows Influx to be used via UDP.

### Grafana
Grafana is a data visulation tool. You can connect to a database and then view the data. 

openLuup includes a Grafana Data Source API, so  in Grafana choose a Graphite connection and point the URL to:

- With openLuup running on same device as Grafana enter:
    http://localhost:3480

- With openLuup on a different device from Grafana enter:
   http://openLuup_IP_address:3480

### Whisper database
Whisper, the industry-standard for a time-based metrics database, is utilised for storage. The data is stored in simple text files.

The choices for the aggregation functions are:
- average
- sum
- last
- max
- min

You can set up a [schema for different variables](https://graphite.readthedocs.io/en/latest/config-carbon.html#storage-schemas-conf) that define the time interval and the maximum number of points to retain.

Refer to files `storage-schemas.conf` and `storage-aggregation.conf`. [This post](https://community.ezlo.com/t/openluup-data-historian/199464/120) is also useful.

Refer also to the file [../cmh-ludl/openLuup/openLuup/servertables.lua](https://github.com/akbooer/openLuup/blob/f5db7abc964595ba4db6f5f373918cae6ec312b8/openLuup/servertables.lua#L229)

Setting a up Whisper file to monitor a variable is a bit cryptic:

A one-off file create can be done with code like this run in Lua Test:

```lua
local whisper = require "openLuup.whisper"

local filename = "history/0.377.Weather1.TodayHighTemp.wsp"
local archives = "10m:7d,1h:30d,3h:1y,1d:10y"

whisper.create (filename,archives,0)

print((whisper.info (filename)).retentions)
```

The key parameters you will need to change are filename and archives. This also assumes that the historian folder is `history/`.

The filename syntax is `0.deviceNumber.shortServiceId.variableName`, and the archives are in the usual Whisper syntax. The example make sense for most measurements, with a 10 minute basic sample rate, but you could, for example, switch that to 5.

If the code is successful, it will echo the archive retentions read from the newly created file, eg:

10m:7d,1h:30d,3h:1y,1d:10y

You have to reload for the historian to start using this new set of archives.

### Graphite Carbon databases
Set the server address.

```lua
-- place this line in in your startup code
luup.attr_set ("openLuup.Historian.Graphite_UDP", "127.0.0.1:2003")
```

### InfluxDB
Set the server address.

```lua
-- place this line in in your startup code
luup.attr_set ("openLuup.Databases.Influx", "172.16.42.129:8089")
```

## AltUI
AltUI allows for any selected variable to be watched and pushed to:

- datayours
- emoncms
- IFTTT
- thingspeak

## DataYours
Plugin by @akbooer. Superseded by native openLuup processes. **Deprecated**.

Refer to storage-schemas.conf in `/etc/cmh-ludl/whisper/`

DataYours will register itself with AltUI, if installed, and be available as a Data Storage Provider under the graphing menu for each variable.

A simple naming convention with a one-letter suffix will, by default, configure a data archive for that variable to be:

```text
    .d - one minute resolution for one day
    .w - five minute resolution for one week
    .m - twenty minute resolution for one month (30 days)
    .q - one hour resolution for one quarter (90 days)
    .y - six hour resolution for one year

    .w1 - one minute resolution for one week
    .m1 - five minute resolution for one month (30 days)
    .q1 - twenty minute resolution for one quarter (90 days)
    .y1 - one hour resolution for one year

    yMin - y axis minimum
    yMax - y axis maximum
```

Files end up in `/etc/cmh-ludl/whisper/`

However the file location can be changed in the plugin. Refer to variable LOCAL_DATA_DIR, which defaults to `whisper/`

Examples: `temp_outside.w.wsp`, `Watts_Solar.d.wsp`

You need to make your entries on the AltUI page and then press the (red) "graph" icon button at the top (the same one you used to enter the page.)

The configuration will not be permanently saved until the next user_data.json checkpoint (might be up to 6 minutes away) or the next Luup reload (whichever is the sooner.)

Uses Google Charts, so an internet connection is required. Example URL to chart:

```http
http://openLuup_IP:3480/data_request?id=lr_render&target={temp_hot_water_pipe.w,temp_outside.w}&title=HW%20pipe&height=750&from=-y&yMin=0&yMax=50
```

## Datamine
A venerable plugin by @Chris Jackson. **Deprecated**.
