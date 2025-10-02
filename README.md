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

# Gemini

Large Language Models (LLMs) are the fancy-schmancy AI technology that have been making all the waves in the AI world recently. Products like:

ChatGPT
Claude
Cursor
Google Gemini
... are all powered by LLMs. For the purposes of this course, you can think of an LLM as a smart text generator. It works just like ChatGPT: you give it a prompt, and it gives you back some text that it believes answers your prompt. We're going to use Google's Gemini API to power our agent in this course. It's reasonably smart, but more importantly for us, it has a free tier.

Tokens
You can think of tokens as the currency of LLMs. They are the way that LLMs measure how much text they have to process. Tokens are roughly 4 characters for most models. It's important when working with LLM APIs to understand how many tokens you're using.

We'll be staying well within the free tier limits of the Gemini API, but we'll still monitor our token usage!

Be aware that all API calls, including those made during local testing, consume tokens from your free tier quota. If you exhaust your quota, you may need to wait for it to reset (typically 24 hours) to continue the lesson. Regenerating your API key will not reset your quota.
Assignment
Create an account on Google AI Studio if you don't already have one
Click the "Create API Key" button. Here are the docs if you get lost.
If you already have a GCP account and a project, you can create the API key in that project. If you don't, AI studio will automatically create one for you.
Copy the API key, then paste it into a new .env file in your project directory. The file should look like this:
GEMINI_API_KEY="your_api_key_here"

Add the .env file to your .gitignore
We never want to commit API keys, passwords, or other sensitive information to git.
Update the main.py file. When the program starts, load the environment variables from the .env file using the dotenv library and read the API key:
import os
from dotenv import load_dotenv

load_dotenv()
api_key = os.environ.get("GEMINI_API_KEY")

Import the genai library and use the API key to create a new instance of a Gemini client:
from google import genai

client = genai.Client(api_key=api_key)

Use the client.models.generate_content() method to get a response from the gemini-2.0-flash-001 model! You'll need to use two named parameters:
model: The model name: gemini-2.0-flash-001 (this one has a generous free tier)
contents: The prompt to send to the model (a string). For now, hardcode this prompt:
"Why is Boot.dev such a great place to learn backend development? Use one paragraph maximum."
The generate_content method returns a GenerateContentResponse object. Print the .text property of the response to see the model's answer.

If everything is working as intended, you should be able to run your code and see the model's response in your terminal!

In addition to printing the text response, print the number of tokens consumed by the interaction in this format:
Prompt tokens: X
Response tokens: Y

The response has a .usage_metadata property that has both:

a prompt_token_count property (tokens in the prompt)
a candidates_token_count property (tokens in the response)
Run and submit the CLI tests.

The Gemini API is an external web service and on occasion it's slow and unreliable. It's possible in this course for you to lose armor because of an API outage on Google's end... just be sure to always run before submitting to minimize the risk of that happening.

