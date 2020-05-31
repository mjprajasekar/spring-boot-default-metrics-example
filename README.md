This repository demonstrates the most important default Spring Boot metrics which are enabled through the Spring Boot
Actuator module:

* Spring MVC metrics
* Outbound request metrics for `RestTemplate` and `WebClient`
* JVM metrics

For a full explanation see the accompanying *Spring Boot default metrics* article at [tomgregory.com](https://tomgregory.com/spring-boot-default-metrics)

## Prerequisites

To run this project you'll need:

* JDK 11+
* Docker

## Running

Build and run the project using Docker compose with the following command:

```
./gradlew assemble docker dockerComposeUp
```

This will startup:

* **Spring Boot application** at localhost:8080 exposing several APIs
  * [API 1](http://localhost:8080/api1) is a simple GET API which adds a small random latency
  * [API 2](http://localhost:8080/api2) is a simple GET API which adds a larger random latency
  * [API 3](http://localhost:8080/api3) is a GET API that makes a request to another application using `RestTemplate` (see below)
  * [Prometheus metrics](http://localhost:8080/actuator/prometheus) for the Prometheus service to scrape (see below)
* **nginx application** at [localhost:8081](http://localhost:8081) to act as the 2nd application to be called using `RestTemplate`.
It always returns a 200 response.
* **Prometheus service** at [localhost:9090](http://localhost:9090) which is preconfigured to scrape from the Spring Boot application

## Generating metric data

To avoid having to hit the APIs by hand in order to demonstrate the Prometheus metrics, a [Gatling](https://gatling.io) script is included
which will hit all the different APIs multiple times over 30 seconds. See *[BasicSimulation.scala](src/gatling/simulations/BasicSimulation.scala)* for details of how this is done.

Just run:

`./gradlew gatlingRun`

## Some sample Prometheus queries

Try out these simple queries on [Prometheus](http://localhost:9090):

#### Inbound request per second rate
`rate(http_server_requests_seconds_count[1m])`
#### Inbound request average duration 
`rate( http_server_requests_seconds_sum[1m]) / rate(http_server_requests_seconds_count[1m])`
#### Outbound request average duration 
`rate(http_client_requests_seconds_sum[1m]) / rate(http_client_requests_seconds_count[1m])`
#### JVM used heap memory
`sum(jvm_memory_used_bytes{area="heap"})`

See more examples in the *Spring Boot default metrics* article at [tomgregory.com](https://tomgregory.com/spring-boot-default-metrics)
## Stopping

To stop all the applications in this project, just run:

```
./gradlew dockerComposeDown
```