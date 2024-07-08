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

![Basic flow](/umls/images/diagram-14825854324745902788.png)

The main problem is to build a player's behavior model based on the limited information - OBP knows about the following
player's actions:

- deposit money to OGP;
- receiving money from OGP.

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

In accordance with the [research](https://medium.com/tapjoy/tapjoy-research-when-do-people-play-mobile-games-8c622c7429f), nighttime is more popular than the morning for gaming.

Mobile gamers are more than twice as likely to play games at night right before they go to bed than in the morning right when they wake up, 59% to 27%. Women are even more likely than men to play at night, with 66% of them reporting they play right before they go to sleep at night. Meanwhile, only 22% of men report playing games right when they wake up in the morning.

Millennials are 24% more likely to play games before they go to sleep at night than older players and 42% more likely to play right when they wake up in the morning as well. In fact, Millennials seem to play more consistently throughout the day, over-indexing for just about every activity. 

### 4.3. Data scrapping

| Operation                | Frequency    |
|--------------------------|--------------|
| Data scrapping per token | once per day | 

#### 4.2.4. Average size of token for data scraping

Let's assume 200 bytes.







 






