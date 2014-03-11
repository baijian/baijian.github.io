---
layout: post
title: "Usage-of-RabbitMQ"
date: 2013-09-08 17:00
comments: true
categories: 
---

We select RabbitMQ as our storm spout, so we will introduce 
a basic usage of RabbitMQ first.All blow description will under ***RabbitMQ 3.1.5***

**Definition from wikipedia**

RabbitMQ is open source message broker software(message-oriented middleware)
that implements the [Advanced Message Queuing Protocol](http://en.wikipedia.org/wiki/Advanced_Message_Queuing_Protocol)
standard. The RabbitMQ server is writen in Erlang and is built 
on the [Open Telecom Platform](http://en.wikipedia.org/wiki/Open_Telecom_Platform)
framework for clustering and failover.

**Components of RabbitMQ Project**

* The RabbitMQ exchange server itself
* Gateways for HTTP, STOMP and MQTT protocols
* AMQP client libraries for **JAVA**, **Erlang** and so on
* A plug-in platform for custom additions, plugins such as ***Management*** plug-in

### Install RabbitMQ

**Mac OS X**

$ brew update

$ brew install rabbitmq

$ export PATH=$PATH;/usr/local/sbin

$ rabbitmq-server -detached

$ rabbitmqctl status

$ rabbitmqctl stop

**RHEL Linux**

You want to install RabbitMQ through ```yum install rabbitmq-server```, 
maybe the version is not the latest stable version. So you'd better download
the latest stable rpm package to install, with ```yum``` you can solve your dependency.

RabbitMQ有依赖于erlang,erlang的版本太老会对RabbitMQ的性能有一些影响,而且
旧一点的erlang版本不支持诸如SSL,基于HTTP的像management这样得插件.所以首先需要安装
比较新一点的erlang版本,RHEL5或者CentOS 5的系统可以用下面的方式装erlang.

$ wget -o /etc/yum.repos.d/epel-erlang.repo http://repos.fedorapeople.org/repos/peter/erlang/epel-erlang.repo

$ yum install erlang || yum update erlang

下面继续安装RabbitMQ Server.

$ wget http://www.rabbitmq.com/releases/rabbitmq-server/v3.1.5/rabbitmq-server-3.1.5-1.noarch.rpm

$ rpm --import http://www.rabbitmq.com/rabbitmq-signing-key-public.asc

$ yum install rabbitmq-server-3.1.5-1.noarch.rpm

Then you can start or stop your RabbitMQ server.

**Ubuntu Linux**

Add blow to /etc/apt/source.list

$ deb http://www.rabbitmq.com/debian testing main

$ wget http://www.rabbitmq.com/rabbitmq-signing-key-public.asc

$ sudo apt-key add rabbitmq-signing-key-public.asc

$ sudo apt-get update

$ sudo apt-get install rabbitmq-server


### One Producer & One Consumer

下面介绍RabbitMQ最简单地通信模式,从生产者发送消息,传递消息给消费者,
以一个生产者一个消费者为例,消息在从生产者传递给消费者的过程中可以路由
这个消息,或者缓存住或者直接丢掉.
Producer就是向一个队列里面发消息,没什么特别的,
所以是一个Producer还是多个Producer其实无所谓的.
下面写一段最简单的Java代码例子,当然你需要启动好RabbitMQ server,下载并安装好
RabbitMQ的java client library.下面的代码中我也会通过一些注释来解释一些概念.

**Sender**

{% highlight java %}
public class RabbitMQSend {

    private final static String QUEUE_NAME = "test";

    public static void main(String[] args) throws Exception {
        
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        /* 
           新建一个client到broker的连接,broker就是RabbitMQ-server
           RabbitMQ的client到broker的连接是TCP连接.
         */
        Connection connection = factory.newConnection();
        /*
           按AMQP协议创建了一个连接后需要创建一个Channel
           并通过它能进行通信.
         */
        Channel channel = connection.createChannel();

        /*
           声明一个队列来给接受生产者发送的消息,除了QUEUE_NAME外的参数下面会介绍.
         */
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);

        String message = "Hello World!";

        //第一个参数是exchange,这里先默认它空,下面再介绍,null这个参数也下面介绍.
        channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
        System.out.println("[RabbitMQ-Sender] Send '" + message + "'");

        channel.close();
        connection.close();
    }

}
{% endhighlight %}

**Receiver**

{% highlight java %}
public class RabbitMQRecv {

    private final static String QUEUE_NAME = "test";

    public static void main(String[] args) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");

        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        System.out.println("[*] Waiting for messages. To exit press CTRL+C");

        QueueingConsumer consumer = new QueueingConsumer(channel);
        channel.basicConsume(QUEUE_NAME, true, consumer);

        while(true) {
            /*
               Consumer.nextDelivery() 将会阻塞直到下一个消息来.
             */
            QueueingConsumer.Delivery delivery = consumer.nextDelivery();
            String message = new String(delivery.getBody());
            System.out.println("[RabbitMQ-Receiver] Received '" + message + "'");
        }
    }
}
{% endhighlight %}

