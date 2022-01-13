# Ansible Playbook for a ELK Setup

The RSyslog-, and Logstas-Playbooks from [robertdebock](https://galaxy.ansible.com/robertdebock) are used.

The Reverse-Proxy is from [hispanico](https://galaxy.ansible.com/hispanico/nginx_revproxy)

Elasticsearch. Metricbeat and Kibana is installed from scratch to have the newest version.

## Prepare

Install requirements: `$ ansible-galaxy install -r roles/requirements.yml`

## Configuration

### RSyslogd

```
 rsyslog_receiver: yes
 rsyslog_remote: "127.0.0.1"
 rsyslog_remote_port: 1514
 rsyslog_remote_tcp: yes
 rsyslog_remote_template: json_output
 rsyslog_rsyslog_d_files:
   01-json_output:
     content: |
       # See https://rsyslog.readthedocs.io/en/latest/configuration/index.html
       template(name="json_output" type="list" option.json="on") {
         constant(value="{")
         constant(value="\"@timestamp\":\"")     property(name="timereported" dateFormat="rfc3339")
         constant(value="\",\"@version\":\"1")
         constant(value="\",\"tag\":\"")         property(name="syslogtag" format="json")
         constant(value="\",\"relayhost\":\"")   property(name="fromhost")
         constant(value="\",\"relayip\":\"")     property(name="fromhost-ip")
         constant(value="\",\"logsource\":\"")   property(name="source")
         constant(value="\",\"host\":\"")        property(name="hostname" caseconversion="lower")
         constant(value="\",\"severity\":\"")    property(name="syslogseverity")
         constant(value="\",\"facility\":\"")    property(name="syslogfacility")
         constant(value="\",\"priority\":\"")    property(name="pri")
         constant(value="\",\"severity_label\":\"")    property(name="syslogseverity-text")
         constant(value="\",\"facility_label\":\"")    property(name="syslogfacility-text")
         constant(value="\",\"priority_label\":\"")    property(name="pri-text")
         constant(value="\",\"programname\":\"") property(name="programname")
         constant(value="\",\"procid\":\"")      property(name="procid")
         constant(value="\",\"message\":\"")     property(name="msg" format="json")
         constant(value="\"}\n")
       }
```

_Use RSyslogd to receive logs from remote systems, reformat and enrich them in json before forwarding them to Logstash._


### Logstash

#### Syslogd input

```
  # Syslog-Json listener on localhost
  logstash_syslog: { port: 1514, host: "127.0.0.1" }
```

#### Beats input

See (Get started with Beats)[https://www.elastic.co/guide/en/beats/libbeat/7.16/getting-started.html] and install one or more beats on your client.

```
  # Beast listener on all interfaces directly
  logstash_beats: { port: 5044, host: "0.0.0.0" }
```

#### Elasticsearch Output

```
 # Default 127.0.0.1:9200
  logstash_elasticsearch_hosts:
    - { host: "127.0.0.1", port: "9200" }
```


### Kibana

```
  kibana_version: "7.16.2"

  # Don't be verbose, be quiet to not flood the whole system on each click/query
  kibana_log_queries: false
  kibana_log_quiet: true

  # Elasticsearch URL
  kibana_elasticsearch_url:
    - "http://localhost:9200"
```

### Elasticsearch

```
  elasticsearch_version: "7.16.2" 
  elasticsearch_data_path: "/var/lib/elasticsearch"
  elasticsearch_cluster_name: "elk-cluster"
  elasticsearch_node_name: "node-01"
  elasticsearch_discovery_seed_hosts:
     - "127.0.0.1"
  elasticsearch_cluster_initial_master_nodes:
     - "node-01"

  # Enable xpack.security
  elasticsearch_security_enabled: true

  # Loglevel for GC-Log: info, debug, ...
  elasticsearch_gc_loglevel: "debug"

  # Number of gc-logifles
  elasticsearch_gc_filecount: 5
```

### Metricbeat

```
  metricbeat_version: "7.16.2"
  metricbeat_kibana_host: { host: "localhost", port: 5601 }
  metricbeat_kibana_username: "Username"
  metricbeat_kibana_password: "Password"

  # Elasticsearch receiver
  metricbeat_elasticsearch_hosts:
    - { host: "localhost", port: 9200 }
  metricbeat_elasticsearch_api_key: "Api-Key"
  metricbeat_elasticsearch_username: "Username"
  metricbeat_elasticsearch_password: "Password"

  # Beats Receiver
  metricbeat_logstash_hosts:
    - { host: "localhost", port: 5044 }
```

## Run

`$ ansible-playbook playbook.yml -i inventory/hosts -b --ask-become-pass`
