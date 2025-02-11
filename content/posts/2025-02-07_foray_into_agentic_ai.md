+++
title = "Foray into Agentic AI"
updated = 2025-02-11

[taxonomies]
tags = ["Research", "AI", "Agents", "Nix"]
+++

I spoke with someone who is tasked with implementing AI systems at their organization.
The goal, for them, is to use AI to help fetch and format data in response to natural
language queries. Someone could ask, for example:

> Compare the number of widgets sold in 2024 and 2025. Break down the data by quarter.
Present the data in both a table as well as a line chart.

The way they are able to accomplish this is by leveraging agentic AI.
<!--more-->

# What is Agentic AI
```bash
ollama run llama3.3 "What is agentic AI?"
```
> Agentic AI refers to artificial intelligence (AI) systems that are capable 
> of autonomous decision-making and action, often with a degree of 
> self-directedness or goal-oriented behavior. The term "agentic" comes from 
> the word "agent," which in this context means an entity that can act 
> independently to achieve its goals.
> 
> Agentic AI systems are designed to perceive their environment, reason 
> about it, and take actions that influence their surroundings. They may be 
> able to learn from experience, adapt to changing situations, and make 
> decisions based on their own objectives, rather than simply following 
> pre-programmed instructions.
>
> ...

That sounds great, but does not help much here. What I am really looking to find out
is how it works.

Here we go again...
```bash
ollama run llama3.3 "How does agentic AI work?"
```

This yields similar text which does not really help me much. But it sounds great!

```bash
ollama run llama3.3 "What libraries are available to implement agentic AI? What are their key differences?"
```

The response is not helpful. That was a bit of a waste of time.

In my conversation the other day, I learned that you create tools the AI can leverage
to interact with the world. For example, you could create a tool for searching the web
or a tool to perform mathematical operations. The AI needs to know about the existence
of these tools and what they are for. Then it is either trained to or somehow figures out
on its own when and how to use the tools.

OpenAI recently created [DeepResearch](https://en.wikipedia.org/wiki/OpenAI_Deep_Research)
which can autonomously research your questions by searching the web. Shortly thereafter
the Hugging Face community came out with 
[Open Deep Research](https://huggingface.co/blog/open-deep-research) which purports to do the
same thing. Just for funsies, I would like to get Open Deep Research running locally. Then
I would like to ask it to research this area for me.

## Open Deep Research
### Setup

***Editor's note:* This section is long and meandering. Feel free to skip to the next section
if a play by play is not your idea of fun.**

```bash
git clone https://github.com/huggingface/smolagents.git
cd smolagents
git checkout gaia-submission-r1
cd examples/open_deep_research
```

Now I have the code. I'm going to set up a shell with python and all of the requirements
using a Nix flake just as I did in [Diving into AI](@/posts/2025-02-06_diving_into_ai.md).
I put the following in a file called `flake.nix` in the `open_deep_research` directory.

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
						pip install -r requirements.txt
						echo "When done with this shell remember to press ctrl+d"
					'';
					
				};
		};
}
```
***Note:* Nix allows flakes to be composed. At some point, I may be able to create a flake
with most of the boilerplate. Then I can have flakes in my individual projects that import it,
and mostly just need to have the specific differences needed for that project. I may get to that
at some point, but I am not sure if I want to do that. It adds more complication. Sometimes it is
nice for an entity to stand alone. Even if there is more boilerplate.**

Now I run `nix develop` and I get the error:
```
error: path '/nix/store/siwymp8gjx9fi4rag6x3x4hhcxgh8zfv-source/examples/open_deep_research/flake.nix' does not exist
```

When nix is running out of a directory that is a git repository, it will only see files
that are added to the git repository. It does not need the changes to be committed, and you
do not have to `git add` again if you change the flake. It would be nice if this error
message could be improved to provide a bit more guidance for the end user.

I run `git add flake.nix`. Now `nix develop` works, the dependencies are installed,
and I have the bash shell in front of me. Time to see if it works.

```bash
(.venv) bash-5.2$ python run.py
Traceback (most recent call last):
  File "/Users/brianbustin/projects/smolagents/examples/open_deep_research/run.py", line 14, in <module>
    from scripts.reformulator import prepare_response
  File "/Users/brianbustin/projects/smolagents/examples/open_deep_research/scripts/reformulator.py", line 5, in <module>
    from smolagents.models import MessageRole, Model