好了,第一个简单的例子完成了,你可以自己编译运行它看看效果,
之所以说它简单是因为,很多默认值隐藏了一些概念,
下面通过其他的例子来引出这些概念.

### Multi-Producer & Multi-Consumer

上面讲一个生产者一个消费者的时候提到有几个生产者其实无所谓,但是有几个
消费者就需要关心了,它是如何给多个消费者分发消息的,如何才能公平的给所有
消费者分发消息,再顺便解释下如何保证消息可靠地被消费者完全的消费了,
如果RabbitMQ Server down了,重启后它的消息和队列是否还能保持呢.

RabbitMQ默认的消息分发规则是轮询,RabbitMQ的消息支持acknowledgment,利用ack机制
可以保证消息被消费者可靠地消费掉了.下面来一段Java代码示例,下面的代码是基于上
面得代码修改的,我会结合代码在注释里进行解释.

**Sender**

{% highlight java %}
public class RabbitMQSend {

    private final static String QUEUE_NAME = "test";

    public static void main(String[] args) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");

        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        /*
           在上面的例子中durable这个参数给的默认值false.
           如何设置为true,当RabbitMQ Server重启或者非正常退出了,
           消息会被保存到磁盘(持久化),当然这样不能百分之百保证消息都保存到磁盘了,
           如果对publish代码运用事务的话会增强guarantee.Send的durable参数为
           true,receiver的durable也必须是true,且需要mark message,如下.
           需要明确的是如果开启消息的持久化,一定会对性能产生一定的影响.
         */
        boolean durable = true;
        channel.queueDeclare(QUEUE_NAME, durable, false, false, null);

        String message = "Hello World!";

        /*
           As queue declare to be durable,so we need to mark the message
           as persistent by setting PERSISTENT_TEXT_PLAIN.
         */
        channel.basicPublish("", QUEUE_NAME, MessageProperties.PERSISTENT_TEXT_PLAIN,
                message.getBytes());
        System.out.println("[RabbitMQ-Send] Send '" + message + "'");

        channel.close();
        connection.close();
    }
}
{% endhighlight %}

**Receiver**

{% highlight java %}
public class RabbitMQRecv {

    private final static String QUEUE_NAME = "test";
    private final static long WAIT_FOR_NEXT_MESSAGE = 1L;

    public static void main(String[] args) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");

        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        /*
           上面说过如果有多个消费者即worker,RabbitMQ将以轮询的方式
           传递消息,有可能某一个消费者接收到的消息比较复杂,处理起来
           比较慢,而其他的消费者接收到的消息处理起来都比较快,但是RabbitMQ
           并不知道,那么这样得轮询就是不公平的,所以我们需要设置basicQos,
           设置了basicQos,这样只有消费者ack了一个消息,RabbitMQ菜会继续传递
           下一个消息给这个消费者,这样就公平了一点了,prefetchCount是消费者
           缓存队列消息的数量,可以设置的大一点.
           这种情况下如何所有的消费者都非常的忙碌,那么队列里面的消息很容易
           暴增,所以需要做一个监控,在适当地时候用增加消费者或者其他办法解决
           队列消息太多的问题.
         */
        int  prefetchCount = 500;
        channel.basicQos(prefetchCount);

