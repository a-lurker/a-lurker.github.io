# Graphing data
openLuup provides a variety of ways to graph data.

## Grafana
openLuup includes a Grafana Data Source API.

## Influx
Set the server address.

```lua
-- place this line in in your startup code
luup.attr_set ("openLuup.Databases.Influx", "172.16.42.129:8089")
```

## Graphite
Set the server address.

```lua
-- place this line in in your startup code
luup.attr_set ("openLuup.Historian.Graphite_UDP", "127.0.0.1:2003")
```

## Whisper
TBA

## Data historian
Set up the directory.
```lua
-- place this line in in your startup code
luup.attr_set ("openLuup.Historian.Directory","history/")
```

## Data request
Uses Google charts.
eg:

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

## DataYours
Plugin by @akbooer. Superseded by native openLuup processes.

## Datamine
A venerable plugin by @Chris Jackson
