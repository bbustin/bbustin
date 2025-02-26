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
```python,linenos,linenostart=51
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

## Allow for using Ollama to run LLM locally

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

# First try using it

I run `nix develop` and it pops up Jupyter in my browser. I navigate to `/src/open_deep_research`
and create a new Jupyter notebook. I mostly copy from the `graph.ipynb` notebook. To create
this:

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

To be continued...
