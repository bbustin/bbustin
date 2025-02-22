+++
title = "Foray into Agentic AI"
updated = 2025-02-21

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

## Open Deep Research
OpenAI recently created [OpenAI Deep Research](https://en.wikipedia.org/wiki/OpenAI_Deep_Research)
which can autonomously research your questions by searching the web. Shortly thereafter
the Hugging Face community came out with
[Open Deep Research](https://huggingface.co/blog/open-deep-research) which purports to do the
same thing. Just for funsies, I would like to get Open Deep Research running locally. Then
I would like to ask it to research this area for me.

***Note:* This section is long and meandering. Feel free to [skip to the results](#results)
if a play by play is not your idea of fun.**

### Setup

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
are Jupyter notebook files I can examine first.

#### Modify the code to run the model locally and use my prompts rather than those from the benchmark

I'm going to use Jupyter while figuring this all out. To do that, I modify the flake.nix file
to install and run Jupyter. To make it a bit better than in my previous post
[Diving into AI](@/posts/2025-02-06_diving_into_ai.md), I made it not run jupyter if
dependency installation fails. Otherwise, there could be a failure you don't notice
causing you to bang your head on the wall trying to figure
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
and they also are doing the same thing to a certain degree. I know at this point that I should really
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
I notice that the output of the model appears to be getting cut off. Changing the first line
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

It also significantly slows down on step 2. Maybe because it has so much more context.
It did work in the end though when I asked `How high is the Eiffel Tower?`. When
I asked a more complicated question `Compare and contrast the differences between
the various Python libraries to implement agentic AI`, it ended up running out of
memory before it couls could complete.

```bash
Error in generating model output:
MPS backend out of memory (MPS allocated: 67.09 GB, other allocations: 22.75 GB, max allowed: 81.60 GB). Tried to
allocate 30.25 KB on private pool. Use PYTORCH_MPS_HIGH_WATERMARK_RATIO=0.0 to disable upper limit for memory
allocations (may cause system failure).
```

I may need to use a smaller model on this machine unless I figure out a way to reduce memory utilization.

Additionally, I keep seeing the error:

```bash
Code execution failed at line 'search_agent(query="Popular Python libraries for implementing agentic AI")' due to:
TypeError:MultiStepAgent.__call__() missing 1 required positional argument: 'request'
```

The agent does autonomously figure out it needs to pass `request` into `search_agent` instead of `query`. It
would be nicer if it did not have to have an extra step. This could potnetially be accomplished by either:

##### Fixing the code so the first call to search_agent has the correct parameter

I followed the code and it looks like it is `MultiStepAgent.__call__` that expects the `request`
parameter. `ToolCallingAgent` extends this class, and in my code I am creating a ToolCallingAgent.

```python
...
text_webbrowser_agent = ToolCallingAgent(
    model=model,
    tools=WEB_TOOLS,
...
```

`show_agents_descriptions` in the code refers to there needing to be a parameter called `request`,
which is used by `format_prompt_with_managed_agents_descriptions` which is in turn used to create
a system prompt in `MultiStepAgent.initialize_system_prompt`, which itself is called in `MultiStepAgent.run`
and `CodeAgent.run`. `CodeAgent` has its own `initialize_system_prompt` method, but it calls
`super().initialize_system_prompt` and just replaces `{{authorized_imports}}`.

This looks like a chicken and egg problem. The prompt is not modified until a step runs. The step
runs after the planning stage. Once the first step runs, the system prompt now reflects that the parameter
is `request` and the LLM adjusts accordingly. I may have analyzed this incorrectly, but if I get this right,
then it stands to reason that I need a system prompt earlier in the game to reflect the correct parameter
to pass to `search_agent`.

In my `answer_single_question` function, I modify `augmented_question` to contain `The search_agent tool
requires a parameter called 'request'.` This works. The first call to `search_agent` uses the `request`
parameter. This small change might drastically reduce the amount of steps needed. Let's see if my more
complex question can be completed now.

Step 3 has an issue. I saw this on the previous run, so it is not a result of the most recent small change.

```bash
Error in generating tool call with model:
Tool call '{'id': '94707', 'type': 'function', 'function': {'name': 'visit_page', 'arguments': {'url':
'https://github.com/vectara/py-vectara-agentic'}}}' has an invalid JSON structure: Expecting property name enclosed
in double quotes: line 1 column 2 (char 1)
```

I think, based on the error, the LLM will be able to figure out it needs to use double quotes instead
of single quotes to create valid JSON. Although, I'm kind of with the AI on this one. If it is constructing
Python to run, it seems single quotes would be appropriate.

It really bogs down on step 4. I am almost definitely going to need to use a smaller model for these
kinds of multi-step queries. I stop it and run it with `Qwen/Qwen2-0.5B-Instruct` instead. This
is a *significantly smaller* model. It may be too small, but I want this to run faster so I
can iterate much quicker.

`Qwen/Qwen2-0.5B-Instruct` does not appear to be up to the task.
````Here is the plan of action that I will follow to solve the task:
```
user
<end_plan>

I'm sorry, but I am unable to assist with that
````

Then it moves on to step 1 and sort of gets stuck there. I'm not sure if it would eventually
recover because I stop it pretty quickly. It did not come up with a plan to follow and it does
not look like the Open Depp Seek code likes that. This migth be an edge case.

The second run produced this initial plan and then got stuck:
```Here is the plan of action that I will follow to solve the task:
````
user

Plan: Compare and contrast the differences between the various Python libraries to implement agentic AI

Output: {
  'agent_learning_libraries': ['PyTorch', 'Keras', 'TensorFlow'],
  'ai_rights': ['DeepMind', 'IBM Watson', 'Google DeepMind', 'Microsoft Cognitive Services', 'IBM Watson AI
Engine'],
  'competitive_libraries': ['OpenAI GPT-4', 'GPT-3', 'BERT', 'EleutherAI/MNIST-10B', 'Stanford NLP'],
  'deprecated_libraries': [],
  'free_libraries': ['PyPI', 'Pypi', 'GitHub', 'Bitbucket', 'Docker Hub']
}
```

Please note that the order of the libraries may vary depending on the input.
````

Trying again with `Qwen/Qwen2.5-0.5B-Instruct`. My initial run was with version 2.
This is 2.5 with the same number of parameters. It had always been my intention to use version 2.5
as this is the same model as we used at the beginning with fewer parameters.

This one is not working either. Let's try a qwen2.5 model with more parameters. How
about `Qwen/Qwen2.5-3B-Instruct`? It looks like it works, but for some reason it is
specifying that the searches are to limit the results to 2023. Maybe 2023 is the current year
according to Qwen? Regardless, this means the analysis is going to be dated. It ends up
taking way too long. This seems to happen after the initial DuckDuckGo search.

##### Search DuckDuckGo using a library

I think having more structured search data could make the context smaller, leading to
faster execution. It might also allow the AI to make better use of the search results.
I'll try using the `duckduckgo_search` library.

First, I modify `flake.nix` to install the library. I try hard to set things up in
a way where, if someone else uses this, it is more likely to work right off the
bat. Otherwise I would just run `pip install duckduckgo_search` and be done.

```nix
...
shellHook = ''
    python -m venv .venv
    source .venv/bin/activate
    pip install accelerate duckduckgo_search jupyter -e ../.. -r requirements.txt \
    #&& jupyter lab
    echo "exiting open deep research shell"
    exit
'';
...
```

Since I will be making a lot of changes, I copy `scripts/text_web_browser.py` to
`scripts/text_web_browser_ddg.py`. I then update the import in the main code I've been
working on from `from scripts.text_web_browser import (` to `from scripts.text_web_browser_ddg import (`.

The main changes I make are:

* Remove `_serpapi_search` method
* Remove all references to the `serpapi_key`
* Modify the `set_address` method to check for `ddg:` instead of `google:` and call `self._ddg_search` instead
* Modify the `_split_pages` method to check for `ddg:` instead of `google:`
* Implement the `_ddg_search` method below

```python
def _ddg_search(self, query: str, filter_year: Optional[int] = None) -> None:
    # If filter year provided, convert to DuckDuck Go's format for a date range
    if filter_year:
        timelimit = f"{filter_year}-01-01..{filter_year}-12-31"
    else:
        timelimit = None

    results = DDGS().text(query, backend="lite", timelimit=timelimit)

    content = (
        f"A DuckDuckGo search for '{query}' found {len(results)} results:\n\n## Web Results\n"
        + results
    )

    self._set_page_content(content)
```

`Qwen/Qwen2.5-3B-Instruct` just can not figure out how to convert the descriptions of the
arguments for a tool into the proper Python format. I need to switch back to
`Qwen/Qwen2.5-7B-Instruct` and see if the memory utilization is reduced by using
the DuckDuckGo search library.

```bash
╭─────────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Calling tool: 'web_search' with arguments: {'query': 'Python libraries for agentic AI', 'filter_year': '2022'}  │
╰─────────────────────────────────────────────────────────────────────────────────────────────────────────────────╯

Error in tool call execution: TypeError: can only concatenate str (not "list") to str
You should only use this tool with a correct input.
As a reminder, this tool's description is the following:

- web_search: Perform a web search query (think a google search) and returns the search results.
    Takes inputs: {'query': {'type': 'string', 'description': 'The web search query to perform.'}, 'filter_year':
{'type': 'string', 'description': "[Optional parameter]: filter the search results to only include pages from a
specific year. For example, '2020' will only include pages from 2020. Make sure to use this parameter if you're
trying to search for articles from a specific date!", 'nullable': True}}
    Returns an output of type: string
```

The same problem happens with `Qwen/Qwen2.5-7B-Instruct` and there are some issues
with my `_ddg_search` method. I will get to the bottom of these problems.

Since the model is just calling the tools, I figure I can iterate more quickly on fixing the issue
by creating some code that directly does the same thing. That way I don't have to wait for the model to do it.

```python
import os
from scripts.text_web_browser_ddg import (
    SearchInformationTool,
    SimpleTextBrowser,
)

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

browser = SimpleTextBrowser(**BROWSER_CONFIG)
search_tool = SearchInformationTool(browser)

search_tool(query="Python libraries for agentic AI", filter_year="2022")
```

The code easily replicated the problem.

```bash
File ~/projects/smolagents/examples/open_deep_research/scripts/text_web_browser_ddg.py:213, in SimpleTextBrowser._ddg_search(self, query, filter_year)
    208     timelimit = None
    210 results = DDGS().text(query, backend="lite", timelimit=timelimit)
    212 content = (
--> 213     f"A DuckDuckGo search for '{query}' found {len(results)} results:\n\n## Web Results\n"
    214     + results
    215 )
    217 self._set_page_content(content)

TypeError: can only concatenate str (not "list") to str
```

This error makes complete sense. The results are a list of dictionaries. Since the data is
nicely machine readable, I should return the results to the agent as JSON.
I modify `text_web_browser_ddg.py` to import json. Then I makr the following change:

```python
content = (
    f"A DuckDuckGo search for '{query}' found {len(results)} results in JSON:\n\n## Web Results\n"
    + json.dumps(results)
)
```

Now I'm getting two successful web searches and then it starts failing with the following
message.

```bash
Error in generating tool call with model:
Tool call '{'id': '72834', 'type': 'function', 'function': {'name': 'web_search', 'arguments': {'query': 'popular
Python libraries for agentic AI in AI/ML documentation'}}}' has an invalid JSON structure: Expecting property name
enclosed in double quotes: line 1 column 2 (char 1)
```

I'm definitely closer.

#### Addressing the elphant in the room - memory utilization

This is never going to run successfully on my local machine if the python process ends up chewing
up 90+ GB of virtual memory. I watched a video last night which may have given me a clue
([AI Agent Fundamentals in 21 Minutes - Tina Huang](https://youtu.be/qU3fmidNbJE)). It is likely
that the model is running multiple times. One for the main agent and then other times for the
subagents it is delgating to. Would it be possible to unload a model that is not currently
being used, but retain its context? Let's see.

To figure this out, I need to dig more into the smolagent code. At this point I do not
really have a great picture of what is going on. For all I know, smolagent has an option
an option to do this and I have just failed to turn it on.

`CodeAgent` does not have an option to do what I want...

```python
help(CodeAgent)

class CodeAgent(MultiStepAgent)
 |  CodeAgent(
 |      tools: List[smolagents.tools.Tool],
 |      model: Callable[[List[Dict[str, str]]], smolagents.models.ChatMessage],
 |      system_prompt: Optional[str] = None,
 |      grammar: Optional[Dict[str, str]] = None,
 |      additional_authorized_imports: Optional[List[str]] = None,
 |      planning_interval: Optional[int] = None,
 |      use_e2b_executor: bool = False,
 |      max_print_outputs_length: Optional[int] = None,
 |      **kwargs
 |  )
 |
 |  In this agent, the tool calls will be formulated by the LLM in code format, then parsed and executed.
 |
 |  Args:
 |      tools (`list[Tool]`): [`Tool`]s that the agent can use.
 |      model (`Callable[[list[dict[str, str]]], ChatMessage]`): Model that will generate the agent's actions.
 |      system_prompt (`str`, *optional*): System prompt that will be used to generate the agent's actions.
 |      grammar (`dict[str, str]`, *optional*): Grammar used to parse the LLM output.
 |      additional_authorized_imports (`list[str]`, *optional*): Additional authorized imports for the agent.
 |      planning_interval (`int`, *optional*): Interval at which the agent will run a planning step.
 |      use_e2b_executor (`bool`, default `False`): Whether to use the E2B executor for remote code execution.
 |      max_print_outputs_length (`int`, *optional*): Maximum length of the print outputs.
 |      **kwargs: Additional keyword arguments.
 ```

It extends `MultiStepAgent`. Let's check there.

```python
from smolagents import MultiStepAgent
help(MultiStepAgent)

Help on class MultiStepAgent in module smolagents.agents:

class MultiStepAgent(builtins.object)
 |  MultiStepAgent(
 |      tools: List[smolagents.tools.Tool],
 |      model: Callable[[List[Dict[str, str]]], smolagents.models.ChatMessage],
 |      system_prompt: Optional[str] = None,
 |      tool_description_template: Optional[str] = None,
 |      max_steps: int = 6,
 |      tool_parser: Optional[Callable] = None,
 |      add_base_tools: bool = False,
 |      verbosity_level: int = 1,
 |      grammar: Optional[Dict[str, str]] = None,
 |      managed_agents: Optional[List] = None,
 |      step_callbacks: Optional[List[Callable]] = None,
 |      planning_interval: Optional[int] = None,
 |      name: Optional[str] = None,
 |      description: Optional[str] = None,
 |      managed_agent_prompt: Optional[str] = None,
 |      provide_run_summary: bool = False
 |  )
 |
 |  Agent class that solves the given task step by step, using the ReAct framework:
 |  While the objective is not reached, the agent will perform a cycle of action (given by the LLM) and observation (obtained from the environment).
 |
 |  Args:
 |      tools (`list[Tool]`): [`Tool`]s that the agent can use.
 |      model (`Callable[[list[dict[str, str]]], ChatMessage]`): Model that will generate the agent's actions.
 |      system_prompt (`str`, *optional*): System prompt that will be used to generate the agent's actions.
 |      tool_description_template (`str`, *optional*): Template used to describe the tools in the system prompt.
 |      max_steps (`int`, default `6`): Maximum number of steps the agent can take to solve the task.
 |      tool_parser (`Callable`, *optional*): Function used to parse the tool calls from the LLM output.
 |      add_base_tools (`bool`, default `False`): Whether to add the base tools to the agent's tools.
 |      verbosity_level (`int`, default `1`): Level of verbosity of the agent's logs.
 |      grammar (`dict[str, str]`, *optional*): Grammar used to parse the LLM output.
 |      managed_agents (`list`, *optional*): Managed agents that the agent can call.
 |      step_callbacks (`list[Callable]`, *optional*): Callbacks that will be called at each step.
 |      planning_interval (`int`, *optional*): Interval at which the agent will run a planning step.
 |      name (`str`, *optional*): Necessary for a managed agent only - the name by which this agent can be called.
 |      description (`str`, *optional*): Necessary for a managed agent only - the description of this agent.
 |      managed_agent_prompt (`str`, *optional*): Custom prompt for the managed agent. Defaults to None.
 |      provide_run_summary (`bool`, *optional*): Wether to provide a run summary when called as a managed agent.
```

That is not what I need. We essentially have a `CodeAgent` that has a managed_agent which is
a `ToolCallingAgent` with access to various tools. Maybe what I seek is in `ToolCallingAgent`?

```python
help(ToolCallingAgent)

class ToolCallingAgent(MultiStepAgent)
 |  ToolCallingAgent(
 |      tools: List[smolagents.tools.Tool],
 |      model: Callable[[List[Dict[str, str]]], smolagents.models.ChatMessage],
 |      system_prompt: Optional[str] = None,
 |      planning_interval: Optional[int] = None,
 |      **kwargs
 |  )
 |
 |  This agent uses JSON-like tool calls, using method `model.get_tool_call` to leverage the LLM engine's tool calling capabilities.
 |
 |  Args:
 |      tools (`list[Tool]`): [`Tool`]s that the agent can use.
 |      model (`Callable[[list[dict[str, str]]], ChatMessage]`): Model that will generate the agent's actions.
 |      system_prompt (`str`, *optional*): System prompt that will be used to generate the agent's actions.
 |      planning_interval (`int`, *optional*): Interval at which the agent will run a planning step.
 |      **kwargs: Additional keyword arguments
```

That is not helping much either. I am going to really need to dig down and see what is
happening. First, I find some places where I was creating duplicate objects in
`create_agent_hierarchy`. I also modify `flake.nix` to install `memory_profiler`
and annotate the `answer_single_question` with `@profile`.

While that runs, I am going to look more at how `MultiStepAgent` works since this seems
to be the parent class for the agents we are using.

I switch to using Ollama directly instead of the Transformers library. Memory
utilization drops like a rock. It is now using around 6GB instead of 90+ GB.
It is also running significantly faster.

Memory problem solved!

What I did:

* Modify flake.nix to install `litellm`
* Edit the imports to import `LiteLLMModel` from `smolagents`
* In `answer_single_question`, change the definition of the model to
`model = LiteLLMModel(model_id, api_base="http://127.0.0.1:11434", num_ctx=99999)`
* Change the model name som it is preceeded by `ollama/` like this
`answer_single_question("Compare and contrast the differences between the various Python libraries to implement agentic AI", "ollama/qwen2.5:7b")`

### Open Deep Research's Answers {#results}

I am going to have Open Deep Research use multiple models to answer the following question:

> What libraries are available to implement agentic AI? What are their key differences?

#### Using qwen2.5:7b

Here is Qwen2.5:7b's answer:

---

Thought: I have gathered information about libraries used for implementing agentic AI and their key differences.
Now, I need to summarize the findings as requested:

1. **Task outcome (short version):**
   Several popular libraries are commonly used for implementing agentic AI, including CAMEL, Lyzr,
Astra-Assistants-API, LangGraph, AutoGen, and CrewAI. These frameworks provide tools for developing autonomous
agents capable of performing complex tasks.

2. **Task outcome (extremely detailed version):**
   - **CAMEL Framework (αlpha)**
     - Description: Open-source library designed for studying autonomous and communicative agents.
     - Documentation: Available at https://github.com/slavakurilyak/awesome-ai-agents
     - Use Cases: Suitable for researchers aiming to understand the behaviors, capabilities, and potential risks of
autonomous systems.

   - **Lyzr Framework**
     - Description: Enterprise-grade AI agent framework designed for easy configuration and deployment.
     - Documentation: Available at https://github.com/juliustago/ai-agents..2
     - Use Cases: Ideal for businesses requiring robust and scalable AI agents in enterprise environments.

   - **Astra-Assistants-API**
     - Description: Backend implementation of the OpenAI Assistants API, supporting features like persistent
threads, files, assistants, and streaming functionalities.
     - Documentation: Details available at https://github.com/slavakurilyak/awesome-ai-agents
     - Use Cases: Useful for integrating advanced AI capabilities into applications that require persistent
communication and dynamic data handling.

   - **LangGraph Framework**
     - Description: One of the top 3 trending agentic AI frameworks, focusing on multi-agent orchestration.
     - Documentation: Comprehensive comparison available at
https://www.datagrom.com/data-science-machine-learning-ai-blog/langgraph-vs-autogen-vs-crewai-comparison-agentic-ai
-frameworks
     - Use Cases: Suitable for developers building large-scale agentic AI systems with a focus on multi-agent
coordination and efficient resource management.

   - **AutoGen Framework**
     - Description: Another prominent agentic AI framework, designed to facilitate the development of autonomous
agents that can handle complex tasks through advanced planning and decision-making mechanisms.
     - Documentation: Available at
https://www.datagrom.com/data-science-machine-learning-ai-blog/langgraph-vs-autogen-vs-crewai-comparison-agentic-ai
-frameworks
     - Use Cases: Ideal for projects requiring highly autonomous agents that can operate effectively in dynamic
environments with minimal human intervention.

   - **CrewAI Framework**
     - Description: The third of the top 3 trending agentic AI frameworks, known for its comprehensive approach to
building and managing multi-agent systems.
     - Documentation: Details available at
https://www.datagrom.com/data-science-machine-learning-ai-blog/langgraph-vs-autogen-vs-crewai-comparison-agentic-ai
-frameworks
     - Use Cases: Suitable for developers working on large-scale projects that require the coordination of multiple
autonomous agents to achieve complex objectives.

3. **Additional context (if relevant):**
   The above frameworks and libraries are part of a broader ecosystem of tools designed for implementing agentic
AI, including Langchain and AI Fairness 360 by IBM, which can be used to ensure ethical development and responsible
deployment of these systems.
   For beginners or those looking for a simpler approach, resources like the GitHub repositories mentioned provide
extensive documentation and practical examples to get started with developing your own agentic AI agents.

---

This is not bad. The information is useful. The documentation links are not great though. At least
they are not the official documentation for each of those libraries.

### Using llama3.2

Let's see what the response looks like if I use llama3.2. It seems to have a lot of issues following the instructions.
It creates code that is not allowed and then I get errors like this:

```bash
Code execution failed at line 'output = web_search(request=request)' due to: InterpreterError:It is not permitted
to evaluate other functions than the provided tools or functions defined/imported in previous code (tried to
execute web_search).
```

I gave up after a little while.

#### Using deepseek-r1:7b

I try Deepseek-r1 and it has even more issues trying to figure out what to do. It
essentially keeps returning the same answer which is not Python code. Deepseek-r1:7b
is one of the distillation models. In this case, Qwen-7b was "re-trained" by
Deepseek.

Unfortunately, this one completely can not handle the task at hand.

#### Using phi4

I try Phi 4 and this goes much better than llama3.2 or deepseek-r1:4b. Its
research approach appears sound and it is properly formatting python code
to call the tools.

To be completely fair, this is a 14 billion parameter model. All the others were
7 billion. If this goes well, I will also try it with qwen2.5:14b. Just so no
one looks at this and says Phi4 is so much better than Qwen when it was an
unfair comparison.

Here is Phi4's answer:

---

To provide an accurate and comprehensive response to the question regarding libraries available for implementing agentic AI and their key differences, a detailed analysis has been conducted. Here is a concise summary of the findings:

### Key Libraries for Agentic AI

1. **Microsoft AutoGen**
   - **Description**: An open-source framework designed to build and coordinate multi-agent conversational systems.
   - **Key Features**:
     - Supports dynamic collaboration among agents, allowing roles adjustment based on real-time demands.
     - Facilitates modular architecture for maintainable processes.
   - **Use Cases**: Best suited for applications that require a flexible and maintainable structure.

2. **OpenAI Swarm**
   - **Description**: An experimental multi-agent framework by OpenAI focusing on team-based task delegation among agents.
   - **Key Features**:
     - Implements a multi-agent design pattern for efficient task delegation.
     - Promotes modularity, scalability, and maintainability in AI systems.
   - **Use Cases**: Ideal for educational purposes and projects needing fluid collaboration among agents.

### Comparative Analysis

| Feature                  | Microsoft AutoGen                                                                 | OpenAI Swarm                                                                                  |
|--------------------------|-----------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------|
| **Design Focus**         | Multi-agent conversational systems.                                               | Team-based multi-agent orchestration framework.                                                |
| **Dynamic Collaboration**| Allows agents to adjust roles based on real-time task demands.                    | Facilitates task delegation among agents using shared message contexts.                        |
| **Modularity & Scalability**| Emphasizes modular architecture for maintainable processes.                     | Promotes modularity, scalability, and maintainability in AI systems.                           |
| **Use Cases**            | Applications requiring flexible and maintainable structure.                       | Educational examples of multi-agent systems; projects needing fluid collaboration.             |
| **Community Support**    | Active GitHub repository; extensive documentation.                                | Experimental framework with growing community support.                                         |

### Recommendations

- **Microsoft AutoGen**: Recommended for developing complex, flexible AI-driven conversational systems that require dynamic agent interactions.
- **OpenAI Swarm**: Suitable for educational purposes and scenarios where task delegation among agents in a collaborative environment is essential.

### Final Answer

**FINAL ANSWER:**

1. Microsoft AutoGen:
   - Open-source framework
   - Multi-agent conversational systems
   - Dynamic collaboration
   - Modular architecture

2. OpenAI Swarm:
   - Experimental multi-agent framework
   - Team-based task delegation
   - Modularity and scalability
   - Educational use cases

---

It is not lost on me that Microsoft's model recommended a library created by Microsoft.

This output is pretty good, but it really only dug into two libraries. My preference
so far is for qwen2.5:7b. What can qwen2.5:14b do?

### Using qwen2.5:14b

Let's see how the 14 billion parameter Qwen2.5 model stacks up to the 7 billion
parameter version I used earlier.

Here is Qwen2.5:14b's answer.

---

Based on the information gathered by the search agent, I have compiled a list of popular Python libraries used for
agentic AI along with their key features and functionalities.

### Task Outcome (Short Version):
Listed several popular Python libraries used for agentic AI.

### Task Outcome (Detailed Version):

1. **TensorFlow**:
   - Introduction: Developed by Google for numerical computation and machine learning.
   - Features: Supports deep learning, reinforcement learning, generative models.
     Wide-ranging applications from natural language processing to computer vision.
   - Functionality: Offers extensive documentation and a large community of developers.
   - Use Cases: Applicable in industries ranging from finance to healthcare.

2. **LangChain (Langchain)**:
   - Introduction: An open-source framework simplifying LLM application development.
   - Features: Integration with OpenAI API and Hugging Face Transformers.
     Supports memory, retrieval, and agent-based systems.
   - Functionality: Enables seamless creation of applications powered by large language models.
   - Use Cases: Commonly used in chatbot development and customer service.

3. **Phidata**:
   - Introduction: A lightweight framework for building multi-modal agents from LLMs.
   - Features: Conversion of LLMs into specialized AI assistants.
     Support for APIs, databases, and other tools.
   - Functionality: Facilitates the creation of intelligent assistants using large language models.
   - Use Cases: Applicable in financial analysis and personalized assistance.

4. **AutoGen**:
   - Introduction: A framework providing a no-code GUI for multi-agent application development.
   - Features: Autonomous workflows with human-in-the-loop support.
     Supports agents using LLMs and tools.
   - Functionality: Facilitates the creation of complex, collaborative workflows in agentic systems.
   - Use Cases: Used for multi-agent collaboration in research and development.

5. **CAMEL (Communicative Agents for Mind Exploration of Large Language Model Society)**:
   - Introduction: A library designed for studying autonomous and communicative agents through scalable techniques.
   - Features: Facilitates role-playing to understand agent behaviors.
     Provides extensive documentation.
   - Functionality: Enables research into the behavior, capabilities, and potential risks of large language
model-driven agents.
   - Use Cases: Used in academic research and advanced AI exploration.

6. **OpenAI Swarm**:
   - Introduction: A lightweight multi-agent orchestration framework released by OpenAI for experimental use.
   - Features: Supports development of agentic systems that handle complex tasks.
     Facilitates collaborative task handling.
   - Functionality: Enables experimental projects in the field of agentic AI and multi-agent coordination.
   - Use Cases: Used in academic research and advanced AI experimentation.

7. **Vectara Agentic**:
   - Introduction: A library enabling creation of specialized tasks-handling AI agents with financial inquiries or
other specific domains.
   - Features: Custom instructions for agent operation.
     Tool integration support.
   - Functionality: Facilitates the development and deployment of custom-tailored agentic systems.
   - Use Cases: Applicable in specialized sectors requiring targeted intelligent assistance.

### Additional Context:
- Documentation Quality: High-quality documentation is available for TensorFlow, LangChain, Phidata, AutoGen,
CAMEL, and Vectara Agentic, with active communities providing support.
- Community Support: Each library benefits from a vibrant community of developers contributing to ongoing
improvements and innovative use cases.
- Common Applications: Applications range from chatbot development and customer service to advanced AI research in
multi-agent collaboration.

---

Not bad. I do not really think TensorFlow can be considered a library for implementing agentic AI.
I guess maybe at a very low level, but what it is implementing is a way to interact with the model,
not really the creation of AI agents.

I feel Qwen2.5:14b delivered the best output. It did not provide incorrect documentation links.
This opinion is unscientific, subjective, and based on a very small sample size. Take it with a
grain of salt.
