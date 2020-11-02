# Telegraf on non-k8s

* [Prerequisites](#prerequisites)
  * [Launch an EC2 instance](#launch-an-ec2-instance)
  * [Install Telegraf](#install-telegraf)
    * [Amazon Linux 2 AMI](#amazon-linux-2-ami)
* [Redis](#redis)
  * [Install redis on Amazon Linux 2](#install-redis-on-amazon-linux-2)
  * [Start redis](#start-redis)
  * [Telegraf configuration for redis](#telegraf-configuration-for-redis)
  * [Start telegraf to collect metrics from redis](#start-telegraf-to-collect-metrics-from-redis)
  * [Observe redis metrics in Sumo Logic](#observe-redis-metrics-in-sumo-logic)
* [Nginx](#nginx)
  * [Install nginx on Amazon Linux 2](#install-nginx-on-amazon-linux-2)
  * [Add status page configuration to nginx](#add-status-page-configuration-to-nginx)
  * [Start nginx](#start-nginx)
  * [Telegraf configuration for nginx](#telegraf-configuration-for-nginx)
  * [Start telegraf to collect metrics from nginx](#start-telegraf-to-collect-metrics-from-nginx)
  * [Observe nginx metrics in Sumo Logic](#observe-nginx-metrics-in-sumo-logic)
* [JMX](#jmx)
  * [Install Java and tomcat](#install-java-and-tomcat)
  * [Start tomcat with jolokia2 jvm agent](#start-tomcat-with-jolokia2-jvm-agent)
  * [Telegraf configuration for jolokia2 agent](#telegraf-configuration-for-jolokia2-agent)
  * [Start telegraf to collect metrics from jolokia2 jvm agent](#start-telegraf-to-collect-metrics-from-jolokia2-jvm-agent)
  * [Observe jolokia2 jvm metrics in Sumo Logic](#observe-jolokia2-jvm-metrics-in-sumo-logic)
* [Tips and tricks](#tips-and-tricks)
  * [See the metrics sent in terminal](#see-the-metrics-sent-in-terminal)
  * [See more information about what's happening in telegraf](#see-more-information-about-whats-happening-in-telegraf)
* [Common pitfalls](#common-pitfalls)
  * [`Error writing to outputs.sumologic`](#error-writing-to-outputssumologic)

## Prerequisites

* created [HTTP collector](https://help.sumologic.com/03Send-Data/Hosted-Collectors/Configure-a-Hosted-Collector)
* created [HTTP Source](https://help.sumologic.com/03Send-Data/Sources/02Sources-for-Hosted-Collectors/HTTP-Source#configure-an-http%C2%A0logs-and-metrics-source)
* link for the HTTP Source created above [help](https://help.sumologic.com/03Send-Data/Sources/02Sources-for-Hosted-Collectors/HTTP-Source/zGenerate-a-new-URL-for-an-HTTP-Source)
* AWS credentials to create an EC2 instance

### Launch an EC2 instance

We're going to use an EC2 instance in order to run telegraf and applications on it.
Please start an instance using `Amazon Linux 2 AMI` with an instance type having
at least 8GB of RAM and 4vCPUs (e.g. `r5.xlarge`).

Ensure you can ssh into it.

### Install Telegraf

When you're logged in to the machine download `telegraf`
After installing `telegraf` with steps below ensure you can run it

```
$ telegraf --version
Telegraf 1.16.1 (git: HEAD 292e285a)
```

In order to use [Sumo Logic output plugin](https://github.com/influxdata/telegraf/tree/master/plugins/outputs/sumologic)
one will need version `v1.16.0` or higher.

#### Amazon Linux 2 AMI

To install `telegraf` on Amazon Linux 2 AMI one can use `yum` and `.rpm`s provided
by `influxdata`.

First download telegraf

```
$ curl -O https://dl.influxdata.com/telegraf/releases/telegraf-1.16.1-1.x86_64.rpm
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 20.6M  100 20.6M    0     0  72.0M      0 --:--:-- --:--:-- --:--:-- 71.8M
```

and then install it using `yum`

```
$ sudo yum localinstall telegraf-1.16.1-1.x86_64.rpm -y
Failed to set locale, defaulting to C
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd
Examining telegraf-1.16.1-1.x86_64.rpm: telegraf-1.16.1-1.x86_64
Marking telegraf-1.16.1-1.x86_64.rpm to be installed
Resolving Dependencies
--> Running transaction check
---> Package telegraf.x86_64 0:1.16.1-1 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=====================================================================================
 Package        Arch         Version           Repository                       Size
=====================================================================================
Installing:
 telegraf       x86_64       1.16.1-1          /telegraf-1.16.1-1.x86_64        69 M

Transaction Summary
=====================================================================================
Install  1 Package

Total size: 69 M
Installed size: 69 M
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : telegraf-1.16.1-1.x86_64                                          1/1
Created symlink from /etc/systemd/system/multi-user.target.wants/telegraf.service to /usr/lib/systemd/system/telegraf.service.
  Verifying  : telegraf-1.16.1-1.x86_64                                          1/1

Installed:
  telegraf.x86_64 0:1.16.1-1

Complete!
```

## Redis

### Install redis on Amazon Linux 2

```
$ sudo amazon-linux-extras install redis4.0 -y
Installing redis
Failed to set locale, defaulting to C
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd
Cleaning repos: amzn2-core amzn2extra-docker amzn2extra-redis4.0
14 metadata files removed
6 sqlite files removed
0 metadata files removed
Failed to set locale, defaulting to C
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd
amzn2-core                                                                                                                                         | 3.7 kB  00:00:00
amzn2extra-docker                                                                                                                                  | 3.0 kB  00:00:00
amzn2extra-redis4.0                                                                                                                                | 3.0 kB  00:00:00
(1/7): amzn2-core/2/x86_64/group_gz                                                                                                                | 2.6 kB  00:00:00
(2/7): amzn2-core/2/x86_64/updateinfo                                                                                                              | 295 kB  00:00:00
(3/7): amzn2extra-redis4.0/2/x86_64/primary_db                                                                                                     |  11 kB  00:00:00
(4/7): amzn2extra-docker/2/x86_64/updateinfo                                                                                                       |   76 B  00:00:00
(5/7): amzn2extra-docker/2/x86_64/primary_db                                                                                                       |  68 kB  00:00:00
(6/7): amzn2extra-redis4.0/2/x86_64/updateinfo                                                                                                     |   76 B  00:00:00
(7/7): amzn2-core/2/x86_64/primary_db                                                                                                              |  46 MB  00:00:00
Resolving Dependencies
--> Running transaction check
---> Package redis.x86_64 0:4.0.10-2.amzn2.0.2 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

==========================================================================================================================================================================
 Package                          Arch                              Version                                          Repository                                      Size
==========================================================================================================================================================================
Installing:
 redis                            x86_64                            4.0.10-2.amzn2.0.2                               amzn2extra-redis4.0                            732 k

Transaction Summary
==========================================================================================================================================================================
Install  1 Package

Total download size: 732 k
Installed size: 2.2 M
Downloading packages:
redis-4.0.10-2.amzn2.0.2.x86_64.rpm                                                                                                                | 732 kB  00:00:00
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : redis-4.0.10-2.amzn2.0.2.x86_64                                                                                                                        1/1
  Verifying  : redis-4.0.10-2.amzn2.0.2.x86_64                                                                                                                        1/1

Installed:
  redis.x86_64 0:4.0.10-2.amzn2.0.2

Complete!
```

### Start redis

Start `redis-server` in a separate terminal window

```
$ redis-server
12664:C 30 Oct 10:03:44.572 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
12664:C 30 Oct 10:03:44.572 # Redis version=4.0.10, bits=64, commit=00000000, modified=0, pid=12664, just started
...
```

### Telegraf configuration for redis

Official `redis` input plugin documentation can be found at https://github.com/influxdata/telegraf/tree/master/plugins/inputs/redis.

Put the below configuration to a file (full redis configuration file
can be found at [redis.conf](/non-k8s/redis/redis.conf))

```conf
[agent]
  interval = "3000ms"
  flush_interval = "3000ms"

[[inputs.redis]]
  servers = ["tcp://localhost:6379"]

[[outputs.sumologic]]
  ## Unique URL generated for your HTTP Metrics Source.
  ## This is the address to send metrics to.
  # url = "https://events.sumologic.net/receiver/v1/http/<UniqueHTTPCollectorCode>"

  data_format = "prometheus"
```

Substitute `url` in `sumologic` output plugin section with the URL that you got When
creating HTTP Source at sumologic

### Start telegraf to collect metrics from redis

```
$ telegraf --config .redis.conf
2020-10-26T11:42:47Z I! Starting Telegraf 1.16.1
2020-10-26T11:42:47Z I! Loaded inputs: redis
2020-10-26T11:42:47Z I! Loaded aggregators:
2020-10-26T11:42:47Z I! Loaded processors:
2020-10-26T11:42:47Z I! Loaded outputs: sumologic
2020-10-30T10:06:46Z I! Tags enabled: host=ip-172-31-10-189.eu-west-1.compute.internal
2020-10-26T11:42:47Z I! [agent] Config: Interval:3s, Quiet:false, Hostname:"pmalek-mac", Flush Interval:3s
...
```

### Observe redis metrics in Sumo Logic

Find your HTTP source in `Collection` view

![Metrics in Sumo Logic](./sumo_collection_metrics1.png "Metrics in Sumo Logic")

Open its metrics view and validate that metrics are coming to Sumo

![Redis metrics in Sumo Logic](./sumo_metrics_collection_view_redis.png "Redis metrics in Sumo Logic")

## Nginx

### Install nginx on Amazon Linux 2

```
$ sudo amazon-linux-extras install nginx1 -y
Installing nginx
Failed to set locale, defaulting to C
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd
Cleaning repos: amzn2-core amzn2extra-docker amzn2extra-nginx1 amzn2extra-redis4.0
18 metadata files removed
8 sqlite files removed
0 metadata files removed
Failed to set locale, defaulting to C
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd
amzn2-core                                                        | 3.7 kB  00:00:00
amzn2extra-docker                                                 | 3.0 kB  00:00:00
amzn2extra-nginx1                                                 | 3.0 kB  00:00:00
amzn2extra-redis4.0                                               | 3.0 kB  00:00:00
(1/9): amzn2-core/2/x86_64/group_gz                               | 2.6 kB  00:00:00
(2/9): amzn2-core/2/x86_64/updateinfo                             | 295 kB  00:00:00
(3/9): amzn2extra-nginx1/2/x86_64/primary_db                      |  19 kB  00:00:00
(4/9): amzn2extra-redis4.0/2/x86_64/updateinfo                    |   76 B  00:00:00
(5/9): amzn2extra-redis4.0/2/x86_64/primary_db                    |  11 kB  00:00:00
(6/9): amzn2extra-docker/2/x86_64/updateinfo                      |   76 B  00:00:00
(7/9): amzn2extra-docker/2/x86_64/primary_db                      |  68 kB  00:00:00
(8/9): amzn2extra-nginx1/2/x86_64/updateinfo                      |   76 B  00:00:00
(9/9): amzn2-core/2/x86_64/primary_db                             |  46 MB  00:00:00
Resolving Dependencies
--> Running transaction check
...
...
Installed:
  nginx.x86_64 1:1.18.0-1.amzn2.0.1

Dependency Installed:
  dejavu-fonts-common.noarch 0:2.33-6.amzn2
  dejavu-sans-fonts.noarch 0:2.33-6.amzn2
  fontconfig.x86_64 0:2.13.0-4.3.amzn2
  fontpackages-filesystem.noarch 0:1.44-8.amzn2
  gd.x86_64 0:2.0.35-26.amzn2.0.2
  gperftools-libs.x86_64 0:2.6.1-1.amzn2
  libX11.x86_64 0:1.6.7-2.amzn2
  libX11-common.noarch 0:1.6.7-2.amzn2
  libXau.x86_64 0:1.0.8-2.1.amzn2.0.2
  libXpm.x86_64 0:3.5.12-1.amzn2.0.2
  libxcb.x86_64 0:1.12-1.amzn2.0.2
  libxslt.x86_64 0:1.1.28-6.amzn2
  nginx-all-modules.noarch 1:1.18.0-1.amzn2.0.1
  nginx-filesystem.noarch 1:1.18.0-1.amzn2.0.1
  nginx-mod-http-geoip.x86_64 1:1.18.0-1.amzn2.0.1
  nginx-mod-http-image-filter.x86_64 1:1.18.0-1.amzn2.0.1
  nginx-mod-http-perl.x86_64 1:1.18.0-1.amzn2.0.1
  nginx-mod-http-xslt-filter.x86_64 1:1.18.0-1.amzn2.0.1
  nginx-mod-mail.x86_64 1:1.18.0-1.amzn2.0.1
  nginx-mod-stream.x86_64 1:1.18.0-1.amzn2.0.1

Complete!
```

### Add status page configuration to nginx

In order to make `nginx` expose metrics on status page so that `telegraf` can
scrape it, we need to add the following configuration to a `.conf` file in `/etc/nginx/conf.d/`

```
server{
  location /nginx_status {
      stub_status on;
      access_log  on;
      allow all;  # REPLACE with your access policy
  }
}
```

### Start nginx

```
$ sudo service nginx start
Redirecting to /bin/systemctl start nginx.service
```

At this point we should get the following when sending a request to the server via `curl`

```
$ curl http://localhost/nginx_status
Active connections: 1
server accepts handled requests
 1 1 1
Reading: 0 Writing: 1 Waiting: 0
```

### Telegraf configuration for nginx

Official `nginx` input plugin documentation can be found at https://github.com/influxdata/telegraf/tree/master/plugins/inputs/nginx.

Put the below configuration to a file e.g. `nginx.conf` (full config with all
possible configuration options - [nginx.conf](/non-k8s/nginx/nginx.conf))

```conf
[agent]
  interval = "3000ms"
  flush_interval = "3000ms"

[[inputs.nginx]]
  urls = ["http://localhost/nginx_status"]

[[outputs.sumologic]]
  ## Unique URL generated for your HTTP Metrics Source.
  ## This is the address to send metrics to.
  # url = "https://events.sumologic.net/receiver/v1/http/<UniqueHTTPCollectorCode>"
  data_format = "prometheus"
```

Substitute `url` in `sumologic` output plugin section with the URL that you got When
creating HTTP Source at sumologic.

### Start telegraf to collect metrics from nginx

```
$ telegraf --config nginx.conf
2020-10-27T12:55:11Z I! Starting Telegraf 1.16.1
2020-10-27T12:55:11Z I! Loaded inputs: nginx
2020-10-27T12:55:11Z I! Loaded aggregators:
2020-10-27T12:55:11Z I! Loaded processors:
2020-10-27T12:55:11Z I! Loaded outputs: file sumologic
2020-10-30T10:06:46Z I! Tags enabled: host=ip-172-31-10-189.eu-west-1.compute.internal
2020-10-27T12:55:11Z I! [agent] Config: Interval:3s, Quiet:false, Hostname:"pmalek-mac", Flush Interval:3s
...
```

### Observe nginx metrics in Sumo Logic

Open its metrics view and validate that metrics are coming to Sumo

![Nginx metrics in Sumo Logic](./sumo_metrics_collection_view_nginx.png "Nginx metrics in Sumo Logic")

## JMX

### Install Java and tomcat

Install java and tomcat

```
$ sudo amazon-linux-extras install java-openjdk11 tomcat8.5 -y
Installing java-11-openjdk
Failed to set locale, defaulting to C
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd
Cleaning repos: amzn2-core amzn2extra-docker amzn2extra-java-openjdk11
              : amzn2extra-nginx1 amzn2extra-redis4.0
22 metadata files removed
8 sqlite files removed
0 metadata files removed
Failed to set locale, defaulting to C
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd
amzn2-core                                                    | 3.7 kB  00:00:00
amzn2extra-docker                                             | 3.0 kB  00:00:00
amzn2extra-java-openjdk11                                     | 3.0 kB  00:00:00
amzn2extra-nginx1                                             | 3.0 kB  00:00:00
amzn2extra-redis4.0                                           | 3.0 kB  00:00:00
(1/11): amzn2-core/2/x86_64/group_gz                          | 2.6 kB  00:00:00
(2/11): amzn2-core/2/x86_64/updateinfo                        | 295 kB  00:00:00
(3/11): amzn2extra-java-openjdk11/2/x86_64/primary_db         |  58 kB  00:00:00
(4/11): amzn2extra-nginx1/2/x86_64/updateinfo                 |   76 B  00:00:00
(5/11): amzn2extra-nginx1/2/x86_64/primary_db                 |  19 kB  00:00:00
(6/11): amzn2extra-docker/2/x86_64/updateinfo                 |   76 B  00:00:00
(7/11): amzn2extra-redis4.0/2/x86_64/updateinfo               |   76 B  00:00:00
(8/11): amzn2extra-redis4.0/2/x86_64/primary_db               |  11 kB  00:00:00
(9/11): amzn2extra-docker/2/x86_64/primary_db                 |  68 kB  00:00:00
(10/11): amzn2extra-java-openjdk11/2/x86_64/updateinfo        |   76 B  00:00:00
(11/11): amzn2-core/2/x86_64/primary_db                       |  46 MB  00:00:00
Resolving Dependencies
...

Complete!
```

### Start tomcat with jolokia2 jvm agent

Download jolokia2 agent

```
$ curl -L -O https://search.maven.org/remotecontent?filepath=org/jolokia/jolokia-jvm/1.6.2/jolokia-jvm-1.6.2-agent.jar
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   134  100   134    0     0    451      0 --:--:-- --:--:-- --:--:--   451
100  454k  100  454k    0     0  1441k      0 --:--:-- --:--:-- --:--:-- 1441k
```

and move it to `/`

```
$ sudo cp jolokia-jvm-1.6.2-agent.jar /
```

Edit `/usr/lib/systemd/system/tomcat.service` so that it includes java agent
configuration

```
[Unit]
Description=Apache Tomcat Web Application Container
After=syslog.target network.target

[Service]
Type=simple
EnvironmentFile=/etc/tomcat/tomcat.conf
Environment="NAME="
EnvironmentFile=-/etc/sysconfig/tomcat
ExecStart=/usr/libexec/tomcat/server start
SuccessExitStatus=143
User=tomcat

# This is the important bit!
Environment="CATALINA_OPTS=-javaagent:/jolokia-jvm-1.6.2-agent.jar=host=0.0.0.0,port=8778"

...
```

and now we can run `tomcat`

```
$ sudo service tomcat start
Redirecting to /bin/systemctl start tomcat.service
```

Now confirm that jolokia2 endpoint is working via (using `jq` for pretty JSON output)

```
$ curl -s http://localhost:8778/jolokia/version | jq
{
  "request": {
    "type": "version"
  },
  "value": {
    "agent": "1.6.2",
    "protocol": "7.2",
    "config": {
      "listenForHttpService": "true",
      "maxCollectionSize": "0",
      "authIgnoreCerts": "false",
      "agentId": "172.17.0.2-1-3496ad19-jvm",
      "debug": "false",
      "agentType": "jvm",
      "policyLocation": "classpath:/jolokia-access.xml",
      "agentContext": "/jolokia",
      "serializeException": "false",
      "mimeType": "text/plain",
      "maxDepth": "15",
      "authMode": "basic",
      "authMatch": "any",
      "discoveryEnabled": "true",
      "streaming": "true",
      "canonicalNaming": "true",
      "historyMaxEntries": "10",
      "allowErrorDetails": "true",
      "allowDnsReverseLookup": "true",
      "realm": "jolokia",
      "includeStackTrace": "true",
      "maxObjects": "0",
      "useRestrictorService": "false",
      "debugMaxEntries": "100"
    },
    "info": {
      "product": "tomcat",
      "vendor": "Apache",
      "version": "9.0.39"
    }
  },
  "timestamp": 1603886739,
  "status": 200
}
```

### Telegraf configuration for jolokia2 agent

Official `jolokia2_agent` input plugin documentation can be found at https://github.com/influxdata/telegraf/tree/master/plugins/inputs/jolokia2.

Put below configuration into a file e.g. `jolokia2.conf` (full configuration
file [jolokia2.conf](/non-k8s/jolokia2/jolokia2.conf))

```conf
[agent]
  interval = "3000ms"
  flush_interval = "3000ms"

[[inputs.jolokia2_agent]]
  # TODO: Tweak exposed metrics to your liking
  urls = ["http://localhost:8778/jolokia"]

  [[inputs.jolokia2_agent.metric]]
    name  = "jvm_runtime"
    mbean = "java.lang:type=Runtime"
    paths = ["Uptime"]

  [[inputs.jolokia2_agent.metric]]
    name  = "jvm_memory"
    mbean = "java.lang:type=Memory"
    paths = ["HeapMemoryUsage", "NonHeapMemoryUsage", "ObjectPendingFinalizationCount"]

  [[inputs.jolokia2_agent.metric]]
    name  = "jvm_garbage_collector"
    mbean = "java.lang:name=*,type=GarbageCollector"
    paths = ["CollectionTime", "CollectionCount"]

  [[inputs.jolokia2_agent.metric]]
    name  = "java_lang_ClassLoading"
    mbean = "java.lang:type=ClassLoading"
    paths = ["LoadedClassCount", "TotalLoadedClassCount", "UnloadedClassCount"]

  [[inputs.jolokia2_agent.metric]]
    name  = "java_lang_Compilation"
    mbean = "java.lang:type=Compilation"
    paths = ["TotalCompilationTime"]

  [[inputs.jolokia2_agent.metric]]
    name  = "java_lang_GarbageCollector"
    mbean = "java.lang:name=*,type=GarbageCollector"
    paths = ["CollectionCount", "CollectionTime", "LastGcInfo"]
    tag_keys = ["name"]

  [[inputs.jolokia2_agent.metric]]
    name  = "java_lang_MemoryPool"
    mbean = "java.lang:name=*,type=MemoryPool"
    paths = ["CollectionUsage", "CollectionUsageThresholdSupported", "PeakUsage", "Usage", "UsageThresholdSupported"]
    tag_keys = ["name"]

  [[inputs.jolokia2_agent.metric]]
    name  = "java_lang_Memory"
    mbean = "java.lang:type=Memory"
    paths = ["HeapMemoryUsage", "NonHeapMemoryUsage", "ObjectPendingFinalizationCount"]

  [[inputs.jolokia2_agent.metric]]
    name  = "java_lang_OperatingSystem"
    mbean = "java.lang:type=OperatingSystem"
    paths = ["AvailableProcessors", "CommittedVirtualMemorySize", "FreePhysicalMemorySize", "FreeSwapSpaceSize", "MaxFileDescriptorCount", "OpenFileDescriptorCount", "ProcessCpuLoad", "ProcessCpuTime", "SystemCpuLoad", "SystemLoadAverage", "TotalPhysicalMemorySize", "TotalSwapSpaceSize"]

  [[inputs.jolokia2_agent.metric]]
    name  = "java_lang_Runtime"
    mbean = "java.lang:type=Runtime"
    paths = ["BootClassPathSupported", "StartTime", "Uptime"]

  [[inputs.jolokia2_agent.metric]]
    name  = "java_lang_Threading"
    mbean = "java.lang:type=Threading"
    paths = ["CurrentThreadCpuTime", "CurrentThreadUserTime", "DaemonThreadCount", "ObjectMonitorUsageSupported", "PeakThreadCount", "SynchronizerUsageSupported", "ThreadContentionMonitoringEnabled", "ThreadContentionMonitoringSupported", "ThreadCount", "ThreadCpuTimeEnabled", "ThreadCpuTimeSupported", "TotalStartedThreadCount"]

[[outputs.sumologic]]
  ## Unique URL generated for your HTTP Metrics Source.
  ## This is the address to send metrics to.
  # url = "https://events.sumologic.net/receiver/v1/http/<UniqueHTTPCollectorCode>"
  data_format = "prometheus"
```

Substitute `url` in `sumologic` output plugin section with the URL that you got When
creating HTTP Source at sumologic.

### Start telegraf to collect metrics from jolokia2 jvm agent

```
$ telegraf --config jolokia2.conf
2020-10-28T12:12:51Z I! Starting Telegraf 1.16.1
2020-10-28T12:12:51Z I! Loaded inputs: jolokia2_agent
2020-10-28T12:12:51Z I! Loaded aggregators:
2020-10-28T12:12:51Z I! Loaded processors:
2020-10-28T12:12:51Z I! Loaded outputs: sumologic
2020-10-28T12:12:51Z I! Tags enabled: host=pmalek-mac
2020-10-28T12:12:51Z I! [agent] Config: Interval:3s, Quiet:false, Hostname:"pmalek-mac", Flush Interval:3s
2020-10-28T12:12:51Z D! [agent] Initializing plugins
2020-10-28T12:12:51Z D! [agent] Connecting outputs
2020-10-28T12:12:51Z D! [agent] Attempting connection to [outputs.sumologic]
2020-10-28T12:12:51Z D! [agent] Successfully connected to outputs.sumologic
2020-10-28T12:12:51Z D! [agent] Starting service inputs
...
```

### Observe jolokia2 jvm metrics in Sumo Logic

Open its metrics view and validate that metrics are coming to Sumo

![Jolokia2 metrics in Sumo Logic](./sumo_metrics_collection_view_jolokia2.png "Jolokia2 metrics in Sumo Logic")

## Tips and tricks

### See the metrics sent in terminal

In order to see the data that is being sent on the terminal one can the the following
output plugin in the configuration

```
...

[[outputs.file]]
  files = ["stdout"]

...
```

After starting `telegraf` with the above included in the configuration we can obverve
metrics in the terminal window

```
2020-10-27T12:59:24Z I! Starting Telegraf 1.16.1
2020-10-27T12:59:24Z I! Loaded inputs: nginx
2020-10-27T12:59:24Z I! Loaded aggregators:
2020-10-27T12:59:24Z I! Loaded processors:
2020-10-27T12:59:24Z I! Loaded outputs: file sumologic
2020-10-27T12:59:24Z I! Tags enabled: host=pmalek-mac
2020-10-27T12:59:24Z I! [agent] Config: Interval:1s, Quiet:false, Hostname:"pmalek-mac", Flush Interval:1s
nginx,host=pmalek-mac,port=8080,server=localhost accepts=10i,handled=10i,requests=13i,reading=0i,writing=1i,waiting=0i,active=1i 1603803565000000000
nginx,host=pmalek-mac,port=8080,server=localhost active=1i,accepts=10i,handled=10i,requests=14i,reading=0i,writing=1i,waiting=0i 1603803566000000000
nginx,host=pmalek-mac,port=8080,server=localhost reading=0i,writing=1i,waiting=0i,active=1i,accepts=10i,handled=10i,requests=15i 1603803567000000000
nginx,host=pmalek-mac,port=8080,server=localhost waiting=0i,active=1i,accepts=10i,handled=10i,requests=16i,reading=0i,writing=1i 1603803568000000000
nginx,host=pmalek-mac,port=8080,server=localhost waiting=0i,active=1i,accepts=10i,handled=10i,requests=17i,reading=0i,writing=1i 1603803569000000000
nginx,host=pmalek-mac,port=8080,server=localhost active=1i,accepts=10i,handled=10i,requests=18i,reading=0i,writing=1i,waiting=0i 1603803570000000000
nginx,host=pmalek-mac,port=8080,server=localhost reading=0i,writing=1i,waiting=0i,active=1i,accepts=10i,handled=10i,requests=19i 1603803571000000000
...
```

### See more information about what's happening in telegraf

To make `telegraf` logs more verbose one can add `--debug` flag when starting it

```
$ telegraf --config nginx.conf --debug
2020-10-27T13:01:37Z I! Starting Telegraf 1.16.1
2020-10-27T13:01:37Z I! Loaded inputs: nginx
2020-10-27T13:01:37Z I! Loaded aggregators:
2020-10-27T13:01:37Z I! Loaded processors:
2020-10-27T13:01:37Z I! Loaded outputs: sumologic
2020-10-27T13:01:37Z I! Tags enabled: host=pmalek-mac
2020-10-27T13:01:37Z I! [agent] Config: Interval:3s, Quiet:false, Hostname:"pmalek-mac", Flush Interval:3s
2020-10-27T13:01:37Z D! [agent] Initializing plugins
2020-10-27T13:01:37Z D! [agent] Connecting outputs
2020-10-27T13:01:37Z D! [agent] Attempting connection to [outputs.sumologic]
2020-10-27T13:01:37Z D! [agent] Successfully connected to outputs.sumologic
2020-10-27T13:01:37Z D! [agent] Starting service inputs
2020-10-27T13:01:41Z D! [outputs.sumologic] Wrote batch of 1 metrics in 1.002110093s
2020-10-27T13:01:41Z D! [outputs.sumologic] Buffer fullness: 0 / 10000 metrics
2020-10-27T13:01:44Z D! [outputs.sumologic] Wrote batch of 1 metrics in 376.138844ms
2020-10-27T13:01:44Z D! [outputs.sumologic] Buffer fullness: 0 / 10000 metrics
2020-10-27T13:01:47Z D! [outputs.sumologic] Wrote batch of 1 metrics in 250.25096ms
2020-10-27T13:01:47Z D! [outputs.sumologic] Buffer fullness: 0 / 10000 metrics
...
```

## Common pitfalls

### `Error writing to outputs.sumologic`

If you see the following error when starting `telegraf`

```
2020-10-28T12:16:56Z E! [agent] Error writing to outputs.sumologic: sumologic: failed sending request to [https://events.sumologic.net/receiver/v1/http/<UniqueHTTPCollectorCode>]: Post "https://events.sumologic.net/receiver/v1/http/%3CUniqueHTTPCollectorCode%3E": dial tcp: lookup events.sumologic.net on 192.168.0.1:53: no such host
```

Most likely you didn't change the `url` in the configuration file

```
# Uncomment this and set it to URL obtained when creating HTTP Source at Sumo
# url = "https://events.sumologic.net/receiver/v1/http/<UniqueHTTPCollectorCode>"
```
