# logs-monitoring
The relay will output logs to a predefined location and the same log file will be used as a source by fluent-bit to parse and transfer data to loki which will later be used via grafana to create dashboards and alerts.

Loki can be configured to use s3 or local storage for data storage needs.

The basic flow is as:

![CentralizedRelayer(1)](https://github.com/bcsainju/logs-monitoring/assets/157450414/1ce1a42e-f525-4f30-9618-f765db7f52d4)


The fluent-bit will use the below regex to extract KV pairs from the logs and json parser to get json keys as well

Regex  ^(?<time>[a-zA-Z0-9_\/\.\-\:]*)\s+(?<log_level>[a-z]*)\t(?<message>[^\{]*)\t(?<json_log>.*)

```
[SERVICE]
    flush 1
    log_level debug
    Daemon       Off
    Buffer       False
    parsers_file /fluent-bit/etc/parsers_multiline.conf

[INPUT]
    Name   tail
    Path    /var/log/service/centralized-relay.log
    Tag    my_tag
    Parser main_log

[FILTER]
    Name   parser
    Match  *
    Reserve_Data On
    Key_Name json_log
    Parser json

[OUTPUT]
    Name loki
    Match *
    host                  loki-srv
    port                   3100
    label_keys  $name,$nid,$event_type
    labels              job=centralized_relayer
    line_format json

[OUTPUT]
    Name stdout
    Match *
```

Alerts,Dashboards and search can be done via grafana. Also, when endpoints are exposed, actions can also be initiated via grafana dashboards. 
A sample dashbaord and interaction is as follows:

![GrafanaLoki](https://github.com/bcsainju/logs-monitoring/assets/157450414/f59ae433-4d63-42e9-9b47-e8ece14ff86c)

    
