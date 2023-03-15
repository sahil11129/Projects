# Watson NLP vs OpenAI


```python
import json
import pandas as pd
import watson_nlp
import random
import string
from watson_nlp import data_model as dm
from watson_nlp.toolkit.entity_mentions_utils import prepare_train_from_json, create_iob_labels
```


```python
# Silence Tensorflow warnings
import tensorflow as tf
tf.get_logger().setLevel('ERROR')
tf.autograph.set_verbosity(0)
```


```python
# Load a syntax model to split the text into sentences and tokens
syntax_model = watson_nlp.load(watson_nlp.download('syntax_izumo_en_stock'))
```

## Watson NLP

<a id="PTbuildModel"></a>
## 1. Pre-Trained Models for Entities Extraction


```python
Text = "my credit card number is 8763923482349237 Black-on-black ware is a 20th- and 21st-century pottery tradition developed by the Puebloan Native American ceramic artists in Northern New Mexico. Traditional reduction-fired blackware has been made for centuries by pueblo artists. Black-on-black ware of the past century is produced with a smooth surface, with the designs applied through selective burnishing or the application of refractory slip. Another style involves carving or incising designs and selectively polishing the raised areas. For generations several families from Kha'po Owingeh and Pohwhóge Owingeh pueblos have been making black-on-black ware with the techniques passed down from matriarch potters. Artists from other pueblos have also produced black-on-black ware. Several contemporary artists have created works honoring the pottery of their ancestors."
Text
```




    "my credit card number is 8763923482349237 Black-on-black ware is a 20th- and 21st-century pottery tradition developed by the Puebloan Native American ceramic artists in Northern New Mexico. Traditional reduction-fired blackware has been made for centuries by pueblo artists. Black-on-black ware of the past century is produced with a smooth surface, with the designs applied through selective burnishing or the application of refractory slip. Another style involves carving or incising designs and selectively polishing the raised areas. For generations several families from Kha'po Owingeh and Pohwhóge Owingeh pueblos have been making black-on-black ware with the techniques passed down from matriarch potters. Artists from other pueblos have also produced black-on-black ware. Several contemporary artists have created works honoring the pottery of their ancestors."




```python
Text1 = "Over the past few months, the United States has turbo-charged the process of granting visas to Indians in unique ways, including using its embassies in other countries to deal with the backlog in India, to reduce visa delays for thousands of applicants. India was one of the very few countries where applications for US visas saw a major upswing after coronavirus-related travel restrictions were lifted. Julie Stufft, deputy assistant secretary for consular affairs, said reducing the wait time was her No 1 priority. Its the first thing I do when I wake up, she said. Its a once in a lifetime situation. The visa operations in India are very different from any other country because of the range of visas and the massive demand."
Text1
```




    'Over the past few months, the United States has turbo-charged the process of granting visas to Indians in unique ways, including using its embassies in other countries to deal with the backlog in India, to reduce visa delays for thousands of applicants. India was one of the very few countries where applications for US visas saw a major upswing after coronavirus-related travel restrictions were lifted. Julie Stufft, deputy assistant secretary for consular affairs, said reducing the wait time was her No 1 priority. Its the first thing I do when I wake up, she said. Its a once in a lifetime situation. The visa operations in India are very different from any other country because of the range of visas and the massive demand.'



<a id="Bilstm"></a>
### 1.1 BiLSTM Model


```python
# Load a workflow consisting of the Syntax and an Entity Mention model for English
entity_workflow_model = watson_nlp.load(watson_nlp.download('entity-mentions_bilstm-workflow_en_stock'))

```


```python
import time

start_time = time.time()
entity_mentions = entity_workflow_model.run(Text)
for i in entity_mentions.mentions:
    print("Text", i.span.text.ljust(15, " "), "Type: ", i.type)

end_time = time.time()

print(f"Time taken: {end_time - start_time} seconds")
```

    Text 20th- and 21st-century Type:  Duration
    Text Puebloan Native American ceramic artists Type:  Organization
    Text Northern New Mexico Type:  Location
    Text pueblo          Type:  Location
    Text artists         Type:  JobTitle
    Text Kha'po Owingeh  Type:  Location
    Text Pohwhóge Owingeh pueblos Type:  Location
    Text matriarch potters Type:  Location
    Text artists         Type:  JobTitle
    Time taken: 0.909630298614502 seconds



