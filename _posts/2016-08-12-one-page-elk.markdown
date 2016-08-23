---
published: true
title: One Page ELK
layout: post
---
A one page guide for searching

\*A large portion of this guide originated from [Elastic]'s own documentation, I just wanted the important stuff curated.

---

# **Installing**
Elasticsearch requires Java. I used Oracle's Java, [but OpenJDK works as well.](https://www.elastic.co/guide/en/elasticsearch/guide/current/_java_virtual_machine.html) 

[Install OpenJDK](http://openjdk.java.net/install/)

[Install Oracle Java](https://docs.oracle.com/javase/8/docs/technotes/guides/install/linux_jdk.html#BJFJJEFG)

# Debian

## **Elasticsearch**

Download and add the GPG signing key:

```
wget -qO - https://packages.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```

Now we need to add the official elasticsearch repository adding it to our sources

```
echo "deb https://packages.elastic.co/elasticsearch/2.x/debian stable main" | sudo tee -a /etc/apt/sources.list.d/elasticsearch-2.x.list
```

Update repositories

```
sudo apt-get update
sudo apt-get install elasticsearch
```

Set to run at startup

```
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable elasticsearch.service
```

## **Logstash**

Download and add the GPG signing key: (If you skipped the Elasticsearch Portion)

```
wget -qO - https://packages.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```

Now we need to add the official elasticsearch repository adding it to our sources

```
echo "deb https://packages.elastic.co/logstash/2.3/debian stable main" | sudo tee -a /etc/apt/sources.list.d/logstash-2.3.list
```

Update repositories and install

```
sudo apt-get update
sudo apt-get install logstash
```

Set to run at startup

```
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable logstash.service
```

## **Kibana**

Download and add the GPG signing key: (If you skipped the Elasticsearch Portion)

```
wget -qO - https://packages.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```

Now we need to add the official elasticsearch repository adding it to our sources

```
echo "deb https://packages.elastic.co/kibana/4.5/debian stable main" | sudo tee -a /etc/apt/sources.list.d/kibana-4.5.list
```

Update repositories and install

```
sudo apt-get update
sudo apt-get install kibana
```

Set to run at startup

```
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable kibana.service
```

# RHEL/CentOS

## **Elasticsearch** 

Download signing key

```
rpm --import https://packages.elastic.co/GPG-KEY-elasticsearch
```

Edit ```elasticsearch.repo``` in ```/etc/yum.repos.d/```  

```
[elasticsearch-2.x]
name=Elasticsearch repository for 2.x packages
baseurl=https://packages.elastic.co/elasticsearch/2.x/centos
gpgcheck=1
gpgkey=https://packages.elastic.co/GPG-KEY-elasticsearch
enabled=1
```

Then install

```
yum install elasticsearch
```

Set to run at startup

```
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable elasticsearch.service
```

## **Logstash** 

Download signing key

```
rpm --import https://packages.elastic.co/GPG-KEY-elasticsearch
```

Edit ```logstash.repo``` in ```/etc/yum.repos.d/```  

```
[logstash-2.3]
name=Logstash repository for 2.3.x packages
baseurl=https://packages.elastic.co/logstash/2.3/centos
gpgcheck=1
gpgkey=https://packages.elastic.co/GPG-KEY-elasticsearch
enabled=1
```

Then install
```
yum install logstash
```


Set to run at startup

```
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable logstash.service
```


## **Kibana** 

Download signing key

```
rpm --import https://packages.elastic.co/GPG-KEY-elasticsearch
```

Edit ```kibana.repo``` in ```/etc/yum.repos.d/```  

```
[kibana-4.5]
name=Kibana repository for 4.5.x packages
baseurl=http://packages.elastic.co/kibana/4.5/centos
gpgcheck=1
gpgkey=http://packages.elastic.co/GPG-KEY-elasticsearch
enabled=1
```

Then install
```
yum install kibana
```


Set to run at startup

```
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable kibana.service
```


# **Configuring**

*You should probably memorize these.*

| Term     | Definition                                                                                                                   |
|----------|------------------------------------------------------------------------------------------------------------------------------|
| Node     | a single server that is part of the cluster, stores data, and participates in the cluster’s indexing and search capabilities |
| Cluster  | a collection of one or more nodes (servers) that provides federated indexing and search capabilities across all nodes        |
| Index    | a collection of documents that have somewhat similar characteristics                                                         |
| Type     | a logical category/partition of the index whose semantics is defined by documents that have a set of common fields           |
| Document | a basic unit of information that can be indexed                                                                              |
| Shard    | a is fully functional piece of an index, which permits an index to be spread across multiple nodes                           |
| Replica Shard  | if the node holding a primary shard dies, a replica is promoted to the role of primary.                           |
| Template | a definition that describes how documents will be mapped in an newly created index. 
                              |




## **Logstash**

The purpose of Logstash is to "massage" incoming data, and convert it into a JSON format that Elasticsearch can use. As such, many plugins exist for Logstash to appropriately ingest data. For the full list, go [here.](https://github.com/logstash-plugins)

A typical traffic flow for incoming data is usually, Source -> Logstash -> Elasticsearch. For this example, I will be using the netflow plugin.

Installing a codec: 

```
cd /opt/logstash
sudo ./bin/logstash-plugin install logstash-codec-netflow
```

Now we have create the config file, located in ```/etc/logstash/conf.d/``` (Note: if ```conf.d``` doesn't exist, create it)

Now create a config for Netflow V9 ingestion

```
cd /etc/logstash/conf.d/    
sudo vim logstash-netflow-v9.conf
```

Here is my conf for V9, with comments

```
input {
  udp {
    port => 9999                            # Listening port
    codec => netflow {
      versions => [9]                     # Only Netflow V9, change to [5] for Netflow V5
    }
  }
}

output {
  #stdout { codec => json }               # output logstash data to console
  elasticsearch {
    index => "netflow-v9-%{+YYYY.MM.dd}"    # Add to the index sorted by day
    hosts => "localhost"                  # elasticsearch host
  }
}
```

To test the syntax of the config file:

```
sudo ./bin/logstash --configtest -f /etc/logstash/conf.f/logstash-netflow-v9.conf
```

then start logstash with 

```
sudo systemctl start logstash
```

For multiple config files, Logstash initiates the pipeline in alphabetic order

## **Elasticsearch**

Elasticsearch is the main component of this project. From the logstash config above, you may have noticed that the output is to elasticsearch, but there is a parameter named "index", reading the definition above, an index, or multiple indices, is how data is stored in an Elasticsearch cluster.

It's important to identify how the documents are to be arranged in the index. To accomplish this, templates are used. 

Adding a templates is accomplished through Elasticsearch's RESTful API on port ```9200```. Below is an example of my Netflow V9 template. 

```
  "netflow-v9" : {
    "order" : 0,
    "template" : "netflow-v9-*",
    "settings" : {
      "index" : {
        "number_of_shards" : "1",
        "number_of_replicas" : "0",
        "refresh_interval" : "5s"
      }
    },
    "mappings" : {
      "_default_" : {
        "_all" : {
          "enabled" : false
        },
        "properties" : {
          "netflow" : {
            "dynamic" : true,
            "type" : "object",
            "properties" : {
              "icmp_type" : {
                "index" : "not_analyzed",
                "type" : "long"
              },
              "dst_mask" : {
                "index" : "analyzed",
                "type" : "integer"
              },
              "in_pkts" : {
                "index" : "analyzed",
                "type" : "long"
              },
              "ipv4_dst_addr" : {
                "index" : "analyzed",
                "type" : "ip"
              },
              "first_switched" : {
                "index" : "not_analyzed",
                "type" : "date"
              },
              "l4_src_port" : {
                "index" : "not_analyzed",
                "type" : "long"
              },
              "ipv4_next_hop" : {
                "index" : "analyzed",
                "type" : "ip"
              },
              "src_mask" : {
                "index" : "analyzed",
                "type" : "integer"
              },
              "version" : {
                "index" : "analyzed",
                "type" : "integer"
              },
              "ipv6_src_addr" : {
                "index" : "analyzed",
                "type" : "ip"
              },
              "ipv4_src_addr" : {
                "index" : "analyzed",
                "type" : "ip"
              },
              "in_bytes" : {
                "index" : "analyzed",
                "type" : "long"
              },
              "protocol" : {
                "index" : "not_analyzed",
                "type" : "integer"
              },
              "last_switched" : {
                "index" : "not_analyzed",
                "type" : "date"
              },
              "out_pkts" : {
                "index" : "analyzed",
                "type" : "long"
              },
              "out_bytes" : {
                "index" : "analyzed",
                "type" : "long"
              },
              "l4_dst_port" : {
                "index" : "analyzed",
                "type" : "long"
              }
            }
          },
          "@timestamp" : {
            "index" : "analyzed",
            "type" : "date"
          },
          "@version" : {
            "index" : "analyzed",
            "type" : "integer"
          }
        }
      }
    },
    "aliases" : { }
  },

```

I was able to create the above template from the [structure of the Netflow v9](https://www.plixer.com/support/netflow_v9.html) record, and the logstash-netflow-codec documentation.

I usually save the template as ```netflow-v9.template.json``` or something similiar. From there, I run

```
curl -XPUT 'http://localhost:9200/_template/netflow-v9' -d@netflow-v9.template.json
```
where Elasticsearch is running on localhost.

If Elasticsearch is not running on localhost, then you need to edit ```/etc/elasticsearch/elasticsearch.yml``` and change network host from ```localhost``` to ```0.0.0.0``` or the appropriate address you would like Elasticsearch to be bound to. 

**IF THIS PARAMETER IS CHANGED, ALL OTHER CONFIGS IN BOTH LOGSTASH AND KIBANA NEED TO BE CHANGED AS WELL.** 


Below are some rapid Elasticsearch commands that you are likely to use quite frequently in administration


#### **Useful [Elasticsearch] Commands**

View all elasticsearch indices 

**Note: A cluster with one node, and only one replica will *always* be yellow. It means that there are no copies of the shard active.**


```
curl 'localhost:9200/_cat/indices?v'
```

Delete an index

```
curl -XDELETE 'http://localhost:9200/index_name’
```

View all elasticsearch templates

```
curl -XGET localhost:9200/_template?pretty
```

View specific elasticsearch templates

```
curl -XGET localhost:9200/_template/template_name?pretty
```

Delete specific elasticsearch template

```
curl -XDELETE localhost:9200/_template/template_name
```

Add template from file

```
curl -XPUT localhost:9200/_template/template_name -d@path/to/template.json
```

Again, this is assuming that Elasticsearch is running on localhost


#### **Elasticsearch states**

| State     | Definition                                                                                                                   |
|----------|------------------------------------------------------------------------------------------------------------------------------|
| Green    | All primary and replica shards are allocated. Your cluster is 100% operational. |
| Yellow      | All primary shards are allocated, but at least one replica is missing. No data is missing, so search results will still be complete. However, your high availability is compromised to some degree. If more shards disappear, you might lose data. Think of yellow as a warning that should prompt investigation.        |
| Red    | At least one primary shard (and all of its replicas) is missing. This means that you are missing data: searches will return partial results, and indexing into that shard will return an exception.                                                         |



## **Kibana**

Kibana is the visual aspect of ELK. By default Kibana assumes that Elasticsearch is running on localhost. To correct this, edit ```/opt/kibana/config/kibana.yml```

Kibana listens on port ```5601``` after Kibana is started, navigate to ```http://localhost:5601``` to view the Kibana dashboard.

Note: Any visualizations that are made in Kibana are stored in Elasticsearch under the ```.kibana``` index. 


## **Beats**

- in progress

## MORE COMING SOON

# -
# -
# -








[Oracle]:http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
[Elastic]:https://www.elastic.co/guide/index.html
