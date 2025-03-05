+++
title = "Open Deep Research using LangChain"

[taxonomies]
tags = ["Research", "AI", "Agents", "Nix", "langchain"]
+++

In [Foray Into Agentic AI](@/posts/2025-02-07_foray_into_agentic_ai.md), I adapted HuggingFace's
Open Deep Research work to run locally. Now I am going to try to do the same using
[LangChain's Open Deep Research code](https://github.com/langchain-ai/open_deep_research).
<!--more-->

# Initial Setup

Clone the repository locally and then cd into it.

```bash
❯ git clone https://github.com/langchain-ai/open_deep_research.git
Cloning into 'open_deep_research'...
remote: Enumerating objects: 388, done.
remote: Counting objects: 100% (120/120), done.
remote: Compressing objects: 100% (60/60), done.
remote: Total 388 (delta 67), reused 66 (delta 55), pack-reused 268 (from 2)
Receiving objects: 100% (388/388), 682.29 KiB | 3.20 MiB/s, done.
Resolving deltas: 100% (174/174), done.

~/projects
❯ cd open_deep_research
```

I am going to do the same as I have before by creating a Nix flake. See
[Diving Into AI](@/posts/2025-02-06_diving_into_ai.md) for more information about Nix and
initially setting it up.

```nix
{
	description = "Shell for running Open Deep Research";

	inputs =
		{
			nixpkgs.url = github:nixos/nixpkgs/nixos-unstable;
		};

	outputs = { self, nixpkgs, ... }:
		let
			system = "aarch64-darwin";
			pkgs = nixpkgs.legacyPackages.${system};
			pythonVersion = "python313";
		in
		{
			devShells.${system}.default = pkgs.mkShell
				{
					buildInputs = [
						pkgs.${pythonVersion}
						pkgs."${pythonVersion}Packages".pip
					];

					shellHook = ''
						python -m venv .venv
						source .venv/bin/activate
						pip install jupyter duckduckgo_search langchain-ollama -e . && jupyter lab
						exit
					'';
				};
		};
}
```

Then `git add flake.nix` or else it will not work.

## Allow searching using DuckDuckGo

The code is setup by default to use the Tavily search API, and then Anthropic or OpenAI to
run the LLM. I want to change this to using DuckDuckGo for search and an LLM running locally.

Here is what I did:

In `src/open_deep_research/configuration.py`, edit the SearchAPI class to:

```python
class SearchAPI(Enum):
    PERPLEXITY = "perplexity"
    TAVILY = "tavily"
    DUCKDUCKGO = "duckduckgo"
```

In `src/open_deep_research/utils.py`:

* Add the following import:

```python
from duckduckgo_search import DDGS
```

* Create a `duckduckgo_search` function following the example of
`perplexity_search`.

```python
@traceable
def duckduckgo_search(search_queries):
    """Search the web using DuckDuckGo.

    Args:
        search_queries (List[SearchQuery]): List of search queries to process

    Returns:
        List[dict]: List of search responses from DuckDuckGo, one per query. Each response has format:
            {
                'query': str,                    # The original search query
                'follow_up_questions': None,
                'answer': None,
                'images': list,
                'results': [                     # List of search results
                    {
                        'title': str,           # Title of the search result
                        'url': str,             # URL of the result
                        'content': str,         # Summary/snippet of content
                    },
                    ...
                ]
            }
    """

    search_docs = []
    for query in search_queries:
        results = DDGS().text(query, backend="lite")

        # match field names used for Tavily
        results = [{'title': result['title'], 'url': result['href'], 'content': result['body']} for result in results]

        # Format response to match Tavily structure
        search_docs.append({
            "query": query,
            "follow_up_questions": None,
            "answer": None,
            "images": [],
            "results": results
        })

    return search_docs
```

In `src/open_deep_research/graph.py`:

* On line 14 add `duckduckgo_search` to the imports
* Update `generate_report_plan` so it can perform a DuckDuckGo search if configured to:
```python
# Search the web
if search_api == "tavily":
    search_results = await tavily_search_async(query_list)
    source_str = deduplicate_and_format_sources(search_results, max_tokens_per_source=1000, include_raw_content=False)
elif search_api == "perplexity":
    search_results = perplexity_search(query_list)
    source_str = deduplicate_and_format_sources(search_results, max_tokens_per_source=1000, include_raw_content=False)
elif search_api == "duckduckgo":
    search_results = duckduckgo_search(query_list)
    source_str = deduplicate_and_format_sources(search_results, max_tokens_per_source=1000, include_raw_content=False)
else:
    raise ValueError(f"Unsupported search API: {configurable.search_api}")
```
* Update `search_web` so it can perform a DuckDuckGo search if configured to"
```python
# Search the web
if search_api == "tavily":
    search_results = await tavily_search_async(query_list)
    source_str = deduplicate_and_format_sources(search_results, max_tokens_per_source=5000, include_raw_content=True)
elif search_api == "perplexity":
    search_results = perplexity_search(query_list)
    source_str = deduplicate_and_format_sources(search_results, max_tokens_per_source=5000, include_raw_content=False)
elif search_api == "duckduckgo":
    search_results = duckduckgo_search(query_list)
    source_str = deduplicate_and_format_sources(search_results, max_tokens_per_source=5000, include_raw_content=False)
else:
    raise ValueError(f"Unsupported search API: {configurable.search_api}")
```

## Allow for using Ollama to run the LLM locally

In `src/open_deep_research/configuration.py`, edit PlannerProvider and WriterProvider to be able to use Ollama:

```python
class PlannerProvider(Enum):
    ANTHROPIC = "anthropic"
    OPENAI = "openai"
    GROQ = "groq"
    OLLAMA = "ollama"

class WriterProvider(Enum):
    ANTHROPIC = "anthropic"
    OPENAI = "openai"
    GROQ = "groq"
    OLLAMA = "ollama"
```

Luckily, LangChain already supports Ollama. So this is the only change needed aside
from installing `langchain-ollama` which `flake.nix` is already taking care of for us.

# Running it

I run `nix develop` and it pops up Jupyter in my browser. I navigate to `/src/open_deep_research`
and create a new Jupyter notebook. I mostly copy from the `graph.ipynb` notebook.

```python
import open_deep_research
print(open_deep_research.__version__)

# We are not going to use Tavily, but it will error out if this is not set
%env TAVILY_API_KEY=NONE

from IPython.display import Image, display
from langgraph.types import Command
from langgraph.checkpoint.memory import MemorySaver
from open_deep_research.graph import builder

memory = MemorySaver()
graph = builder.compile(checkpointer=memory)
display(Image(graph.get_graph(xray=1).draw_mermaid_png()))

import uuid
from IPython.display import Markdown

# local config
thread = {"configurable": {"thread_id": str(uuid.uuid4()),
                           "search_api": "duckduckgo",
                           "planner_provider": "ollama",
                           "planner_model": "qwen2.5-coder:7b",
                           "writer_provider": "ollama",
                           "writer_model": "llama3.3",
                           "max_search_depth": 2,
                           }}

# Create a topic
topic = "Overview of the AI inference market with focus on Fireworks, Together.ai, Groq"

# Run the graph until the interruption
async for event in graph.astream({"topic":topic,}, thread, stream_mode="updates"):
    if '__interrupt__' in event:
        interrupt_value = event['__interrupt__'][0].value
        display(Markdown(interrupt_value))

# Pass feedback to update the report plan
async for event in graph.astream(Command(resume="Include a revenue estimate (ARR) in the sections focused on Groq, Together.ai, and Fireworks"), thread, stream_mode="updates"):
    if '__interrupt__' in event:
        interrupt_value = event['__interrupt__'][0].value
        display(Markdown(interrupt_value))

# Pass True to approve the report plan
async for event in graph.astream(Command(resume=True), thread, stream_mode="updates"):
    print(event)
    print("\n")

final_state = graph.get_state(thread)
report = final_state.values.get('final_report')
Markdown(report)
```

When I run it, I get the following error:
```bash
ValidationError: 2 validation errors for Queries
queries.0
  Input should be a valid dictionary or instance of SearchQuery [type=model_type, input_value='AI inference market trends and growth', input_type=str]
    For further information visit https://errors.pydantic.dev/2.10/v/model_type
queries.1
  Input should be a valid dictionary or instance of SearchQuery [type=model_type, input_value='Fireworks, Together.ai, ...utions and applications', input_type=str]
    For further information visit https://errors.pydantic.dev/2.10/v/model_type
During task with name 'generate_report_plan' and id '2238eb30-2a86-de8d-2b1f-f3988c67325c'
```

I think this means the model is not generating the search queries properly when coming up
with a research plan.

# Troubleshooting the validation errors
If I change the code to operate at the message level and have it print each message,
I should be able to get a better idea of what is happening:

```python
# Run the graph until the interruption
import pprint
async for event in graph.astream({"topic":topic,}, thread, stream_mode="messages"):
    pprint.pprint(event)
    # if '__interrupt__' in event:
    #     interrupt_value = event['__interrupt__'][0].value
    #     display(Markdown(interrupt_value))
```

Among the huge amount of output, I saw this:
```python
tool_calls=[{'name': 'Queries', 'args': {'queries': ['Overview of AI inference market trends and key players 2023', 'Detailed analysis of Fireworks, Together.ai, and Groq in AI inference market including their technologies and competitive landscape']}, 'id': '7c25ca51-0200-4302-b455-8307f36147c6', 'type': 'tool_call'}]
```

It looks like it did, indeed, generate two queries and those are not in the format the validation
error is specifying. I revert the code changes. Right now, I'm not sure if it is the model that is
supposed to format them properly or some other code.

LangChain appears to have really good documentation.
I read [How to return structured data from a model](https://github.com/langchain-ai/langchain/blob/master/docs/docs/how_to/structured_output.ipynb).

First, the [model must support `.with_structured_output()`](https://github.com/langchain-ai/langchain/tree/master/docs/docs/integrations/chat).
It looks like [Ollama does support structured output](https://github.com/langchain-ai/langchain/blob/master/docs/docs/integrations/chat/ollama.ipynb).
It mentions installing `langchain-ollama` which I already did. It also mentions updating the `ollama` library
to make sure the latest version supporting structured outputs is installed. That is worth trying.

I open a terminal in Jupyter and type `pip install -U ollama`. Then I go back to my notebook
and choose `Restart Kernel and Run All Cells...` from the `Kernel` menu. That does not help,
but it was worth a shot.

There really aren't any more clues in the LangChain Ollama documentation. I either need to debug
the code or create sample code. I am going to choose the latter.

Here is my test code to see what is happening.

```python
from typing import List
from langchain_ollama import ChatOllama
from pydantic import BaseModel, Field
from langchain_core.messages import HumanMessage, SystemMessage

llm = ChatOllama(
    model="qwen2.5-coder:7b",
    temperature=0,
)

class SearchQuery(BaseModel):
    search_query: str = Field(None, description="Query for web search.")

class Queries(BaseModel):
    queries: List[SearchQuery] = Field(
        description="List of search queries.",
)

structured_llm = llm.with_structured_output(Queries)

results = structured_llm.invoke([SystemMessage(content="You are a helpful research assistant who will be drafting a report. The subject of the report will be provided by the human.")]+[HumanMessage(content="Generate search queries that will help with planning sections of a report about LangChain.")])
print(results)
```

I am getting similar errors with this code. That is very helpful because now I've pared this
down quite a bit.

If I change `structured_llm = llm.with_structured_output(Queries)` to `structured_llm = llm.with_structured_output(Queries, include_raw=True)`,
then I can see the raw output. It looks like this:

```python
{'parsed': None,
 'parsing_error': 4 validation errors for Queries
queries.0
  Input should be a valid dictionary or instance of SearchQuery [type=model_type, input_value='report structure', input_type=str]
    For further information visit https://errors.pydantic.dev/2.10/v/model_type
queries.1
  Input should be a valid dictionary or instance of SearchQuery [type=model_type, input_value='section planning', input_type=str]
    For further information visit https://errors.pydantic.dev/2.10/v/model_type
queries.2
  Input should be a valid dictionary or instance of SearchQuery [type=model_type, input_value='content organization', input_type=str]
    For further information visit https://errors.pydantic.dev/2.10/v/model_type
queries.3
  Input should be a valid dictionary or instance of SearchQuery [type=model_type, input_value='time management for report writing', input_type=str]
    For further information visit https://errors.pydantic.dev/2.10/v/model_type,
 'raw': AIMessage(content='', additional_kwargs={}, response_metadata={'model': 'qwen2.5-coder:7b', 'created_at': '2025-02-27T18:01:38.990692Z', 'done': True, 'done_reason': 'stop', 'total_duration': 1125161416, 'load_duration': 30422375, 'prompt_eval_count': 161, 'prompt_eval_duration': 162000000, 'eval_count': 71, 'eval_duration': 930000000, 'message': Message(role='assistant', content='', images=None, tool_calls=None)}, id='run-e4fef1f2-8f57-4c95-9f8e-16c452dafe7b-0', tool_calls=[{'name': 'Queries', 'args': {'queries': ['report structure', 'section planning', 'content organization', 'time management for report writing']}, 'id': '8e9cd5e8-c02b-44b2-a330-c71e75a6848f', 'type': 'tool_call'}], usage_metadata={'input_tokens': 161, 'output_tokens': 71, 'total_tokens': 232})}
```

In the `tool_calls` field, there is a `Queries` tool call with a list of search queries. The
validation errors are saying
`Input should be a valid dictionary or instance of SearchQuery [type=model_type, input_value='report structure', input_type=str]`.
It is true the output is not a dictionary, but `SearchQuery` defines a single field that is a `str`.
I suppose it wants the output to look like this?

```python
[{'search_query': 'report structure'}, {'search_query': 'section planning'}, {'search_query': 'content organization'}, {'search_query': 'time management for report writing'}]
```

Time to learn more about [pydantic](https://docs.pydantic.dev/latest/). Yeah, that is the problem. Let's test my assumption above
while taking LangChain completely out of the loop.

```python
from typing import List
from pydantic import BaseModel, Field, ValidationError
import json

class SearchQuery(BaseModel):
    search_query: str = Field(None, description="Query for web search.")

class Queries(BaseModel):
    queries: List[SearchQuery] = Field(
        description="List of search queries.",
)

query_output_from_llm = {'queries': ['report structure', 'section planning', 'content organization', 'time management for report writing']}

print("Original Data")
try:
    Queries(**query_output_from_llm)
    print("Success!")
except ValidationError as e:
    print("Error..........")
    print(e.errors())

possibly_corrected_query_output = {'queries': [{'search_query': 'report structure'}, {'search_query': 'section planning'}, {'search_query': 'content organization'}, {'search_query': 'time management for report writing'}]}
print("-" * 80)
print("\nUpdated Data")
try:
    Queries(**possibly_corrected_query_output)
    print("Success!")
except ValidationError as e:
    print("Error..........")
    print(e.errors())
```

The output I get from running this is:
```
Original Data
Error..........
[{'type': 'model_type', 'loc': ('queries', 0), 'msg': 'Input should be a valid dictionary or instance of SearchQuery', 'input': 'report structure', 'ctx': {'class_name': 'SearchQuery'}, 'url': 'https://errors.pydantic.dev/2.10/v/model_type'}, {'type': 'model_type', 'loc': ('queries', 1), 'msg': 'Input should be a valid dictionary or instance of SearchQuery', 'input': 'section planning', 'ctx': {'class_name': 'SearchQuery'}, 'url': 'https://errors.pydantic.dev/2.10/v/model_type'}, {'type': 'model_type', 'loc': ('queries', 2), 'msg': 'Input should be a valid dictionary or instance of SearchQuery', 'input': 'content organization', 'ctx': {'class_name': 'SearchQuery'}, 'url': 'https://errors.pydantic.dev/2.10/v/model_type'}, {'type': 'model_type', 'loc': ('queries', 3), 'msg': 'Input should be a valid dictionary or instance of SearchQuery', 'input': 'time management for report writing', 'ctx': {'class_name': 'SearchQuery'}, 'url': 'https://errors.pydantic.dev/2.10/v/model_type'}]
--------------------------------------------------------------------------------

Updated Data
Success!
```

This confirms my suspicions. The model needs to be informed how to format the data. It
seems like the tooling should do it automatically, but that does not appear to be happening.

I find some details on [how Ollama works with structured outputs](https://ollama.com/blog/structured-outputs).
It appears Ollama has a format parameter which accepts JSON. `ChatOllama.with_structured_output` accepts
a parameter called `method`. If we add `method="json_schema"`, maybe the call will work?

To test this theory, I go back to some earlier test code and modify the `.with_structured_output`
call.

```json
from typing import List
from langchain_ollama import ChatOllama
from pydantic import BaseModel, Field
from langchain_core.messages import HumanMessage, SystemMessage

llm = ChatOllama(
    model="qwen2.5-coder:7b",
    temperature=0,
)


class SearchQuery(BaseModel):
    search_query: str = Field(None, description="Query for web search.")

class Queries(BaseModel):
    queries: List[SearchQuery] = Field(
        description="List of search queries.",
)

structured_llm = llm.with_structured_output(Queries, method="json_schema")
result = structured_llm.invoke([SystemMessage(content="You are a helpful research assistant who will be drafting a report.")]+[HumanMessage(content="Generate search queries that will help with planning the sections of the report.")])

print(result)
```

This works! Now I go back to `src/open_deep_research/graph.py` and modify the code so that if `planner_provider`
is `ollama`, then `method="json_schema"` is included `.with_structured_output` call.
I make the same change for writer_provider. I could just omit this conditional logic, but I want
this to still be compatible with other LLM providers.

```python
async def generate_report_plan(state: ReportState, config: RunnableConfig):
    """ Generate the report plan """
    # Set writer model (model used for query writing and section writing)
    writer_provider = get_config_value(configurable.writer_provider)
    writer_model_name = get_config_value(configurable.writer_model)
    writer_model = init_chat_model(model=writer_model_name, model_provider=writer_provider, temperature=0)
    structured_llm = writer_model.with_structured_output(Queries,
        method="json_schema" if writer_provider == "ollama" else None)
    ## skipping code for post ##
    # Set the planner model
    if planner_model == "claude-3-7-sonnet-latest":
        planner_llm = init_chat_model(model=planner_model, model_provider=planner_provider, max_tokens=20_000, thinking={"type": "enabled", "budget_tokens": 16_000})
        # with_structured_output uses forced tool calling, which thinking mode with Claude 3.7 does not support. Use bind_tools to generate the report sections
        structured_llm = planner_llm.bind_tools([Sections])
        report_sections = structured_llm.invoke([SystemMessage(content=system_instructions_sections)]+[HumanMessage(content="Generate the sections of the report. Your response must include a 'sections' field containing a list of sections. Each section must have: name, description, plan, research, and content fields.")])
        tool_call = report_sections.tool_calls[0]['args']
        report_sections = Sections.model_validate(tool_call)
    else:
        planner_llm = init_chat_model(model=planner_model, model_provider=planner_provider)
        structured_llm = planner_llm.with_structured_output(Sections,
            method="json_schema" if planner_provider == "ollama" else None)
        report_sections = structured_llm.invoke([SystemMessage(content=system_instructions_sections)]+[HumanMessage(content="Generate the sections of the report. Your response must include a 'sections' field containing a list of sections. Each section must have: name, description, plan, research, and content fields.")])
    ## skipping code for post ##

def generate_queries(state: SectionState, config: RunnableConfig):
    """ Generate search queries for a report section """
    ## skipping code for post ##
    # Generate queries
    writer_provider = get_config_value(configurable.writer_provider)
    writer_model_name = get_config_value(configurable.writer_model)
    writer_model = init_chat_model(model=writer_model_name, model_provider=writer_provider, temperature=0)
    structured_llm = writer_model.with_structured_output(Queries,
        method="json_schema" if writer_provider == "ollama" else None)
    ## skipping code for post ##

def write_section(state: SectionState, config: RunnableConfig) -> Command[Literal[END, "search_web"]]:
    """ Write a section of the report """
    ## skipping code for post ##
    # Feedback
    structured_llm = writer_model.with_structured_output(Feedback,
        method="json_schema" if writer_provider == "ollama" else None)
    feedback = structured_llm.invoke([SystemMessage(content=section_grader_instructions_formatted)]+[HumanMessage(content="Grade the report and consider follow-up questions for missing information:")])
    ## skipping code for post ##
```

Time to test! I go back to my original Jupyter notebook, go to the `Kernel` menu
and select `Restart Kernel and Run all Cells`. Now there is a RateLimit error
with the DuckDuckGo search API. I must've been sending out a ton of search queries
to hit that.

## Implementing simple rate limiting for DuckDuckGo search
This is not going to be a production-grade fix here. I am simply going to make the
`duckduckgo_search` wait 1 second between searches. If it hits a rate limit, it will
wait 5 seconds and try the search that did not succeed again.

I make the following changes to to `src/open_deep_research/utils.py`:

* Import the exception, and time:
```python
import time
from duckduckgo_search.exceptions import DuckDuckGoSearchException
```
* Modify `duckduckgo_search`:
```python
@traceable
def duckduckgo_search(search_queries):
    """Search the web using DuckDuckGo.

    Args:
        search_queries (List[SearchQuery]): List of search queries to process

    Returns:
        List[dict]: List of search responses from DuckDuckGo, one per query. Each response has format:
            {
                'query': str,                    # The original search query
                'follow_up_questions': None,
                'answer': None,
                'images': list,
                'results': [                     # List of search results
                    {
                        'title': str,           # Title of the search result
                        'url': str,             # URL of the result
                        'content': str,         # Summary/snippet of content
                    },
                    ...
                ]
            }
    """
    def perform_search(query):
            max_retries = 2
            retry_count = 0

            while retry_count < max_retries:
                try:
                    results = DDGS().text(query, backend="lite")
                    # match field names used for Tavily
                    return [{'title': result['title'], 'url': result['href'], 'content': result['body']} for result in results]
                except DuckDuckGoSearchException as e:
                    if "202 RateLimit" in str(e):
                        retry_count += 1
                        # if this is the last retry, propagate the original exception
                        if retry_count == max_retries:
                            raise
                        # wait before trying again
                        time.sleep(5)
                    else:
                        raise

    search_docs = []
    for query in search_queries:
        time.sleep(1)
        results = perform_search(query)

        # Format response to match Tavily structure
        search_docs.append({
            "query": query,
            "follow_up_questions": None,
            "answer": None,
            "images": [],
            "results": results
        })

    return search_docs
```

## Third time's the charm

It works... kind of. I think it has problems incorporating data from the search results when
constructing sections. I was watching outbound traffic and saw it connect to DuckDuckGo many times.
What I did not see was it following any of the links.

I will need to circle back soon to better understand what is going on.

Here is what it generated. The original research question was
`Overview of the AI inference market with focus on Fireworks, Together.ai, Groq`.

---

# Overview of the AI Inference Market with Focus on Fireworks, Together.ai, Groq

The AI inference market is experiencing rapid growth driven by the increasing demand for real-time decision-making across various industries such as healthcare, finance, and autonomous vehicles. This report delves into the current trends, growth projections, and key players in this dynamic sector, with a particular focus on Fireworks, Together.ai, and Groq. By examining their technology features, use cases, and financial performance, we aim to provide insights that can guide stakeholders in making informed decisions.

## Summary

The AI inference market is poised for significant expansion, fueled by advancements in hardware and software technologies. Fireworks, Together.ai, and Groq are leading players, each offering unique solutions tailored to diverse applications. Below is a comparative table summarizing their key attributes:

| Feature/Company | Fireworks | Together.ai | Groq |
|-----------------|-----------|-------------|------|
| Technology      | Distributed AI infrastructure | Collaborative AI platform | Custom silicon for high-performance inference |
| Use Cases       | Large-scale AI model deployment, cloud services | Team collaboration and shared AI models | Real-time AI processing in edge devices |
| ARR (Approx.)   | $10M+     | $5M+        | $20M+|

These platforms represent different approaches to addressing the challenges of AI inference, from distributed computing to specialized hardware solutions. As the market continues to evolve, understanding these distinctions is crucial for leveraging AI effectively.

Next steps include further analysis of emerging trends and potential partnerships within the industry to capitalize on growth opportunities.

Certainly! To generate a report section, I'll need you to provide the specific details or sources you want included in the report. Please share the relevant information or documents so that I can craft an accurate and detailed section for your report.

Certainly! To generate a report section, I'll need you to provide the specific details or sources you want included in the report. Please share the relevant information or documents so that I can craft an accurate and detailed section for your report.

Certainly! To generate a report section, I'll need you to provide the specific details or sources you want included in the report. Please share the relevant information or documents so that I can craft an accurate and detailed section for your report.

Certainly! To generate a report section, I'll need you to provide the specific details or sources you want included in the report. Please share the relevant information or documents so that I can craft an accurate and detailed section for your report.

Certainly! To generate a report section, I'll need you to provide the specific details or sources you want included in the report. Could you please share the relevant information or documents? This could include data points, statistics, quotes from experts, or any other pertinent material that should be incorporated into the report section.

# Overview of the AI Inference Market with Focus on Fireworks, Together.ai, and Groq

The AI inference market is experiencing rapid growth driven by advancements in artificial intelligence technology and increasing demand for real-time data processing across various industries. This report delves into the key trends shaping this market and provides a detailed analysis of three prominent players: Fireworks, Together.ai, and Groq. By examining their unique offerings, use cases, and financial performance, we aim to offer insights that can guide stakeholders in making informed decisions.

## Summary

The AI inference market is witnessing significant expansion, fueled by the need for efficient and scalable solutions. Among the leading companies, Fireworks stands out with its cloud-based infrastructure designed for seamless deployment of machine learning models. Together.ai offers a collaborative platform that simplifies model sharing and experimentation, catering to teams looking to streamline their workflow. Groq, on the other hand, focuses on hardware acceleration, providing ultra-fast inference capabilities through custom silicon solutions.

| Company     | Technology Focus         | Use Cases                          | Annual Recurring Revenue (ARR) |
|-------------|--------------------------|------------------------------------|----------------------------------|
| Fireworks   | Cloud-based infrastructure | Real-time analytics, customer service | Not publicly disclosed           |
| Together.ai | Collaborative platform     | Model sharing, experimentation       | Not publicly disclosed           |
| Groq        | Hardware acceleration      | High-performance inference         | Not publicly disclosed           |

These platforms represent different strategies within the AI inference market, each addressing specific needs and challenges. As the landscape continues to evolve, understanding these distinctions is crucial for leveraging AI effectively.

Next steps include further analysis of emerging trends and potential partnerships that could shape the future of AI inference solutions.
