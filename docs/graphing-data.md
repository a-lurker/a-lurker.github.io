# Graphing data
openLuup provides a variety of ways to graph data.

## Data historian
The data historian monitors pretty much every openLuup variable and saves it to a cache. Optionally it saves the same data to disk. All graphing uses this data.

The data historian also uses the Graphite Finder standard as an interface to the WSAPI CGI graphite_cgi, which may then be accessed by dashboard applications or servers like Grafana.

Note: ONLY numeric variable values are supported by the historian.

The Historian maintains an in-memory cache of the 1000, or so, most recent values for numeric variables by default. Note that an openLuup restart will empty the cache.

To save the data to disk, enable the archiving by placing this line in the start code. The data on disk can then be used by Grafana and influx.

```lua
-- place this line in in your startup code
luup.attr_set ("openLuup.Historian.Directory","history/")
```

### Quick graph view
In the openLuup console go to "Home-->Historian-->Cache. Just click the little graph icon next to the variable of interest.

### Choosing what variables to monitor
If the on disk archiving has been enabled:

Then all the variable checkboxes will show what is being retained on disk. Note: these checkboxes are read only.

To see the variable archiving possibilities refer to the console:  Historian --> Rules page.

If the on disk archiving has been enabled the Data Historian Grafana Data Source API can be used.

Data may be easily visualised if you already have a Grafana installation.

A similar arrangement allows Influx to be used via UDP.

### Whisper database
Whisper, the industry-standard for a time-based metrics database, is utilised for storage. The data is stored in simple text files. See also DataYours below for more info.

### Influx
Set the server address.

```lua
-- place this line in in your startup code
luup.attr_set ("openLuup.Databases.Influx", "172.16.42.129:8089")
```

### Grafana
openLuup includes a Grafana Data Source API.

### Graphite Carbon databases
Set the server address.

```lua
-- place this line in in your startup code
luup.attr_set ("openLuup.Historian.Graphite_UDP", "127.0.0.1:2003")
```

## Console
A quick chart of the data cached by the Data Historian.

Refer to page /home/cache.

## AltUI
AltUI allows for any selected variable to be watched and pushed to:

- datayours
- emoncms
- IFTTT
- thingspeak

## DataYours
Plugin by @akbooer. Superseded by native openLuup processes. Deprecated.

Refer to storage-schemas.conf in /etc/cmh-ludl/whisper/

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

Files end up in /etc/cmh-ludl/whisper/

Examples: temp_outside.w.wsp, Watts_Solar.d.wsp

You need to make your entries on the AltUI page and then press the (red) "graph" icon button at the top (the same one you used to enter the page.)

The configuration will not be permanently saved until the next user_data.json checkpoint (might be up to 6 minutes away) or the next Luup reload (whichever is the sooner.)

Uses Google charts. eg:

```http
http://openLuup_IP:3480/data_request?id=lr_render&target={temp_hot_water_pipe.w,temp_outside.w}&title=HW%20pipe&height=750&from=-y&yMin=0&yMax=50
```

## Datamine
A venerable plugin by @Chris Jackson. Deprecated.
