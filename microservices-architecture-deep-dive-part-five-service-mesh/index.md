# Microservices Architecture Deep Dive Part Five: Service Mesh


{{< admonition note "Table Of Contents" >}}

:(fa-solid fa-arrow-right-long): Part 1 - [Microservices Architecture Deep Dive](https://danish-mehmood.github.io/microservices-architecture-deep-dive-part-one/)

:(fa-solid fa-arrow-right-long): Part 2 - [API Gateways and Backend For Frontend Pattern](https://danish-mehmood.github.io/microservices-architecture-deep-dive-part-two--api-gateways-and-bff-pattern/)

:(fa-solid fa-arrow-right-long): Part 3 - [Microservices Communication](https://danish-mehmood.github.io/microservices-architecture-deep-dive-part-three-communications/)

:(fa-solid fa-arrow-right-long): Part 4 - [Service Discovery](https://danish-mehmood.github.io/microservices-architecture-deep-dive-part-three-communications/)

:(fa-solid fa-arrow-right-long): Part 5 - **Service Mesh** :(fa-solid fa-arrow-left-long): you are here :(fa-solid fa-location-crosshairs):
{{< /admonition >}}

## The Problem

Imagine you are working with a microservices architecture and you have 10s of services running, And these services work together by communicating with each other to achieve a goal. Now imagine every team working on different microservices built using different tech stack need to write the inter service communication logic by hand. Thats a waste of time and effort. Isn't it?.

Because not only you have to call the services apis but you also need to handle things like retry logic, load balancing, api sercurity,service discovery, rate limiting, authentication, observeability and much more. Writing all this logic for every service in the system is **cumbersome** and **error prone**. Because now alot of logic is going in to writing the communication middleware for the service and not the actual logic of the service.

## The Solution

Service meshes present a very good solution to this problem, service mesh is a peice of software that abstracts away all the communication logic from the application layer and takes it to the network, now the application doesn't need to worry about the communication detail and only need to focus on the service logic itself.

> a service mesh is a dedicated infrastructure layer for facilitating service-to-service communications between services or microservices using a proxy.

## API Gateway Vs Service Mesh

API gateways and service meshes are two different architectural components that address different concerns in a microservices-based application. While they may have some overlapping functionality, they serve distinct purposes and are often used together for a comprehensive solution.

### API Gateway:

- **Edge component:** An API gateway is an entry point for external clients (such as web browsers, mobile apps, or other services) to access the microservices in the application. It acts as a reverse proxy and a single point of entry for all incoming requests, abstracting the underlying microservices from the clients.
- **Request routing:** It routes requests from clients to the appropriate microservices, based on the API path and other criteria.
- **Authentication and authorization:** API gateways often handle authentication and authorization for incoming requests, ensuring that only authorized clients can access the microservices.
- **Rate limiting and throttling:** It can enforce rate limiting and throttling policies on incoming requests to protect the application from being overwhelmed by excessive traffic.
- **API transformation and aggregation:** An API gateway can modify or transform requests and responses, as well as aggregate data from multiple microservices to provide a cohesive response to the client.

### Service mesh:

- **Internal communication:** A service mesh is primarily focused on managing, controlling, and observing communication between microservices within the distributed system, rather than external client requests.
- **Sidecar proxies:** A service mesh uses lightweight network proxies (sidecars) deployed alongside each microservice, which handle inter-service traffic, enabling features like load balancing, traffic routing, and security enforcement.
- **Resilience and fault tolerance:** It provides resiliency features like circuit breaking, retries, and timeouts to enhance the fault tolerance of the microservices communication.
- **Observability:** A service mesh offers built-in observability for metrics, logs, and tracing, which enables in-depth monitoring and troubleshooting of microservices interactions.
- **Security:** It can enforce security policies such as mutual TLS, ensuring secure and encrypted communication between microservices.

> In summary, an API gateway is focused on managing external client access to microservices, whereas a service mesh manages communication between microservices within the distributed system. Both components can complement each other, with the API gateway serving as the ingress point for external requests and the service mesh ensuring reliable and secure communication within the application.

## Key Concepts:

Service mesh architecture is designed to provide a dedicated infrastructure layer for managing, controlling, and observing communication between microservices in a distributed system. Key components and concepts of service mesh architecture include:

1. **Data Plane**
   The data plane is responsible for managing the traffic between microservices. It consists of lightweight network proxies, called sidecars, that are deployed alongside each microservice instance. Sidecars intercept and manage inter-service communication, handling tasks such as load balancing, traffic routing, and enforcing security policies.

2. **Control Plane**
   The control plane is the central management layer of the service mesh, responsible for configuring and monitoring the data plane. It provides an interface for users to define and enforce policies, configurations, and traffic rules. The control plane also collects metrics, logs, and tracing information from the sidecars to provide observability into the microservices’ communication.

3. **Sidecar Proxy**
   A sidecar proxy is a lightweight network proxy deployed alongside each microservice instance, which intercepts and manages the traffic between microservices. Sidecar proxies are typically implemented using technologies like Envoy or Linkerd. They are responsible for load balancing, traffic routing, circuit breaking, retries, timeouts, and enforcing security policies such as mutual TLS.

4. **Traffic Management**
   A service mesh enables fine-grained traffic management, allowing users to control and manipulate the flow of traffic between microservices. This includes features such as request routing based on criteria like headers, weights, or versions, load balancing algorithms, fault injection for testing purposes, and traffic shifting for canary deployments or blue-green rollouts.

5. **Observability**
   A service mesh provides built-in observability for the entire microservices ecosystem. It typically includes collecting metrics for performance and resource usage, distributed tracing for end-to-end visibility of request flows, and log aggregation for troubleshooting and analysis. This information can be consumed by monitoring and visualization tools, helping users understand the behavior and health of their microservices.

6. **Security**
   A service mesh can enhance the security of microservices communication by providing features like mutual TLS for encrypting traffic and ensuring the identity of communicating services. Additionally, it can enforce fine-grained access control policies based on various criteria such as service identity, request attributes, or metadata.

7. **Resiliency**
   The service mesh architecture helps increase the overall resiliency of microservices-based applications by providing features like circuit breaking, retries, and timeouts, which mitigate the impact of failures, delays, and network issues. These features help improve the fault tolerance of the system and ensure the availability of critical services.

> In a service mesh architecture, the data plane and control plane work together to provide a comprehensive solution for managing, controlling, and observing microservices communication, allowing developers to focus on their application’s business logic while the service mesh handles the complexities of inter-service communication.

## Advantages Of Service Mesh

1. **Advanced Traffic Management**

Enables fine-grained control over traffic routing, including canary deployments, blue-green deployments, and traffic splitting based on specific conditions.

2. **Service Discovery and Load Balancing**

Automatically discovers services and ensures optimal load balancing between instances.

3. **Observability**

Provides deep insights into system behavior through distributed tracing, metrics, and logging without modifying application code.

4. **Enhanced Security**

Facilitates mutual TLS (mTLS) encryption for service-to-service communication and enforces security policies at the network level.

5. **Fault Tolerance**

Includes built-in mechanisms such as retries, timeouts, circuit breakers, and rate limiting to improve resilience.

6. **Policy Enforcement**

Allows the implementation of custom policies for traffic control, authentication, authorization, and rate limiting.

7. **Decoupling Communication Logic**

Removes communication-related concerns (e.g., retries, encryption, and telemetry) from application code, simplifying the development process.

## Disadvantages Of Service Mesh

1. **Increased Complexity**

Adds an additional layer to the architecture, requiring expertise to configure, deploy, and maintain effectively.

2. **Performance Overhead**

The sidecar proxies in a service mesh introduce latency and consume additional CPU and memory resources.

3. **Operational Overhead**

Requires setting up and maintaining the service mesh infrastructure, which may include upgrades, troubleshooting, and monitoring.

4. **Learning Curve**

Teams must invest time to understand service mesh concepts, tools, and configuration.

5. **Cost Implications**

More resources (compute, network, and storage) are needed to run the service mesh components, potentially increasing operational costs.

6. **Risk of Overengineering**

For small-scale systems or simple use cases, implementing a service mesh might be unnecessary and lead to overengineering.

7. **Compatibility Issues**

Integrating a service mesh with existing legacy systems or specific protocols might be challenging.

## The Patterns

### Sidecar Proxy Pattern

This is the most popular and foundational pattern in service mesh architecture.

A proxy (like Envoy) runs alongside each application instance in the same pod (in Kubernetes) or host.
The proxy intercepts all inbound and outbound traffic, handling routing, observability, and security.

### Service Mesh with eBPF

This Uses [EBPF](https://ebpf.io/) (Extended Berkeley Packet Filter) in the Linux kernel to handle communication.
Avoids traditional proxies by running network-related operations directly in the kernel.

### Sidecar-Less Mesh Pattern

Removes the need for individual sidecars by embedding proxy logic into the node or network infrastructure.
The mesh logic can be integrated into the application runtime or centralized.

## Examples Of Service Mesh Software

- Likerd
- Consul
- Istio
- Cilium

