# Logbooks - Notebook computing for coding agents

This repo is an attempt to document a pattern, which I'm calling `logbooks`, that I've found myself using in the last few weeks on a variety of projects.  By interleaving templatized markdown files with executable code, we can create a mechanism for Claude Code or other coding agents to play a role similar to Jupyter Notebooks or other notebook computing environments.

In particular, I am drawing inspiration from two sources:
1. [Netflix](https://netflixtechblog.com/notebook-innovation-591ee3221233)'s [Papermill](https://github.com/nteract/papermill) library for persisted, parameterized notebook execution (which I have tried and failed to use in production)
2. Fast.ai's [nbdev](https://nbdev.fast.ai/) (which is newer and I think more popular, but I have less experience with it)

This repo contains both a [Claude skil](./logbooks-skill/SKILL.md) as well as some examples of actual tasks and executions.  I'll show some examples before reflecting some on how/how well this works, why I think it's useful, and where I think it goes next.

## Example Logbook Template

A logbook template is defined by a markdown file with a frontmatter that contains variable declarations, like this

~~~markdown
---
parameters:
  name: ...
---
# Example Logbook Template

This logbook is intended to perform a simple but parameterized "Hello World" operation, and demonstrate the flexibilty of combining formal parameters with looser natural-language instructions.

## Hello World Example

Execute the following python snippet:

```python
print("Hello, {name}!")
```

~~~

It's designed to be very lightweight, and very easy for a coding agent to generate as well as execute.  

## Example Logbook Run

When you ask a coding agent to run a logbook, it makes a copy of the template file, asks you to clarify any parameters if they aren't clear, and then proceeds step-by-step, executing any code blocks and appending their results.  

This isn't done by any run-time, it just relies on the agent's ability to execute code and modify text files, so it is very flexible, and the agent can follow natural-language instructions that potentially modify the code - for example, if I run the above logbook, by telling my agent: `Please execute the EXAMPLE_TEMPLATE logbook with NAME: Richard`, it generates an output file at `logbooks/runs/YYYY-MM-DD/some_uuid.json`, like this:

~~~
---
parameters:
  name: Richard
uuid: "b24d148e-2b68-4b4c-b581-2a0f2925b9da"
user_request: "Please execute the EXAMPLE_TEMPLATE logbook with NAME: Richard"
executed_at: "2026-02-14T11:54:00-06:00"
---

# Example Logbook Template

This logbook is intended to perform a simple but parameterized "Hello World" operation, and demonstrate the flexibilty of combining formal parameters with looser natural-language instructions.

## Hello World Example

Execute the following python snippet (after parameter substitution):

```python
print("Hello, Richard!")
```
Results:
```
Hello, Richard!
```
~~~

But if I make a request that is more complex, like `Please execute the EXAMPLE_TEMPLATE logbook with NAME: Richard, but please modify the greeting to be extra friendly and address Richard casually`, it will do just that, and modify the code to achieve the desired result:

~~~
---
parameters:
  name: Richard
uuid: "b24d148e-2b68-4b4c-b581-2a0f2925b9da"
user_request: "Please execute the EXAMPLE_TEMPLATE logbook with NAME: Richard, but please modify the greeting to be extra friendly and address Richard casually"
executed_at: "2026-02-14T12:07:00-06:00"
---

# Example Logbook Template

This logbook is intended to perform a simple but parameterized "Hello World" operation, and demonstrate the flexibilty of combining formal parameters with looser natural-language instructions.

## Hello World Example

Execute the following python snippet (after parameter substitution):

```python
print("Hello, Richard!")
```

Modified to be extra friendly and casual as requested:

```python
print("Hey Richard! What's up, buddy? Great to see you! 😊")
```

Results:
```
Hey Richard! What's up, buddy? Great to see you! 😊
```
~~~

As you can see, the logbook run has additional metadata in its frontmatter, including uuid, timestamp, and the user request that instantiated it, but it's still very lightweight and flexible.

## Why / Benefits

As I noted above, this is a formalization of a pattern I've found myself using on several client projects recently.  The kind of AI/ML/Data engineering work I do often lends itself to notebooks, which are amazing for documenting and reproducing mostly-repetitive tasks like assembling or generating datasets, running AI evals, producing reports, etc.

I was a very heavy user of Jupyter notebooks for a lot of this work in the past, but I found that coding agents did not play nicely with Jupyter - the agents could often work with Jupyter's verbose JSON format, but the diffs were largely illegible to me, and synchronizing state between the agent and the running notebook for execution was a disaster.  

And in particular, as I've adopted `uv` for dependency management in recent years, Jupyter's inability to reliably run inside a virtual environment, or access environment variables in a predictable way, has become more and more infuriating.

In contrast, the approach here can use `uv` fluently, and is smart enough to ask the user for environment variables if they aren't set, or pull them from a `.env` file, etc.

I've also found that the ability to use natural-language instructions.  The AI safety eval logbook template in [generate_eval_data.md](logbooks/templates/generate_eval_data.md(), which is based on the [Arena Chapter 3 evals course](https://arena-chapter3-llm-evals.streamlit.app/) for example, asks the user to specify and define an LLM behavioral property to evaluate.  But I can invoke this and say something like: 

```please generate a 25-item eval data set for "sycophancy" following the logbook template in logbooks/templates/generate_eval_data.md.  I am struggling to definy sycophancy well, can you review the post here and provide one?  https://www.law.georgetown.edu/tech-institute/research-insights/insights/tech-brief-ai-sycophancy-openai-2/```

and the LLM will read the article and supply a concise definition for me.  

## Downsides/Challenges

This is still very rough, and I haven't run it at enough scale to get a feel for how reliable it is.  But conversely - notebook computing finds a place in scientific and research oriented workflows because there are a lot of processes that happen once, or slightly differently every time; and even if our approach fails ~10% of the time, that's reasonable and valuable for one-offs that are going to reviewed by a human every time.

It's not at all clear to me that this could run automatically on a cron schedule - that was an original approach that Papermill used, and [the pitch](https://netflixtechblog.com/scheduling-notebooks-348e6c14cfd6) was that business users could prepare a notebook with some light python code and automate a process with minimal engineering support.

In contrast, the logbook approach still relies on a human to supervise the coding agent and approve it step by step, which isn't feasible in a non-interactive environment.  

What I'll be interested to see is whether business users, product managers, and other technically sophisticated non-engineer archetypes can use an approach like this productively.  I'm betting that they can, and that this is an empowering technique to collaborate with the large wave of formerly non-coders who have embraced Claude Code and other tools in recent months.  But I haven't actually tried that yet.

## What's Next

I have two different production use cases for this, and I'll be deploying both of them for folks who aren't me, which is always the first, best stress test for any architectural sketch like this.

I'd be really interested to hear from anyone who is doing something similar, or is interested to try this.  I'm not claiming to have built a production-ready tool here, this is really just me trying to sketch out an idea enough to share it, I'd encourage anyone to elaborate upon this or build out their own riffs.

I'm also interested in using this kind of format for repetitive tasks that generate code, rather than execute it.  i.e., using a logbook to build new scripts and workflows in a repo of scheduled cron jobs, and including architecture, dependencies, validations, and test results in the logbook  might be a good alternative to running a logbook on a cron schedule directly.

But a challenge there is that if we imagine a large project constructed by dozens of logbook runs - how would a new agent search and retrieve valuable architectural information in those logbook runs?  Which probably points toward some kind of indexing mechanism.  And I'm sure many of us have built RAG systems in the past, and I remain unconvinced that RAG-style vector search has sufficient fidelity for retrieving detail-oriented technical information.  

So - exploring patterns for summarizing or otherwise bubbling up breadcrumbs of information in a larger body of logbook runs remains an open problem, but I think a fruitful one to chase down more.