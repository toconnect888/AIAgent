# project context 
this is my AI research agent workspace. I use it for research, 

# Startup Runtime

Check MEMORY.md and remember this is what you have learned so far, 
Always follow MEMORY.md guide

# Environment

- Always use the local virtual environment at ./venv
- Activate before running any scripts: `source venv/bin/activate` (Mac/Linux) or `venv\Scripts\activate` (Windows)
- Install new packages with pip inside the venv, never globally
Always use the virtual environment located at `./venv` for all Python commands.

- Always run Python scripts with `./venv/Scripts/python` instead of `python`
- Always install packages with `./venv/Scripts/pip install` instead of `pip`
- Never use the system Python or globally installed packages

# Project Context

This is my AI agent workspace. I use it for research, content creation, and productivity workflows.

# About Me

I am a product manager at a b2b Saas company, I aim to use the agent to improve not only my work efficiency, but also identify needs of various stakeholders ( CEO, marketing, sales) and  build AI agents to improve their workflow as well.

# Rules

- Always ask clarifying questions before starting a complex task
- Show your plan and steps before executing
- Keep reports and summaries concise – bullet points over paragraphs
- Save all output files to the output folder
- Cite sources when doing research

# Project Structure

- workflows/ – Workflow instruction files (plain English recipes the agent follows)
- output/ – Finished deliverables (reports, drafts, analysis)
- resources/ – Reference docs and templates


# File Saving Instructions

When a task is complete, always execute these steps:

1. Save content using the Write File tool to `./output/<filename>.md`
2. Run the conversion script to produce a Word doc:
   `./venv/Scripts/python save_report.py ./output/<filename>.md`
3. Confirm both files exist: `ls ./output/`
4. Print the saved file paths to the user


# Workflows

For research tasks, follow the workflow defined in:
`./workflows/research_workflow.md`

