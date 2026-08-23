+++
title = 'Pydantic 2'
date = 2026-08-22T07:15:06+01:00
draft = true
+++

Pydantic-ai is async first. So, instead of agent.runsync we can call the agent.run method as long as you are in an async context and that is going to apply tipically in for example a FastAPI application. Using these async contexts and event loops and things like that are going to make your applications more scalable. So for example instead of synchronously waiting around for the agent to return its response, while that happens the evnt loop can pass control to other tasks and check the status of other tasks. So its much more scalable and it allows you to do it much more concurrently if you use asynchronous contexts


*Getting Structured, Validated Outputs*

A string response is fine for simple Q&A, but most production applications need the LLM to return data in a shape your code can immediately consume — a typed object, not a blob of text to parse. Pydantic AI handles this via the output_type argument. See the structured output docs for a detailed overview. (I think I need this to build real time plots)

*Giving Your Agent Tools*

Language models have no access to the outside world. Tools bridge this gap: you register Python functions that the LLM can invoke during its reasoning cycle, receive the results, and continue reasoning before producing its final output.


*Dependency Injection in Practice*

Hardcoding some pandas DF directly in the module works for demos, but in production your agent needs access to things created at runtime: a live database connection, an authenticated API client, or user session data. Pydantic AI’s dependency injection pattern handles this cleanly via a typed RunContext.

------------



