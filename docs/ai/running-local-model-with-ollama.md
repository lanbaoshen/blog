# Running Local Model with Ollama
Sometimes, I need to run a model locally on my machine. This is useful for debugging and testing purposes. 

This guide will show you how to run a model locally using Ollama.

## Install Ollama
We can install Ollama from [web page](https://ollama.com/download)

If you are using linux, you can install it using the following command:

```shell
curl -fsSL https://ollama.com/install.sh | sh
```

## Download Model
I will use [llama3.2](https://ollama.com/library/llama3.2:3b) model in the following example. So we need to download it first:

<!-- termynal -->

```shell
# 1B parameters, may take up 1.3GB of disk space
# ollama pull llama3.2:1b
# 3B parameters (default), may take up 2GB of disk space
$ ollama pull llama3.2
---> 100%
success
```

## Run Model
We can run the model using a command and interact with it using the terminal:

<!-- termynal -->

```shell
$ ollama run llama3.2
>>> hello
Hello! How can i assist you today?


>>> Tell me a joke
Why don't eggs tell jokes?

Because they'd crack each other up!


>>> /bye
```

## Interact via API
We can interact with the model using the API:
```shell
curl http://localhost:11434/api/chat -d '{
  "model": "llama3.2",
  "messages": [
    {
      "role": "user",
      "content": "Tell me i joke"
    }
  ],
  "stream": false
}'
```

## Interact via Code
We can interact with the model using third party lib `ollama`, we need to install it first:
```shell
pip install -U ollama
```

And then we can execute the following code:
```python
import ollama


response = ollama.chat(
  'llama3.2',
  messages=[{'role': 'user', 'content': 'Hello'}],
)
print(response.message.content)  # How can I assist you today?
```

## Example
Here is a sample example of how to use `ollama` to collect statistical data:

```python
import ollama


csv_content = """
name, score
Mindy Luo, 97
Sovin Yang, 91
Leo Li, 90
Lanbao Shen, 92
Chang Li, 99
Jarvan Shi, 93
"""

prompt = """
Find the student with the highest score and calculate the average score in the following CSV file.
Only return result, no need to show the process.

# CSV File
{input}
"""

response = ollama.chat(
  'llama3.2',
  messages=[{'role': 'user', 'content': prompt.format(input=csv_content)}],
)
print(response.message.content)
```

When you run the code above, you may find the result is not right.
This is the problem we will address later, and it will not be elaborated on here.

!!! note
    As of this writing, `ollama` has supported function calls since version **0.4.0**.
    You can refer to [functions-as-tools](https://ollama.com/blog/functions-as-tools) for more information.
