---
layout: post
title: "AI Distributed Response & Evaluation System"
subtitle: "Architecture for Parallel LLM Querying with Quality Assessment"
date: 2025-04-20
categories: design architecture ai
---

## Project Overview

This project was initiated during my part-time engagement at an AI startup. I independently proposed the idea, designed the architecture, and implemented the system. The development was self-directed, unconstrained by existing infrastructure or directives, and focused on identifying and solving a core problem that had not yet been addressed within the team.

1. At the time, there was no internal metric or feedback mechanism to determine whether an LLM-generated answer was actually "good." I identified this gap and proposed an evaluation system to score LLM responses.
2. Although the product was gaining users, monetization remained unstable, and the infrastructure cost (especially token usage) was too high relative to revenue. This system aimed to deliver better quality responses with fewer tokens.

Design principles included:

1. Seamless insertion into an existing LLM + RAG + prompt pipeline without disrupting the flow.
2. Evaluation components should remain modular and loosely coupled from the main flow.
3. If possible, modular components should be deployable in a microservice architecture.
4. The existing vector search and RAG components should be reused, minimizing additional computation.
5. Use smaller, lower-cost LLMs instead of the ones currently in production to reduce token expenses.

## System Components

1. **Vector Search Module**

   * Replicates and integrates the existing vector search engine and RAG index from the core business system.

2. **Evaluation Module**

   * A standalone scoring engine for evaluating the quality of responses.
   * Uses LangChain's `StructuredOutputParser` with evaluation prompts to assign scores between 0–100 based on understanding, accuracy, and relevance.

3. **Low-Cost LLM Response Generator**

   * Generates answers using cheaper, lightweight LLMs for reduced token cost.

4. **Asynchronous Distributed Executor**

   * Executes response generation as concurrent background tasks within a single container, while measuring latency.

5. **Response Comparison & Final Selector**

   * Compares the scored responses and selects the best one, with logic to fall back or ensemble if necessary.

## Process Flow

1. User submits a question.
2. The system determines the user profile.
3. It detects the platform type of the question (e.g., web, app, chatbot).
4. Based on these, it loads the corresponding prompt and RAG data.
5. Low-cost LLMs are used to generate multiple candidate answers in parallel.
6. Each response is evaluated using a dedicated scoring prompt and structured scoring logic.
7. The best-scoring response is selected.
8. The final answer is returned to the user, along with internal evaluation metadata.

## Architecture Design

The system is implemented in a single-container environment, with external RAG components integrated directly into the runtime. The entire flow is executed sequentially in one process, optimized for fast development and internal testing.

1. **Single API Runtime**

   * All operations are handled within a single container—from receiving the question to selecting and returning the final response.

2. **RAG Integration**

   * The system replicates the existing business vector search and RAG modules and executes them locally within the container.

3. **Parallel LLM Execution**

   * Low-cost LLMs are called in parallel via asynchronous background tasks. No distributed worker nodes or clusters are used.

4. **Evaluation Logic**

   * Each generated response is passed into a scoring prompt and evaluated using an LLM, with results parsed via StructuredOutputParser.

5. **Response Selection**

   * The highest-scoring response is returned, with fallback or composition logic if needed.

**Characteristics and Limitations:**

* Easy to maintain and test due to single-container architecture.
* Not fully packaged for production; integration was conditional upon internal review.
* Designed to allow future migration to modular or microservice-based deployment.

## Ops & Evaluation Strategy

1. 100 real user questions were sampled from production, excluding invalid or malformed input.
2. Each question was tested across four configurations:

   * GPT-3.5 (single response)
   * GPT-4.0 (single response)
   * GPT-3.5 multi-response (4 candidates + selection)
   * GPT-4.0 multi-response (4 candidates + selection)
3. Each question triggered six LLM calls:

   * 4 response generations
   * 1 evaluation
   * 1 final selection validation
4. Around 400 results were collected in total, each with a score and response latency.

| Model Config     | Avg. Score | Avg. Latency |
| ---------------- | ---------- | ------------ |
| GPT-3.5 (Single) | \~55       | \~3.6 sec    |
| GPT-3.5 (Multi)  | \~68       | \~8.0 sec    |
| GPT-4.0 (Single) | \~72       | \~7.0 sec    |
| GPT-4.0 (Multi)  | \~76       | \~10.0 sec   |

### Business Feedback

GPT-3.5 multi showed the best cost-performance ratio compared to GPT-4.0 (single). However, it was rejected. I was told by the CEO that quality was more important than time or cost.

Running GPT-4.0 in multi-response mode was also tested, but the cost would increase 3–5x per query. Although quality matters most, this level of cost increase was not acceptable, so that setup was discarded.

## Retrospective & Improvements

What mattered most was the evaluation logic. I no longer work at that company, so I don't know if my evaluation pipeline is still in use. But I firmly believe that setting criteria for evaluating responses is more important than anything else in AI.

At the time, the CEO was more focused on user acquisition and wasn't very responsive to new product proposals. From an engineer's perspective, I felt that without a solid product, users would just leave no matter how many we acquired.

Instead of building another LLM chatbot as a business, I now think AI should be applied to more diverse and grounded use cases across other industries.
