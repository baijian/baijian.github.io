---
layout: post
title: "storm-java-alog-count"
date: 2013-08-23 11:59
comments: true
categories: 
---


### Design a Topology

<img style="width:620px;" src="/images/topology.png" />
    
### Data source & RabbitMQ

**pycat.py**

{% highlight python %}
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import sys
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters(host='localhost'))
channel = connection.channel()
channel.queue_declare(queue='rabbit-alog')

line = sys.stdin.readline()
while line:
    line = line.rstrip('\n')
    channel.basic_publish(exchange='', routing_key='rabbit-alog', body=line)
    line = sys.stdin.readline()
connection.close()
{% endhighlight %}

**Collector Data**

$ tail -F /var/log/access.log | python pycat.py

### AlogSpout

**Key Codes**

{% highlight java %}
public class AlogSpout extends BaseRichSpout {

    private SpoutOutputCollector _collector;
    private final Scheme _scheme;
    public AlogSpout(Scheme scheme) {
        _scheme = scheme;
        //init some server info
    }
    @Override
    public void declareOutputFields(OutputFieldsDeclarer declarer) {
        declarer.declare(_scheme.getOutputFields);
    }
    @Override
    public void open(Map map, TopologyContext context, SpoutOutputCollector spoutCollector) {
        _collector = spoutCollector;
        try {
            connectAMQP();
        } catch (Exception e) {
        }
    }
    @Override
    public void nextTuple() {
        try {
            byte[] message = zmqDelivery.getBody();
            List<Object> msg = _scheme.deserizlize(message);
            _collector.emit(msg);
        } catch (Exception e) {
        }
    }
}
{% endhighlight %}

**Explanation**

One stream is a just a tuple, all of the tuple will be created and handled
throughout a cluster parallelly. You should define each field of the tuple.
Like:

{% highlight java %}
Fields f = new Fields("str");
Fields ff = new Fields("url_id", "time_local");
{% endhighlight %}

Method *nextTuple* should be non-block, storm emit msg in single thread.
Method *ack* and *fail* is used in message reliablity.

### FilterUrlBolt

**Key Codes**

{% highlight java %}
public class FilterUrlBolt extends BaseRichBolt {
    private OutputCollector _collector;
    public FilterUrlBolt() {
    }
    @Override
    public void prepare(Map stormConf, TopologyContext context, OutputCollector collector) {
        _collector = collector;
    }
    @Override
    public void execute(Tuple input) {
        //update url hash map from mysql every 3 miniute
        //...
        // foreach of all urls in a hashmap
        if (Pattern.matches(pattern, line)) {
            _collector.emit(new Values(id, time));
            break;
        }
        _collector.ack(input);
    }
    @Override
    public void declareOutputFields(OutputFieldDeclarer declarer) {
        declarer.declare(_scheme.getOutputFields());
    }
}
{% endhighlight %}

**Explanation**

Bolt is used to handle messages and emit new messages to the next Bolt.
Method *ack* must be invoked to tell storm the message is handled.

### CountBolt

**Key Code**

{% highlight java %}
public class CountBolt extends BaseRichBolt {
    @Override
    public execute(Tuple input) {
        int url_id = input.getInteger(0);
        String time_str = input.getString(1);

        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        Date dt = null;
        try {
            dt = simpleDateFormat.parse(time_str);
        } catch (Exception e) {
        }

        SimpleDateFormat miniuteFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm");
        String miniute_begin = miniuteFormat.format(dt);
        String miniute_begin_time = miniute_begin + ":00";

        if (!_timer.containsKey(url_id)) {
            _timer.put(url_id, miniute_begin_time);
        }
        if (!_counter.containsKey(url_id)) {
            _counter.put(url_id, 0);
        }

        Date dt_miniute_begin = null;
        try {
            dt_miniute_begin = miniuteFormat.parse(_timer.get(url_id));
        } catch (Exception e) {
        }

        if (dt.getTime() - dt_miniute_begin.getTime() > 60000) {
            recordCount(url_id, _timer.get(url_id), _counter.get(url_id));
            _timer.remove(url_id);
            _timer.put(url_id, miniute_begin_time);
            _counter.remove(url_id);
            _counter.put(url_id, 0);
        }

        if (_counter.containsKey(url_id)) {
            int count = _counter.get(url_id);
            count++;
            _counter.remove(url_id);
            _counter.put(url_id, count);
        }

        _collector.ack(input);
    }
}
{% endhighlight %}

**Explanation**

CountBolt just increase by 1 and persistent count number to 
mysql every miniute based on message's id and time. And do
not emit message any more.

### AlogCountTopology

**Key Code**

{% highlight java %}
public class AlogCountTopology {
    
    public static void main(String[] args) {
        TopologyBuilder builder = new TopologyBuilder();
        builder.setSpout("Alog", new AlogSpout(...), 10);
        builder.setBolt("FilterAlog", new FilterUriBolt(...), 10)
            .shuffleGrouping("Alog");
        builder.setBolt("GroupCount", new CountBolt(...), 20)
            .fieldsGrouping("FilterAlog", new Fields("url_id"));

        Config config = new Config();

        if (args != null && args.length > 0) {
            config.setNumWorkers(10);
            StormSubmitter.submitTopology(args[0], config, builder.createTopology());
        } else {
        }
    }
}
{% endhighlight %}

**Explanation**

One Topology will be executed in one single or severial wokers.
Spout or Bolt will be split into severial tasks to execute.
And each task will map to a node+port. You can set worker numbers 
of your Topology and set parallelism(number of task) of each Spout or Bolt,
sum of them as parallelism of Topology.

Stream Grouping defines how to emit messages from a couple of tasks to 
another couple of tasks. Below are some types of ***Stream Grouping***:

* Shuffle Grouping
* Fields Grouping
* All Grouping
* Global Grouping
* Non Grouping
* Direct Grouping
* Local or shuffle Grouping

You can use Config to config your *Numbus* *Supervisor* and *Topology*.

[SourceCode](https://github.com/baijian/storm-java)