```python
import time
start_time = time.time()

entity_mentions = entity_workflow_model.run(Text1)
for i in entity_mentions.mentions:
    print("Text", i.span.text.ljust(15, " "), "Type: ", i.type)
    
end_time = time.time()
print(f"Time taken: {end_time - start_time} seconds")
```

    Text past few months Type:  Duration
    Text United States   Type:  Location
    Text Indians         Type:  Location
    Text India           Type:  Location
    Text India           Type:  Location
    Text US              Type:  Location
    Text Julie Stufft    Type:  Person
    Text deputy assistant secretary for consular affairs Type:  JobTitle
    Text first           Type:  Ordinal
    Text India           Type:  Location
    Time taken: 0.48821377754211426 seconds


<a id="PreSire"></a>
### 1.2 Sire Model


```python
# let's download and run our SIRE entity mentions model.
sire_mentions_model = watson_nlp.load(watson_nlp.download('entity-mentions_sire_en_stock-wf'))

```


```python
import time
start_time = time.time()

sire_entity_mentions = sire_mentions_model.run(Text)
for i in sire_entity_mentions.mentions:
    print("Text", i.span.text.ljust(15, " "), "Type: ", i.type)

    
end_time = time.time()
print(f"Time taken: {end_time - start_time} seconds")

```

    Text Puebloan        Type:  Location
    Text American        Type:  Location
    Text Northern New Mexico Type:  Location
    Text past century    Type:  Date
    Text Kha             Type:  Organization
    Text Owingeh         Type:  Person
    Text Pohwhóge Owingeh Type:  Person
    Time taken: 0.1568772792816162 seconds



```python
import time
start_time = time.time()

sire_entity_mentions = sire_mentions_model.run(Text1)
for i in sire_entity_mentions.mentions:
    print("Text", i.span.text.ljust(15, " "), "Type: ", i.type)

    
end_time = time.time()
print(f"Time taken: {end_time - start_time} seconds")
```

    Text past few months Type:  Date
    Text United States   Type:  Location
    Text India           Type:  Location
    Text US              Type:  Location
    Text Julie Stufft    Type:  Person
    Text first           Type:  Ordinal
    Text India           Type:  Location
    Time taken: 0.38111257553100586 seconds


<a id="Prebert"></a>
### 1.3 BERT Model


```python
# let's download and run our Bert entity mentions model.
bert_entity_mentions = watson_nlp.load( watson_nlp.download('entity-mentions_bert_multi_stock'))
```


```python
import time
start_time = time.time()

syntax_result =syntax_model.run(Text)
entity_mentions = bert_entity_mentions.run(syntax_result)
for i in entity_mentions.mentions:
    print("Text", i.span.text.ljust(15, " "), "Type: ", i.type)

    
end_time = time.time()
print(f"Time taken: {end_time - start_time} seconds")

```

    Text 20th- and       Type:  Date
    Text 21st-century    Type:  Date
    Text Puebloan Native American ceramic Type:  Organization
    Text Northern New Mexico Type:  Location
    Text past century    Type:  Date
    Text Kha'po Owingeh  Type:  Location
    Text Pohwhóge Owingeh Type:  Location
    Time taken: 0.9805474281311035 seconds



```python
import time
start_time = time.time()

syntax_result =syntax_model.run(Text1)
entity_mentions = bert_entity_mentions.run(syntax_result)
for i in entity_mentions.mentions:
    print("Text", i.span.text.ljust(15, " "), "Type: ", i.type)

    
end_time = time.time()
print(f"Time taken: {end_time - start_time} seconds")
```

    Text past few months Type:  Duration
    Text United States   Type:  Location
    Text India           Type:  Location
    Text India           Type:  Location
    Text US              Type:  Location
    Text Julie Stufft    Type:  Person
    Text deputy assistant secretary for consular affairs Type:  JobTitle
    Text first           Type:  Ordinal
    Text India           Type:  Location
    Time taken: 0.4609248638153076 seconds


## OpenAI 


