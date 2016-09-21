[🏠](README.md) › Service Architecture

# Service Architecture

Our system is based on a microservice architecture.  

**What is a microservice?**  
A microservice is a single business unit that can be developed & deployed in isolation.  

> "services are built around business capabilities and independently deployable"  
<cite>_[Martin Fowler](http://martinfowler.com/articles/microservices.html)_</cite>

Our microservices are seperated into different types, including `service` and `edge`.

### edge services
- Edge services are an API gateway, they implement the aggregation strategy & handle cross crossing concerns.  
- They compose primitive services into a useful api for more robust consumption (aggregation strategy).

<img src="https://cdn.auth0.com/blog/microservices/aggregation.png" alt="aggregation strategy" width="50%" />

> "Requests also support the aggregation strategy for splitting requests among several microservices: a single public endpoint may aggregate data from many different internal endpoints (microservices). All returned data is in JSON format."  
<cite>_[Sebastián Peyrott (Auth0)](https://auth0.com/blog/an-introduction-to-microservices-part-2-API-gateway/)_</cite>

- Concerns shared by many microservices are abstracted to an API gateway. In an [AuthO Blog Post](), they use an authentication service using jwt tokens with a `/login` endpoint as an example.  

> "Microservices are developed almost in isolation. Cross-cutting concerns are dealt with by upper layers in the software stack. The API gateway is one of those layers."  
<cite>_[Sebastián Peyrott (Auth0)](https://auth0.com/blog/an-introduction-to-microservices-part-2-API-gateway/)_</cite>

### services service

- A service is a http interface for a database or some kind of authoritative source of data.  
- They encapsulate the db schema known as the gatekeeper pattern [Tammer Saleh](https://youtu.be/I56HzTKvZKc?t=6m15s) allowing services to depend on same database without being tightly coupled (the API is the contract not the schema). One service can safely alter the schema if it doesn't alter existing API calls.  

**additional reading**  
1. [Microservices](http://martinfowler.com/articles/microservices.html)
2. [API Gateway. An Introduction to Microservices, Part 2](https://auth0.com/blog/an-introduction-to-microservices-part-2-API-gateway/)
3. [Microservices Anti-Patterns](https://youtu.be/I56HzTKvZKc?t=6m15s)
