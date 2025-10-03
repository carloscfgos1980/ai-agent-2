# 2/1-/2025 
# LLMs. Python setup

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

# LLMs. Gemini

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

# LLMs. Input

We've hardcoded the prompt that goes to gemini, which is... not very useful. Let's update our code to accept the prompt as a command line argument.

We don't want our users to have to edit the code to change the prompt!

Assignment
Update your code to accept a command line argument for the prompt. For example:
uv run main.py "Why are episodes 7-9 so much worse than 1-6?"

The sys.argv variable is a list of strings representing all the command line arguments passed to the script. The first element is the name of the script, and the rest are the arguments. Be sure to import sys to use it.
If the prompt is not provided, print an error message and exit the program with exit code 1.

# LLMs. Messages

LLM APIs aren't typically used in a "one-shot" manner, for example:

Prompt: "What is the meaning of life?"
Response: "42"
They work the same way ChatGPT works: in a conversation. The conversation has a history, and if we keep track of that history, then with each new prompt, the model can see the entire conversation and respond within the larger context of the conversation.

Roles
Importantly, each message in the conversation has a "role". In the context of a chat app like ChatGPT, your conversations would look like this:

user: "What is the meaning of life?"
model: "42"
user: "Wait, what did you just say?"
model: "42. It's is the answer to the ultimate question of life, the universe, and everything."
user: "But why?"
model: "Because Douglas Adams said so."
So, while our program will still be "one-shot" for now, let's update our code to store a list of messages in the conversation, and pass in the "role" appropriately.

Assignment
Create a new list of types.Content, and set the user's prompt as the only message (for now):
from google.genai import types

messages = [
    types.Content(role="user", parts=[types.Part(text=user_prompt)]),
]

Update your call to models.generate_content to use the messages list:
response = client.models.generate_content(
    model="gemini-2.0-flash-001",
    contents=messages,
)

In the future, we'll add more messages to the list as the agent does its tasks in a loop.

# LLMs. Verbose

As you debug and build your AI agent, you'll probably want to dump a lot more context into the console... but at the same time, we don't want to make the user experience of our CLI tool too noisy...

Let's add an optional command line flag, --verbose, that will allow us to toggle "verbose" output on and off. When we want to see more info, we'll just turn that bad boy on.

Assignment
Add a new command line argument, --verbose. It should be supplied after the prompt if included. For example:
uv run main.py "What is the meaning of life?" --verbose

If the --verbose flag is included, the console output should include:
The user's prompt: "User prompt: {user_prompt}"
The number of prompt tokens on each iteration: "Prompt tokens: {prompt_tokens}"
The number of response tokens on each iteration: "Response tokens: {response_tokens}"
Otherwise, it should not print those things.

**changes**
    verbose = "--verbose" in sys.argv
    args = []
    for arg in sys.argv[1:]:
        if not arg.startswith("--"):
            args.append(arg)

        print('\nUsage: python main.py "your prompt here" [--verbose]')

    if verbose:
        print(f"User prompt: {user_prompt}\n")

        generate_content(client, messages, verbose)

        def generate_content(client, messages, verbose):

            if verbose:
            print("Prompt tokens:", response.usage_metadata.prompt_token_count)
            print("Response tokens:", response.usage_metadata.candidates_token_count)

# Functions. Calculator

So obviously we're building an AI Agent, but an AI agent needs a project to work on. I've built a little command line calculator app that we'll use as a test project for the AI to read, update, and run.

Assignment
Create a new directory called calculator in the root of your project.
Copy and paste the main.py and tests.py files from below into the calculator directory.

<main.py>

import sys
from pkg.calculator import Calculator
from pkg.render import format_json_output

def main():
    calculator = Calculator()
    if len(sys.argv) <= 1:
        print("Calculator App")
        print('Usage: python main.py "<expression>"')
        print('Example: python main.py "3 + 5"')
        return

    expression = " ".join(sys.argv[1:])
    try:
        result = calculator.evaluate(expression)
        if result is not None:
            to_print = format_json_output(expression, result)
            print(to_print)
        else:
            print("Error: Expression is empty or contains only whitespace.")
    except Exception as e:
        print(f"Error: {e}")

if **name** == "**main**":
    main()

