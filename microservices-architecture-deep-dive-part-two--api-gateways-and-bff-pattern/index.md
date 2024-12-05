# Microservices Architecture Deep Dive Part Two : API Gateways and Backend For Frontend Pattern


API gateway is a common pattern majority of microservices out in the world follow, to address some issues and Backend for frontend is a pattern which is a natural extension of API gateway pattern which helps the microservices scale very smoothly.

<!--more-->

{{< admonition note "Table Of Contents" >}}

:(fa-solid fa-arrow-right-long): Part 1 - [Microservices Architecture Deep Dive](https://danish-mehmood.github.io/microservices-architecture-deep-dive-part-one/)

:(fa-solid fa-arrow-right-long): Part 2 - **API Gateways and Backend For Frontend Pattern** :(fa-solid fa-arrow-left-long): you are here :(fa-solid fa-location-crosshairs):

:(fa-solid fa-arrow-right-long): Part 3 - [Microservices Communication](https://danish-mehmood.github.io/microservices-architecture-deep-dive-part-three-communications/)

:(fa-solid fa-arrow-right-long): Part 4 - [Service Discovery](https://danish-mehmood.github.io/microservices-architecture-deep-dive-part-four-service-discovery/)
{{< /admonition >}}

### The Problem

People who are new to learning about microservices architecture and when they first come across the API gateway pattern, naturally they would want to go a little deep into what gateways are, and when they do that they are often confused, because they get different uses and definitions of API gateway that they no longer could figure out what problem is it trying to solve.

> let me tell you API gateways exist to solve a very specific, single problem which is related to communication between the clients and services, everything else that you hear about API gateways being capable of doing are just some nice **sideeffects** we use to our advantage.

Imagine yourself being a frontend engineer and you are creating the UI for the ecommerce application, you go to the backend engineer and ask him that you are developing the login experience **_what API endpoint should you use on the client side?_**
and the backend engineer would tell you that the **Authentication Service** is the one which would handle the login, so they give you the endpoint for authentication service, now after some days you start developing the Cart functionality on the frontend and you get to know from the backend team that, you would need to make use of two more microservices the payment and shipping service to implement that and this continues for the life time of the project.

**_Do you see the problem?_**

The clients are directly communicating with the microservices, utilizing the API exposed by the services, and this is a problem, the client is highly **coupled** with the backend services, if some endpoint changes the client would need to change too. and coupling is bad as i discussed in my [last](https://danish-mehmood.github.io/microservices-architecture-deep-dive-part-one/) post.

We need an abstraction between the clients and the services, and that abstraction is **API Gateway**.

The API gateways sits between the clients and services and now clients only need to know about the gateway and only talk to it, the clients don't need to worry about hundreds of services to talk to, gateway will handle that. This also decouples the clients from services, because now microservices can change all they want to change and client won't need changing because now there is a level of indirection between clients and services.

### The Side-Effects

This communication problem is the primary problem that gateways exist to solve, but now that we have single point of entry to the microservice architecture we have, we can accomplish alot of other good things for our infrastructure.
Now that gateway is the entrypoint for our infra we can accomodate the **cross cutting** features in the gateway itself, because these features are required by all the services in the infra, obviously it would be a madness to implement these for each service individually in different techstacks.

Following sections will go into some of the features API Gateways implement (the features which are commonly attributed to API gateways)

#### Authentication

This is the major and obvious one, every microservice needs some kind of authentication and identity management, so why do it separately for every service, so the common pattern is to have authentication placed in an **Edge Service** like API Gateway.

#### Protocol Translation

As i did discuss in the [last](https://danish-mehmood.github.io/microservices-architecture-deep-dive-part-one/) post one of the advantage of having microservices is flexibility with choosing the technology for the implementation of the service, one microservice can have an **REST** api, one could have **graphQL** the other could have a **grpc** implemented. If clients want to talk to them, they have to speak all these languages (protocols) to implement the client side successfully which is madness.

So API gateway solves this problem as well by sitting between the services and clients, it acts as a protocol interpreter because as services are only talking to the gateway they only need to speak one protocol mainly **REST** and API gateway would take care of talking to diverse services in diverse protocols.

#### Rate Limiting

Another cross cutting concern is rate limiting, API gateway can implement the cutting edge rate limiting algorithms on the edge instead of these algorithms being part of every service which saves us from redundancy.

#### Request Routing and Load Balancing

API gateway is also responsible to Route incoming API requests to appropriate backend services based on the request path, headers, or other criteria.
API gateway is also used to implement the load balancer for the services to Distribute traffic across multiple service instances for high availability and load balancing.

#### Caching

API gateway is naturally a good candidate for caching the service responses to reduce latency.

#### Monitoring And Logging

API gateways being the entrypoint are very suitable for the implementation of monitoring and logging solutions. whenever the gateway comes into action simply log the stuff and trigger the monitoring middleware.

#### Others

Some other common functionalities of API gateways include

- Service discovery
- Security features
- CORS policies
- API versioning

### How to scale

![GWLB](images/posts/microservices/parttwo/gwlb.png)

Now i am sure one question must have popped in your mind, that **_wouldn't the API gateway become a bottle neck as all the traffic is going through the gateway and it is acting like a funnel?_** you are right! there is a potential of gateway becoming a bottle neck, but there is a simple solution.
We can horizontally scale the API gateway and then put a load balancer in front of it, now the clients request will come to load balancer first and then the load balancer will distribute the request among the gateways.

### Backend For Frontend

![BFF](images/posts/microservices/parttwo/bff.gif)

#### The Problem

During the whole blog post i have been mentioning the word **"Client"**, but the client itself is a generic term and it could be many things like a client could be a PC, a mobile, an IOT device or another microservice.
All these clients are different in many ways, a PC for instance has all the resources in abundance like big screen, alot of memory, big compute power e.t.c, mobiles on the other hand have small screen real estate, small memory, small compute and also limited power supply.

This is the reason the backend APIs they are consuming have to be tailor made because all these clients have their own strengths and weaknesses and our goal as engineers is to exploit their strengths and mitigate the weaknesses as much as we can.

The mobile clients and PC clients have different resources hence they can't just have same API endpoints available to consume, mobile clients show limited data and try to make as few HTTP round trips to fetch data as possible because each http request drains battery and increases latency, technology like graphql is more suitable for mobiles then the HTTP REST api.

Following is the depiction of difference between simple API Gateway and the gateways following the BFF architecture

![BFF](images/posts/microservices/parttwo/contrast.png)

You see now all the clients have a tailored gateway now they will get what they need in the most efficient way possible. All the data fetching and then aggregation will now move to the tailored gateway and the client will focus on the frontend logic only.

### Useful Links

- [sam newman BFF](https://samnewman.io/patterns/architectural/bff/)
- [amazon api gateway](https://docs.aws.amazon.com/apigateway/latest/developerguide/welcome.html)
- [microsoft api gateway architecture](https://learn.microsoft.com/en-us/azure/architecture/microservices/design/gateway)

