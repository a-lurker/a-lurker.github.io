# Graphing data
openLuup provides a variety of ways to graph data.

## Influx
Set the server address.

```lua
-- place this line in in your startup code
luup.attr_set ("openLuup.Databases.Influx", "172.16.42.129:8089")
```

## Grafana
openLuup includes a Grafana Data Source API.

### Graphite Carbon databases
Set the server address.

```lua
-- place this line in in your startup code
luup.attr_set ("openLuup.Historian.Graphite_UDP", "127.0.0.1:2003")
```

### Whisper
Database

## Data historian
Set up the directory.
```lua
-- place this line in in your startup code
luup.attr_set ("openLuup.Historian.Directory","history/")
```

## DataYours
Plugin by @akbooer. Superseded by native openLuup processes.

Refer to storage-schemas.conf  in   /etc/cmh-ludl/whisper/

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

You need to make your entries on the AltUI page and then press the (red) "graph" icon button at the top (the same one you used to enter the page.)

The configuration will not be permanently saved until the next user_data.json checkpoint (might be up to 6 minutes away) or the next Luup reload (whichever the sooner.)

Uses Google charts. eg:

```http
http://openLuup_IP:3480/data_request?id=lr_render&target={temp_hot_water_pipe.w,temp_outside.w}&title=HW%20pipe&height=750&from=-y&yMin=0&yMax=50
```


## Console
A quick chart of cached data.

Refer to page /home/cache.

## AltUI
AltUI allows for any selected variable to be watched and pushed to:

- datayours
- emoncms
- IFTTT
- thingspeak

## Datamine
A venerable plugin by @Chris Jackson
