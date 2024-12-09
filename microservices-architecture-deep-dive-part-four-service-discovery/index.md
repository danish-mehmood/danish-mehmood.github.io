# Microservices Architecture Deep Dive Part Four: Service Discovery


{{< admonition note "Table Of Contents" >}}

:(fa-solid fa-arrow-right-long): Part 1 - [Microservices Architecture Deep Dive](https://danish-mehmood.github.io/microservices-architecture-deep-dive-part-one/)

:(fa-solid fa-arrow-right-long): Part 2 - [API Gateways and Backend For Frontend Pattern](https://danish-mehmood.github.io/microservices-architecture-deep-dive-part-two--api-gateways-and-bff-pattern/)

:(fa-solid fa-arrow-right-long): Part 3 - [Microservices Communication](https://danish-mehmood.github.io/microservices-architecture-deep-dive-part-three-communications/)

:(fa-solid fa-arrow-right-long): Part 4 - **Service Discovery** :(fa-solid fa-arrow-left-long): you are here :(fa-solid fa-location-crosshairs):

:(fa-solid fa-arrow-right-long): Part 5 - [Service Mesh](https://danish-mehmood.github.io/microservices-architecture-deep-dive-part-five-service-mesh/)
{{< /admonition >}}

## The Problem

As i discussed in the [last](https://danish-mehmood.github.io/microservices-architecture-deep-dive-part-three-communications/) blog post, the communication patterns in microservices are way different than the patterns in monoliths, in monoliths the modules communicate using **_method calls_** but in case of microservices, services communicate using the **_network calls_**.

This small difference has **massive** implications, now the services which are involved in communication are living potentially on different machines and to communicate over the network the services would need to **know** each others IP addresses.

The naive solution is to hard code the IP addresses of every service the communicating service needs to communicate with in the code, but this is not prudent in case of highly distributed environments like microservices.

**_But why?_**

the reason is this that modern microservices are deployed in **containers** and they are also auto scaled, scaled up and down, both ways. As a result all these microservice instances running are highly **_ephemeral_** and tools like **kubernetes** are used to make sure all the services are up all the time. And the way it makes sure that the services are up and running is by running new instances of the service when it notices some instance go down or being problematic.

Due to all this the IP addresses of the services are no more static and they keep on changing. And our solution of **_hard coding_** the servies IP addresses is rendered obsolete.

## The Solution: Service Registry

The solution is **service registry**, we would have an additional service running at all times, which would act as a registry for all the services in the environment. Every new service which would be available in the environment would register to the service registry.

### The Process

- The service registry is running on the known IP.
- each service in the environment would register itself to the registry, for instance there is this new service called **_service A_**, on the startup this service is required to register to the service registry first, so that it is locateable by other services in the environment.
- Now if **_service B_** wants to talk to **_service A_** it would go to service registry and service registry would give service B the address of service A so that they can communicate with each other.

> you see now we would not face any problem if the IP addresses of the service container are dynamic and ephemeral, because whenever the service comes up it is required to register to service registry.

### Health Checks

You may be wondering that how does the service registry know that some instance of the service has gone down and now past IP address is no more valid?
for that service registry employs some method of service health check.

#### Heart Beat Check

some service registeries have a heart beat mechanism built in. service registry sends the periodic heart beats to registered services to know whether they are up or not, if they do not respond then the service IP is deemed invalid.

#### Periodic Registration

some service registeries require registered services to register again after a regular interval of time, if they fail to do so the IP address associated with the service which failed to register would be deemed invalid.

### The Types Of service Registry

There are two main types of service registries, they differ in a way they go about the process of service discovery, the types are following

1. client side service registery
2. server side service registery

#### Client Side Service Registry

Consider the following example.

You want to order pizza, you know the place you want to order from but you don't know how to contact them, you go to **_google_** and search for the place and get their number, then you call them and order pizza.

This is how the client side service registry works, in this example consider yourself **service A** the pizza place **service B** and google **service registry**. In client side registry you would ask the registry for the IP address and it will give you the IP address and then its you who contact the other service.

#### Server Side Service Registry

Consider the following example.

You want to contact the **CEO** of some company, you don't have the contact information of the CEO but you do know the **exchange** number of the company, you call the exchange and let them know you want to talk to CEO, they will not give you the CEO's phone number but instead would forward your call to the CEO themselves.

This is exactly how the server side registry would work, **service A** would contact the **registry** and tell it that it wants to contact the **service B** unlike the client side registry it will not give the IP of service B to you instead it will contact service B it self and act like a proxy.

### Well Known Service Registries

the following are well known service registries

1. Consul
1. Eureka
1. ETCD
1. Zookeeper