ModuleNotFoundError: No module named 'smolagents'
```

I'm surprised that is not in the `requirements.txt`. When I look at the `visual_vs_textbrowser.ipynb` file, it starts with the line `!pip install "smolagents[litellm]" -q`. I modify the flake to install the optional smolagents with
the optional litellm dependency `pip install "smolagents[litellm]" -r requirements.txt`.

Second try.
```bash
(.venv) bash-5.2$ python run.py
/Users/brianbustin/projects/smolagents/examples/open_deep_research/.venv/lib/python3.13/site-packages/pydub/utils.py:170: RuntimeWarning: Couldn't find ffmpeg or avconv - defaulting to ffmpeg, but may not work
  warn("Couldn't find ffmpeg or avconv - defaulting to ffmpeg, but may not work", RuntimeWarning)
processor_config.json: 100%|███████████████████| 483/483 [00:00<00:00, 1.03MB/s]
Chat templates should be in a 'chat_template.jinja' file but found key='chat_template' in the processor's config. Make sure to move your template to its own file.
preprocessor_config.json: 100%|████████████████| 460/460 [00:00<00:00, 1.84MB/s]
tokenizer_config.json: 100%|███████████████| 1.64k/1.64k [00:00<00:00, 8.31MB/s]
tokenizer.model: 100%|███████████████████████| 493k/493k [00:00<00:00, 5.22MB/s]
tokenizer.json: 100%|██████████████████████| 1.80M/1.80M [00:00<00:00, 6.78MB/s]
added_tokens.json: 100%|██████████████████████| 92.0/92.0 [00:00<00:00, 503kB/s]
special_tokens_map.json: 100%|█████████████| 1.04k/1.04k [00:00<00:00, 3.65MB/s]
Traceback (most recent call last):
  File "/Users/brianbustin/projects/smolagents/examples/open_deep_research/run.py", line 33, in <module>
    from smolagents import (
    ...<6 lines>...
    )
ImportError: cannot import name 'MANAGED_AGENT_PROMPT' from 'smolagents' (/Users/brianbustin/projects/smolagents/examples/open_deep_research/.venv/lib/python3.13/site-packages/smolagents/__init__.py)
```

There are two issues here. The first is that ffmpeg is not available. For that, I just add
a line with `pkgs.ffmpeg` to the buildInputs section. The second issue may be related to
[this bug report](https://github.com/huggingface/smolagents/issues/522). It turns out the
smolagents library has changes in the branch we checked out from what is published to the net.
At the time of this writing, this code has been out for only 3 days. We need to install the
version of the library in this branch.

I change the flake.nix to have `pip install -e ../.. -r requirements.txt`. This tells
pip to install smolagents in development mode from the root of the git repository we cloned
as well as the requirements needed for open deep research.

```bash
(.venv) bash-5.2$ python run.py
Chat templates should be in a 'chat_template.jinja' file but found key='chat_template' in the processor's config. Make sure to move your template to its own file.

    _|    _|  _|    _|    _|_|_|    _|_|_|  _|_|_|  _|      _|    _|_|_|      _|_|_|_|    _|_|      _|_|_|  _|_|_|_|
    _|    _|  _|    _|  _|        _|          _|    _|_|    _|  _|            _|        _|    _|  _|        _|
    _|_|_|_|  _|    _|  _|  _|_|  _|  _|_|    _|    _|  _|  _|  _|  _|_|      _|_|_|    _|_|_|_|  _|        _|_|_|
    _|    _|  _|    _|  _|    _|  _|    _|    _|    _|    _|_|  _|    _|      _|        _|    _|  _|        _|
    _|    _|    _|_|      _|_|_|    _|_|_|  _|_|_|  _|      _|    _|_|_|      _|        _|    _|    _|_|_|  _|_|_|_|

