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
