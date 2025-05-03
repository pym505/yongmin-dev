---
layout: post
title: "Coin Bot Architecture"
subtitle: "Scalable Trading System with Django, Celery, and Redis"
date: 2025-04-20
categories: design architecture trading
---

# Coin Bot Architecture

## 1. Overview

As part of an internal side project, the company president had long expressed interest in building an automated cryptocurrency trading bot. However, the idea remained vague without concrete planning.

In the past, I had personal experience writing simple automated trading scripts, but never structured one within a full-fledged framework.

Given this background, I was asked to temporarily pause my infrastructure-related duties and take full ownership of the bot system’s planning and development. The objective was not to build a simple script, but to design a framework-based strategy automation system.

---
## 2. Requirements and Proposed Expansion

### Initial Requirements
- Implement arbitrage trading between exchanges
- Start planning entirely from scratch

Although the requirement was simple, it lacked concrete standards or strategy. Based on my previous experience with trading scripts, I determined a platform-based design was essential.

### Proposed Direction
- Build on a framework + DB integration rather than a simple script
- Support for long/short/leverage strategies in addition to arbitrage
- Structure to accumulate and analyze all trading data
- Enable flexible strategy definition and switching via DB

---
## 3. Tech Stack and Framework Selection

Because database integration and modeling were core aspects, I chose the Django framework, which I’m most familiar with.

Django’s ORM and serializers can be heavy, which may not be ideal for real-time systems requiring quick responses. However, I prioritized the following advantages:

- Powerful and flexible data modeling via ORM
- Easy setup and management through Django Admin
- High maintainability and extendability for strategy testing

---
## 4. Data Models

- Coin: Logs real-time price data for each coin
- Indicator: Stores market indicators, sentiment, long/short ratios, etc.
- Strategy: Defines trade entry/exit conditions and links to script logic
- Cron: Manages periodic tasks (DB-driven scheduling)
- Log: Common logging (to file and DB)
- User: User structure for future platform expansion
- Position: Tracks current holdings and links them to strategies
- System: Stores system-wide configuration
- Script: Manages metadata for script files

---
## 5. Infrastructure

- AWS RDS: Centralized database
- Redis (Local): Celery worker queue
- AWS EC2: Main bot execution server
- VPN Server: Handles exchange IP restrictions and enables distributed operation

While based on a single-instance architecture, the system is designed with future support for multi-container or distributed deployment in mind.

---
## 6. Data Collection Flow

1. Retrieve schedule info from the Cron model
2. Register Celery workers based on the retrieved schedule
3. Execute workers at the designated time
4. Fetch price and indicator data via external APIs
5. Store the data into Coin and Indicator models

---
## 7. Trading Execution Flow

1. Retrieve active strategies from the Strategy model
2. Execute entry condition script → Check Indicator values
3. If conditions are met, execute the trade via the defined method
4. Record the resulting position in the Position model
5. Monitor the position and exit when conditions from the Strategy model are satisfied

---
## 8. System Implementation Details

- Customized Django Admin: Added filtering, search, CSV export, etc.
- Unified logging utility: Logs to console, file, and DB
- Script management: Strategy logic stored externally and referenced via DB
- Deployment: GitHub Actions + Git Flow

---
## 9. Design Principles

Since there’s no absolute “correct” strategy for profit in crypto trading, the system was designed for high adaptability, strategy modularity, and data-driven iteration. The structure had to support experimentation even in the absence of initial clear specifications.

---
## 10. Operation Status and Reflection (as of May 2025)

- Data collection and indicator crawling are stable, and workers run without errors.
- Initially, only a basic arbitrage algorithm is being used, with limited real trading capital.
- Strategy modules are being expanded to support more complex, multi-layered logic.

The system is currently positioned as an internal platform, focusing on structural stability and preparing for wider strategy experimentation.

Managing this project while working full-time in Japan was physically demanding, especially since I handled everything from planning to implementation. However, my prior experience in crypto trading and strong interest in system design enabled me to deliver a scalable, coherent system.

While the core infrastructure and strategy logic are mostly in place, actual trading has not yet been activated. The system is still under development, with data collection running continuously and strategy scripts in progress.

Looking back, I feel reasonably satisfied with the outcome so far. But I do wonder how I’ll perceive this work a few years from now.