        boolean durable = true;
        /*
           In Sender we have define the queue to be durable, so Receiver
           must also define as durable.
           So queue Declare code should be same with Sender.
         */
        channel.queueDeclare(QUEUE_NAME, durable, false, false, null);
        System.out.println("[*] Waiting for messages. To exit press CTRL+C");

        QueueingConsumer consumer = new QueueingConsumer(channel);
        boolean autoAck = false;
        /*
           之前的例子autoAck这个参数给的true,即RabbitMQ发送完一条消息后
           会自动将它ack,即将它从内存移去,如果设置成false,就需要消费者
           在消息处理完后通知RabbitMQ.
         */
        channel.basicConsume(QUEUE_NAME, autoAck, consumer);

        while(true) {
            /*
               上面提到consumer.nextDelivery()将会阻塞住,所以这里给一个
               1ms默认timeout值,即当没有消息1ms后就返回null,在某些应用场景下是不希望
               阻塞住的,设置一个比较小的timeout是比较可取的做法.
             */
            QueueingConsumer.Delivery delivery = consumer.nextDelivery(WAIT_FOR_NEXT_MESSAGE);
            String message = new String(delivery.getBody());
            /*
               如何autoAck设置的false,下面这个操作就非常重要了!
               ack the message which have been received.
             */
            channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
            System.out.println("[RabbitMQ-Receiver] Received '" + message + "'");
        }
    }
}
{% endhighlight %}

### Pub-Sub & Routing

下面会讲解Pub-Sub通信模式,在讲解Pub-Sub模式时会引入一个重要的概念:exchange,前面只所以没有提到那是
因为用了默认的exchange,也就隐藏了这个概念.然后我会在Pub-Sub的基础上深入路由的概念.

生产者发送的消息实际上并没有直接被传递到队列,而是被发送到了Exchange,Exchange会根据一些规则将消息路由
到不同的队列,路由的规则是消费者连接Broker的时候Bind上去的,且路由规则可以分为*fanout* *direct*
*topic* *headers* 几种类型.

**fanout**

这是最简单的exchange,fanout类型的exchange仅仅将接收到的消息广播到所有队列.下面看代码:

{% highlight java %}
public class Sender {
    private static final String EXCHANGE_NAME = "test";

    public static void main(String[] args) throws Exception{
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        //声明fanout类型的exchange
        channel.exchangeDeclare(EXCHANGE_NAME, "fanout");
        String message = "hello world";
        //fanout类型的exchange会将接收到的消息发送给所有的队列,所以这里队列名参数给空就可以了.
        //这时如果没有队列绑定上来,exchange接收到的消息将会直接丢掉.
        channel.basicPublish(EXCHANGE_NAME, "", null, message.getBytes());
        channel.close();
        connection.close();
    }
}

public class Recv {
    private static final String EXCHANGE_NAME = "logs";
    public static void main(String[] args) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        channel.exchangeDeclare(EXCHANGE_NAME, "fanout");
        //这里的队列名字是随机的,声明了一个非持久化的,独立消费的,自动删除的队列.
        String queueName = channel.queueDeclare().getQueue();
        //这里是fanout类型的exchange,所以第三个参数bindKey可以忽略,直接为空就可以了
        channel.queueBind(queueName, EXCHANGE_NAME, "");
        QueueingConsumer consumer = new QueueingConsumer(channel);
        //这里广播消息还是autoAck的好,广播消息有丢失正常,如果不想丢失可选用下面其他类型的exchange
        //fanout类型的exchange灵活性不是很好,一般就用于广播消息.
        channel.basicConsumer(queueName, true, consumer);
        while(True) {
            QueueingConsumer.Delivery delivery = consumer.nextDelivery();
            String msg = new String(delivery.getBody());
            System.out.println(msg);
        }
    }
}
{% endhighlight %}

