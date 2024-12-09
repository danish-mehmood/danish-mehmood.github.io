# Microservices Architecture Deep Dive Part Three: Communications


<!--more-->

{{< admonition note "Table Of Contents" >}}

:(fa-solid fa-arrow-right-long): Part 1 - [Microservices Architecture Deep Dive](https://danish-mehmood.github.io/microservices-architecture-deep-dive-part-one/)

:(fa-solid fa-arrow-right-long): Part 2 - [API Gateways and Backend For Frontend Pattern](https://danish-mehmood.github.io/microservices-architecture-deep-dive-part-two--api-gateways-and-bff-pattern/)

:(fa-solid fa-arrow-right-long): Part 3 - **Microservices Communication** :(fa-solid fa-arrow-left-long): you are here :(fa-solid fa-location-crosshairs):

:(fa-solid fa-arrow-right-long): Part 4 - [Service Discovery](https://danish-mehmood.github.io/microservices-architecture-deep-dive-part-four-service-discovery/)

:(fa-solid fa-arrow-right-long): Part 5- [Service Mesh](https://danish-mehmood.github.io/microservices-architecture-deep-dive-part-five-service-mesh/)
{{< /admonition >}}

## Preface

In monoliths the application is divided into modules, those modules serve as **_logical_** separation, in microservices the application is divided into services which serve as **_physical_** separation, This changes things drastically, before the modules in monoliths used to communicate with each other using the **_method calls_** but in case of microservices the services would need to communicate using **_network calls_**.
This presents different challenges and trade offs, we have to carefully manage these tradeoffs, there are different patterns which the industry leaders in microservices have comeup with which manage the tradeoffs really well in live environments. But still when deciding on the communication patterns for your microservices you have to pay painstaking attention to detail. Because you have to remember these communication patterns are playing with tradeoffs and you have to choose which fits best your design goals and business domain.

## The mistake

There is a mistake that engineers make very often when they are trying to move to a microservices architecture from a monolith application.
They convert the application modules which were present in monolith to the services and then simply try to convert the inter module function calls in monolith to the **RPC** calls in microservices.

This happens because the engineers have wrong assumptions about microservices (or distributed systems). They are still assuming somethings which they assumed for monolithic architecture like **_they have a reliable network_** (which is not true, networks are never reliable) and another assumption thy could be making is **_latency is still zero_** just like in monoliths, both assumptions are far from truth, and these false assumptions and many other fallacies are coverd in wikipedia entry [Fallacies of distributed computing](https://en.wikipedia.org/wiki/Fallacies_of_distributed_computing?useskin=vector).

If you translate all the inter module function calls to RPC calls, your services would be very chatty, which is the last thing we want. In a monolith, network was not being used to communicate unlike microservices, and every call now on the network has a latency which we want to reduce.

## Challenges

The common challenges in microservices communications include following

- **Network Latency**:Network calls are slower than internal function calls.
- **Service Discovery**: Services need mechanisms to locate one another dynamically.
- **Fault Tolerance**: Failures in communication can cause cascading issues.
- **Data Consistency**: Ensuring consistency across distributed services.
- **Security**: Protecting communication channels and data in transit.
- **Protocol Complexity**: Choosing between REST, gRPC, or message brokers.

## Communication Types

![communication types](images/posts/microservices/partthree/async.jpg)

> Client and services can communicate through many different types of communication, each one targeting a different scenario and goals. Initially, those types of communications can be classified in two axes.

**_The first axis defines if the protocol is synchronous or asynchronous:_**

- HTTP is a **synchronous protocol** for communication. The client sends the request and waits for the response to continue. Even if the http requests are being **_pipelined [^1]_**, even then the responses will be in sequence and the client which sent the request has to wait to get the response and would only then continue doing some thing else, you can think of it as a blocking communication mechanism.

- Protocols like **_AMQP[^2]_** are **_asynchronous_**, In this type of communication the client sends the message/request and does not wait for the response from the reciever(s) to continue doing other things (you can think of this like email communication).

**_The second axis defines if the communication has a single receiver or multiple receivers:_**

- **Single receiver**. Each request must be processed by exactly one receiver or service. An example of this communication is the Command pattern.
<!-- todo-add links to pub/sub and eda -->
- **Multiple receivers**. Each request can be processed by zero to multiple receivers. This type of communication must be asynchronous. An example is the **_publish/subscribe_** mechanism used in patterns like **_Event-driven architecture_**. This is based on an event-bus interface or message broker when propagating data updates between multiple microservices through events; it's usually implemented through a service bus or similar artifact.

> don't get bogged down by the different concepts like pub/sub and event-driven architecture, i will go into the detail of how these things work in future posts.

A microservice-based application will often use a combination of these communication styles. The most common type is single-receiver communication with a synchronous protocol like HTTP/HTTPS when invoking a regular Web API HTTP service. Microservices also typically use messaging protocols for asynchronous communication between microservices.

These axes are good to know so you have clarity on the possible communication mechanisms, but they're not the important concerns when building microservices. Neither the asynchronous nature of client thread execution nor the asynchronous nature of the selected protocol are the important points when integrating microservices. What is important is being able to integrate your microservices asynchronously while maintaining the independence of microservices.

## Asynchronous Integration

As mentioned, the important point when building a microservices-based application is the way you integrate your microservices. Ideally, you should try to minimize the communication between the internal microservices. The fewer communications between microservices, the better. But in many cases, you'll have to somehow integrate the microservices. When you need to do that, the critical rule here is that the communication between the microservices should be asynchronous. That doesn't mean that you have to use a specific protocol (for example, asynchronous messaging versus synchronous HTTP). It just means that the communication between microservices should be done only by propagating data asynchronously, but try not to depend on other internal microservices as part of the initial service's HTTP request/response operation.

If possible, never depend on synchronous communication (request/response) between multiple microservices, not even for queries. **_The goal of each microservice is to be autonomous and available to the client consumer_**, even if the other services that are part of the end-to-end application are down or unhealthy. If you think you need to make a call from one microservice to other microservices (like performing an HTTP request for a data query) to be able to provide a response to a client application, you have an architecture that won't be resilient when some microservices fail.

![integration](images/posts/microservices/partthree/integration.png)

Moreover, having HTTP dependencies between microservices, like when creating long request/response cycles with HTTP request chains, as shown in the first part of the diagram, not only makes your microservices not autonomous but also their performance is impacted as soon as one of the services in that chain isn't performing well.

The more you add synchronous dependencies between microservices, such as query requests, the worse the overall response time gets for the client apps.

> In summary if client requests something from microservice A and this service can't fulfill the request on its own and it talks to microservice B and consequently B needs to talk to microservice C to fulfill the request, then it means that these services are not **_autonomous_**. And if the communication between all these services is synchronous , the latency would keep on adding with each request in the chain and if any single service fails in the chain the failure would cascade and the whole chain would fail.

As shown in the above diagram, in synchronous communication a "chain" of requests is created between microservices while serving the client request. This is an anti-pattern. In asynchronous communication microservices use asynchronous messages or http polling to communicate with other microservices, but the client request is served right away.

If your microservice needs to raise an additional action in another microservice, if possible, do not perform that action synchronously and as part of the original microservice request and reply operation. Instead, do it asynchronously (using asynchronous messaging or integration events, queues, etc.). But, as much as possible, do not invoke the action synchronously as part of the original synchronous request and reply operation.

And finally (and this is where most of the issues arise when building microservices), if your initial microservice needs data that's originally owned by other microservices, do not rely on making synchronous requests for that data. Instead, replicate or propagate that data (only the attributes you need) into the initial service's database by using eventual consistency.

> duplicating some data across several microservices isn't an incorrect design or anti pattern

## References

[^1]: [pipelining](https://en.wikipedia.org/wiki/HTTP_pipelining?useskin=vector)
[^2]: [AMQP](https://en.wikipedia.org/wiki/Advanced_Message_Queuing_Protocol?useskin=vector)

