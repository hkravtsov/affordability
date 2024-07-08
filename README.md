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

```
Hypothesis: Scheduled data scrapper + ETL
```

```
Hypothesis: Data Lake for weakly structured data
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

## 5. The back-of-envelop calculation

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

##### 6.1.1.2. Comparison of ETL and ELT

| Approach  | ETL | ELT |
|-----------|-----|-----|
| Pros      |     |     |
| Cons      |     |     |

### 6.2. Basic flows

#### 6.2.1. Scrapping (replication) flows

![Scrapping flow](/umls/images/diagram_data_scrapping.png)

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

Please, follow [NIST SP 800-171](https://csrc.nist.gov/pubs/sp/800/171/r2/upd1/final) recommendations in security monitoring.









 






