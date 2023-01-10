# Rest_API_Scaling_Stragegy

What is scaling?

To begin with ‘Scalability’ is a somewhat vague term. A scalable system is able to grow to accommodate required growth, changes, or requirements. Scaling refers to the methods, technologies, and practices that allow an app to grow.

I think we’re now ready to discuss “Strategy to scale REST API”.

Ps: I will be citing python as a language of reference even though I’m agnostic across Go lang, Javascript and C++.

Most of the time Scalability challenges are always a result of resource crunch and if we talk in context of API then it can be CPU overload, I/O waiting & Queuing, Memory, Network resources e.t.c

As recent study by Akamai claims

A 1 second delay on your page load can cause a 7% reduction in conversations and if applications take longer than 3 seconds then approx 50% of the users will uninstall the mobile application

Scaling API is highly dependent on case to case basis as there is not a single way to achieve and improve the performance of your API. Identifying the bottleneck is one of the key things here and basis on that you would be able to take a call and tune that part to scale your overall API.

You can analyse production metrics and look for outliers to help identify what potential bottle necks there can be.

Key performance indicators can be your application/infrastructures request/sec, latency, request duration, CPU time, memory usage, heap usage, garbage collection e.t.c

But as per my experience below are the few things which work to improve the overall API performance -

1. In-Memory Cache — Databases are one of the biggest bottleneck and choking point of any application. If you hit DB for everything then not only it takes more time but puts extra burden on the DB server as well.


Here use the power of in-memory cache like Redis and hit DB only as a last resort.

Let’s see on example — if you are implementing one Quiz having 100 questions in that and millions of users will come and attempt that Quiz. In normal scenario to get the questions, your applications will be interacting with DB millions of time and this is when, when you are getting all the questions of the quiz in a single request. What if you have to take one questions at a time, may be you will be creating billions of interactions with DB to serve those millions of users. Isn’t it useless and there are chances of CPU overload and Queuing on your DB server if number of concurrent users are too high.

Can we reduce this unnecessary hits from DB? Yes we can, just store the questions in Redis and serve it from Redis instead of DB.

2. De-normalisation and Pre-aggregation — Gone the days when we were told to normalise data as much as possible and i think what make sense behind this logic was

- The storage cost was really expensive at that time and optimising the database to consume less storage make sense.

- Data were not as huge as we have now hence SQL query with multiple JOINS were not too expensive.

But now the situations is different, storage is quite cheaper and data is huge huge… so we should optimise our application to be processing intensive.

- Use denormalization wherever possible and reduce the joins which are processing intensive and can degrade the API performance.

- Pre-aggregate all of the data wherever required instead of calculation it on run time

3. Every bit matters so return only required data in API response — If your response size is large it will take more time to

- Server to transmit data

- Client to download data

but if your response size is small then not only your response time will speed up but you can save bandwidth too. What i have seen in many API that there are too much data is returned in API response which has no use on the client side.

For e.g. Python-Django Rest framework append all the related objects inside the API response and most of the time developer does not care about this so best practice is to always optimise the response and remove the extra fields which is not required further.

4. Add pagination to large API payloads — There are situations where data is really much larger for e.g. consider a page of “Customer with a long history of orders” and API returns order with detailed line items of each order and their related transactions. As the payload size is really large hence it will take time as we discussed in point 3 “Every bit matters…”.

So what should we do here to reduce the payload size, i think we should follow below approach to overcome this problem

— Introduce Pagination — Instead of returning the large API response we should introduce pagination to limit the number of records.

— Break the API into 2 parts — Instead of sending item details in first API just make another API call to get item details based on Order Id.

5. Connection Pooling — The idea behind using connection pooling is to minimise the time spent in creating and closing a Database Connection so use connection pooling to connect the database which will overall speedup the API response time.

6. Read only slaves pattern — Read/write master database and read-only slaves pattern is also “common” pattern to scale reads. However it does not scale the writes though but it can significantly improve performance of application if you have write intensive application.

7. Horizontal partitioning — It basically involves breaking up the big database into many smaller databases that share nothing and can be spread across multiple servers. Overall it will reduce the index size and DB server load.

8. Auto scaling and Load Balancer — No matter how optimised your Code and DB is, without proper resources/infrastructure it will crash under a heavy load. So deploy you API on cloud in an auto scale group and setup with appropriate LB to scale up and scale down your applications.

9. Set-up Workers — Figuring out the optimal number of (Gunicorn, if you’re using python frameworks) workers to run is an exercise in try-it-and-see-what-happens, but a good rule of thumb is to start off with (2 * number_of_cpu_cores) + 1 workers. Let’s assume that you’re running on really cheap, commodity hardware and we’ve only got 4 cores per server. That means you will start off with 9 workers per server.

10. Concurrency & Parallelism — Scaling across processors is done with multithreading. This means you can run code in parallel with threads, which are contained in a single process. You should note that codes will run in parallel only if there is more than one CPU available.

With multiple CPUs, which is one of the best options for scalability, Python offers two options for spreading your workload across multiple local CPUs: threads and processes.

Threading: A good way to run a function concurrently is with threads. If there are multiple CPUs available of course. Threads can be scheduled on multiple processing units, scheduling is usually determined by the operating system. In python, there is only one thread which is the main, by default. This thread runs your Python application by default. To start another thread, Python offers the threading module.
Processes: When work can be needs to be parallelized for a certain amount of time, it’s better to use multiprocessing and fork jobs. This spreads the workload among several CPU cores.
You use multi threaded when you have a need to have something producing and something consuming and both will be doing a good mix of I/O and non-I/O.

You use multi-process when you have at least one part of your application likely to be CPU heavy.

For example — I have a multi-threaded application where each thread is doing some I/O and lots of waiting — and some of those threads spawn off worker processes to do CPU heavy work.

The other advantage of multi-processing is if you want to take advantage of a multi-CPU then you need multi-processing.

Although, multiprocessing and multithreading both adds performance to the system. Multiprocessing is adding more number of or CPUs/processors to the system which increases the computing speed of the system. Multithreading is allowing a process to create more threads which increase the responsiveness of the system.