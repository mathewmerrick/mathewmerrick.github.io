---
published: true
title: One Page ELK
layout: post
---
A one page guide for ctrl+f'ing

\*A large portion of this guide originated from [Elastic]'s own documentation, I just wanted a tl:dr.

---

# ***Installing***
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
```

Install elasticsearch

```
sudo apt-get install Elasticsearch
```


















[Oracle]:http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
[Elastic]:https://www.elastic.co/guide/index.html