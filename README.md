MBassador
=========

MBassador is a light-weight, high-performance message (event) bus implementation based on the [publish subscribe pattern](https://en.wikipedia.org/wiki/Publish-subscribe_pattern). It is designed for ease of use and aims to be feature rich and extensible while preserving resource efficiency and performance. 

The core of MBassador's high performance is a specialized data structure that minimizes lock contention such that performance degradation of concurrent read/write access is minimal. The advantages of this design are illustrated in the [eventbus-performance](https://github.com/bennidi/eventbus-performance) github repository.

Read this introduction to get an overview of MBassadors features. There is also some documentation in the Wiki - although admittedly not enough to make a developer happy. Additionally, you can browse the [javadoc](http://bennidi.github.io/mbassador/)

There is a [spring-extension](https://github.com/bennidi/mbassador-spring) available to support CDI-like transactional message sending in a Spring environment. This is a good example of integration with other frameworks.

>  ################   NOTE #################### 

> [15.10.2015] My spare time programming efforts have shifted to another open source project - [openmediaid](http://www.openmediaid.org). I will not be able to actively push development of this library anymore. Any developers interested in becoming co-maintainers of this library... you are very welcome! 

Table of contents:
  *  [Usage](#usage)
  *  [Features](#features)
  *  [Installation](#installation)
  *  [Wiki](#wiki)
  *  [Release Notes](#release-notes)
  *  [Integrations](#integrations)
  *  [Credits](#credits)
  *  [Contribute](#contribute)
  *  [License](#license)

<h2>Usage</h2>

Using MBassador in your project is very easy. Create as many instances of MBassador as you like (usually a singleton will do) ` bus = new MBassador()`, mark and configure your message handlers with `@Handler` annotations and finally register the listeners at any MBassador instance `bus.subscribe(aListener)`. Start sending messages to your listeners using one of MBassador's publication methods `bus.post(message).now()` or `bus.post(message).asynchronously()`.

As a first reference, consider this illustrative example. You might want to have a look at the collection of [examples](./examples) to see its features on more detail.

      // Define your listener
     class SimpleFileListener{
     
       @Handler
       public void handle(File msg){
          // do something with the file
       }
     
     }
  
      // somewhere else in your code
  
     MBassador bus = new MBassador();
     Object listener = new SimpleFileListener();
     bus.subscribe (listener);
     bus.post(new File("/tmp/smallfile.csv")).now();
     bus.post(new File("/tmp/bigfile.csv")).asynchronously();
   

##Features


> Annotation driven

|Annotation|Function|
|:-----|:-----|
|`@Handler`|Defines and customizes a message handler. Any well-formed method annotated with `@Handler` will cause instances of the defining class to be treated as message listeners|
|`@Listener`|Can be used to customize listener wide configuration like the used reference type|
|`@Enveloped`|A message envelope can be used to pass messages of different types into a single handler|
|`@Filter`|Add filtering to prevent certain messages from being published|

> Delivers everything

Messages do not need to implement any interface and can be of any type. The class hierarchy of a message is considered during message delivery, such that handlers will also receive subtypes of the message type they consume for - e.g. a handler of Object.class receives everything. Messages that do not match any handler result in the publication of a `DeadMessage` object which wraps the original message. DeadMessage events can be handled by registering listeners that handle DeadMessage.

> Synchronous and asynchronous message delivery

A handler can be invoked to handle a message either synchronously or asynchronously. This is configurable for each handler via annotations. Message publication itself supports synchronous (method blocks until messages are delivered to all handlers) or asynchronous (fire and forget) dispatch

> Configurable reference types

By default, MBassador uses weak references for listeners to relieve the programmer of the need to explicitly unsubscribe listeners that are not used anymore and avoid memory-leaks. This is very comfortable in container managed environments where listeners are created and destroyed by frameworks, i.e. Spring, Guice etc. Just stuff everything into the message bus, it will ignore objects without message handlers and automatically clean-up orphaned weak references after the garbage collector has done its job. Instead of using weak references, a listener can be configured to be referenced using strong references using `@Listener(references=References.Strong)`. Strongly referenced listeners will stick around until explicitly unsubscribed.

> Filtering

MBassador offers static message filtering. Filters are configured using annotations and multiple filters can be attached to a single message handler. Since version 1.2.0 Java EL expressions in `@Handler` are another way to define conditional message dispatch. Messages that have matching handlers but do not pass the configured filters result in the publication of a FilteredMessage object which wraps the original message. FilteredMessage events can be handled by registering listeners that handle FilteredMessage.

> Enveloped messages

Message handlers can declare to receive an enveloped message using `Enveloped`. The envelope can wrap different types of messages to allow a single handler to handle multiple, unrelated message types.

> Handler priorities

A handler can be associated with a priority to influence the order in which messages are delivered when multiple matching handlers exist

> Custom error handling

Errors during message delivery are sent to all registered error handlers which can be added to the bus as necessary.

> Extensibility

MBassador is designed to be extensible with custom implementations of various components like message dispatchers and handler invocations (using the decorator pattern), metadata reader (you can add your own annotations) and factories for different kinds of objects. A configuration object is used to customize the different configurable parts, see [Features](https://github.com/bennidi/mbassador/wiki/Components#Feature)

<h2>Installation</h2>
MBassador is available from the Maven Central Repository using the following coordinates:
```xml
    <dependency>
        <groupId>net.engio</groupId>
        <artifactId>mbassador</artifactId>
        <version>{see.git.tags.for.latest.version}</version>
    </dependency>
```

You can also download binary release and javadoc from the [maven central repository](http://search.maven.org/#search|ga|1|mbassador). Of course you can always clone the repository and build from source.

<h2>Wiki</h2>
There is ongoing effort to extend documentation and provide code samples and detailed explanations of how the message bus works. Code samples can also be found in the various test cases. Please read about the terminology used in this project to avoid confusion and misunderstanding.

<h2>Release Notes</h2>

Release notes moved to the [changelog](./changelog).


<h2>Credits</h2>
The initial inspiration for creating this component comes from Google Guava's event bus implementation.
I liked the simplicity of its design and I trust in the code quality of google libraries. The main reason it proved to be unusable for our scenario was that it uses strong references to the listeners.

I want to thank the development team from [friendsurance](http://www.friendsurance.de) for their support and feedback on the bus implementation and the management for allowing me to publish the component as an open source project.

I also want to thank all githubbers who have made [contributions](https://github.com/bennidi/mbassador/pulls?q=is%3Apr+is%3Aclosed). It was always a pleasure to see how users got engaged into the libraries development and support. Special thanks go to
+ [arne-vandamme](http://github.com/arne-vandamme) for adding support for [meta-annotations](https://github.com/bennidi/mbassador/pull/74)
+ [Bernd Rosstauscher](http://github.com/Rossi1337) for providing an initial integration with JUEL
+ [David Sowerby](http://github.com/davidsowerby) for answering user questions, his tutorial on [guice integration](bennidi/mbassador/wiki/guice-integration) and his various PRs
+ [dorkbox](http://github.com/dorkbox) for various PRs and his incredible [work on performance tuning](http://github.com/bennidi/eventbus-performance/issues/1) which is still to be integrated
+ [durron597](http://github.com/durron597) for his many PRs and the help he offered to other users

Many thanks also to ej-technologies for providing me with an open source license of 
[![JProfiler](http://www.ej-technologies.com/images/banners/jprofiler_small.png)](http://www.ej-technologies.com/products/jprofiler/overview.html) and Jetbrains for a license of [IntelliJ IDEA](http://www.jetbrains.com/idea/)

Mbassador uses the following open source projects:

* [jUnit](http://www.junit.org)
* [maven](http://www.maven.org)
* [mockito](http://www.mockito.org)
* [slf4j](http://www.slf4j.org)
* [Odysseus JUEL](http://juel.sourceforge.net/guide/start.html)

Special thanks also to [Sonatype](http://www.sonatype.com/) for the hosting of their [oss nexus repository](https://oss.sonatype.org/).


##Contribute

Please feel invited to contribute by creating a pull request to submit the code you would like to be included. Make your PRs small and provide test code! Take a look at [this issue](bennidi/mbassador#109) for a good example.

Sample code and documentation are both very appreciated contributions. Especially integration with different frameworks is of great value. Feel free and welcome to create Wiki pages to share your code and ideas. Example: [Guice integration](https://github.com/bennidi/mbassador/wiki/Guice-Integration)

<h2>License</h2>

This project is distributed under the terms of the MIT License. See file "LICENSE" for further reference.