logstash role
===============

This role is used to add log aggregation and forwarding capability to
a given server.

Configuration
--------------

### Inventory

Normally, we put all aggregation on a single machine. However,
it is possible to decouple them, and provision separate machines
for any aggregators we might have. This example demonstrate how
to create a dedicated logstash aggregator.

```ini
[logstash:children]
logstash-somedc-prod

[logstash-somedc-prod]
logstash1.somedc.prod         ansible_ssh_host=10.0.1.111

[logstash-somedc-prod:vars]
# Specify the port used by logstash to receive syslog data
logstash_syslog_port = 514

# Support collectd codec
# https://www.elastic.co/guide/en/logstash/current/plugins-codecs-collectd.html
collectd_forward_to_logstash = false

# Use ElasticSearch output
# https://www.elastic.co/guide/en/logstash/current/plugins-outputs-elasticsearch.html
use_elasticsearch = false

# Depending on the environment you will be deploying to,
# auto-discovery might not work (e.g. in environments where
# multicast is not working). Your ElasticSearch cluster
# configuration should tell you this (by default, auto-discovery
# is on; the inventory should tell you if it is turned off)
#
# If the cluster is auto-discovery capable, set:
#
# logstash_elasticsearch_cluster
#
# if it is not, set:
#
# logstash_elasticsearch_host
#
# Don't worry, in both cases full cluster discovery will be made,
# and logstash should end using all nodes
#
logstash_elasticsearch_cluster=elasticsearch1-somedc-prod
logstash_elasticsearch_host=node1.elasticsearch1.somedc.prod
```

### Integration with external services

When provisioning with this role, you can also add
[Papertrail](https://papertrailapp.com),
[Loggly](https://www.loggly.com),
[Logentries](https://logentries.com/) and
[Graylog](https://www.graylog.org/) support.

As you should not add any credentials to your inventory,
it should be done by using `--extra-vars` as described below.

#### Papertail

```bash
--extra-vars="use_papertrail=false" \
--extra-vars="papertrail_host=CUSTOM_HOST" \
--extra-vars="papertrail_port=CUSTOM_PORT"
```

#### Loggly

```bash
--extra-vars="use_loggly=false" \
--extra-vars="loggly_key=SOME_KEY"
```

#### Logentries

```bash
--extra-vars="use_logentries=false" \
--extra-vars="logentries_port=CUSTOM_PORT"
```

#### Graylog

```bash
--extra-vars="use_graylog=false" \
--extra-vars="graylog_host=HOST" \
--extra-vars="graylog_port=CUSTOM_PORT"
```

### The log aggregation process

By default, the described inventory would forward all its syslog
to this server. However, it is very possible to send logs from
your own role or application.

The following input format are supported:

* syslog, on port 514
* gelf (Graylog's custom format), on port 12201
