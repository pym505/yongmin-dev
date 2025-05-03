---
layout: post
title: "Distributed Crawling System"
subtitle: "Scalable Web Crawling Architecture with Docker and Selenium"
date: 2025-04-20
categories: design architecture crawling
---

## Project Overview

This project was carried out at a Japanese company to improve the crawling system for a commercial EC site. The original implementation used a single server to collect product information over HTTP, but after a certain number of requests, the IP was blocked, making further collection impossible.

The goal of this project was to solve the IP blocking issue and design a crawling system that is both scalable and reliable for continuous operation.

---

## Background and Requirements

**Context Summary**

1. Internal product data needed to be crawled regularly, but the process started failing intermittently due to IP blocking.
2. The system was built on a Flask-based MSA architecture that made API calls, but DB access was restricted due to firewall settings.
3. Approximately 10,000–30,000 product items needed to be crawled daily, ideally within 4 hours.

**Requirements**

* Identify and resolve the root cause of the crawl failure.
* Automate and stabilize the process using a realistic, maintainable solution.

---

## Investigation Process

1. Tested different request intervals to identify the IP blocking conditions:

   * 0s interval: blocked after \~40 requests
   * 5s interval: blocked after \~60 requests
   * 10s+ interval: over 200 requests with no block
   * Once blocked, the IP remained restricted for 30+ minutes

2. Conclusion: Crawling is only feasible with either 10s+ intervals or multiple IPs. However, using time intervals alone would make full crawling infeasible within a day.

3. Tried Tor browser to test with multiple IPs, but the target site only allowed domestic IPs, making this approach unviable.

---

## Solution Design: IP Source Selection

1. **Tor Browser + Distributed Instances**

   * Eliminated due to high switching latency and lack of control

2. **Domestic Paid IP Purchase**

   * Overseas providers were hard to contract with, and domestic IP services were expensive or hardware-dependent

3. **Decision: AWS-Based Infrastructure**

   * Decided to use AWS EC2 instances with distributed IPs for reliability and control

---

## Solution Design: Instance Type Evaluation

| Type      | Pros                           | Cons                                |
| --------- | ------------------------------ | ----------------------------------- |
| Lambda    | Lowest per-call cost           | No fixed IP, complex to orchestrate |
| Lightsail | UI and deployment independence | Highest cost                        |
| Spot      | Cheapest option                | Unstable, may be terminated anytime |
| On-Demand | Stable, fixed IP               | Moderate cost, but manageable       |

**Final Choice**: AWS On-Demand instances for their stability, IP consistency, and manageable cost.

---

## Final Summary

The system was implemented as a distributed crawler architecture using AWS On-Demand instances. It successfully mitigated the IP blocking issue and enabled stable, large-scale product crawling.

The structure also leaves room for future enhancements, such as autoscaling, scheduling, and distributed task monitoring. Basic scaling logic was implemented based on the product volume: the number of EC2 instances provisioned is dynamically determined to ensure that crawling can complete within the 4-hour operational window.

---

## System Components

1. **Main Server** – Middleware for reading product data from the DB
2. **Crawling API** – Performs actual product crawling
3. **AWS Instances** – Hosts the crawling API; results are uploaded to S3
4. **Load Balancer** – Distributes product numbers to instances using round-robin
5. **S3** – Intermediate storage for crawl results
6. **DB** – Final destination for crawled product data

---

## Process Flow

1. Main server reads DB to determine number of products to crawl
2. Based on product count, the server dynamically creates EC2 instances via boto3
3. Server attaches the instances to a load balancer
4. Crawling API is deployed to each instance
5. Load balancer assigns product numbers to each instance
6. Each instance stores its assigned product numbers locally
7. Instances crawl the products sequentially with configurable delays (up to 4 hours max)
8. Results are saved to S3 upon completion
9. Each instance reports completion back to the main server
10. Main server tracks completed instances and terminates them
11. Once all are complete, the main server aggregates results from S3
12. Aggregated data is written to the DB

---

## Architecture Design

The system is composed of a single main server coordinating multiple AWS EC2 On-Demand instances. The architecture is as follows:

* The main server retrieves the list of products to crawl and dynamically provisions EC2 instances based on product volume
* Each instance is equipped with a crawling API and connected to a load balancer
* Product numbers are evenly distributed via round-robin to each instance
* Each instance performs crawling with rate-limiting and uploads results to S3
* Once completed, the main server gathers all results from S3 and writes them to the DB

This architecture provides an effective solution to IP bans, enables parallel execution, and is scalable while remaining simple to maintain. Basic autoscaling is already implemented based on estimated product volume.

---

## Development Status (as of May 2025)

The system design has been completed, and I delegated the implementation task to a team member at the Japanese company as part of an internal learning opportunity.  
However, due to that member being occupied with higher-priority assignments, development has not progressed beyond the planning stage.  
The project remains on hold until the assigned developer becomes available.

It is likely that the implementation will begin around June, or I may take over the development myself if the team member becomes overloaded or steps away from the task.

---

## Retrospective and Improvements

This project was carried out at a Japanese company and involved the following practical constraints:

1. **Access Control**: As a new employee and a foreign national, I had limited permissions on the infrastructure. It took time to gain sufficient access.
2. **Procurement Restrictions**: Though using overseas IP providers might have been more efficient, company policy mandated domestic IP addresses, restricting implementation options.
3. **Legacy Tech Stack**: The company operated on a legacy stack. Docker was not used; deployments were handled via simple Git pull. Instead of Terraform, infrastructure automation was done with boto3. In this context, keeping the architecture simple and maintainable took priority over using modern tools.