Enter your token (input will not be visible): 
```

This is great except why do I need a token to have it log in to Hugging Face? My goal is
to run it completely locally. Time to dig into the code in `run.py` to see what is
happening.

It turns out there is a benchmarking dataset called `gaia-benchmark/GAIA`. I really do not have
any interest in running the benchmark. I look more deeply at the file and it is completely built
around these test datasets. I could probably modify `run.py` to do what I want; however, there
are type Jupyter notebook files I can examine first.

#### Modify the code to run the model locally and use my prompts rather than those from the benchmark

I'm going to use Jupyter while figuring this all out. To do that, I modify the flake.nix file
to install and run Jupyter. To make it a bit better than in my previous post
[Diving into AI](@/posts/2025-02-06_diving_into_ai.md), I made it not run jupyter if 
dependency installation fails. Otherwise, there could be a failure you don't notice 
causing you bang your head on the wall trying to figure
out why nothing is working. Here is the relevant part.

```nix
shellHook = ''
    python -m venv .venv
    source .venv/bin/activate
    pip install jupyter -e ../.. -r requirements.txt \
    && jupyter lab
    echo "exiting open deep research shell"
    exit
'';
```

Now when I run `nix develop` jupyter lab opens. I look at both `analysis.ipynb` and `visual_vs_text_browser.ipynb`
and they also are doing the same thing to a certain degree. I know at this point, that I should really
go back to the start, examine LangChain/LangGraph, smolagents, and then find what other libraries. But,
franky, I find this fun.

The game plan? Create a new Jupyter notebook with snippets from existing code and try to figure out how
it all fits together. In the end, I get it to work running locally.

Here is the flake.nix so far
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
						pkgs.ffmpeg
					];

					shellHook = ''
						python -m venv .venv
						source .venv/bin/activate
						pip install accelerate jupyter -e ../.. -r requirements.txt \
						&& jupyter lab
						echo "exiting open deep research shell"
						exit
					'';
				};
		};
}
```

Here is the python code I have so far. I have this in a jupyter notebook, but you could
just as easily put it in a .py file and run it. (If you do that, you can edit the flake to not launch
Jupyter or exit on its own.)

