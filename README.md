# Assignment

The task is to build a Calculator Manager service.
The service should have three endpoints as described in the [openapi spec](spec.yml) in this project.

The service allows a client to execute calculations.
Your service will not need to implement the calculation itself, for this a calculator service is provided via a docker image.


## Calculator Service
This service is provided to you via a docker image. It has only one endpoint which is described [here](calculator-service.yml).
This `calculate` endpoint receives as input for the calculation an object with two properties:
- `category` which indicates how the calculator needs to process
- `value` the start value for the calculation

Remarks
- An individual calculation can take just few milli-seconds or several minutes for other inputs
- the service is thread safe
- the calculation is not necessarily deterministic, the same input can lead to different results at different times
- it is not important for this assignment to understand the internal logic of the calculator, it just returns values
- for this assignment you can for now assume that the service is reliable and does not throw any exceptions


## Calculator Manager Service
This service needs to be implemented by you.
It introduces the concept of a `Context` against which calculations are executed.
The process is as follows:

### 1) context is implicitly created

a context is created implicitly by calling the endpoint `calculate`.

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

### 4) optional deletion
A context can be deleted

### Further service requirements

- A closed context cannot be closed again
- A closed context should not accept further calculations
- once a context was deleted it can be recreated

See also the spec for the expected http status codes, payloads and other requirements.


## Implementation requirements
todo


## Files summary

- [calculator-service.yml](calculator-service.yml) openapi spec of the Calculator Service implemented already, accessible via a docker image
- [spec.yml](spec.yml) openapi spec of the service to be built by you