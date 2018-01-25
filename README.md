# ELK

#### Elasticsearch
Elasticsearch is a highly scalable, centralized data storage repository. It provides a RESTful interface in order to get data out, and put data in. Each node in an Elasticsearch cluster contributes storage, so there is no need for monolithic storage arrays and shared storage between Elasticsearch nodes.

#### Logstash
Logstash is a data ingestion and processing tool. It is flexible in nature and allows you to format data to meet your business needs, and send data to different types of endpoints. In the case of this document, we will be using Elasticsearch as the final endpoint and Redis as an intermediate endpoint (queue) to send Logstash data. We will also configure a Logstash receiver (listens for incoming data, send to queue) and a Logstash indexer (parse the data, send to the Elasticsearch index).

#### Kibana
Kibana is the user interface that interacts directly with Elasticsearch. Kibana will display the Elasticsearch indexed data in a visual manner to help end users identify trends.

## Part 1: Logstash
The Logstash event processing pipeline has three stages: inputs → filters → outputs. Inputs generate events, filters modify them, and outputs ship them elsewhere. Inputs and outputs support codecs that enable you to encode or decode the data as it enters or exits the pipeline without having to use a separate filter.

### Configuration
Our Logstash receives messages from Tower, performs aggregation and filtering, and then forwards parsed messages to Elasticsearch to be indexed. Tower's logging settings are straight-forward and support shipping messages via HTTP/HTTPS, TCP, or UDP. As you can see from the settings below, you need only to choose the logging aggregator type (Logstash) and enter the host and port of the syslog destination.

Logstash is configured via plugins, through the following file:
```
# /etc/logstash/conf.d/logstash.json

input {
  http {
    port => 5055
    tags => "tower"
  }
}

filter {
  json {
    source => "message"
    remove_field => [ "headers" ]
  }
}

output {
  elasticsearch {
    hosts => "azwus2imetwrd3:9200"
  }
}
```

### Plugins
#### Input
You use inputs to get data into Logstash. In the above Logstash config, I've setup an HTTP input on port 5055 to match Tower's HTTP log shipping method, and any messages it receives have a new field added: `tags: [ tower ]`. Additional inputs can be configured on other ports.
[See the full list of input types](https://www.elastic.co/guide/en/logstash/current/input-plugins.html).

#### Filtering
Filters are intermediary processing devices in the Logstash pipeline. You can combine filters with conditionals to perform an action on an event if it meets certain criteria. By default, Tower ships messages in a standard syslog format with the `message` field containing JSON. The above filter specifies to parse the `message` field as raw JSON.
[See the full list of plugin types](https://www.elastic.co/guide/en/logstash/current/filter-plugins.html).

#### Output
Outputs are the final phase of the Logstash pipeline. An event can pass through multiple outputs, but once all output processing is complete, the event has finished its execution. After receiving and filtering messages, Logstash outputs to Elasticsearch for daily indexing.
[See the full list of output types](https://www.elastic.co/guide/en/logstash/current/output-plugins.html).

#### Codecs
Though not used [yet] in our setup, codecs are basically stream filters that can operate as part of an input or output and enable you to easily separate the transport of your messages from the serialization process.
[See the full list of codec types](https://www.elastic.co/guide/en/logstash/current/codec-plugins.html).

### Message Results
After receiving and filtering Tower syslogs, Logstash forwards the parsed messages to Elasticsearch. Below is an example of an indexed fact collection message.

```
{
  "_index": "logstash-2017.09.12",
  "_type": "logstash",
  "_id": "AV53BdXOS2azg4aDEJWy",
  "_score": 1,
  "_source":
  {
    "play": "collect device facts and display OS version",
    "parent": null,
    "role": "network_facts",
    "stdout": "\u001b[0;32mok: [tusredrwecn3p1]\u001b[0m",
    "start_line": 231,
    "event_data": {
      "play": "collect device facts and display OS version",
      "event_loop": null,
      "remote_addr": "tusredrwecn3p1",
      "res": {
        "_ansible_no_log": false,
        "ansible_facts": {
          "hostname": "TUSREDRWECN3P1"
        },
        "changed": false
      },
      "role": "network_facts",
      "task_args": "hostname=TUSREDRWECN3P1",
      "pid": 3,
      "play_pattern": "all",
      "playbook_uuid": "703e7b8b-f22f-46f2-838d-e855f3ece15e",
      "task": "set hostname fact",
      "host": "tusredrwecn3p1",
      "task_path": "/var/lib/awx/projects/_7__ansible_network/roles/network_facts/tasks/cisco-nxos.yml:19",
      "task_uuid": "000d3af9-c8e6-2aaa-f2e7-0000000000dd",
      "play_uuid": "000d3af9-c8e6-2aaa-f2e7-000000000022",
      "playbook": "facts.yml",
      "task_action": "set_fact"
    },
    "type": "logstash",
    "uuid": "341c2bc0-c2b9-4a55-bddf-a4050003049f",
    "event_display": "Host OK",
    "end_line": 232,
    "@version": "1",
    "host": 26,
    "modified": "2017-09-12T16:57:04.000Z",
    "id": 7996,
    "logger_name": "awx.analytics.job_events",
    "event": "runner_on_ok",
    "playbook": "facts.yml",
    "level": "INFO",
    "created": "2017-09-12T16:57:04.000Z",
    "failed": false,
    "counter": 272,
    "message": "Job event data saved.",
    "tags": [ "tower" ],
    "@timestamp": "2017-09-12T16:57:04.382Z",
    "parent_uuid": "000d3af9-c8e6-2aaa-f2e7-0000000000dd",
    "task": "set hostname fact",
    "cluster_host_id": "localhost",
    "job": 82,
    "verbosity": 0,
    "host_name": "tusredrwecn3p1",
    "changed": false
  },
  "fields": {
    "modified": [
      1505235424000
    ],
    "created": [
      1505235424000
    ],
    "@timestamp": [
      1505235424382
    ]
  }
}
```

Note that data from any/all fields can be queried independently.
