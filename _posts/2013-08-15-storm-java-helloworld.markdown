---
layout: post
title: "storm-java-HelloWorld"
date: 2013-08-15 13:30
comments: true
categories: 
---

### Preface

Recently, our team is considering how to process our log data centralized.
We want to process our data real-time and our log will be big in the near future.
So storm may be the best technology solution.

So, I will begin to do a research on storm to meet our requirements and solve 
some problems may appear in the future.And I will record my learning process in 
the blog and also provide some codes. Hope it will be helpful for you.

### What is storm

OK, the first question of every thing and I think is the hardest question to answer.

    nathanmarz says: Storm is a distributed realtime computation system. Similar to how
    hadoop provides a set of general primitives for doing batch processing. Storm provides 
    a set of general primitives for doing realtime computation. Storm is simple, can be used
    with any programming language, and is a lot of fun to use!

Recommend some useful resources for you first:

* [Storm Source Code](https://github.com/nathanmarz/storm)
* [Storm Wiki Page](https://github.com/nathanmarz/storm/wiki)
* [Topology Demo](https://github.com/nathanmarz/storm-starter)
* [Java API Doc](http://nathanmarz.github.io/storm/doc/overview-summary.html)
* [Storm Google Group](https://groups.google.com/forum/#!forum/storm-user)

Storm is write in Clojure mostly, and provide entire Java API, and you can write applications 
use almost any languages, ofcourse you need one adapter to connect to it.If you want to read
source code of Storm you must know Clojure and Java, ofcourse I think it is necessary.

[Some Concepts You Should Know](https://github.com/nathanmarz/storm/wiki/Concepts)

In this blog I will just show you how to install a storm cluster, how to develop a topology
and how to submit a topology to the cluster step by step.

### Install a storm cluster

Before we install storm cluster, let us see how storm cluster running.
                                                              
                                                  ------------
                                                  |supervisor|
                                                  ------------
                                                  ------------
                          -----------             |supervisor|
       ----------         |zookeeper|             ------------
       |        |         -----------             ------------
       | Nimbus |  <----> -----------  <------->  |supervisor|
       |        |         |zookeeper|             ------------
       ----------         -----------             ------------
                          -----------             |supervisor|
                          |zookeeper|             ------------
                          -----------             ------------
                                                  |supervisor|
                                                  ------------

Numbus is the master node and supervisor is the worker node, and zookeeper
is the import thing to record the states of master and worker node.

OK, begin to install, recommend *ZeroMQ-2.1.7* [This JZMQ](https://github.com/nathanmarz/jzmq) *Java6* *Python-2.6.6*

$ wget http://cloud.github.com/downloads/nathanmarz/storm/storm-0.8.1.zip

$ wget http://mirrors.cnnic.cn/apache/zookeeper/stable/zookeeper-3.4.5.tar.gz

$ wget http://download.zeromq.org/zeromq-2.1.7.tar.gz

**Config and startup Zookeeper**

$ vim conf/zoo.cfg

    tickTime=2000
    initLimit=10
    syncLimit=5
    dataDir=/storm/zookeeper-data
    clientPort=2181
    dataLogDir=/storm/log
    server.1=xx.xx.xx.xx:2888:3888
    server.2=oo.oo.oo.oo:2888:3888

$ echo '1' > xx.xx.xx.xx:/storm/zookeeper-data/myid

$ echo '2' > oo.oo.oo.oo:/storm/zookeeper-data/myid

$ bin/zkServer.sh start

**Write *storm.yaml* config file**

    storm.zookeeper.servers:
        - "xx.xx.xx.xx"
        _ "oo.oo.oo.oo"

    storm.local.dir: "/storm/storm-state"

    nimbus.host: "xx.xx.xx.xx"

    supervisor.slots.port:
        - 6701
        - 6702
        - 6703
        - 6704

**Run Some Daemons**

$ nohup bin/storm numbus &

$ nohup bin/storm supervisor &
    
$ nohup bin/storm ui &

At last, open your browser to view xx.xx.xx.xx:8080.
                                                
### Develop a HelloWorld topology in local

Before we write topology,at lease we should know three concepts, ***Topology*** , ***Bolts***, ***Spouts***.

OK, then I will write a simple topology just append *World* to the end, and also write contents to file 
to see the result.

**Spout**

{% highlight java %}
public class HelloWorldSpout extends BaseRichSpout {

    SpoutOutputCollector collector;

    @Override
    public void declareOutputFields(OutputFieldsDeclarer declarer) {
        declarer.declare(new Fields("hello"));
    }

    @Override
    public void open(Map map, TopologyContext context, SpoutOutputCollector spoutOutputCollector) {
        collector = spoutOutputCollector;
    }

    @Override
    public void nextTuple() {
        Utils.sleep(1000);
        collector.emit(new Values("Hello"));
    }
}
{% endhighlight %}

**Bolt**

{% highlight java %}
public class HelloWorldBolt extends BaseBasicBolt {

    @Override
    public void execute(Tuple tuple, BasicOutputCollector collector) {
        String msg = tuple.getString(0);
        System.out.println("=====before write file=====");
        try {
            File file = new File("/tmp/storm.txt");
            if (!file.exists()) {
                file.createNewFile();
            }
            FileWriter fw = new FileWriter(file.getAbsoluteFile(), true);
            BufferedWriter bw = new BufferedWriter(fw);
            bw.write(msg + "\n");
            bw.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
        System.out.println("=====after write file=====");
        collector.emit(new Values(msg + " World"));
    }

    @Override
    public void declareOutputFields(OutputFieldsDeclarer declarer) {
        declarer.declare(new Fields("world"));
    }
}
{% endhighlight %}

**Topology**

{% highlight java %}
public class HelloWorldTopology {

    public static void main(String[] args) throws Exception {
        TopologyBuilder builder = new TopologyBuilder();
        builder.setSpout("Hello", new HelloWorldSpout(), 12);
        builder.setBolt("World", new HelloWorldBolt(), 12).shuffleGrouping("Hello");
        builder.setBolt("WorldTwo", new HelloWorldBolt(), 12).shuffleGrouping("World");

        Config config = new Config();
        config.setDebug(true);

        if (args != null && args.length > 0) {
            config.setNumWorkers(6);
            config.setNumAckers(6);
            config.setMaxSpoutPending(100);
            config.setMessageTimeoutSecs(20);
            StormSubmitter.submitTopology(args[0], config, builder.createTopology());
        } else {
            LocalCluster cluster = new LocalCluster();
            cluster.submitTopology("Hello-World-BaiJian", config, builder.createTopology());
            Utils.sleep(10000);
            cluster.killTopology("Hello-World-BaiJian");
            cluster.shutdown();
        }
    }
}
{% endhighlight %}

### Test it in your local computer and then submit to cluster

$ mvn package

$ mvn exec:java -Dexec.classpathScope=compile \

-Dexec.mainClass=package.name.to.your.main.class

$ storm jar /path/to/jar package.name.your.main.class TopologyName 

$ storm kill TopologyName

[My Source Code](https://github.com/baijian/storm-java)