**direct**

这里提到的direct类型的exchange就需要用到binding key的参数了,队列在绑定到exchange
的时候需要指定一个bindKey,这样exchange就知道如何分配源头过来的消息了.

{% highlight java %}
public class Send {

    private static final String EXCHANGE_NAME = "test";

    public static void main(String[] argv) throws Exception {

            ConnectionFactory factory = new ConnectionFactory();
            factory.setHost("localhost");
            Connection connection = factory.newConnection();
            Channel channel = connection.createChannel();

            channel.exchangeDeclare(EXCHANGE_NAME, "direct");

            String routinKey = "type1";
            String message = "hello world";

            channel.basicPublish(EXCHANGE_NAME, routinKey, null, message.getBytes());

            channel.close();
            connection.close();
        }
}

public class Recv {

    private static final String EXCHANGE_NAME = "test";

    public static void main(String[] args) throws Exception {

           ConnectionFactory factory = new ConnectionFactory();
           factory.setHost("localhost");
           Connection connection = factory.newConnection();
           Channel channel = connection.createChannel();

           channel.exchangeDeclarePassive(EXCHANGE_NAME, "direct");
           String queueName = channel.queueDeclare().getQueue();

           channel.queueBind(queueName, EXCHANGE_NAME, "type1");

           QueueingConsumer consumer = new QueueingConsumer(channel);
           channel.basicConsume(queueName, true, consumer);

           while (true) {
               QueueingConsumer.Delivery delivery = consumer.nextDelivery();
               String message = new String(delivery.getBody());
               String routingKey = delivery.getEnvelope().getRoutingKey();

               System.out.println(" [x] Received '" + routingKey + "':'" + message + "'");
           }
   }
}
{% endhighlight %}

上面Receiver的代码声明exchange的时候用到了*exchangeDeclarePassive(exchange, type)*,那么就顺便讲解下
几个不同的exchange声明的区别.

{% highlight java %}
channel.exchangeDeclare(EXCHANGE_NAME, "direct");
{% endhighlight %}

上面的代码是声明一个non-autodelete, non-durable且没有额外参数的exchange.

{% highlight java %}
channel.exchangeDeclare(EXCHANGE_NAME, "direct", durable);
{% endhighlight %}

这比上面的例子多了一个可以自定义的选择参数durable,是否持久化.

{% highlight java %}
channel.exchangeDeclare(EXCHANGE_NAME, "direct", durable, autoDelete, Map<String, Object> args);
{% endhighlight %}

这又比上面的例子多了两个可控制的参数.

{% highlight java %}
channel.exchangeDeclare(EXCHANGE_NAME, "direct", durable, autoDelete,internal, args);
{% endhighlight %}

这里又多了一个参数internal,如果设置为true,那么这个exchange将不能直接pub消息到client.
这是参数最全的一种情况,这里是完全由自己去设置exchange的参数.

{% highlight java %}
channel.exchangeDeclarePassive(EXCHANGE_NAME);
{% endhighlight %}

这里会先检查EXCHANGE_NAME是否存在,不存在会报404:channel异常.


**topics**

topics类型的exchange提供了更大的灵活性,它的灵活性体现在routingKey上面,它的routinKey是
由 *.* 隔开的word,像 *name.age.hobby* 这样,绑定的时候bindingKey可以是 *name.\**  *name.age.hobby* 
*name.#* 等, * 代表一个明确的word,#代表>=0个word,这样在bind队列的时候灵活性就更大了.就不给示例代码了.


### RPC

上面讲的通信模式可以运用在master-worker的场景下,比如Storm的数据源头用RabbitMQ,消息进入队列,Strom的
spout共享一个队列共同消费这个队列的消息,利用RabbitMQ的ack机制可以确保Strom将队列的消息完整的处理了.

这里的RPC即Remote procedure call可以运用在将没有线程概念且是串行执行的任务并行化执行(当然这样的任务
是需要拿到执行的返回结果的),并行话执行就可以将时间花销从线性相加降低到并行执行最长的那个时间,这里暂时不做详细解释.
