# Gambling Risk Score: the system design

The modal verbs in the document are used in accordance with
[RFC-2119](https://datatracker.ietf.org/doc/html/rfc2119).

## 1. The project purpose

This project is aimed at minimizing the risk of players conducting problem and at-risk gambling.

## 2. Definitions

This part of the document aims to provide non-casuistic definitions of the terms used in the document.

| Term                         | Definition                                                                                                                                                                                                                                                                                                                              |
|------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Bank                         | An institution that deals in money and its substitutes and provides other money-related services. In its role as a financial intermediary, a bank accepts deposits and makes loans. [Source](https://www.britannica.com/money/bank)                                                                                                     |
| Open Banking Platform (OBP)  | Open banking is a financial services model that allows third-party developers to access financial data in traditional banking systems through application programming interfaces (APIs). This model completely changes the way financial data is shared and accessed [Source](https://stripe.com/resources/more/open-banking-explained) |
| Player                       | Someone who takes part in a game or sport [Source](https://dictionary.cambridge.org/dictionary/english/player)                                                                                                                                                                                                                          |
| Online gambling              | Online gambling refers to betting or playing games of chance or skill for money, by using a remote device such as a tablet, computer, smartphone, or any mobile phone with an internet connection. [Source](https://www.britannica.com/money/bank)                                                                                      |
| Problem and at-risk gambling | roblem gambling means gambling to a degree that compromises, disrupts, or damages family, personal or recreational pursuits. [Source](https://www.gamblingcommission.gov.uk/about-us/guide/page/problem-and-at-risk-gambling)                                                                                                           |
| Gambling Risk Score (GRS)    | The indicator is based on measuring the risky behavior of people with gambling problems.                                                                                                                                                                                                                                                |

## 3. The problem statement

The following diagram shows a flow of interactions between player, bank, open banking platform and online gambling
platform:

![Basic flow](/umls/images/diagram_general_flow.png)

The main problem is to build a player's behavior model based on the limited information - OBP knows about the following
player's actions:

- deposit money to OGP (PayIn, Deposit);
- receiving money from OGP (PayOut, Withdraw).

Under the player's behavior model we understand a class of the players depends on the weak structured data.
Understanding different nature of the data and, potentially, different quality leads us to using the ensemble methods.

```
Hypothesis: K-nearest neigborns + random forest + neural network for weakly structured data
```

## 4. Preliminary requirements

### 4.1. Multiple bank account

Player can have multiple bank accounts in the different banks.

```
Question: How player activities from the different banks can be merged?
```

```
Hypothesis: Player can be identified by socialID
```

### 4.2. Player activities

#### 4.2.1. Player's bank operations

| Operation                                      | Frequency             |
|------------------------------------------------|-----------------------|
| Deposit (PayIn) transaction (Player -> OGP)    | up to 5 times per day | 
| Withdraw  (PayOut) transaction (OGP -> Player) | up to 2 per month     |

#### 4.2.2. Average bank transaction payload size

Average size of a credit card data transaction is 500
bytes  ([Source](https://www.quora.com/What-is-the-average-data-size-of-a-credit-card-data-transaction))

#### 4.2.3. Average size of player activity records

Player activity record is a record in the bank's database with all needed meta-data.
Let's assume that record has size less than 5KB.

#### 4.2.4. Timing

In accordance with
the [research](https://medium.com/tapjoy/tapjoy-research-when-do-people-play-mobile-games-8c622c7429f), nighttime is
more popular than the morning for gaming.

Mobile gamers are more than twice as likely to play games at night right before they go to bed than in the morning right
when they wake up, 59% to 27%. Women are even more likely than men to play at night, with 66% of them reporting they
play right before they go to sleep at night. Meanwhile, only 22% of men report playing games right when they wake up in
the morning.

Millennials are 24% more likely to play games before they go to sleep at night than older players and 42% more likely to
play right when they wake up in the morning as well. In fact, Millennials seem to play more consistently throughout the
day, over-indexing for just about every activity.

Thus, most operation will be made in **2-3 hours per day**.

### 4.3. Data scrapping

| Operation                | Frequency    |
|--------------------------|--------------|
| Data scrapping per token | once per day | 

#### 4.3.1. Average size of token for data scraping

Let's assume 200 bytes.

#### 4.3.2. Total players

Let's assume up to 20M players.

## 5. Back-of-the-envelop calculation

| Metric                                                 | Unit                    | Value | Comment |
|--------------------------------------------------------|-------------------------|-------|---------|
| Total players                                          | persons per months      | 20M   |         | 
| Total deposit transactions                             | transactions per month  | 3B    |         |
| Total withdraw transactions                            | transactions per months | 40M   |         | 
| Data Lake capacity (+30%)                              | PB per month            | 1.5   |         |
| Data Lake capacity (+30%)                              | PB per year             | 18    |         |
| Embedding size (25 features x 4 bytes)                 | Bytes                   | 100   |         | 
| Total embeddings                                       | record                  | 20M   |         |
| Vector DB capacity (+30%)                              | GB per year             | 240   |         | 
| Scrapping (replication) throughput (Bank -> Data Lake) | GB per second           | 4.63  |         | 
| Vector database throughput                             | MB per second           | 2.8   |         | 

## 6.Architectural design

### 6.1. Architectural concepts and approaches

#### 6.1.1. ETL vs ELT

ETL (Extract, Transform, Load) and ELT (Extract, Load, Transform) stand as the two predominant methodologies for
designing such king of the system. [Resource](https://www.datacamp.com/blog/etl-vs-elt)

##### 6.1.1.1. ETL

ETL, as the acronym suggests, consists of three primary steps:

- **Extract** - Data is gathered from different source systems.
- **Transform** - Data is then transformed into a standardized format. The transformation can include cleansing,
  aggregation, enrichment, and other processes to make the data fit for its purpose.
- **Load** - The transformed data is loaded into a target data warehouse or another repository.

##### 6.1.1.2. ELT

ELT takes a slightly different approach:

- **Extract** - Just as with ETL, data is collected from different sources.
- **Load** - Instead of transforming it immediately, raw data is directly loaded into the target system.
- **Transform** - Transformations take place within the data warehouse.

**ELT is more suitable** for the project due to:

- **Flexibility** - As raw data is loaded first, businesses can decide on the transformation logic later, offering the
  ability to adapt as requirements change.
- **Efficiency** - Capitalizing on the robust power of modern cloud warehouses, transformations are faster and more
  scalable.
- **Suitability for large datasets** - ELT is generally more efficient for large datasets as it leverages the power of
  massive parallel processing capabilities of cloud data warehouses.

#### 6.1.2. Vector persistence

In the context of this project the best choice for the persistence layer is vector databases.

#### 6.1.2.1. Replication

Replicate vectors across multiple servers or data centers to ensure data availability and redundancy.
In the context of the project, the bidirectional replication seems like the most effective.

#### 6.1.2.2. Partitioning

The primary use of partitioning is scalability. The persistence layer should be partitioned by geoID.

#### 6.1.2.3. Transactions

No needed.

#### 6.1.2.4. Integrity

Data integrity exists to ensure the data remains accurate and uncompromised throughout this process.
No special requirements

#### 6.1.2.5. PACELC

In the context of the PACELC theorem, we can assume:

- network splitting at the replica level;
- eventually consistency due to inter-cluster replication via a queue.

Thus, we need to select a PA/EC vector database.

#### 6.1.2.6. Load profile

| Operation | Cases                     | Frequency                | Unit                 | Value  |
|-----------|---------------------------|--------------------------|----------------------|--------|
| Create    | Save token for scoring    | per PayIn token          | operations per month | 3B     |
| Read      | re-evaluate score per geo | per geo location per day | operations per month | 30-120 |
| Update    | update risk score         | per PayIn token          | operation per month  | 3B     |
| Delete    | delete outdated vectors   | per PayIn token          | operations per month | 31     |

As the table shows the profile is **WRITE-HEAVY**.

The recommended database: Apache Cassandra (write-heavy, PA/EL).

#### 6.1.2.6. Vector structure

### 6.2. Basic flows

#### 6.2.1. Scrapping (extracting and loading) flows

replication
The main goal of this step is to collect raw data from integrated banks and store it in the Data Lake, enriched with
Open Banking Platform metadata.

![Scrapping flow](/umls/images/diagram_data_scrapping.png)

The comparison of the data format for Data Lake:

| Options                  | Avro  | Parquet | ORC    |
|--------------------------|-------|---------|--------|
| Schema evolution support | Best  | Good    | Better |
| Compression of files     | Good  | Better  | Best   |
| Splittability support    | Good  | Good    | Best   |
| Row/ Column based        | Row   | Column  | Column |
| Read/Write intensive     | Write | Read    | Write  |

Considering that the mechanism for converting raw data into embeddings/vectors may change over time, Parquet is a better
choice because it is designed to be read-intensive.

#### 6.2.2. Transforming flows

//TODO:

## 6.3. Monitoring, metrics and alerting

This part of the document defines the requirementS and the best practices in monitoring, metrics and alerting of User
Auth Module. This module is responsible for the typical business operation related to user registration,
authentication.

### 6.3.1. Performance monitoring

This involves tracking metrics like response times, request rates, resource utilization, and error rates to ensure
optimal performance.

The metrics:

| Metric               | Description                                                                                                                                | Unit   | Percentile         |
|----------------------|--------------------------------------------------------------------------------------------------------------------------------------------|--------|--------------------|
| Response time        | This metric measures the time taken for the app to respond to a user request. It helps in evaluating the app's speed and efficiency.       | ms     | 50%, 75%, 95%, 99% |
| Request throughput   | The number of requests the app can handle per unit of time, indicating its capacity and scalability.                                       | rps    |                    |
| Error rate           | Monitoring the percentage of unsuccessful requests or errors occurring within the app provides crucial insights into its reliability.      | %      |                    |
| Apdex score          | A metric that combines response time and error rate to evaluate user satisfaction with the app's performance.                              | rps    |                    |
| Round Trip Time      | The time taken for a request to travel from the client to the server and back.                                                             | ms     | 50%, 75%, 95%, 99% |
| Latency              | The time used by server for for a request processing.                                                                                      | ms     | 50%, 75%, 95%, 99% |
| Resource utilization | Tracking CPU, memory, disk usage, queues lengths, and other resources helps ensure efficient resource allocation and prevents bottlenecks. | %, mpq |                    |

### 6.3.2. Health and liveness checks (probes)

Regularly checking the app's status and components to detect any potential issues or failures.
The requirements:

| Requirement class                 | Description                                                                                                                   |
|-----------------------------------|-------------------------------------------------------------------------------------------------------------------------------|
| Frequency                         | Regularly schedule health checks to monitor the app's status and components at predefined intervals.                          |
| Timeouts                          | Set appropriate timeouts for health checks to prevent them from causing delays or bottlenecks in the app.                     |
| Thresholds                        | Define thresholds for health check results to determine when an app component is considered unhealthy or  unresponsive.       |
| Customization                     | Allow for customization of health checks to include specific checks for different app components or services.                 |
| Logging and reporting             | Log the results of health checks and provide detailed reports to track app performance and status over time.                  |
| Alerting                          | Configure alerts to notify administrators or stakeholders when an app component fails a health check or becomes unresponsive. |
| Integration with monitoring tools | Integrate health and liveness checks with existing monitoring tools or systems to streamline monitoring processes.            |
| Automatic recovery                | Implement automatic recovery processes based on health check results to ensure prompt resolution of issues.                   |

### 6.3.3. Alerting

Setting up alerts for specific thresholds or conditions to notify when performance degrades or errors occur.

What to alert on:

- Alerts should be actionable and relevant to users.
- Some examples of things to alert on are availability, latency, and integrity/durability.
- Alerts are the “start” of an action or an investigation; they may only represent a small portion of what you monitor.

When to alert on things:

- Alerts can be more or less urgent depending on how long the issue has been going on and how severe the impact is.
  Consider that impact changes over time.
- You can use alerts to give people enough time to act before there are consequences for issues such as quota
  consumption.

Who to alert, and how:

- Only notify people you intend to act in response, and trust them to inform more people if needed.
- Page the person on-call if the situation requires an immediate response.
- Consider not paging the person on-call and creating a ticket if the issue is not urgent.

### 6.3.4. Logging

Collecting and analyzing logs to track app behavior, diagnose issues, and troubleshoot problems.

- Know What to Log
- Know When to Use Each Log Level
- Use English Language and Friendly Log Messages
- Have a Consistent Structure Across All Logs
- Understand Metrics
- Make Each Log Message Unique
- Always Provide Context
- Reporting Alerts and Exception Handling
- Write Log Parsers and Proactively Monitor Logs

### 6.3.5. Security monitoring

Monitoring for suspicious activities, potential intrusions, and vulnerabilities to protect the app from security
threats.

Please, follow [NIST SP 800-171](https://csrc.nist.gov/pubs/sp/800/171/r2/upd1/final) recommendations in security
monitoring.

## 7. High-Level Architectural Design

![Architectural design](/pictures/player_risk_score.drawio.png)

Multi-DC solutions play a critical role in ensuring **high availability**, **fault tolerance**, and **scalability**.

| Key aspects                          | Description                                                                                                                                                                                                                                                                                                                                             |
|--------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Redundancy                           | Redundancy is achieved by replicating critical infrastructure components, such as servers, storage, networking equipment, and power sources, across multiple data centers. This redundancy ensures that if one data center fails or becomes unavailable, the workload can be seamlessly transferred to another data center without disrupting services. |
| Load Balancing                       | Load balancing distributes incoming network traffic across multiple servers or data centers to ensure optimal resource utilization, maximize throughput, minimize response time, and avoid overload on any single server or data center. It can be implemented using hardware or software-based solutions.                                              |
| Data Replication and Synchronization | Data replication involves copying data across multiple data centers in real-time or near-real-time to ensure data availability and consistency. Synchronization mechanisms are employed to keep data consistent across different data centers, often utilizing techniques like synchronous or asynchronous replication.                                 |
| Global Traffic Management            | Global Traffic Management solutions direct user requests to the nearest or most suitable data center based on factors such as geographical location, network latency, server load, or other customized criteria. This ensures optimal performance and user experience.                                                                                  |
| Disaster Recovery (DR)               | Multi-DC solutions often include disaster recovery plans and procedures to mitigate the impact of catastrophic events such as natural disasters, cyber-attacks, or hardware failures. These plans typically involve automated failover processes to switch traffic to backup data centers in case of a disaster.                                        |
| Consistency Models                   | Multi-DC architectures may implement different consistency models depending on the specific requirements of the application. These models include strong consistency, eventual consistency, and causal consistency, each offering trade-offs between data consistency, availability, and partition tolerance.                                           |











 