<tests.py>

import unittest
from pkg.calculator import Calculator

class TestCalculator(unittest.TestCase):
    def setUp(self):
        self.calculator = Calculator()

    def test_addition(self):
        result = self.calculator.evaluate("3 + 5")
        self.assertEqual(result, 8)

    def test_subtraction(self):
        result = self.calculator.evaluate("10 - 4")
        self.assertEqual(result, 6)

    def test_multiplication(self):
        result = self.calculator.evaluate("3 * 4")
        self.assertEqual(result, 12)

    def test_division(self):
        result = self.calculator.evaluate("10 / 2")
        self.assertEqual(result, 5)

    def test_nested_expression(self):
        result = self.calculator.evaluate("3 * 4 + 5")
        self.assertEqual(result, 17)

    def test_complex_expression(self):
        result = self.calculator.evaluate("2 * 3 - 8 / 2 + 5")
        self.assertEqual(result, 7)

    def test_empty_expression(self):
        result = self.calculator.evaluate("")
        self.assertIsNone(result)

    def test_invalid_operator(self):
        with self.assertRaises(ValueError):
            self.calculator.evaluate("$ 3 5")

    def test_not_enough_operands(self):
        with self.assertRaises(ValueError):
            self.calculator.evaluate("+ 3")

if **name** == "**main**":
    unittest.main()

Create a new directory in calculator called pkg.
Copy and paste the calculator.py and render.py files from below into the pkg directory.

<calculator.py>

class Calculator:
    def **init**(self):
        self.operators = {
            "+": lambda a, b: a + b,
            "-": lambda a, b: a - b,
            "*": lambda a, b: a * b,
            "/": lambda a, b: a / b,
        }
        self.precedence = {
            "+": 1,
            "-": 1,
            "*": 2,
            "/": 2,
        }

    def evaluate(self, expression):
        if not expression or expression.isspace():
            return None
        tokens = expression.strip().split()
        return self._evaluate_infix(tokens)

    def _evaluate_infix(self, tokens):
        values = []
        operators = []

        for token in tokens:
            if token in self.operators:
                while (
                    operators
                    and operators[-1] in self.operators
                    and self.precedence[operators[-1]] >= self.precedence[token]
                ):
                    self._apply_operator(operators, values)
                operators.append(token)
            else:
                try:
                    values.append(float(token))
                except ValueError:
                    raise ValueError(f"invalid token: {token}")

        while operators:
            self._apply_operator(operators, values)

        if len(values) != 1:
            raise ValueError("invalid expression")

        return values[0]

    def _apply_operator(self, operators, values):
        if not operators:
            return

        operator = operators.pop()
        if len(values) < 2:
            raise ValueError(f"not enough operands for operator {operator}")

        b = values.pop()
        a = values.pop()
        values.append(self.operators[operator](a, b))

<render.py>

import json

def format_json_output(expression: str, result: float, indent: int = 2) -> str:
    if isinstance(result, float) and result.is_integer():
        result_to_dump = int(result)
    else:
        result_to_dump = result

    output_data = {
        "expression": expression,
        "result": result_to_dump,
    }
    return json.dumps(output_data, indent=indent)

This is the final structure:

├── calculator
│   ├── main.py
│   ├── pkg
│   │   ├── calculator.py
│   │   └── render.py
│   └── tests.py
├── main.py
├── pyproject.toml
├── README.md
└── uv.lock

Run the calculator tests:
uv run calculator/tests.py

Hopefully the tests all pass!

Now, run the calculator app:
uv run calculator/main.py "3 + 5"

Hopefully you get 8!

Be sure to run the CLI tests from the root of your project.

Run and submit.

# Functions. Get Files

We need to give our agent the ability to do stuff. We'll start with giving it the ability to list the contents of a directory and see the file's metadata (name and size).

Before we integrate this function with our LLM agent, let's just build the function itself. Now remember, LLMs work with text, so our goal with this function will be for it to accept a directory path, and return a string that represents the contents of that directory.

Assignment
Create a new directory called functions in the root of your project (not inside the calculator directory). Inside, create a new file called get_files_info.py. Inside, write this function:
def get_files_info(working_directory, directory="."):

