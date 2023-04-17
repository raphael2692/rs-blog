
---
title : "Natural query against tabular data with LangChain and Pandas" 
tags : ["coding", "python", "langchain", "pandas"] 
date : "2023-04-14" 
draft : true
---


## Introduction

In this tutorial, we will explore how LangChain can be used to retrieve information from a dataframe for a specific query using the `create_pandas_dataframe_agent` function. 

This script utilizes pandas and Langchain, using the OpenAI API, to create a dataframe agent that can answer natural language questions based off of the data provided by a CSV file. 

## Step 1: Import Libraries

Let's begin by importing the necessary libraries. 

```python
import pandas as pd
from langchain.agents import create_pandas_dataframe_agent
from langchain.llms import OpenAI
```

## Step 2: Create a DataFrame
We will be reading in data from a CSV file to create a DataFrame. In this case we will be using a dataset containing information about the python packages listend in pypi as of 2019.

```python
df = pd.read_csv("/content/package-manifest.csv")
```

If you are running this code on a notebook you can view the first few entries of the DataFrame by using the `.head()` method.

```python
df.head()
```

## Step 3: Create DataFrame Agent

Now we will use the LangChain library to create a DataFrame agent which can answer natural language questions based off of the CSV data. 

```python
agent = create_pandas_dataframe_agent(OpenAI(temperature=0), df, verbose=True)
```

Here, we have created the agent by passing two arguments. 
  - First is `OpenAI(temperature=0)`, which is an instance of OpenAI with the desired level of temperature (in this case, 0). The temperature represents the degrees of freedom the LLM (chatgpt-turbo 3.5 in this case) uses. We set zero to force coherence to data.
  - Second is the `dataframe`, a pandas DataFrame instance that we want to search for information.


## Step 4: Ask Questions 

Now you are all set to ask questions to the agent. 

```python
agent.run("what are the best packages for data visualization?")
```

Here, we passed a free text query `"what are the best packages for data visualization?"` to the agent, which returned a pandas DataFrame containing information about the packages for data visualization. You should see the agent's response printed in the terminal, like this:

```shell
> Entering new AgentExecutor chain...
Thought: I need to find packages that are related to data visualization
Action: python_repl_ast
Action Input: df[df['summary'].str.contains('visualization')]
Observation: Cannot mask with non-boolean array containing NA / NaN values
Thought: I need to filter out the NA/NaN values
Action: python_repl_ast
Action Input: df[df['summary'].str.contains('visualization', na=False)]
Observation:     package_name version                                            summary  \
62        altair   3.2.0  Altair: A declarative statistical visualizatio...   
145   datashader   0.7.0  Data visualization toolchain based on aggregat...   
224     graphviz   0.8.4          Open Source graph visualization software.   
317    missingno   0.4.2      Missing data visualization module for Python.   
425     pyLDAvis   2.1.2  Interactive topic model visualization. Port of...   
519      seaborn   0.9.0                     Statistical data visualization   
604         vida     0.3        Python binding for Vida data visualizations   
605       visvis  1.11.2  An object oriented approach to visualization o...   
607          vtk   8.1.2  VTK is an open-source toolkit for 3D computer ...   

          license metadata_source  
62   BSD 3-clause            PyPI  
145  BSD-3-Clause        Anaconda  
224      EPL v1.0        Anaconda  
317           NaN            PyPI  
425           MIT            PyPI  
519  BSD 3-Clause        Anaconda  
604       UNKNOWN            PyPI  
605  BSD 3-Clause        Anaconda  
607           BSD            PyPI  
Thought: I now know the best packages for data visualization
Final Answer: altair, datashader, graphviz, missingno, pyLDAvis, seaborn, vida, visvis, and vtk.

> Finished chain.
altair, datashader, graphviz, missingno, pyLDAvis, seaborn, vida, visvis, and vtk.
```

Obviusly you can skip the logging part and just fetch this string:

```
altair, datashader, graphviz, missingno, pyLDAvis, seaborn, vida, visvis, and vtk.
```

## Behind the scenes
The agent uses an LLMChains and "tools" to firstly embed the dataset and make it searchable against a query in NLP (the user's prompt). I might  cover in detail how an agent works in the context of LLMs, in the meant time you can check out the documentation these resources:

- [Nice introduction to the topic from Haystack](https://haystack.deepset.ai/blog/introducing-haystack-agents)
- [Agents in LangChain documentation](https://python.langchain.com/en/latest/modules/agents/getting_started.html)

## Conclusion
In this tutorial we have seen how we can use LangChain to extract information from a pandas DataFrame for a specific query using an NLP tool with few lines of python code and an OpenAI subscription.