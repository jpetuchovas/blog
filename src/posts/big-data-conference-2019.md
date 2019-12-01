---
title: "Notes from BIG DATA Conference 2019"
path: "/big-data-conference-2019"
date: "2019-12-01"
excerpt: "On November 26–28, 2019, I attended the BIG DATA Conference. It was my first time participating in this conference, so I cannot compare it…"
---

On November 26–28, 2019, I attended the [BIG DATA Conference](https://bigdataconference.lt/). It was my first time participating in this conference, so I cannot compare it with the previous ones, but overall it was an interesting experience.

The part of the conference I enjoyed the most was the workshop because it allowed me to dive deeper into a particular topic ([stream processing](#stream-processing)) and apply the newly learned knowledge during hands-on labs.

As for the talks, even though I am skeptical about learning technical topics by listening to other people talking about them, the speakers mentioned many new concepts, ideas, and technologies that I hadn’t had experience with, so by taking note of them, I can always learn more on my own time if something seems useful. Also, there were multiple instances during the conference when the speakers shared interesting ways of looking at things that I found particularly useful, e.g., [a step-by-step process of optimizing a data pipeline](#pipeline-optimization) or [thinking about data in terms of logs](#kafka) (a database is just a cache of a subset of a log).

One criticism I have is that the opening keynotes seemed a bit too basic for a conference focused on big data. Of course, that probably depends on the background of the listeners, so maybe that’s what the organizers wanted.

The slides of the talks should be available on the conference’s website sometime soon. In the meantime, these are my cleaned-up notes of some of the talks I took during the conference. I did not include the notes from some of the talks I did not find very insightful. If something catches your eye, I encourage you to take a look at the talks yourself since I do not have much experience with some of the topics (e.g., stream processing), so I may have misinterpreted or missed something important.

<h2 id="pipeline-optimization">“Optimize Your Data Pipeline Without Rewriting It” by Magnus Runesson</h2>

“It’s not fast enough!” may mean a lot of different things. If you don’t know how fast the old pipeline was, you won’t be able to tell whether it is fast enough. Make sure to have the relevant metrics and use them to make your stand.

The optimization process:

1. **Visualize**. When you have a problem, make a small sketch of it to understand it better. For instance, draw the relevant databases and tasks. Also, add the time perspective to understand what takes up the most time.
2. **Understand the requirements**. Sometimes requirements may be very specific, but at other times the stakeholders may not provide good requirements (e.g., “Make it as fast as possible.”). In that case, you have to reason out the requirements yourself (e.g., the data should be processed within half an hour).
3. **Break things down**. For example, figure out how many tables are used.
4. **Parallelize**. If the data is read from each table sequentially, then the pipeline can be made faster using parallelism. However, running everything simultaneously may not be feasible because of infrastructure limitations (the database may not be able to handle the high load). Nevertheless, maybe some tables can be read simultaneously.
5. **Remove idle time**. Use dependency-based scheduling (tools like Apache Airflow can help here).
6. **Find bottlenecks**. Focus on the bottleneck. Make sure you understand why it is worth optimizing a particular component (metrics help here). Check whether increasing the load on some component is feasible.
7. **Zoom in**. Most junior engineers jump into the code at first. However, you need to understand the bigger picture to know what to optimize.

You may need to repeat the steps 3 to 6 multiple times and maybe even try to understand the requirements better.

The most important step: stop when good enough. If you know your requirements, you will know when it’s good enough. If you don’t know them, you will not know when to stop.

---

Various nuggets from the Q&A session:

- Each task should be idempotent—you should be able to be run it over and over again. Tasks shouldn’t be broken down too much because then the overhead of starting a task may increase too much.
- It’s worth talking with users to understand their expectations. A product manager’s and user’s expectations need to be aligned.
- Airflow’s [`ExternalTaskSensor`](https://airflow.readthedocs.io/en/stable/_modules/airflow/sensors/external_task_sensor.html) operator can be used to integrate with other people’s pipelines.
- Important metrics:
  - CPU, memory usage, etc.
  - Within DAGs: when data is delivered; how long each task takes; how long the full DAG takes.

Slides are available on [SlideShare](https://www.slideshare.net/MagnusRunesson/optimise-your-data-pipeline-without-rewriting-it-big-data-conference-vilnius).

<h2 id="stream-processing">“Stream Processing Essentials” Workshop by Vladimir Schreiner</h2>

Stream processing is not something massively new. It is an evolution of the tools.

**Batch processing** (collect -> process -> use):

- Linear process.
- No overlap means huge latency.
- Use cases:
  - Post-mortem analysis.
  - ML training/data science.
  - Offline transaction processing.
  - ETL.
- Tools:
  - Hadoop.
  - Hive for SQL people.
  - Often custom.

The SQL approach means storing and computing in the same cluster, which is convenient.

When the data storage layer is decoupled from the processing layer, separate clusters are used for storage and for processing. Each cluster can have its optimized hardware (hard disks for storage, CPUs for processing) and can be scaled separately (e.g., add more CPUs to scale processing). Also, people who write processing code prefer languages like Java not SQL. When you decouple the processing layer from the storage layer, you can use different languages for each of those layers.

**Stream processing** (infinite data -> continuous processing -> continuous use):

- Characteristics:
  - Querying made proactive.
  - Preprocessing data before storing/using it (reduces access times when you need the results).
- Benefits:
  - Low latency.
  - Continuous programming model for infinite datasets. Real world data does not come in batches!
  - Event time, not ingestion time.
  - Batch is just a finite stream.

Stream processing and databases:

- A database allows clients to pull data by querying its state (**pull model**).
- A stream processor runs a continuous query and pushes updates to consumers (**push model**).

What stream processing brings:

- to traditional ETL (data pumps): scale.
- to batch analytics (MapReduce): continuous programming, reduces latency.
- to complex event processing: scale.

Stream processing issues:

- How frequently to push data?
- Dealing with time.

Use cases:

- Continuous ETL:
  - ETL in the 21st century:
      - Maintains the derived data (keeps it in sync).
      - Performance – adapt data to various workloads.
      - Modularization – microservices own the data.
  - Why continuous ETL?
      - Latency.
      - Global operations (no after hours).
      - Continuous resource consumption.
- Analytics and decision making:
  - Real-time dashboards.
  - Statistics (gaming, infrastructure monitoring).
  - Decision making.
  - Recommendations.
  - Prediction – often based on algorithmic prediction (push a stream through an ML model).
  - Complex event processing.
- Event-driven applications:
  - Event sourcing. Sequence of change events as a shared database (simpler than replicating every database to every service).
  - Applications publish and subscribe to the shared event log.
  - Application state is a cache of the event store.
  - Stream processor is the event handler (consumes events from the event store, updates the application state).

**Stream processing big picture**: Data sources (live streams (e.g., Apache Kafka), databases, files, application data) are read by a stream processor, which writes/sends the data to data sinks (live streams, databases, files, applications).

Stream processors come with many connectors to connect to different data sources, which are interchangeable.

Pipelines and jobs:

- Pipeline:
  - Declaration (code) that defines and links sources, transforms, and sinks.
  - Platform-specific SDK is used.
  - It does not run. A client submits a pipeline to the stream processing engine (SPE).
- Job:
  - A running instance of a pipeline in an SPE.
  - SPE executes the pipeline:
      - Code execution.
      - Data routing.
      - Flow control.
  - Parallel and distributed execution.

Imperative vs. declarative:

- Imperative (how):
  - Example: iteration.
  - The user controls the flow.
- Declarative (what):
  - Example: Java streams.
  - Code defines logic  not flow.
  - The runtime can optimize the code.

SPEs use the declarative programming model because they handle the “how”:

- Data routing and partitioning. There is no point in reading data from a file from multiple instances because they’ll compete for the same resource. If the data is partitioned, multiple instances can read multiple partitions (data-level parallelism).
- Invoking pipeline stages.
- Running your pipeline in parallel (task-level parallelism).

A stream processing engine makes an execution plan and tries to use all CPUs as well as minimize data shuffling. DAG is a representation of an execution plan.

---

The lab code is available on [GitHub](https://github.com/hazelcast/hazelcast-jet-training).

Check out [the reference card written by Vladimir](https://dzone.com/refcardz/understanding-stream-processing) to learn more about stream processing.

One problem with stream processing is that a streaming source that produces data continuously is needed. Even though you can use something like Apache Kafka, it may not make sense to introduce new technologies to an existing project. One interesting project mentioned during the workshop is [Debezium](https://debezium.io/), which, as far as I understand, makes it possible to transform a database into a streaming source. It has a set of change data capture (CDC) connectors to various databases.

<h2 id="kafka">“Everything You Wanted to Know About Apache Kafka but You Were Too Afraid to Ask!” by Ricardo Ferreira</h2>

Apache Kafka is not just a messaging technology. It is a distributed streaming platform. A stream is an unbounded set of events (it’s infinite and never stops). Infinite streams pose a lot of challenges, which current technologies (e.g., databases) were not designed to address.

Previously, there was a clear separation between the stages of gathering, storing, and processing data. However, the paradigm shift that we are observing now is that you do not need to store the data before starting to process it. How do you deal with things that never stop?

Databases:

- Are extremely limited. You need to move data to the data warehouse/data lake using batch jobs so that analysts can start analyzing it.
- Make a lot of mess. Databases used for transactions or analytics should exist. However, often data is grabbed from a transactional database, processed and put in another database, and then delivered to a database where the data is used for analytics. Such in-the-middle (incidental) databases should not exist. This is a consequence of database limitations, and that‘s why data architectures are so complicated.

Because databases are limited, workarounds were developed:

- Big data solves the problem of **volume**. Hadoop was designed to provide horizontal scalability and does a perfect job of ingesting and storing data. However, once you start processing data, it gets very complex.
- NoSQL databases were designed to handle volume. They partition data. They do not deal with the problem of processing.
- Messaging technologies are all about grabbing data from point A and delivering it to point B. However, they were designed just to deliver data.

All in all, we still have a mess that just uses a different set of technologies.

The fundamental problem: minimize the window between acquiring and effectively processing data.

ETL/data integration (what happened in the world):

- Pros:
  - Highly scalable.
  - Durable.
  - Persistent.
  - Ordered.
- Cons:
  - Batch.
  - Expensive.
  - Time consuming.

Messaging (what is happening in the world):

- Pros:
  - Fast (low latency).
- Cons:
  - Difficult to scale. When subscriber count is increased, messaging systems slow down.
  - No persistence after consumption.
  - No replay.

Distributed streaming platforms have the best of both worlds:

- Highly scalable.
- Durable.
- Persistent.
- Ordered.
- Fast (low latency).

Databases are good for storing state. When you design your system based on events (which have context and meaning) instead of states, the subscribed applications can get events.

A database is just a cache of a subset of a log. Any database underneath has a log, but someone in the past thought that it’s a very complex technology, so only tables were exposed to developers.

Apache Kafka structures data as a log. When using Kafka, think about logs and change. You won’t completely understand Kafka if you think of it as a messaging platform. A table in Kafka is ALIVE (no need to wait for a batch job to complete).

If the use case of a database is pure analytics, don’t replace it with Kafka. Transactional databases used for concurrent processing are a very good use case for Kafka.

## “More Than a Query Language: SQL in the 21st Century” by Markus Winand

Relational data model:

- Atomic types.
- Schema that is independent of the processing purposes.

Data does not change as quickly as what you want to do with it. Relational operations allow you to transform data for each particular processing purpose.

The 1999 SQL standard introduced rich types:

- Arrays.
- Nested tables.
- Composite types.

In 2003, schemaless support (XML) was introduced. In 2016, JSON support was added.

Other interesting additions introduced over the years:

- Time traveling.
- Match recognize.

The speaker mentioned many more examples of various SQL features introduced over the years, but the main idea of the talk is that SQL has evolved beyond the relational data model. Sadly, not all new SQL features are available in all databases because SQL standards just define what database creators could do, but they are not forced to include those features.

According to the speaker, self-joins are a symptom that you are stuck using old features because self-joins are slower, more complex to write, maintain, and understand compared to more recent SQL syntax.

---

To learn more about modern SQL, check out the speaker’s website <https://modern-sql.com/>.

## “Overview of Generative Adversarial Networks (GANs)” by Jakub Langr

Generative modeling is an unsupervised learning task, so it does not require labels. Generative models generate new distributions. Generation is hard because distributions are complex.

Generative adversarial networks (GANs) allow us to implicitly model complicated distributions, but that also means we don’t get a closed-form solution.

GANs are networks with 2 parts: a discriminator and a generator. They are called “adversarial” because a discriminator and a generator compete against each other (analogy: detective vs. forger). They both have their own loss functions. For the discriminator, the problem is pretty much binary classification.

The two loss functions can be thought of as opposites of each other. Using a stochastic gradient descent analogy, you could think of two players: one wants to go downhill while the other—uphill.

Generator:

- Input: A vector of random numbers.
- Output: Fake examples that strive to be as convincing as possible.
- Goal: Generate fake examples that are indistinguishable from members of the training dataset.

Discriminator:

- Input:
  - Real examples coming from the training dataset.
  - Fake examples coming from the generator.
- Output: Likelihood that the input example is real.
- Goal: Distinguish between fake examples coming from the generator and real examples coming from the training dataset.

Adversarial examples are inputs to ML models that are designed in such a way so as to make a model make a mistake. For example, adding a little bit of noise to an image can fool a computer vision algorithm into misclassifying the image.

If your application is mission critical, you need to think about adversarial attacks. However, in order for an adversarial attack to be successful, you need specialized knowledge and pretty much have access to models. So, if your application has this vector of attack, you probably have simpler vectors of attack as well.

Some examples of where GANs are used in practice:

- Generating logos.
- Matching styles of clothes.
- Domain adaptation in movies.

## “Machine Learning Engineering” by Paweł Zawistowski

ML engineering is the act of building successfully working systems that encompass predictive models.

Shipping checklist:

- Offline results meet the acceptance criteria.
- Implementation is ready and code quality is good.
- Training and serving pipelines are set up.
- Monitoring and logging are in place so that debugging is possible.
- Data dependencies are explicitly stated (e.g., a specific dictionary can only contain 5 entries).
- A/B experimentation pipeline is ready.

Consistent versioning:

- Major – different assumptions with regards to inputs and outputs.
- Minor – changing hyperparameters, some internal improvements.
- Patch - bugfixes, small tweaks.

Example: if v2.13.6 is running in production, you can substitute it with v2.*.*.

AI does not give you a competitive advantage. The thing that matters is what you build using AI.

---

Interesting papers mentioned during the talk:

- [Stop Explaining Black Box Machine Learning Models for High Stakes Decisions and Use Interpretable Models Instead](https://arxiv.org/abs/1811.10154).
- [Hidden Technical Debt in Machine Learning Systems](https://papers.nips.cc/paper/5656-hidden-technical-debt-in-machine-learning-systems).

## “Deep Learning for Lazy People… Neural Architecture Search with Automated Machine Learning” by Diego Hueltes

The time complexity of neural architectural search (NAS) is O(nt), where n is the number of neural architectures tested, and t is the average time spent evaluating every architecture. You can reduce the aforementioned time complexity by either reducing n or t.

Reducing n:

- Reducing the search space.
- Using a better search strategy:
  - Bayesian optimization.
  - Genetic programming.
  - Reinforcement learning. A neural network’s accuracy is treated as a reward.

Reducing t:

- Performance estimation strategies:
  - Lower fidelity estimates.
  - Learning curve extrapolation.
  - One-shot models/weight sharing. Train a big neural network and use its subgraphs as different neural network architectures.
  - Weight inheritance/network morphisms. Retrain a neural network by changing only a part of a network and training only for a small number of epochs.

---

[AutoKeras](https://autokeras.com/) was mentioned as a library that allows to automatically search for ANN architectures.

Slides are available on [the speaker’s blog](https://www.hueltes.com/nas).

## “Runtime Modifications of Distributed Big Data AI Models” by Elena Lazovik

The talk proposed an interesting idea of updating the data sources, parameters, and even models used in ML systems at runtime without the need to stop everything. The library that was developed to achieve this (using Apache Spark) should be open-sourced sometime in Spring 2020.
