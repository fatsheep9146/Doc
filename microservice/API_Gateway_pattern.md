API Gateway Pattern.

API Gateway is a common pattern used in microservice design. When your business are partitioned into serveral different microservice, you can not just expose each microservice's endpoint directly to client to call. Because there may be several disadvantages in this case:

1. The endpoint of each service may change, how to inform this information to client?
2. The partition of the microservices may be changed later, if they are directly exposed to client, then client must handles this change on its own.

So to satisfy this demand, the API Gateway is developed, it is the only entry for different clients. This brings many adventages[1]:

1. Insulates the clients from how the application is partitioned into microservices
2. Insulates the clients from the problem of determining the locations of service instances
3. Provides the optimal API for each client
4. Reduces the number of requests/roundtrips. For example, the API gateway enables clients to retrieve data from multiple services with a single round-trip. Fewer requests also means less overhead and improves the user experience. An API gateway is essential for mobile applications.
5. Simplifies the client by moving logic for calling multiple services from the client to API gateway
6. Translates from a “standard” public web-friendly API protocol to whatever protocols are used internally 

## API Gateway Open Source

kong: https://github.com/Mashape/kong/   Notes: Written in Lua, recommended by nginx.
tyk:  https://github.com/TykTechnologies/tyk   Notes: Written in Go.



## Reference

1. http://microservices.io/patterns/apigateway.html
