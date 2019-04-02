# storj-influxdb

Logs disk usage by Storj to an influxdb database using telegraf (to be displayed by e.g. Grafana)

## Install telegraf
See [telegraf documentation](https://docs.influxdata.com/telegraf/v1.10/introduction/installation)

For ubuntu:
```bash
wget -qO- https://repos.influxdata.com/influxdb.key | sudo apt-key add -
source /etc/lsb-release
echo "deb https://repos.influxdata.com/${DISTRIB_ID,,} ${DISTRIB_CODENAME} stable" | sudo tee /etc/apt/sources.list.d/influxdb.list
sudo apt-get update && sudo apt-get install telegraf
sudo service telegraf start
```

## Editing the config
```
sudo service telegraf stop
rm /etc/telegraf/telegraf.conf
nano /etc/telegraf/telegraf.conf
```
Copy paste the config below, make sure to edit the influxdb address.
```
[agent]
  interval = "30s"
  round_interval = true
  metric_batch_size = 1000
  metric_buffer_limit = 10000
  collection_jitter = "0s"
  flush_interval = "60s"
  flush_jitter = "0s"
  precision = ""
  debug = true
  quiet = false
  logfile = ""
  hostname = ""
  omit_hostname = false
  
[[inputs.exec]]
  commands = ["sudo docker exec storagenode du -s /app/config"]
  timeout = "10s"

  data_format = "grok"
  grok_patterns = ["%{FILESIZE:used_storage:int}"]
  grok_custom_patterns = '''
FILESIZE \d*
'''

[[outputs.influxdb]]
  urls = ["http://your-address:8086"]
  database = "storj"
  # username = ""
  # password = ""
```

## Grafana graph example
```json
{
  "aliasColors": {},
  "bars": false,
  "dashLength": 10,
  "dashes": false,
  "datasource": "Storj",
  "fill": 1,
  "gridPos": {
    "h": 9,
    "w": 12,
    "x": 0,
    "y": 148
  },
  "id": 80,
  "legend": {
    "avg": false,
    "current": false,
    "max": false,
    "min": false,
    "show": true,
    "total": false,
    "values": false
  },
  "lines": true,
  "linewidth": 1,
  "links": [],
  "nullPointMode": "null",
  "percentage": false,
  "pointradius": 5,
  "points": false,
  "renderer": "flot",
  "seriesOverrides": [],
  "spaceLength": 10,
  "stack": false,
  "steppedLine": false,
  "targets": [
    {
      "groupBy": [
        {
          "params": [
            "$__interval"
          ],
          "type": "time"
        },
        {
          "params": [
            "null"
          ],
          "type": "fill"
        }
      ],
      "measurement": "exec",
      "orderByTime": "ASC",
      "policy": "default",
      "refId": "A",
      "resultFormat": "time_series",
      "select": [
        [
          {
            "params": [
              "used_storage"
            ],
            "type": "field"
          },
          {
            "params": [],
            "type": "mean"
          }
        ]
      ],
      "tags": [],
      "alias": "Disk usage"
    }
  ],
  "thresholds": [],
  "timeFrom": "1w",
  "timeRegions": [],
  "timeShift": null,
  "title": "Storj",
  "tooltip": {
    "shared": true,
    "sort": 0,
    "value_type": "individual"
  },
  "type": "graph",
  "xaxis": {
    "buckets": null,
    "mode": "time",
    "name": null,
    "show": true,
    "values": []
  },
  "yaxes": [
    {
      "format": "bytes",
      "label": null,
      "logBase": 1,
      "max": null,
      "min": null,
      "show": true
    },
    {
      "format": "short",
      "label": null,
      "logBase": 1,
      "max": null,
      "min": null,
      "show": true
    }
  ],
  "yaxis": {
    "align": false,
    "alignLevel": null
  }
}
```
