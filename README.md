# Assignment

The task is to build a Calculator Manager service.
The service should have three endpoints as described in the [openapi spec](spec.yml) in this project.

The service allows a client to execute calculations.
Your service will not need to implement the calculation itself, for this a calculator service is provided via a docker image.
Your service will use the calculator and will aggregate results per day.

Please host your completed example at a publically available git URL -- you can use a public GitHub repository for this.

## Calculator Service
This service is provided to you via a docker image. It has only one endpoint which is described [here](calculator-service.yml).

The docker image is pubically available on Docker Hub as `gillius/buildit-calculator-service`. The service runs on port 8080 internally, but it should be exposed on the host as port 8888 to be consistent with the graded environment (`-p 8888:8080`). If you are running the service correctly, the following links should work:

- UI: http://localhost:8888/swagger-ui.html
- OpenAPI spec: http://localhost:8888/v3/api-docs

### the calculate endpoint

This `calculate` endpoint receives as input for the calculation an object with two properties:
- `category` which indicates how the calculator needs to process
- `value` the start value for the calculation

The services then calculates an `index` for the given input which is the value we are interested in.

Here a sample request:
```json
{
    "category" : "cat-77",
    "value" : 40478
}
```

And a sample response:
```json
{
  "category": "cat-77",
  "duration": 1065,
  "index": 860214
}
```
The `index` is the calculated result we are interested in.

### Remarks
- An individual calculation can take just few milli-seconds or up to several seconds
- the service can process multiple concurrent requests
- the calculation is not necessarily deterministic, the same input can lead to different results at different times
- it is not important for this assignment to understand the internal logic of the calculator, it just returns values
- for this assignment you can for now assume that the service is reliable and does not throw any exceptions


## Daily Calculator Service
This service needs to be implemented by you. It will use the above Calculator Service but adds the concept of a daily context to it:
Calculation requests are always executed against a given date.
At end of day a `close` request can be invoked which returns the aggregated totals for the given day.
A closed context should not accept further calculation requests.

The process is as follows:

### 1) context is implicitly created

A context is created implicitly by calling the endpoint `calculate` with a date and a calculation input.

### 2) calculations are executed
Then the clients can trigger many calculations concurrently against this context (again via the same endpoint `calculate``).
Each of the calculation requests will need to be forwarded to the Calculation Service but without blocking the calling client.
 
### 3) context is closed
Finally the context will be closed by a client.
This endpoint should return:
- the number of calculations executed against this context
- the sum of all calculations against this context

The returned values here needs to take all calculations into account which where triggered before the `close` endpoint was called:
any still running calculations must complete before the close result can be delivered.

### the endpoints

All date formats are `yyyy-MM-dd`

#### calculate endpoint

Sample input for a `POST`:
```json
{
    "category" : "cat-77",
    "value" : 105
}
```
There is no response body as the service should return right away.


#### close endpoint

The `close` endpoint returns the daily totals for the given context:

```json
{
    "sum": 101313,
    "count": 417
}
```

with
- `sum` the sum of all calculations of the given context
- `count` the number of calls against the given context


#### delete endpoint
A context can be deleted and even recreated if needed.
On recreation, it should start from scratch i.e., any previous values should be discarded.

### Further service requirements

- A closed context cannot be closed again
- A closed context should not accept further calculations
- once a context was deleted it can be recreated

See also the spec for the expected http status codes, payloads and other requirements.

## Assignment requirements
You should provide a readme including your design decisions and instructions how to start the service.
For Java-based projects, you can assume the evaluator has JDK installed. JDK 11 is preferred but latest JDK is allowed. Maven wrapper (`mvnw`) or Gradle wrapper (`gradlew`) are the allowed build systems for Java-based projects.
The implementation should follow good practices.


## Files summary

- [calculator-service.yml](calculator-service.yml) openapi spec of the Calculator Service implemented already, accessible via a docker image
- [spec.yml](spec.yml) openapi spec of the service to be built by you