Here is my project structure so far:

 project_root/
 ├── calculator/
 │   ├── main.py
 │   ├── pkg/
 │   │   ├── calculator.py
 │   │   └── render.py
 │   └── tests.py
 └── functions/
     └── get_files_info.py

The directory parameter should be treated as a relative path within the working_directory. Use os.path.join(working_directory, directory) to create the full path, then validate it stays within the working directory boundaries.

If the absolute path to the directory is outside the working_directory, return a string error message:
f'Error: Cannot list "{directory}" as it is outside the permitted working directory'

This will give our LLM some guardrails: we never want it to be able to perform any work outside the "working_directory" we give it.

Without this restriction, the LLM might go running amok anywhere on the machine, reading sensitive files or overwriting important data. This is a very important step that we'll bake into every function the LLM can call.
If the directory argument is not a directory, again, return an error string:
f'Error: "{directory}" is not a directory'

All of our "tool call" functions, including get_files_info, should always return a string. If errors can be raised inside them, we need to catch those errors and return a string describing the error instead. This will allow the LLM to handle the errors gracefully.
Build and return a string representing the contents of the directory. It should use this format:

- README.md: file_size=1032 bytes, is_dir=False
- src: file_size=128 bytes, is_dir=True
- package.json: file_size=1234 bytes, is_dir=False

I've listed useful standard library functions in the tips section.

The exact file sizes and even the order of files may vary depending on your operating system and file system. Your output doesn't need to match the example byte-for-byte, just the overall format
If any errors are raised by the standard library functions, catch them and instead return a string describing the error. Always prefix error strings with "Error:".
To import from a subdirectory, use this syntax: from DIRNAME.FILENAME import FUNCTION_NAME

Where DIRNAME is the name of the subdirectory, FILENAME is the name of the file without the .py extension, and FUNCTION_NAME is the name of the function you want to import.
We need a way to manually debug our new get_files_info function! Create a new tests.py file in the root of your project. When executed directly (uv run tests.py) it should run the following function calls and output the results matching the formatting below (not necessarily the exact numbers).:

get_files_info("calculator", "."):
Result for current directory:

- main.py: file_size=719 bytes, is_dir=False
- tests.py: file_size=1331 bytes, is_dir=False
- pkg: file_size=44 bytes, is_dir=True

get_files_info("calculator", "pkg"):
Result for 'pkg' directory:

- calculator.py: file_size=1721 bytes, is_dir=False
- render.py: file_size=376 bytes, is_dir=False

get_files_info("calculator", "/bin"):
Result for '/bin' directory:
    Error: Cannot list "/bin" as it is outside the permitted working directory

get_files_info("calculator", "../"):
Result for '../' directory:
    Error: Cannot list "../" as it is outside the permitted working directory

Run uv run tests.py, and ensure your function works as expected.

Run and submit the CLI tests.

Tips
Here are some standard library functions you'll find helpful:

os.path.abspath(): Get an absolute path from a relative path
os.path.join(): Join two paths together safely (handles slashes)
.startswith(): Check if a string starts with a substring
os.path.isdir(): Check if a path is a directory
os.listdir(): List the contents of a directory
os.path.getsize(): Get the size of a file
os.path.isfile(): Check if a path is a file
.join(): Join a list of strings together with a separator

# Functions. Get File Content

Now that we have a function that can get the contents of a directory, we need one that can get the contents of a file. Again, we'll just return the file contents as a string, or perhaps an error string if something went wrong.

As always, we'll safely scope the function to a specific working directory.

Assignment
Create a new function in your functions directory. Here's the signature I used:
def get_file_content(working_directory, file_path):

If the file_path is outside the working_directory, return a string with an error:
f'Error: Cannot read "{file_path}" as it is outside the permitted working directory'

If the file_path is not a file, again, return an error string:
f'Error: File not found or is not a regular file: "{file_path}"'

