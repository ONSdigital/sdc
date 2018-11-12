# Action Service Performance Degradation

## Introduction

It was noted that there had been a considerable degradation in the performance of the Action service since a major piece of refactoring had occurred. 350,000 actions are presently taking circa 14 hours to process, when before parts of the process had taken a tiny fraction of the time.

## Creation of Actions

The first stage in the action processing is the creation of the actions themselves, based on the cases and the action rules in place at the time of execution.

Originally the actions had been created by a stored procedure called `action.createactions`, which was called via a JPA repository using the `@Procedure` JPA annotation on the function `ActionCaseRepository.createActions()`.

Then, the actions were created by 50 lines of native SQL hard-coded into the JPA repository using the `nativeQuery = true` option.

Finally, the actions are now created by code which:
 1. Retrieves all the cases and all the action rules
 2. Checks every rule against every case (cartesian join product)
 3. Checks to see if an action _has already been created_ - database roundtrip.
 4. Retrieves the ActionType entity from the DB - database roundtrip.
 5. Writes individual Action entities to the DB using `saveAndFlush` - database roundtrip.
 
In total, for every action that is created, there will be 3 roundtrips.


### Benchmarking

Inserting 350,000 new rows into the `action.action` table using Spring Data JPA and Hibernate with a PostgreSQL database was benchmarked on a Macbook Pro with i7 processor and 16Gb of RAM as taking 9 hours - 11 writes per second.

The performance of Hibernate inserting rows into Postgres was surprisingly slow, but the design of the entire process has numerous performance problems, such as roundtripping to check that actions don't already exist, and roundtripping to grab a JPA entity which could easily be cached.

### Conclusion

The stored procedure or native SQL in the code was able to perform the same function as the current code in a few seconds, as opposed to many hours. Although we might somehow be able to distribute the cases across multiple threads/processes, it is the author's opinion - and that of SDC tech leads and other experienced colleagues - that it's preferable to re-instate the stored procedure instead of spending a substantial amount of time and effort to refactor this part of the process.

## Distribution of Actions

Once the actions have been created, they are sent via Rabbit queue to the Action Exporter service or the Notify Gateway, depending on whether a physical letter or an email is required.

Originally, a Redis distributed list had been used, which stored a list of IDs of actions which were being processed. Multiple instances of the Action service were able to grab a block of Actions from the database, which excluded rows being worked on by other concurrent processes.

Then, the use of a PostgreSQL native feature `FOR UPDATE SKIP LOCKED` was used to turn the database into a kind of queueing mechanism. Batches of records were retrieved by each of the Action service instances, thus distributing the load across processes.

Finally, the use of any kind of parallel/concurrent processing was abandoned, and a single process holds a global lock, preventing the undesirable situation where two processes distribute the same action, but losing all horizontal scalability.

The current code has the following steps:
 1. Retrieve all the actions which have not been distributed
 2. Retrieve each action in a different transaction context - database roundtrip
 3. Retrieve all the parties for the case - REST API -> database roundtrip (party service)
 4. Send Rabbit message
 
The process of action distribution is by no means the slowest part of the process, and it is easily fast enough for 350,000 actions, but at full scale - i.e. 25 million actions - it will be unscalable and insufficiently performant.

### Benchmarking

The slowest part of the process is roundtripping to the Party service to get the respondent details. The marshalling of the JSON message, serialising the HTTP request, deserialising the HTTP request, unmarshalling the JSON, making the DB query, constructing the response, marshalling the response, serialising the HTTP response, deserialising the response, unmarshalling the JSON and constructing the POJO representation is expensive. The design of the Party service is such that the quickest possible roundtrip expected, with all processes running on instances connected by the same switch, is still likely to be circa 100ms. For 350,000 roundtrips, which equates to 9.7 hours.

Putting the party details into Redis allowed queries to be completed in a roundtrip of circa 10ms, which would reduce the amount of time for a single-threaded, single-process instance to distribute the actions to the Rabbit queue in around 1 hour.

### Conclusion

The Action distribution process needs to use a mechanism of distributing the processing across threads/processes, in order to achieve scalability. Performance is currently bottlenecked by the roundtrips to the Party service, which queries the database for every request.

Substantial performance improvements could be achieved by caching the party details in memory, resulting in a very substantial performance increase. In combination with multiple instances/processes/threads working concurrently, it seems feasible that this could be scaled to process 25 million actions within a reasonable timeframe.

In the author's opinion - and that of SDC tech leads and other experienced colleagues - the use of the `FOR UPDATE SKIP LOCKED` feature of PostgreSQL could be easily re-instated, allowing multiple processes to work concurrently, although consideration should be paid to the manual steps required to unlock rows in the event of a process crash, unexpected termination etc. where the lock would be held indefinitely and not released.

## Final Conclusions

The use of stored procedures, native database features and use of a database as a queue are all - in the author's opinion - highly undesirable. However, in the interests of maximum performance gain for minumum development effort, the cost:benefit ratio does not justify the greater effort spent on re-architecting a substantial part of the system, in order to achieve cleaner and more elegant code, and move us away from the antiquated and outdated concept of batch processing.

Ideally, the SDC system will move towards an event-driven model, making widespread use of queues, making any cron/scheduled/timer jobs redundant. However, the conclusion of this spike is that we should re-instate both the stored procedure and the `FOR UPDATE SKIP LOCKED` code, to quickly resolve the performance degradation which was relatively recently introduced.