node-red-contrib-semaphoreservice
=====================

This is a node that creates a semaphore service to act as a distributed lock/synchronization mechanism.

## Install

Run the following command in your Node-RED user directory - typically `~/.node-red`

        npm install node-red-contrib-semaphoreservice

## Information


This is a subflow to create a semaphore service to act as a distributed lock/synchronization mechanism. It uses flow variables in order to store the state of a semaphore. Changes in the semaphore value are  uninterruptable at the level of the node.js process. This is due to the fact that the latter executes them in the single threaded eventloop and does not interrupt their execution unless an async method or worker thread is used in the function's implementation.

The subflow defines 5 endpoints:
 * `POST /semaphore` for new semaphore creation. The body should contain name and initialization value of the created semaphore: `{"name":"<semname>","value":1}`. The value needs to be a positive integer. If not positive, a `400 HTTP error code` is returned. A positive float will be converted to integer. If the semaphore already exists, a `303 HTTP error code`  is returned. 
 * `DELETE /semaphore` needs to have only the name attribute in the body (`{"name":"<semname>"}`). If the semaphore does not exist a `404 HTTP error code`  is returned
 * `GET /semaphore/value/:name` for retrieving the current value. If the semaphore does not exist a `404 HTTP error code`  is returned
 * `POST /semaphore/up` for increasing the value by 1 with a body of the semaphore name `{"name":"<semname>"}` . If the semaphore does not exist a `404 HTTP error code`  is returned.
 * `POST /semaphore/down` for decreasing the value by 1 with a body of the semaphore name `{"name":"<semname>"}`. If the semaphore is already at 0,a relevant message `Semaphore locked` with a `409 HTTP error code`  is returned. If the semaphore does not exist a `404 HTTP error code`  is returned.

A created semaphore can be used as a lock (if initialized at 1). As in any semaphore related library, it is the responsibility of the clients of the distributed application to correctly use a call sequence that will indicate if the client can proceed or not to what is considered the critical section or to correctly use the up/down methods. 

For example, a semaphore locked by one client (with a down at 1, resulting to the value being 0) can be unlocked by another client that performs afterwards an up from 0 at the same semaphore. There is no notion of lock ownership by a specific client that performed the initial down. Compared to the typical semaphore libraries, this implementation does not have the ability to make the calling process sleep, if the semaphore is locked. 

 A created semaphore can also be used as a synchronization counter (any initialization value>0 can be used) in a type of producer/consumer problem. However the lock gets activated at 0, like commonly in semaphores, thus a reverse semantics semaphore needs to be used. For example defining the max available slots and then reducing by one for each producer client.  

The GET method is included only for informative reasons. There is no guarantee that the value might not change by the time the response is received by the client.

Notifying the calling processes when a semaphore gets unlocked is not implemented at the moment (e.g. through a callback URL) but it is a feature that can be added in the future if needed. Add a github comment if you think it would be useful.  
 