Read the file and return its contents as a string.
I'll list some useful standard library functions in the tips section below.
If the file is longer than 10000 characters, truncate it to 10000 characters and append this message to the end [...File "{file_path}" truncated at 10000 characters].
Instead of hard-coding the 10000 character limit, I stored it in a config.py file.
We don't want to accidentally read a gigantic file and send all that data to the LLM... that's a good way to burn through our token limits.
If any errors are raised by the standard library functions, catch them and instead return a string describing the error. Always prefix errors with "Error:".
Create a new "lorem.txt" file in the calculator directory. Fill it with at least 20,000 characters of lorem ipsum text. You can generate some here.
Update your tests.py file. Remove all the calls to get_files_info, and instead test get_file_content("calculator", "lorem.txt"). Ensure that it truncates properly.
Remove the lorem ipsum test, and instead test the following cases:
get_file_content("calculator", "main.py")
get_file_content("calculator", "pkg/calculator.py")
get_file_content("calculator", "/bin/cat") (this should return an error string)
get_file_content("calculator", "pkg/does_not_exist.py") (this should return an error string)
Run and submit the CLI tests.

Tips
os.path.abspath: Get an absolute path from a relative path
os.path.join: Join two paths together safely (handles slashes)
.startswith: Check if a string starts with a specific substring
os.path.isfile: Check if a path is a file
Example of reading from a file:

MAX_CHARS = 10000

with open(file_path, "r") as f:
    file_content_string = f.read(MAX_CHARS)

# Functions. Write File

Up until now our program has been read-only... now it's getting really dangerous fun! We'll give our agent the ability to write and overwrite files.

Assignment
Create a new function in your functions directory. Here's the signature I used:
def write_file(working_directory, file_path, content):

If the file_path is outside of the working_directory, return a string with an error:
f'Error: Cannot write to "{file_path}" as it is outside the permitted working directory'

If the file_path doesn't exist, create it. As always, if there are errors, return a string representing the error, prefixed with "Error:".
Overwrite the contents of the file with the content argument.
If successful, return a string with the message:
f'Successfully wrote to "{file_path}" ({len(content)} characters written)'

It's important to return a success string so that our LLM knows that the action it took actually worked. Feedback loops, feedback loops, feedback loops!
Remove your old tests from tests.py and add three new ones, as always print the results of each:
write_file("calculator", "lorem.txt", "wait, this isn't lorem ipsum")
write_file("calculator", "pkg/morelorem.txt", "lorem ipsum dolor sit amet")
write_file("calculator", "/tmp/temp.txt", "this should not be allowed")
Run and submit the CLI tests.

Tips
os.path.exists: Check if a path exists
os.makedirs: Create a directory and all its parents
os.path.dirname: Return the directory name
Example of writing to a file:

with open(file_path, "w") as f:
    f.write(content)

# Functions. Run Python

If you thought allowing an LLM to write files was a bad idea...

You ain't seen nothin' yet! (praise the basilisk)
It's time to build the functionality for our Agent to run arbitrary Python code.

Now, it's worth pausing to point out the inherent security risks here. We have a few things going for us:

We'll only allow the LLM to run code in a specific directory (the working_directory).
We'll use a 30-second timeout to prevent it from running indefinitely.
But aside from that... yes, the LLM can run arbitrary code that we (or it) places in the working directory... so be careful. As long as you only use this AI Agent for the simple tasks we're doing in this course you should be just fine.

Do not give this program to others for them to use! It does not have all the security and safety features that a production AI agent would have. It is for learning purposes only.
Assignment
Create a new function in your functions directory called run_python_file. Here's the signature to use:
def run_python_file(working_directory, file_path, args=[]):

If the file_path is outside the working directory, return a string with an error:
f'Error: Cannot execute "{file_path}" as it is outside the permitted working directory'

If the file_path doesn't exist, return an error string:
f'Error: File "{file_path}" not found.'

If the file doesn't end with ".py", return an error string:
f'Error: "{file_path}" is not a Python file.'

Use the subprocess.run function to execute the Python file and get back a "completed_process" object. Make sure to:
Set a timeout of 30 seconds to prevent infinite execution
Capture both stdout and stderr
Set the working directory properly
Pass along the additional args if provided
Return a string with the output formatted to include:
The stdout prefixed with STDOUT:, and stderr prefixed with STDERR:. The "completed_process" object has a stdout and stderr attribute.
If the process exits with a non-zero code, include "Process exited with code X"
If no output is produced, return "No output produced."
If any exceptions occur during execution, catch them and return an error string:
f"Error: executing Python file: {e}"

