

+++ 
title = "Creating a Wikipedia Q&A using OpenAI, LangChain and Trafilatura" 
tags = ["coding", "python", "datascience", "ai", "llm", "langchain"] 
date = "2023-04-14" 

+++

# Introduction

In this tutorial, we will be building a Python program that searches for a specified topic on Wikipedia, extracts the text from the top search result, and generates a summary using OpenAI and Trafilatura. We will use the LangChain library to connect these different functionalities.

## Required Libraries

We will be using the following external libraries:

- `loguru`
- `trafilatura`
- `openai`
- `langchain`

Before we dive deeper, ensure that all these libraries are installed in your system. 

```python
# standard
import os
import json
import re

from loguru import logger
import trafilatura

from langchain import LLMChain
from langchain.prompts.prompt import PromptTemplate
from langchain.chat_models import ChatOpenAI
from langchain.prompts.chat import (
    ChatPromptTemplate,
    HumanMessagePromptTemplate,
)
from langchain.chains import SimpleSequentialChain
from langchain.chains import LLMChain
from langchain.chains.base import Chain
from typing import Dict, List
```

## Authenticating OpenAI

We will be using OpenAI's API to generate a summary of the text. To access the API, you will have to create an account and obtain an API key. Once you have the key, you can store it in a `secrets.json` file and read it into the program as shown below:

```python
if __name__ == "__main__":
    logger.info("Starting app...")
    # read settings
    with open("secrets.json", "r") as jsonfile:
        secrets = json.load(jsonfile)

    # set openai key
    os.environ["OPENAI_API_KEY"] = secrets["openai_api_key"]
```

Make sure to replace `openai_api_key` with your OpenAI API key.

## Creating a URL Chain

Using the `LangChain` library, we can create a chain that prompts the user to enter the topic they would like to search on Wikipedia. This chain will then use OpenAI to generate a URL to the most relevant article on Wikipedia.

```python
    # init chat object
    chat = ChatOpenAI(temperature=0)

    # setup url chain
    url_chain_human_prompt_template = HumanMessagePromptTemplate(
        prompt=PromptTemplate(
            template = URL_TEMPLATE['text'],
            input_variables = URL_TEMPLATE['input_variables'],
        )
    )
    url_chain_chat_prompt_template = ChatPromptTemplate.from_messages(
        [url_chain_human_prompt_template])

    url_chain = LLMChain(llm=chat, prompt=url_chain_chat_prompt_template, output_key="url", verbose=True)
```

This chain prompts the user for input and generates a URL by using OpenAI to search for the specified topic on Wikipedia. It then stores the URL in the `url_chain` variable.

## Creating a custom Chain for web scraping

We will use Trafilatura to extract the text from the Wikipedia page corresponding to the URL generated in the previous step. The `TrafilaturaChain` defined in the following code block uses the `trafilatura` library to extract and clean the text, then splits the text into paragraphs of maximum length 500 characters.

```python
class TrafilaturaChain(Chain):
    input_key: str = "url"  # :meta private:
    output_key: str = "text" # :meta private:

    @property
    def input_keys(self) -> List[str]:
        return [self.input_key]

    @property
    def output_keys(self) -> List[str]:
        return [self.output_key]

    def _get_text(self, url):
        downloaded = trafilatura.fetch_url(url)
        result = json.loads(trafilatura.extract(
            downloaded, output_format='json', include_links=False))
        text = result["text"].replace("\n", "")
        paragraph = self._get_paragraphs_from_text(text, 500)
        return "".join(paragraph[0:10])

    def _get_paragraphs_from_text(self, text, max_length):
        sentences = re.findall('[^\.!?]+[\.!?]', text)
        current_paragraph = ""
        current_length = 0
        paragraphs = []
        for sentence in sentences:
            if current_length + len(sentence) <= max_length:
                current_paragraph += sentence.strip() + " "
                current_length += len(sentence)
            else:
                paragraphs.append(current_paragraph.strip())
                current_paragraph = sentence.strip() + " "
                current_length = len(sentence)
        paragraphs.append(current_paragraph.strip())
        return paragraphs

    def _call(self, inputs: Dict[str, str]) -> Dict[str, str]:
        return {self.output_key: self._get_text(inputs['url'])}
   
trafilatura_chain = TrafilaturaChain()
```

The `TrafilaturaChain` takes in the URL generated by the `url_chain` and outputs a cleaned version of the corresponding Wikipedia page text. 

## Creating a Summary Chain

The `SummaryChain` defined in the following code block uses OpenAI to generate a summary of the cleaned text generated by the `TrafilaturaChain`.

```python
    # setup summary chain
    summary_chain_human_prompt_template = HumanMessagePromptTemplate(
        prompt=PromptTemplate(
            template = SUMMARY_TEMPLATE['text'],
            input_variables = SUMMARY_TEMPLATE['input_variables'],
        )
    )
    summary_chain_chat_prompt_template = ChatPromptTemplate.from_messages(
        [summary_chain_human_prompt_template])

    logger.info("Composing chain...")
    summary_chain = LLMChain(llm=chat, prompt=summary_chain_chat_prompt_template, output_key="summary", verbose=True) 
```

The `SummaryChain` prompts the user to enter the length of the summary they would like to generate, and then uses OpenAI to generate the summary of the cleaned text generated by the `TrafilaturaChain`. 

## Creating the Sequential Chain

Finally, we use the `SimpleSequentialChain` library to connect the individual chains (`url_chain`, `trafilatura_chain`, and `summary_chain`) into a single chain.

```python
    simple_sequential_chain = SimpleSequentialChain(
        chains=[url_chain, trafilatura_chain, summary_chain],
        verbose=True
    )
    logger.info("Fetching user input...")
    user_prompt = input("\n What do you want to search in Wikipedia?: ")
    simple_sequential_chain.run(user_prompt)
```

The `SimpleSequentialChain` takes in the user's input, runs it through each of the individual chains in order, and produces the final summary.

## Conclusion

In this tutorial, we learned how to create a Python program that searches for a topic on Wikipedia, extracts the corresponding text, and generates a summary. We used OpenAI and Trafilatura to perform the text processing and LLMChain to connect the different functionalities. By following this tutorial and tweaking the variables to your liking, you can create your own customized summary generator for any topic on Wikipedia.

### Resources
- The code for this tutorial [on my GitHub](https://github.com/raphael2692/langchain-prompt-to-wikipedia)