```python
pip install openai
```

    Collecting openai
      Downloading openai-0.27.2-py3-none-any.whl (70 kB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m70.1/70.1 kB[0m [31m3.9 MB/s[0m eta [36m0:00:00[0m
    [?25hRequirement already satisfied: tqdm in /opt/conda/envs/Python-3.10-Premium/lib/python3.10/site-packages (from openai) (4.64.0)
    Requirement already satisfied: requests>=2.20 in /opt/conda/envs/Python-3.10-Premium/lib/python3.10/site-packages (from openai) (2.28.1)
    Requirement already satisfied: aiohttp in /opt/conda/envs/Python-3.10-Premium/lib/python3.10/site-packages (from openai) (3.8.1)
    Requirement already satisfied: charset-normalizer<3,>=2 in /opt/conda/envs/Python-3.10-Premium/lib/python3.10/site-packages (from requests>=2.20->openai) (2.0.4)
    Requirement already satisfied: certifi>=2017.4.17 in /opt/conda/envs/Python-3.10-Premium/lib/python3.10/site-packages (from requests>=2.20->openai) (2022.12.7)
    Requirement already satisfied: idna<4,>=2.5 in /opt/conda/envs/Python-3.10-Premium/lib/python3.10/site-packages (from requests>=2.20->openai) (3.3)
    Requirement already satisfied: urllib3<1.27,>=1.21.1 in /opt/conda/envs/Python-3.10-Premium/lib/python3.10/site-packages (from requests>=2.20->openai) (1.26.11)
    Requirement already satisfied: yarl<2.0,>=1.0 in /opt/conda/envs/Python-3.10-Premium/lib/python3.10/site-packages (from aiohttp->openai) (1.8.1)
    Requirement already satisfied: aiosignal>=1.1.2 in /opt/conda/envs/Python-3.10-Premium/lib/python3.10/site-packages (from aiohttp->openai) (1.2.0)
    Requirement already satisfied: attrs>=17.3.0 in /opt/conda/envs/Python-3.10-Premium/lib/python3.10/site-packages (from aiohttp->openai) (21.4.0)
    Requirement already satisfied: frozenlist>=1.1.1 in /opt/conda/envs/Python-3.10-Premium/lib/python3.10/site-packages (from aiohttp->openai) (1.2.0)
    Requirement already satisfied: async-timeout<5.0,>=4.0.0a3 in /opt/conda/envs/Python-3.10-Premium/lib/python3.10/site-packages (from aiohttp->openai) (4.0.1)
    Requirement already satisfied: multidict<7.0,>=4.5 in /opt/conda/envs/Python-3.10-Premium/lib/python3.10/site-packages (from aiohttp->openai) (5.2.0)
    Requirement already satisfied: typing-extensions>=3.6.5 in /opt/conda/envs/Python-3.10-Premium/lib/python3.10/site-packages (from async-timeout<5.0,>=4.0.0a3->aiohttp->openai) (4.3.0)
    Installing collected packages: openai
    Successfully installed openai-0.27.2
    Note: you may need to restart the kernel to use updated packages.



```python
import os
import openai
```


```python
openai.api_key = "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"

```

## OpenAI Models


```python
import time
start_time = time.time()

response = openai.Completion.create(
  model="text-davinci-003",
  prompt="Extract entities from this text:\n\nBlack-on-black ware is a 20th- and 21st-century pottery tradition developed by the Puebloan Native American ceramic artists in Northern New Mexico. Traditional reduction-fired blackware has been made for centuries by pueblo artists. Black-on-black ware of the past century is produced with a smooth surface, with the designs applied through selective burnishing or the application of refractory slip. Another style involves carving or incising designs and selectively polishing the raised areas. For generations several families from Kha'po Owingeh and Pohwhóge Owingeh pueblos have been making black-on-black ware with the techniques passed down from matriarch potters. Artists from other pueblos have also produced black-on-black ware. Several contemporary artists have created works honoring the pottery of their ancestors.",
  temperature=0.5,
  max_tokens=60,
  top_p=1.0,
  frequency_penalty=0.8,
  presence_penalty=0.0
)

print(response)
    
end_time = time.time()
print(f"Time taken: {end_time - start_time} seconds")
```

    {
      "choices": [
        {
          "finish_reason": "length",
          "index": 0,
          "logprobs": null,
          "text": "\n\nEntities: \n- Puebloan Native American ceramic artists \n- Northern New Mexico \n- Kha'po Owingeh and Pohwh\u00f3ge Owingeh pueblos \n- 20th century \n- 21st century \n- Reduction firing"
        }
      ],
      "created": 1678741265,
      "id": "cmpl-6tjZxde9nMuaR9sexuwKjvM4sIBHA",
      "model": "text-davinci-003",
      "object": "text_completion",
      "usage": {
        "completion_tokens": 60,
        "prompt_tokens": 185,
        "total_tokens": 245
      }
    }
    Time taken: 2.7185773849487305 seconds


### Model 1 - davinci


```python
import time
start_time = time.time()

response = openai.Completion.create(
  model="text-davinci-003",
  prompt="Extract keywords from this text:\n\nOver the past few months, the United States has turbo-charged the process of granting visas to Indians in unique ways, including using its embassies in other countries to deal with the backlog in India, to reduce visa delays for thousands of applicants. India was one of the very few countries where applications for US visas saw a major upswing after coronavirus-related travel restrictions were lifted.Julie Stufft, deputy assistant secretary for consular affairs, said reducing the wait time was her No 1 priority. It's the first thing I do when I wake up, she said. It's a once in a lifetime situation. The visa operations in India are very different from any other country because of the range of visas and the massive demand.",
  temperature=0.5,
  max_tokens=60,
  top_p=1.0,
  frequency_penalty=0.8,
  presence_penalty=0.0
)

print(response)
    
end_time = time.time()
print(f"Time taken: {end_time - start_time} seconds")
```

    {
      "choices": [
        {
          "finish_reason": "stop",
          "index": 0,
          "logprobs": null,
          "text": "\n\n- United States \n- India \n- Visa \n- Coronavirus \n- Julie Stufft \n- Consular Affairs \n- Wait Time \n- Travel Restrictions"
        }
      ],
      "created": 1678366477,
      "id": "cmpl-6sA4zmJwZ1ySzVYUhFIvvWY301uc3",
      "model": "text-davinci-003",
      "object": "text_completion",
      "usage": {
        "completion_tokens": 43,
        "prompt_tokens": 157,
        "total_tokens": 200
      }
    }
    Time taken: 1.6687746047973633 seconds


###  Model 2 - curie 


```python
import time
start_time = time.time()

response = openai.Completion.create(
  model="text-curie-001",
  prompt="Extract keywords from this text:\n\nOver the past few months, the United States has turbo-charged the process of granting visas to Indians in unique ways, including using its embassies in other countries to deal with the backlog in India, to reduce visa delays for thousands of applicants. India was one of the very few countries where applications for US visas saw a major upswing after coronavirus-related travel restrictions were lifted.Julie Stufft, deputy assistant secretary for consular affairs, said reducing the wait time was her No 1 priority. It's the first thing I do when I wake up, she said. It's a once in a lifetime situation. The visa operations in India are very different from any other country because of the range of visas and the massive demand.",
  temperature=0.5,
  max_tokens=60,
  top_p=1.0,
  frequency_penalty=0.8,
  presence_penalty=0.0
)

print(response)
    
end_time = time.time()
print(f"Time taken: {end_time - start_time} seconds")
```

    {
      "choices": [
        {
          "finish_reason": "stop",
          "index": 0,
          "logprobs": null,
          "text": "\n\n-Turbocharged process \n-Visa delays \n-Upswing in applications \n-No. 1 priority \n-Coronavirus-related travel restrictions \n-Visa operations in India"
        }
      ],
      "created": 1678366536,
      "id": "cmpl-6sA5w61Vwfs8vkKMYcHX3TaIx5NkX",
      "model": "text-curie-001",
      "object": "text_completion",
      "usage": {
        "completion_tokens": 46,
        "prompt_tokens": 157,
        "total_tokens": 203
      }
    }
    Time taken: 0.8292214870452881 seconds


###  Model 3 - babbage


```python
import time
start_time = time.time()

response = openai.Completion.create(
  model="text-babbage-001",
  prompt="Extract keywords from this text:\n\nOver the past few months, the United States has turbo-charged the process of granting visas to Indians in unique ways, including using its embassies in other countries to deal with the backlog in India, to reduce visa delays for thousands of applicants. India was one of the very few countries where applications for US visas saw a major upswing after coronavirus-related travel restrictions were lifted.Julie Stufft, deputy assistant secretary for consular affairs, said reducing the wait time was her No 1 priority. It's the first thing I do when I wake up, she said. It's a once in a lifetime situation. The visa operations in India are very different from any other country because of the range of visas and the massive demand.",
  temperature=0.5,
  max_tokens=60,
  top_p=1.0,
  frequency_penalty=0.8,
  presence_penalty=0.0
)

print(response)
    
end_time = time.time()
print(f"Time taken: {end_time - start_time} seconds")
```

    {
      "choices": [
        {
          "finish_reason": "stop",
          "index": 0,
          "logprobs": null,
          "text": "\n\n-US visas being granted to Indians in unique ways \n-The US embassy in India dealing with the backlog in India \n- reduction of visa delays for thousands of applicants"
        }
      ],
      "created": 1678366596,
      "id": "cmpl-6sA6ujX1Z4yGBvTj2prTTDIa3lRqh",
      "model": "text-babbage-001",
      "object": "text_completion",
      "usage": {
        "completion_tokens": 37,
        "prompt_tokens": 157,
        "total_tokens": 194
      }
    }
    Time taken: 0.33603572845458984 seconds


###  Model 4 - ada


```python
import time
start_time = time.time()

def ada_class (text):
    response = openai.Completion.create(
        engine="text-ada-001",
        prompt=f"Extract classfication, Category and PII from the following text:\n{text}\n---\nENTITY_TYPE: ENTITY_NAME",
        max_tokens=50,
        n=1,
        stop=None,
        temperature=0.5,
    )

    classes = response.choices[0].text.strip().split("\n")
    return classes

print(ada_class(Text))

end_time = time.time()
print(f"Time taken: {end_time - start_time} seconds")
```

    [': Black-on-black ware', '', 'PII: 20th- and 21st-century pottery tradition developed by the Puebloan Native American ceramic artists in Northern New Mexico', '', 'Category: Black-on-black ware']
    Time taken: 0.33174633979797363 seconds


# NLP Tasks with OpenAI

## 1. Entities/Keywords Extraction 


```python
def extract_entity(text):
    response = openai.Completion.create(
        engine="text-davinci-003",
        prompt=f"Extract entities from the following text:\n{text}\n---\nKEYWORD_TYPE: KEYWORD_NAME",
        max_tokens=50,
        n=1,
        stop=None,
        temperature=0.5,
    )

    entity = response.choices[0].text.strip().split("\n")
    return entity
```


```python
extract_entity(Text)
```




    ['Credit Card Number: 8763923482349237',
     'Pottery Tradition: Black-on-black ware',
     'Century: 20th- and 21st-century',
     'Ceramic Artists: Puebloan Native American',
     'Northern New']




```python
extract_entity(Text)
```




    ['Credit card number: 8763923482349237',
     '20th century: 20th-century',
     '21st century: 21st-century',
     'Puebloan Native American: Puebloan Native American',
     'Northern New Mexico: Northern New']



#### Embed rank keywords extraction model in Watson NLP


```python
from watson_nlp import data_model as dm 

# Load Noun Phrases, Embedding and Keywords models for English
noun_phrases_model = watson_nlp.load(watson_nlp.download('noun-phrases_rbr_en_stock'))
use_model = watson_nlp.load(watson_nlp.download("embedding_use_en_stock"))
keywords_model = watson_nlp.load(watson_nlp.download('keywords_embed-rank_multi_stock'))

# Run the Noun Phrases model
noun_phrases = noun_phrases_model.run(Text)

# Get document embeddings
# No need to run any Syntax model since the 'raw_text' embed style will be used for doc embedding
syntax_analysis = dm.SyntaxPrediction(text=Text)
doc_embeddings = use_model.run(syntax_analysis, doc_embed_style='raw_text')

# Get embeddings for noun phrases
noun_phrases_analysis = [dm.SyntaxPrediction(text=c.span.text) for c in noun_phrases.noun_phrases]
noun_phrase_embeddings = use_model.run_batch(noun_phrases_analysis, doc_embed_style='raw_text')

# Run the keywords model
keywords = keywords_model.run(doc_embeddings, noun_phrases, noun_phrase_embeddings, limit=2, beta=0.5)
print(keywords)
```

    {
      "keywords": [
        {
          "text": "Native American ceramic artists",
          "relevance": 1.0,
          "mentions": [
            {
              "begin": 134,
              "end": 165,
              "text": "Native American ceramic artists"
            }
          ],
          "count": 1
        },
        {
          "text": "21st-century pottery tradition",
          "relevance": 0.926207488615385,
          "mentions": [
            {
              "begin": 77,
              "end": 107,
              "text": "21st-century pottery tradition"
            }
          ],
          "count": 1
        }
      ],
      "producer_id": {
        "name": "Embed Rank Keywords",
        "version": "0.0.2"
      }
    }


#### Text rank Keywords extraction model in Watson NLP


```python
# #Watson NLP

# Load Syntax, Noun Phrases and Keywords models for English
noun_phrases_model = watson_nlp.load(watson_nlp.download('noun-phrases_rbr_en_stock'))
keywords_model = watson_nlp.load(watson_nlp.download('keywords_text-rank_en_stock'))

# Run the Syntax and Noun Phrases models
syntax_prediction = syntax_model.run(Text, parsers=('token', 'lemma', 'part_of_speech'))
noun_phrases = noun_phrases_model.run(Text)

# Run the keywords model
keywords = keywords_model.run(syntax_prediction, noun_phrases, limit=2)
keywords
```




    {
      "keywords": [
        {
          "text": "Black-on-black ware",
          "relevance": 0.878974,
          "mentions": [
            {
              "begin": 42,
              "end": 61,
              "text": "Black-on-black ware"
            },
            {
              "begin": 637,
              "end": 656,
              "text": "black-on-black ware"
            },
            {
              "begin": 759,
              "end": 778,
              "text": "black-on-black ware"
            }
          ],
          "count": 3
        },
        {
          "text": "Traditional reduction-fired blackware",
          "relevance": 0.689187,
          "mentions": [
            {
              "begin": 190,
              "end": 227,
              "text": "Traditional reduction-fired blackware"
            }
          ],
          "count": 1
        }
      ],
      "producer_id": {
        "name": "Text Rank Keywords",
        "version": "0.0.1"
      }
    }




```python
# #Watson NLP
# entity_mentions = entity_workflow_model.run(Text1)
# for i in entity_mentions.mentions:
#     print("Text", i.span.text.ljust(15, " "), "Type: ", i.type)
```

## 2. PII Extraction 


```python
def extract_PII(text):
    response = openai.Completion.create(
        engine="text-davinci-003",
        prompt=f"Extract PII from the following text:\n{text}\n---\nENTITY_TYPE: ENTITY_NAME",
        max_tokens=50,
        n=1,
        stop=None,
        temperature=0.5,
    )

    PII = response.choices[0].text.strip().split("\n")
    return PII
```


```python
Text2 = "My name is Robert Aragon, and my social security number is 489-36-8350. Here's the number to my Visa credit card number 378282246310005, my driving licence number is A45343748 and email id is sample.mail@email.com, I am working as analysts, my tax identification number is 986-85-4164"
Text2
```




    "My name is Robert Aragon, and my social security number is 489-36-8350. Here's the number to my Visa credit card number 378282246310005, my driving licence number is A45343748 and email id is sample.mail@email.com, I am working as analysts, my tax identification number is 986-85-4164"




```python
extract_PII(Text2)
```




    ['Name: Robert Aragon',
     'Social Security Number: 489-36-8350',
     'Visa Credit Card Number: 378282246310005',
     'Driving License Number: A45343748',
     'Email ID: sample.mail']



## 3. Emotions-Sentiment and Tone analysis 


```python
def extract_emotions(text):
    response = openai.Completion.create(
        engine="text-davinci-003",
        prompt=f"Extract emotions, Tone and sentiment from the following text:\n{text}\n---\nENTITY_TYPE: ENTITY_NAME",
        max_tokens=50,
        n=1,
        stop=None,
        temperature=0.5,
    )

    emotions = response.choices[0].text.strip().split("\n")
    return emotions
```


```python
Text3 = "I am really excited to go on vacation to the beach next week!"
```


```python
extract_emotions(Text3)
```




    ['Emotion: Excitement', 'Tone: Positive', 'Sentiment: Positive']



## 4. Text Classifications 


```python
def extract_class(text):
    response = openai.Completion.create(
        engine="text-davinci-003",
        prompt=f"Extract Classifications from the following text:\n{text}\n---\nENTITY_TYPE: ENTITY_NAME",
        max_tokens=50,
        n=1,
        stop=None,
        temperature=0.5,
    )

    classes = response.choices[0].text.strip().split("\n")
    return classes
```


```python
Text4="Apple, Facebook, Fedex, Taj Mahel, London"

Text5="Ravichandran Ashwin Creates History In Test Cricket, Earns Massive Praise From Sourav Ganguly"

Text6="I want to buy new laptop"
```


```python
extract_class(Text4)
```




    ['1. Company: Apple',
     '2. Company: Facebook',
     '3. Company: Fedex',
     '4. Landmark: Taj Mahel',
     '5. City: London']




```python
extract_class(Text5)
```




    ['1. Person: Ravichandran Ashwin',
     '2. Person: Sourav Ganguly',
     '3. Event: Creates History In Test Cricket',
     '4. Reaction: Earns Massive Praise']



## 5. Category Extraction 


```python
def extract_Category(text):
    response = openai.Completion.create(
        engine="text-davinci-003",
        prompt=f"Extract Category from the following text:\n{text}\n---\nENTITY_TYPE: ENTITY_NAME",
        max_tokens=50,
        n=1,
        stop=None,
        temperature=0.5,
    )

    Category = response.choices[0].text.strip().split("\n")
    return Category
```


```python
extract_Category(Text5)
```




    ['Sports: Ravichandran Ashwin']




```python
extract_class(Text6)
```




    ['Laptop: New Laptop']



## 6. Extract Adjectives and pronouns 


```python
def extract_adj(text):
    response = openai.Completion.create(
        engine="text-davinci-003",
        prompt=f"Extract Adjectives, Adverbs and pronouns from the following text:\n{text}\n---\nENTITY_TYPE: ENTITY_NAME",
        max_tokens=50,
        n=1,
        stop=None,
        temperature=0.5,
    )

    adj = response.choices[0].text.strip().split("\n")
    return adj
```


```python
extract_adj(Text1)
```




    ['Adjectives: turbo-charged, unique, major, No 1, once in a lifetime',
     'Adverbs: past few months, including, reducing, very',
     'Pronouns: its, she, its, her, I, its']



## 7. Topic Modeling


```python
def extract_Topic(text):
    response = openai.Completion.create(
        engine="text-davinci-003",
        prompt=f"Extract Topic clustering from the following text:\n{text}\n---\nENTITY_TYPE: ENTITY_NAME",
        max_tokens=50,
        n=1,
        stop=None,
        temperature=0.5,
    )

    Topic = response.choices[0].text.strip().split("\n")
    return Topic
```


```python
extract_Topic(Text1)
```




    ['Topic: Reducing Visa Delays for Indians in the United States ',
     'Clustering: US Embassies in Other Countries, Visa Applications Upswing, Priority of Deputy Assistant Secretary, Range of Visas, Massive Demand']



## 8. Text Summerization


```python
def extract_Summary(text):
    response = openai.Completion.create(
        engine="text-davinci-003",
        prompt=f"Extract text summarization from the following text:\n{text}\n---\nENTITY_TYPE: ENTITY_NAME",
        max_tokens=50,
        n=1,
        stop=None,
        temperature=0.5,
    )

    Summary  = response.choices[0].text.strip().split("\n")
    return Summary 
```


```python
Text7 = """As the sanctions typically prohibit entry into the US, and freeze assets with any connection to the country, 
they violate the right to freedom of movement, and the right to not be arbitrarily deprived of property. 

"Fear of US sanctions has led many foreign companies and financial institutions to over-comply in order to reduce their risks. 
This only exacerbates the impact of sanctions on human rights," she said. 

Furthermore, human rights are infringed when US trade bans against certain countries penalize foreign companies for conducting business there. 

“These policies affect labour rights, freedom of movement, and the rights of foreign individuals who may be associated with these companies,
" she said, pointing to the harm caused to the human rights of ordinary citizens who rely on the goods or services these companies provide, 
such as medicines and medical equipment. 

In questioning “the compatibility of this type of imposition of extraterritorial jurisdiction with international human rights standards,
” she called for reflecting on how it impacts the international principle of non-interference in domestic affairs. """
```


```python
extract_Summary(Text7)
```




    [': US sanctions have been imposed on foreign companies and financial institutions, resulting in infringement of human rights such as freedom of movement, labour rights, and the rights of individuals associated with these companies. This has caused harm to ordinary citizens who rely on the goods']



## 9. Concept and Relation Extraction


```python
def extract_conRe(text):
    response = openai.Completion.create(
        engine="text-davinci-003",
        prompt=f"Extract Concept and Relation from the following text:\n{text}\n---\nENTITY_TYPE: ENTITY_NAME",
        max_tokens=50,
        n=1,
        stop=None,
        temperature=0.5,
    )

    con  = response.choices[0].text.strip().split("\n")
    return con 
```


```python
extract_conRe(Text7)
```




    ['Concept: Sanctions, Freedom of Movement, Right to not be arbitrarily deprived of property, US Trade Bans, Labour Rights, Non-interference in Domestic Affairs',
     'Relation: Violate the right to freedom of movement, Violate the']



## 10. Language Detection 


```python
def extract_Language (text):
    response = openai.Completion.create(
        engine="text-davinci-003",
        prompt=f"Extract Language from the following text:\n{text}\n---\nENTITY_TYPE: ENTITY_NAME",
        max_tokens=50,
        n=1,
        stop=None,
        temperature=0.5,
    )

    lang  = response.choices[0].text.strip().split("\n")
    return lang 
```


```python
Text8="Je veux acheter une nouvelle maison d'ici l'année prochaine"
Text9="मैं अगले साल तक एक नया घर खरीदना चाहता हूं"
```


```python
extract_Language(Text8)
```




    ['French']




```python
extract_Language(Text9)
```




    ['Language: Hindi']



## 11. Text Similarity


```python
def extract_same (text):
    response = openai.Completion.create(
        engine="text-davinci-003",
        prompt=f"Extract text similarity from the following text:\n{text}\n---\nENTITY_TYPE: ENTITY_NAME",
        max_tokens=50,
        n=1,
        stop=None,
        temperature=0.5,
    )

    same  = response.choices[0].text.strip().split("\n")
    return same 
```


```python
extract_same(Text1)
```




    ['Text Similarity: High']



## 12. HTML Text Processing 


```python
def extract_HTML (text):
    response = openai.Completion.create(
        engine="text-davinci-003",
        prompt=f"Extract HTML tags and text from the following text:\n{text}\n---\nENTITY_TYPE: ENTITY_NAME",
        max_tokens=50,
        n=1,
        stop=None,
        temperature=0.5,
    )

    HTML  = response.choices[0].text.strip().split("\n")
    return HTML 
```


```python
Text10= """
<Html> <!-- This tag is compulsory for any HTML document. -->   
<Head>  
<!-- The Head tag is used to create a title of web page, CSS syntax for a web page, and helps in written a JavaScript code. -->  
</Head>  
<Strong>

</Strong>
<P> This is the sample text.
</P>
<Body>  
<!-- The Body tag is used to display the content on a web page. In this example we do not specify any content or any tag, so in output nothing will display on the web page. -->  
</Body>  
</Html>  """
```


```python
extract_HTML(Text10)
```




    ['HTML tags: <Html>, <Head>, <Strong>, <P>, <Body>',
     'Text: This is the sample text.']



## 13. Embedding 


```python
def extract_Embedd (text):
    response = openai.Completion.create(
        engine="text-davinci-003",
        prompt=f"Extract Embedding  from the following text:\n{text}\n---\nENTITY_TYPE: ENTITY_NAME",
        max_tokens=50,
        n=1,
        stop=None,
        temperature=0.5,
    )

    Embedd  = response.choices[0].text.strip().split("\n")
    return Embedd 
```


```python
extract_Embedd(Text1)
```




    [':',
     'Embedding: US, India, Julie Stufft, deputy assistant secretary for consular affairs, coronavirus, visas, travel restrictions, wait time, visa operations']



## 14. Language Translation  


```python
def Language_Translator (text):
    response = openai.Completion.create(
        engine="text-davinci-003",
        prompt=f"Language Translation English to French from the following text:\n{text}\n---\nENTITY_TYPE: ENTITY_NAME",
        max_tokens=50,
        n=1,
        stop=None,
        temperature=0.5,
    )

    trans  = response.choices[0].text.strip().split("\n")
    return trans 
```


```python
Text11= "I want to buy new mobile"
```


```python
Language_Translator(Text11)  #Hindi
```




    ['मुझे नया मोबाइल खरीदना है।']




```python
Language_Translator(Text11) #French
```




    ['Je veux acheter un nouveau mobile.']