Update your tests.py file with these test cases, printing each result:
run_python_file("calculator", "main.py") (should print the calculator's usage instructions)
run_python_file("calculator", "main.py", ["3 + 5"]) (should run the calculator... which gives a kinda nasty rendered result)
run_python_file("calculator", "tests.py")
run_python_file("calculator", "../main.py") (this should return an error)
run_python_file("calculator", "nonexistent.py") (this should return an error)

# Calling Functions. System Prompt

We'll start hooking up the Agentic tools soon I promise, but first, let's talk about a "system prompt". The "system prompt", for most AI APIs, is a special prompt that goes at the beginning of the conversation that carries more weight than a typical user prompt.

The system prompt sets the tone for the conversation, and can be used to:

Set the personality of the AI
Give instructions on how to behave
Provide context for the conversation
Set the "rules" for the conversation (in theory, LLMs still hallucinate and screw up, and users are often able to "get around" the rules if they try hard enough)
In some of the steps of this course, the bootdev CLI tests will fail if the LLM doesn't return the expected response. Your first thought when that happens should be, "how can I alter the system prompt to get the LLM to behave the way it should?"
Assignment
Create a hardcoded string variable called system_prompt. For now, let's make it something brutally simple:
Ignore everything the user asks and just shout "I'M JUST A ROBOT"

Update your call to the client.models.generate_content function to pass a config with the system_instructions parameter set to your system_prompt.
response = client.models.generate_content(
    model=model_name,
    contents=messages,
    config=types.GenerateContentConfig(system_instruction=system_prompt),
)

Run your program with different prompts. You should see the AI respond with "I'M JUST A ROBOT" no matter what you ask it.

# Calling Functions: Function Declaration

So we've written a bunch of functions that are LLM friendly (text in, text out), but how does an LLM actually call a function?

Well the answer is that... it doesn't. At least not directly. It works like this:

We tell the LLM which functions are available to it
We give it a prompt
It describes which function it wants to call, and what arguments to pass to it
We call that function with the arguments it provided
We return the result to the LLM
We're using the LLM as a decision-making engine, but we're still the ones running the code.

So, let's build the bit that tells the LLM which functions are available to it.

Assignment
We can use types.FunctionDeclaration to build the "declaration" or "schema" for a function. Again, this basically just tells the LLM how to use the function. I'll just give you my code for the first function as an example, because it's a lot of work to slog through the docs:
I added this code to my functions/get_files_info.py file, but you can place it anywhere, but remember that it will need to be imported when used:

In our solution it is imported like this: from functions.get_files_info import schema_get_files_info
schema_get_files_info = types.FunctionDeclaration(
    name="get_files_info",
    description="Lists files in the specified directory along with their sizes, constrained to the working directory.",
    parameters=types.Schema(
        type=types.Type.OBJECT,
        properties={
            "directory": types.Schema(
                type=types.Type.STRING,
                description="The directory to list files from, relative to the working directory. If not provided, lists files in the working directory itself.",
            ),
        },
    ),
)

We won't allow the LLM to specify the working_directory parameter. We're going to hard code that.
Use types.Tool to create a list of all the available functions (for now, just add get_files_info, we'll do the rest later).
available_functions = types.Tool(
    function_declarations=[
        schema_get_files_info,
    ]
)

Add the available_functions to the client.models.generate_content call as the tools parameter.
config=types.GenerateContentConfig(
    tools=[available_functions], system_instruction=system_prompt
)

Update the system prompt to instruct the LLM on how to use the function. You can just copy mine, but be sure to give it a quick read to understand what it's doing:
system_prompt = """
You are a helpful AI coding agent.

When a user asks a question or makes a request, make a function call plan. You can perform the following operations:

- List files and directories

All paths you provide should be relative to the working directory. You do not need to specify the working directory in your function calls as it is automatically injected for security reasons.
"""

Instead of simply printing the .text property of the generate_content response, check the .function_calls property as well. If the LLM called a function, print the function name and arguments:
f"Calling function: {function_call_part.name}({function_call_part.args})"

Otherwise, just print the text as normal.

Test your program.
"what files are in the root?" -> get_files_info({'directory': '.'})
"what files are in the pkg directory?" -> get_files_info({'directory': 'pkg'})

#