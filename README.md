# Ansible Playbook for a ELK Setup

The ELK-Playbooks from [robertdebock](https://galaxy.ansible.com/robertdebock) are used.

The Reverse-Proxy is from [hispanico](https://galaxy.ansible.com/hispanico/nginx_revproxy)

## Prepare

Install requirements: `$ ansible-galaxy install -r roles/requirements.yml`

## Configuration

### RSyslogd

```
 rsyslog_receiver: yes
 rsyslog_remote: "127.0.0.1"
 rsyslog_remote_port: 1514
 rsyslog_remote_tcp: yes
```

_Even use rsyslogd as a receiver and pass through the logs, or use logstash (see below) directly._


### Logstash

#### Syslogd input

```
 # Default to 514
 logstash_syslog_port: "514"
 # Default to 127.0.0.1
 logstash_syslog_host: "127.0.0.1"
```

#### Beats input

See (Get started with Beats)[https://www.elastic.co/guide/en/beats/libbeat/7.16/getting-started.html] and install one or more beats on your client.

```
 # Default to 5044
 logstash_beats_port: "5044"
 # Default to 127.0.0.1
 logstash_beats_host: "0.0.0.0"
```

#### Elasticsearch Output

```
 # Default 127.0.0.1:9200
 logstash_elasticsearch_hosts:
   - { host: "127.0.0.1", port: "9200" }
```


### Kibana

```
 kibana_log_queries: false
 kibana_log_quiet: true
 kibana_elasticsearch_url:
   - "http://localhost:9200"
```

### Elasticsearch

```
 elasticsearch_cluster_name: "elk-cluster"
 elasticsearch_node_name: "node-01"
 elasticsearch_discovery_seed_hosts:
    - "127.0.0.1"
 elasticsearch_cluster_initial_master_nodes:
    - "node-01"
 # Loglevel for GC-Log: info, debug, ...
 elasticsearch_gc_loglevel: "debug"
 # Number of gc-logifles
 elasticsearch_gc_filecount: 5
```

## Run

`$ ansible-playbook playbook.yml -i inventory/hosts -b --ask-become-pass`
