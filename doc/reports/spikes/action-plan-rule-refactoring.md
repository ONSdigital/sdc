# Action Service Refactoring Spike Report
### Proposed ways to increase the performance and scalability of the Action Service

## Introduction
The Action Service was heavily refactored as part of the work to remove the BI cases from the SDC system. The refactoring had known performance flaws, which were highly undesirable, but because of the complexity of removing the BI case logic, the performance and scalability issues were supposed to be addressed soon after removal of BI cases had gone live.

The current, flawed, logic *continuously* attempts to create 'actions' based on *all* the cases associated with an Action Plan. Because the number of cases in the Action Service constantly grows, along with the number of Action Plans, the amount of work that the Action Service is doing keeps growing, as the number of collection exercises and surveys grows.

Because the current logic attempts to apply all Action Plans to all Cases, this creates the cartesian join product, which is to say that working with a large number of cases will create a very great deal of work for the Action Service to do, and place a heavy demand on the database to supply all the cases, repeatedly.

There is a strange concept of a 24-hour window, where an Action Rule is 'triggering' but it will not actually create any 'actions' until the Cases have reached their Action Plan start date.

The net result is that as soon as a collection exercise is set up, a sample loaded, and the collection exercise set to "Ready for Live" then a copy of all the cases will exist in the Action Service database, along with an Action Plan and Action Rule(s). Usually samples are loaded approximately 1 week before the MPS date. This means that the Action Service will be performing a great deal of unnecessary and very resource-intensive processing for a week.

Further, when the 24-hour window where the Action Rule is 'triggering' then even more unnecessary and resource-intensive processing begins taking place. The 'actions' cannot be created until the Case `actionplanstartdate` is reached. So, the Action service will be attempting to apply Action Plans to Cases, but finding it is unable to do so, because we're not 'ready' to create the 'actions' yet.

There has been a bug introduced, where changes to the collection exercise event dates (e.g. MPS, Reminder 1, Reminder 2) will not correctly be reflected in the Case's `actionplanstartdate`. Therefore, if an MPS date is chosen incorrectly, the sample loaded, and then the date is changed, the 'actions' still will not be created until the _original_ MPS date is reached.

## Overall Flaws With Current Solution
Retrieving large amounts of data from the database and then performing complex in-memory calculations, repeatedly, is a highly undesirable anti-pattern.

Repeatedly doing work before it is needed is a waste of resources, putting undue strain onto infrastructure like our Postgres database, which wastes capacity and causes us to require more CPU horsepower than we really need.

The design of repeatedly checking to see if each case's 'start date' has been reached, for a period of 24-hours is not good software engineering. While the author of this spike can understand that it _works_ it certainly is not elegant, and it causes a significant amount of resource demand during the arbitrary 24-hour window.

Further, 'actions' are transient objects, which should never exist as database entities - the Action table is a crude queueing mechanism, lacking the performance and scalability which we could have if we used an _actual_ message queue, allowing us to fan out the processing of the actions to many queue consumer instances.

There is an `ActionPlanJob` table which is completely superfluous. This table has approximately 8 million rows in it, none of which are any use at all. This table will continue to grow at a linear rate, using up disk space and providing no value. It's a confusing and pointless part of the Action Service.

## Simple and Obvious Refactoring
The idea of the 24-hour 'triggering' period of Action Rules should be refactored, so that the `actionrule` table has a `hastriggered` boolean column, to know that an Action Rule has completed its execution and all the 'actions' are ready to be 'distributed' (i.e. sent to the ActionExporter or NotifyGateway, depending on whether it's a notification letter or a reminder email that needs to be sent).

This simple change should ensure that the resource-intensive operation of applying an Action Rule to a large number of cases, is only done once, and it is only done as close as possible to the date and time when the Action Plan 'triggers'.

By marking Action Rules as having 'triggered' we can then disregard them - allowing us to ignore the Action Rules which have been processed.

## Refactoring for Scalability
Currently, an exclusive lock is held on an Action Plan, so that only one single instance of the Action Service can be processing the Action Plan/Rule and creating the 'actions' from the Cases, at any one time. This is not a scalable solution.

It would be a better design, to enqueue every single Case that is associated with an Action Plan, when the Action Rule 'triggers'.

Then, when we have a queue of Cases which are candidates potentially requiring an 'action' we can use a number of queue consumers to concurrently check to see whether we should create an 'action' or not. Those 'actions' should themselves be enqueued, ready for the process we call 'distribution'.

Using queues in this way, would allow multiple threads/processes/containers to concurrently consume messages of the queues, giving much greater scalability and performance.

Using queues would allow us to deprecate the `action` table, because that data is only transient, and maintaining an ever-growing amount of transient data is an unnecessary waste of disk space, plus it will cause performance to gradually degrade as the indexes get bigger. There are currently 650,000 rows in the `action` table, which serve no purpose.

## Refactoring for Performance
Currently, each case is processed sequentially, when order of execution is not not important and there is no reason why cases can't be processed concurrently.

If we didn't do the queue refactoring, we could use multithreading to allow multiple cases to be processed concurrently, within an instance of the Action Service.

If we used the queues as suggested for scalability, we could also use multiple consuming threads, in order to further increase performance.

## Conclusion
Presently, the processing of an ASHE-size 110,000 sample will take 19 hours. At least 3 or 4 hours of the processing time is the creation of the 'actions'. Spike prototypes have shown that it would be possible to create as many as 25 million queue messages in a few minutes.

An experimental branch using multithreading, but not using queues, was able to create 110,000 'actions' in 5 to 10 minutes, which is a vast reduction in elapsed time and would be easily fast enough for samples of 350,000, but will not scale up to 25 million.

The use of Rabbit queues is the most elegant and scalable solution, but it might be considered 'over-engineered' for SDC's planned capacity needs, once we have optimised the Action Service to not perform unnecessary processing.

There may be some operational changes, in how ops support currently 'see' what the Action Service is doing - we might change things, such that the process of checking certain database tables and columns is no longer viable, because we will be deprecating parts of the database that store unnecessary transient data.

Also, the mechanism to manually re-trigger an action, or force it to run earlier etc. by hacking at database entries, would be slightly different.

Overall, the reliability of the solution should be the same or better, but the performance will be drastically improved. If we implement the queueing, we can also horizontally scale up to perhaps 25 million actions, by using multiple instances of the Action Service.