```python
# imports
import os
from datetime import datetime
from typing import List

from scripts.reformulator import prepare_response
from scripts.run_agents import (
    get_single_file_description,
    get_zip_description,
)
from scripts.text_inspector_tool import TextInspectorTool
from scripts.text_web_browser import (
    ArchiveSearchTool,
    FinderTool,
    FindNextTool,
    PageDownTool,
    PageUpTool,
    SearchInformationTool,
    SimpleTextBrowser,
    VisitTool,
)

from smolagents import (
    MANAGED_AGENT_PROMPT,
    CodeAgent,
    #HfApiModel,
    #LiteLLMModel,
    TransformersModel,
    Model,
    ToolCallingAgent,
)

# defines additional imports CodeAgent can use
AUTHORIZED_IMPORTS = [
    "requests",
    "zipfile",
    "os",
    "pandas",
    "numpy",
    "sympy",
    "json",
    "bs4",
    "pubchempy",
    "xml",
    "yahoo_finance",
    "Bio",
    "sklearn",
    "scipy",
    "pydub",
    "io",
    "PIL",
    "chess",
    "PyPDF2",
    "pptx",
    "torch",
    "datetime",
    "fractions",
    "csv",
]

# this is used in a bunch of places. I will need to figure out what it is for.
custom_role_conversions = {"tool-call": "assistant", "tool-response": "user"}

# create the browser and the agents that will use it
user_agent = "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/119.0.0.0 Safari/537.36 Edg/119.0.0.0"

BROWSER_CONFIG = {
    "viewport_size": 1024 * 5,
    "downloads_folder": "downloads_folder",
    "request_kwargs": {
        "headers": {"User-Agent": user_agent},
        "timeout": 300,
    },
}

os.makedirs(f"./{BROWSER_CONFIG['downloads_folder']}", exist_ok=True)


def create_agent_hierarchy(model: Model):
    text_limit = 100000
    ti_tool = TextInspectorTool(model, text_limit)

    browser = SimpleTextBrowser(**BROWSER_CONFIG)

    WEB_TOOLS = [
        SearchInformationTool(browser),
        VisitTool(browser),
        PageUpTool(browser),
        PageDownTool(browser),
        FinderTool(browser),
        FindNextTool(browser),
        ArchiveSearchTool(browser),
        TextInspectorTool(model, text_limit),
    ]
    text_webbrowser_agent = ToolCallingAgent(
        model=model,
        tools=WEB_TOOLS,
        max_steps=20,
        verbosity_level=2,
        planning_interval=4,
        name="search_agent",
        description="""A team member that will search the internet to answer your question.
    Ask him for all your questions that require browsing the web.
    Provide him as much context as possible, in particular if you need to search on a specific timeframe!
    And don't hesitate to provide him with a complex search task, like finding a difference between two webpages.
    Your request must be a real sentence, not a google search! Like "Find me this information (...)" rather than a few keywords.
    """,
        provide_run_summary=True,
        managed_agent_prompt=MANAGED_AGENT_PROMPT
        + """You can navigate to .txt online files.
    If a non-html page is in another format, especially .pdf or a Youtube video, use tool 'inspect_file_as_text' to inspect it.
    Additionally, if after some searching you find out that you need more information to answer the question, you can use `final_answer` with your request for clarification as argument to request for more information.""",
    )

    manager_agent = CodeAgent(
        model=model,
        #tools=[visualizer, ti_tool],
        tools=[ti_tool],
        max_steps=12,
        verbosity_level=2,
        additional_authorized_imports=AUTHORIZED_IMPORTS,
        planning_interval=4,
        managed_agents=[text_webbrowser_agent],
    )
    return manager_agent
    
# ability to actually answer a question
def answer_single_question(question, model_id):
    model = TransformersModel(model_id)
    
    document_inspection_tool = TextInspectorTool(model, 100000)

    agent = create_agent_hierarchy(model)

    augmented_question = """You have one question to answer. It is paramount that you provide a correct answer.
Give it all you can: I know for a fact that you have access to all the relevant tools to solve it and find the correct answer (the answer does exist). Failure or 'I cannot answer' or 'None found' will not be tolerated, success will be rewarded.
Run verification steps if that's needed, you must make sure you find the correct answer!
Here is the task:
""" + question

    start_time = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    try:
        # Run agent 🚀
        final_result = agent.run(augmented_question)

        agent_memory = agent.write_memory_to_messages(summary_mode=True)

        final_result = prepare_response(augmented_question, agent_memory, reformulation_model=model)

        output = str(final_result)
        for memory_step in agent.memory.steps:
            memory_step.model_input_messages = None
        intermediate_steps = [str(step) for step in agent.memory.steps]

        # Check for parsing errors which indicate the LLM failed to follow the required format
        parsing_error = True if any(["AgentParsingError" in step for step in intermediate_steps]) else False

        # check if iteration limit exceeded
        iteration_limit_exceeded = True if "Agent stopped due to iteration limit or time limit." in output else False
        raised_exception = False

    except Exception as e:
        print("Error on ", augmented_question, e)
        output = None
        intermediate_steps = []
        parsing_error = False
        iteration_limit_exceeded = False
        exception = e
        raised_exception = True
    end_time = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    annotated_example = {
        "agent_name": model.model_id,
        "question": question,
        "augmented_question": augmented_question,
        "prediction": output,
        "intermediate_steps": intermediate_steps,
        "parsing_error": parsing_error,
        "iteration_limit_exceeded": iteration_limit_exceeded,
        "agent_error": str(exception) if raised_exception else None,
        "start_time": start_time,
        "end_time": end_time,
    }
    return annotated_example
    
answer_single_question("How high is the Eiffel Tower?", "Qwen/Qwen2.5-7B-Instruct")
```

