# 2/1-/2025 
# Python setup

Python Setup
Let's set up a virtual environment for our project.

Virtual environments are Python's way to keep dependencies (e.g. the Google AI libraries we're going to use) separate from other projects on our machine.

Assignment
Use uv to create a new project. It will create the directory and also initialize git.
uv init your-project-name
cd your-project-name

Create a virtual environment at the top level of your project directory:
uv venv

Always add the venv directory to your .gitignore file.
Activate the virtual environment:
source .venv/bin/activate

You should see (your-project-name) at the beginning of your terminal prompt, for example, mine is:

(aiagent) wagslane@MacBook-Pro-2 aiagent %

Always make sure that your virtual environment is activated when running the code or using the Boot.dev CLI.
Use uv to add two dependencies to the project. They will be added to the file pyproject.toml:
uv add google-genai==1.12.1
uv add python-dotenv==1.1.0

This tells Python that this project requires google-genai version 1.12.1 and the python-dotenv version 1.1.0.

To run the project using the uv virtual environment, you use:

uv run main.py

In your terminal, you should see Hello from YOUR PROJECT NAME
