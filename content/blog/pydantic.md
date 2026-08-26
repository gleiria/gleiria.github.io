+++
title = '"Hello world" version of an AI-data-analyst'
date = 2026-08-10T18:36:58+01:00
draft = false
+++

I am developing an LLM-powered data analyst for a fictional SaaS company. The idea is that a user can ask questions about the business in natural language, and the agent can use the database to answer them if needed. I am using Pydantic-AI to connect the agent to Google's Gemini models and Django to handle the application and data layer. 

**The SaaS Company**

I created a fictional SaaS company and generated synthetic data for it. The goal is not to create a perfect simulation of a real business but to create a small coherent world that I can use to explore the AI-agent architecture and that the agent can reason about. As the project evolves, I will gradually add more data and relationships. This is literally just me putting the pieces together.


For the first version, I generated six months of data (01-01-2026 to 30-06-2026) for 10,000 customers. The initial data model looks like this:


```
Customer
---------
customer_id
segment
signup_date
     │
     │ 1:1
     ▼
Subscription
------------
subscription_id
customer_id
plan
price
start_date
status

Customer
   │
   │ 1:many
   ▼
DailyUsage
----------
customer_id
date
usage

ProductIncident
---------------
incident_id
date
severity
```
---

**The Agent**

Pydantic-AI integrates with most LLM providers. For this project I picked Google Gemini-3.1-flash-lite-preview which is speed optimised and cheaper than the pro models. It also allows you to perform hundreds of API calls a day without providing your credit card details. I got the API key from [aistudio.com](https://aistudio.google.com).

With the API key in place we can already interact programatically with the LLM:
```python
from pydantic_ai import Agent

agent = Agent(
    "google:gemini-3.1-flash-lite-preview",
    instructions="Provide concise answers",
)

result = agent.run_sync("What is a SaaS company?")
print(result)
```

Answer:
```bash
A SaaS (Software as a Service) company hosts applications on its servers and delivers them to customers over the internet, typically through a web browser or app.
```
Cool! We can interact with the LLM through its API, ask questions, and get responses. Now, the magic happens when the agent is actually capable of performing tasks within the software. We can provide it **tools** to query databases, make API calls, run Python functions, perform calculations, or interact with other parts of the application. The model can then decide when one of these **tools** is needed to answer a user's question. Notice that it is the **tool** itself, and not the LLM, that does the actual work. 
When this functionality is in place, the user can ask the LLM business related questions such as:

* How many customers do we have as of today?
* How many customers do we have per segment?
* Which subscription plan is generating the most revenue?
* Have there been any major product incidents recently?


The rest of the post shows how we get there
---


This is a Django project and the file structure follows Django's MVT architecture. However, before wiring things up with Django I simply wanted to see if I could perfrom a "hand-shake" between my application, the LLM, and the database.

**Function Tools**

This is copy and pasted from Pydantic-AI docs:
"Function tools provide a mechanism for models to perform actions and retrieve extra information to help them generate a response. They are useful when you want to enable the model to take some action and use the result, when it is impractical or impossible to put all the context an agent might need into the instructions, or when you want to make agents’ behavior more deterministic or reliable by deferring some of the logic required to generate a response to another (not necessarily AI-powered) tool.". Basicaly, the LLM can choose to call a **tool** during the interaction to get real data instead of hallucinating an answer.

Here is the 'hello word' version of my agent:

```python

from pydantic_ai import Agent
from agent_app.models import Customer

agent = Agent(
    "google:gemini-3.1-flash-lite-preview",
    instructions="You are a data analyst for a SaaS company. Answer questions using the available tools when needed.",
)

# register function with agent as an available tool
@agent.tool_plain 
def get_customer_count() -> int:
    """Return the total number of customers in the database."""
    return Customer.objects.count()
```

Pydantic-AI inspects the function's signature and docstring to build a tool schema (name, parameters, description) that gets sent to the LLM alongside the prompt we write.
The dockstring **"""Return the total number of customers in the database."""** it's not just documentation. Pydantic uses it as the tool's description in the schema sent to the model. This is what the LLM reads to decide when to call function.

When we ask "how many customers do we have?", the model sees the available tool and its description and decides that get_customer_count() is appropriate for answering the question. Pydantic then executes the Python function. The function runs a normal Django ORM query against the database and returns the result to the agent which uses it to formulate a response back to the user.

The return type annotation **-> int** also gives the tool a clear contract. Pydantic AI uses the type information to describe and validate the tool's output, so the value returned by the function is expected to be an integer. This is one of the places where Pydantic's data-validation capabilities become useful in an agent. In addintion to being a python function, the **tool** has a defined interface between the application and the LLM.

It is important to note that the LLM does not query the database itself. It decides that a particular **tool** is needed, but the actual database operation is performed deterministically by our Python function through the Django ORM. The LLM receives the result and uses it to construct the answer.

---

And that is it. We have a very small "hello world" version of an LLM-powered data analyst. It doesn't do anything particularly impressive yet. It can answer one simple question about the business, but the architecture is already there: 

user asks question → LLM → tool selection → Python function → Django ORM → database → tool result → LLM → answer to the user.

Next, I will write a few more tools and add more data to the SaaS company. The goal is to gradually move from this simple get_customer_count() example towards an agent that can answer more meaningful business questions by combining data from multiple tables in the database.

Stay tunned! :rocket: :rocket: :rocket:







