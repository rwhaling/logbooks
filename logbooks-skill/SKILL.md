---
name: logbooks
description: Apply principles of notebook computing and literate computing to data-intensive operations.  Create logbooks (templatized markdown files with code snippets), and then execute logbooks (creating a copy of the template, filling out the results of each step)
---
If you are reading this skill, logbooks are enabled on this repo.  First, locate the logbooks/ directory, which should be at the root of the repo.  It has two subdirectories:

1) logbooks/templates/ contains files with named templates for repeated tasks, called TEMPLATE_NAME.md, each one describes a logical workflow and is referred to by name.  It is conventional for a template file to begin with a description of the overall functionality, followed immediately by an Instructions and Parameters section containing both formal parameters and looser instructions that may be supplied at invocation time.

2) logbooks/runs/ contains the results of running a logbook, which is a copy of the template with more details filled out, and the results of each executable step appended immediately after code snippets or other placeholders.  If you run a code-snippet as is, you do not need to include it, but if you modify or design original code, please include it as well as the output. Naming convention is "logbooks/runs/YYYY-MM-DD/UUID.md" with a random uuid, please use python to generate a random uuid properly.

Because accurate skill invocation is critical, any time you begin the process of either 1) creating a logbook, or 2) executing a logbook, please clearly begin your response indicating that you are "Creating a logbook according to the skill in logbooks-skill/SKILL.md" etc.

Before proceeding, please examine the examples in this directory, EXAMPLE_TEMPLATE.md, and EXAMPLE_RUN_1.md, and EXAMPLE_RUN_2.md, to see how logbooks are typically structured and executed.

A few notes/hints:
- if a user's request is vague or does not provide complete parameters to invoke the workflow properly, you can ask the user more questions, and if there is any ambiguity at all, presenting the user with the formal parameters and asking for their approval before beginning is required.  

- the metadata in the frontmatter is important for runs.  The metadata for a logbook template look like this:
~~~
---
parameters:
  name: ...
---
~~~
But when executed, look like this:

~~~
---
parameters:
  name: Richard
uuid: "b24d148e-2b68-4b4c-b581-2a0f2925b9da"
user_request: "Please execute the EXAMPLE_TEMPLATE logbook with NAME: Richard"
executed_at: "2026-02-14T11:54:00-06:00"
---
~~~
- it is generally a good idea to execute and write out a notebook run one step at a time, rather than doing it all at once.

- it's not the most efficient or effective to use code for everything.  for example, for data generation, if you are writing the result of a previous step to a file, you can just write the file yourself.

- you are permitted to create temporary files as you wish in ./logbooks/run/YYYY-MM-DD/UUID.scratch/

- for easily parallelizable tasks that would be time-consuming, you can ask the user if you should run them in parallel, and suggest a number of sub-agents.  Do this for sub-agent parallelism, but not for python/subprocess parallelism, since you do not have advanced subprocess management capabilities yet