This code all comes from `run.py`. I made some changes to run the model locally instead
of calling an API to a cloud provider. I also removed all of the parts that rely on
the GAIA benchmark. My goal here is to be able to ask a question and then have it go out
and research the answer.

There are still two large problems at this point:

1. It is using the CPU, not the GPU. I will need to try to get it to use the MPS
(Metal Performance Shaders), if possible.
2. I am getting this error:
````
Here is the plan of action that I will follow to solve the task:
```
.imgur
user

inspect_file_as_text(file_path="EiffelTowerHeight.txt",
```

Output message of the LLM:
user                                                                                                               
thought: I will use the `inspect_file_as_text` tool to read the text    

Error in code parsing:
Your code snippet is invalid, because the regex pattern ```(?:py|python)?\n(.*?)\n``` was not found in it.
Here is your code snippet:
ynchronized
user
thought: I will use the `inspect_file_as_text` tool to
Make sure to include code with the correct pattern, for instance:
Thoughts: Your thoughts
Code:
```py
# Your python code here
```<end_code>
Make sure to provide correct code blobs.
````
There is no `EiffelTowerHeight.txt` file. `inspect_file_as_text` refers to the 'TextInspectorTool' 
class in `scripts/text_inspector_tool.py`. I think the model just does not understand that the
file is not actually present. Maybe it would work around this if allowed to run long enough,
or I may need to use a different model.

#### Get it running on the GPU

First I write a method that returns the most appropriate device to have pytorch use. 
You'll need to also add `import torch` with the rest of the imports for the code
below to work.

```python
# Check for CUDA and MPS availability
def get_device():
    """
    Determine the available device for computation.
    
    Returns:
        torch.device: The device to use (CUDA, MPS, OpenCL, or CPU).
    """
    if torch.cuda.is_available():
        device = torch.device("cuda")
        print("Using CUDA")
    elif torch.backends.mps.is_available():
        device = torch.device("mps")
        print("Using MPS")
    elif torch.backends.opencl.is_available():  # Example for an additional device (e.g., OpenCL)
        device = torch.device("opencl")
        print("Using OpenCL")
    else:
        device = torch.device("cpu")
        print("Using CPU")
    return device

device = get_device()
```

Then, we need to modify the code creating the smolagent model to pass the device down the
stack to Pytorch. This is easy. In method `answer_single_question` change the first
line to `model = TransformersModel(model_id, device_map=get_device())`.

Success!

#### Fix the error with the text_inspector_tool

The error I am seeing is coming from `src/smolagents/agents.py` in the `step` method of
`MultiStepAgent` when it is calling `fix_final_answer_code(parse_code_blobs(model_output))`.
I notice that the output of the model appears to be getting cut off. Changing the fist line
in `answer_single_question` to 
`model = TransformersModel(model_id, device_map=get_device(), max_new_tokens=8192)` solves
this problem.

Now the problem I am encountering is that to search Google the serpapi is being used
and this requires an API key. My plan is to modify this to use DuckDuckGo search.

#### Change to using Duck Duck Go search

Right now it is using serpapi to use Google search. This needs an API key and
relies on an intermediary between us and the search. I am going to experiment
with addressing this in two ways:

1. Modify the code to just perform a regular DuckDuckGo search
2. Use a Python library that hits the DuckDuckGo search API

##### Perform a regular Duck Duck Go Search

In `scripts/text_web_browser.py` I change line 389 from
`self.browser.visit_page(f"google: {query}", filter_year=filter_year)`

```python
url = f"https://duckduckgo.com/html/?q={query}"
if filter_year:
    url += f"&df={filter_year}-01-01..{filter_year}-12-31"
self.browser.visit_page(url)
```

This is likely not going to be as good as structured output, but these LLMs
often surprise me with their capacity to do unstructured tasks.

I notice that as each step completes, the RAM utilization of the Python process
goes up. There is likely a bug in my code where the model is getting loaded for each step
rather than just staying loaded. I should not be using 70GB for Qwen2.5-7B-Instruct.

It also appears to always get stuck on step 2.

To be continued...