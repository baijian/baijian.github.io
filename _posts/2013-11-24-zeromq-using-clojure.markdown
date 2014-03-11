---
layout: post
title: "zeroMQ-using-clojure"
date: 2013-11-24 18:27
comments: true
categories: 
---

### ZeroMQ 

ZeroMQ is much more than a queuing system, it is a generalized communication
layer that allows communations between application threads, whereever they are
located, maybe in different nodes in a network or in different processes being executed
in the same machine or same application.ZeroMQ is the abstraction of good old sockets,
a very different kind of sockets.

### Features

* asynchronous and data can be queued

* P2P(peer to peer)

* multi-pattern communications

* TCP and IPC support

* simplify BSD sockets development

### Using ZeroMQ from Clojure

Clojure is a JVM based language, and it is designed for concurrent.Write multi-threads
applications using Clojure is very simple(if you want to know how simple it is see
this demo [multi-wget](https://github.com/baijian/multi-wget)), that's why I use
Clojure here. And ofcourse we will use Clojure to develop our Storm Topology.

ZeroMQ is a native library that is packaged in a Java API via JNI,So you should install
some libs as follows.

* OSX System

You can use `homebrew` or install ZeroMQ from source code like me.
After installing it, input `ls -all /usr/local/lib/libzmq*` to check it.
Then install java native library of zeromq.

$ git clone git://github.com/zeromq/jzmq.git

$ cd jzmq

$ ./autogen.sh

$ ./configure

$ make && sudo make install

$ ls -all /usr/local/lib/libjzm\*

### Trouble Shooting

When I write [zmq-clj](https://github.com/baijian/zmq-clj) which just use
push/pull communication pattern with n-1 fan-in message queue and select
inproc transport to exchange data among threads in the same JVM(and you know
select tcp transport means exchange data among threads in different JVMs).
It always report `can't find jzmq in java.library.path`, then I ask teacher
`Google`, I got a config `:native-path "/usr/local/lib"`, then I run it, It also
report the same exception, then I am crazy, why the demo is normal in my linux 
but not normal in my mac. I don't know how to do next, reinstall
zeromq jzmq again and again,that's no use. Then I stop, and think for a while, use
the following codes to check the actual value of my `java.library.path`.

```clojure
(println (. System getProperty "java.library.path"))
```

Then I found the setting is no use on my mac, `/usr/local/lib` path is not in
my `java.library.path` of my mac OSX, and on default mac OSX is not put `/usr/local/lib`
to `java.library.path`, so I realized that the problem is the setting
`:native-path "/usr/local/lib"`. Then I report a issue on github. Quickyly I get
a reply,[this link](https://github.com/technomancy/leiningen/blob/master/sample.project.clj#L256)
give me the answer, the correct settting is
`:jvm-opts ["-Djava.library.path=/usr/local/lib"]`, this thing tell me that you should
not believe everyone, especially about technique knowledge, everyone may make mistakes.
So you should see official documents. **Yeah, as a engineer, write documents of your
projects is the basic quality.**

### Common Patterns in ZeroMQ

[https://github.com/baijian/zmq-clj](https://github.com/baijian/zmq-clj)

* request/reply

```clojure
(ns zmq-clj.rrclient
 (:use [org.zeromq.clojure :as zmq])
 (:gen-class))

(defn -main []
 (let [context (make-context 1)]
  (println "Connecting to server...")
  (with-open [socket (doto (make-socket context +req+)
                      (connect "tcp://127.0.0.1:5000"))]
   (dotimes [i 10]
    (let [request "Hello"]
     (println "Sending hello" i "...")
     (send- socket (.getBytes request))
     (let [msg (recv socket)]
      (println "Received " (String. msg) " " i)))))))
```

```clojure
(ns zmq-clj.rrserver
 (:use [org.zeromq.clojure :as zmq])
 (:gen-class))

(defn -main []
 (let [context (make-context 1)]
  (with-open [socket (doto (make-socket context +rep+)
                      (bind "tcp://*:5000"))]
   (while (not (.. Thread currentThread isInterrupted))
    (let [msg (recv socket)]
     (println "Receive: " (String. msg))
     (Thread/sleep 1000)
     (send- socket (.getBytes "World")))))))
```

It is just a simple rpc examples, if the client reach the limit
in the buffer of outgoing messages or any other condition, will
block. The server will drop messages.

* publish/subscribe

```clojure
(ns zmq-clj.pubsub
 (:use [org.zeromq.clojure :as zmq])
 (:gen-class))

(def ^:dynamic *ctx* (make-context 1))

(defn -main []
 (future 
  (let [pub (make-socket *ctx* +pub+)]
   (bind pub "tcp://*:5000")
   (loop [msg 0]
    (send- pub (.getBytes (str "Pub " msg)))
    (Thread/sleep 1000)
    (recur (inc msg)))))
 (doseq [i (range 0 5)]
  (future 
   (let [sub (make-socket *ctx* +sub+)]
    (.subscribe sub (.getBytes ""))
    (connect sub "tcp://127.0.0.1:5000")
    (loop [msg (recv sub)]
     (println (str "Sub" i " recv: " (String. msg)))
     (recur (recv sub)))))))
```

Client can subscribe to different publishers and severial clients
can subscribe to the same publisher. If the publisher reaches the
limit in the outgoing buffer limit, it will start dropping messages.
Pub只能发消息，Sub只能接消息，哪一端bind还是connect理论上是无所谓的.
From zmqv3.x, filtering happens at the publisher side when using a
connected protocol(tcp: or ipc:).

* push/pull

```clojure
(ns zmq-clj.pushpull
 (:use [org.zeromq.clojure :as zmq])
 (:gen-class))

(defn work [socket]
 (let [workload (inc (rand-int 100))]
  (send- socket (.getBytes (str workload)))
  workload))

(defn -main[]
 (future
  (let [ctx (make-context 1)]
   (let [push (doto (make-socket ctx +downstream+)
               (bind "tcp://*:5000"))
    sink (doto (make-socket ctx +downstream+)
        (connect "tcp://*:5001"))]
    (send- sink (.getBytes "0"))
    (let [times (repeatedly 100 (partial work push))]
     (printf "Total expected cost: %d msec\n" (apply + times))))))
 (future
  (let [ctx (make-context 1)]
   (let [pull (doto (make-socket ctx +upstream+)
               (connect "tcp://127.0.0.1:5000"))
    sender (doto (make-socket ctx +downstream+)
        (connect "tcp://127.0.0.1:5001"))]
    (while (not (.. Thread currentThread isInterrupted))
     (let [string (recv pull)]
      (println (str "Pull Recv(5000):" (String. string)))
      (Thread/sleep 1000)
      (send- sender (.getBytes "")))))))
    (future
     (let [ctx (make-context 1)]
      (let [receiver (doto (make-socket ctx +upstream+)
                      (bind "tcp://*:5001"))]
       (recv receiver)
       (let [start (System/currentTimeMillis)]
        (dotimes [i 100]
         (recv receiver)
         (if (zero? (mod i 10))
          (print ":")
          (print ".")))
        (println "Total elapsed time:" (- (System/currentTimeMillis) start) "msec"))))))

```

It builds a pipeline of nodes where data is pushed to at least
one connected socket that pulls data. If the memory limit is reached,
the push socket will block. Then messages will not drop.

### Conclusion

ZeroMQ是对底层socket通信的一种封装,理所当然它就很快且效率很高,同时它的这种封装简化了
socket通信的开发难度,掌握常用的通信模式并对这些通信模式进行组合就可以解决不同的应用场景问题,
当然ZeroMQ也不是要取代如RabbitMQ这种东西,我觉得还是要看具体应用场景,应该选择最合适的或者结合使用,当然也可以
看看这篇文档[http://wiki.github.com/rabbitmq/rmq-0mq/](http://wiki.github.com/rabbitmq/rmq-0mq/),

Storm`0.9.0`之前的版本的底层通信采用的是zeromq,但是zeromq在内存控制方面存在一定的
问题,所以`0.9.0`之后的版本将支持netty作为底层传输,目前`0.9.0`还处于`RC`阶段,
ZeroMQ2.x及以下版本在内存管理方面确实存在一些问题,但是随着版本不断的更新会好很多,个人还是很喜欢ZeroMQ的,
现在最新的稳定发布版已经到`4.x`了,也许你使用`3.x`或`4.x`会好很多,
ZeroMQ的高性能等因素也是我想在Storm里运用ZeroMQ的原因,在某些实时
数据分析的场景里运用ZeroMQ做数据源是一个很不错的选择.
