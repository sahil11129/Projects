# Summarize the document using Watsonx.ai API

Watsonx.ai is an integral component of the IBM WatsonX platform, which seamlessly integrates cutting-edge generative AI capabilities, harnessed from foundational models and traditional machine learning, into a dynamic studio encompassing the entire AI development lifecycle. With Watsonx.ai, you can train, validate, fine-tune, and deploy generative AI, foundational models, and machine learning capabilities. This streamlined approach allows you to construct AI applications in significantly less time and with reduced data requirements.

Summarization is a valuable approach that enables us to cut through the noise and extract the essence of complex information. Whether dealing with lengthy emails, extensive documents, or dynamic chat interactions, the ability to generate concise summaries is a game-changer. This blog explores how Watsonx.ai takes summarization to the next level. Here we leverage the Watsonx.ai API within a Jupyter Notebook to demonstrate two use-case as chat summarization and mail summarization.

# Prerequisites

## To follow the steps in this tutorial, you need:

### Signing up for IBM Watsonx as a Service

Embarking on the journey of leveraging IBM Watsonx.ai is an exciting prospect, offering a gateway to powerful tools for working with foundation models. Before diving into the Prompt Lab of Watsonx.ai, it's essential to understand the mandatory prerequisites for signing up. Follow these steps for a seamless onboarding experience with IBM Watsonx.ai or refer to the [signup guide](https://dataplatform.cloud.ibm.com/docs/content/wsj/getting-started/signup-wx.html?context=wx&audience=wdp#personal).

1. Go to [Try IBM watsonx.ai](https://eu-de.dataplatform.cloud.ibm.com/registration/stepone?context=wx&apps=data_science_experience,watson_data_platform,cos&uucid=0b526de8c1c419db&utm_content=WXAWW?context=wx&audience=wdp&preselect_region=true)
2. Select the IBM Cloud service region. You can select the Dallas or Frankfurt region.
3. Enter your IBM Cloud account username and password. If you don't have an IBM Cloud account, create one.
4. If you see the <b>Select account</b> screen, select the account and resource group where you want to use watsonx. If you belong to an account with existing services, you can select it instead of your account. The <b>Select account</b> screen does not display if you have only one account and resource group.
5. Click <b>Continue</b>. The account activation process begins.


# Estimated time

It should take you approximately 60 minutes to complete this tutorial.


# Steps

## Generate the Summary Using Watsonx.ai API (Notebook)

## 1. Mail Summarization (Document Summarization)


Before we get started, make sure to create your personal API key (YOUR_ACCESS_TOKEN) on IBM Cloud. Check out the [documentation](https://cloud.ibm.com/docs/account?topic=account-userapikey&interface=ui) for a user-friendly guide on this important step. To execute these exercises, it's essential to set up and run a [Jupyter Notebook](https://github.com/sahil11129/Projects/blob/main/WatsonX/Summarisation%20%20Using%20WatsonX.ai.ipynb).


1.1 Read the document

Here, we can upload the document slated for summarization. Specifically, we have a file named [HealthCare Email.txt](https://github.com/sahil11129/Projects/blob/main/WatsonX/HealthCare%20Email.txt) containing the email content that we'll be summarizing. 


```
with open("HealthCare Email.txt", 'r') as file:
    doc = file.read()
```

1.2 Set header with YOUR_ACCESS_TOKEN

```
headers = {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer YOUR_ACCESS_TOKEN',
}
```

1.3 Set Model ID and Model Parameters 

In the upcoming step, we'll focus on configuring the model by setting its unique Model ID. This involves defining specific model details and instrumentation, providing essential guidance to the model for task specification. Additionally, we'll leverage the document object loaded earlier as the input, coupled with adjusting parameters such as `max_new_tokens`, `min_new_tokens`, and `decoding_method`. This strategic customization ensures that the model is finely tuned to generate responses aligned with the desired outcomes.

```
json_data = {

    'model_id': 'google/flan-ul2',


    'inputs': ['Summarize the email text. \\n\\ntext: '+doc],

        "parameters": {
        "decoding_method": "greedy",
        "max_new_tokens": 100,
        "min_new_tokens": 50,
        "repetition_penalty": 1.5
      },
}
```
1.4 Send API request to the Watsonx.ai server 

This step involves sending a POST request to the Watsonx.ai server's text generation endpoint using the provided URL. The headers variable contains any necessary information for the request, and json_data holds the payload with details required for text generation. The response variable stores the server's response, allowing access to the generated text or additional information provided by the Watsonx.ai server.

```
response = requests.post('https://us-south.ml.cloud.ibm.com/ml/v1-beta/generation/text?version=2023-05-29', headers=headers, json=json_data)
```

1.5 Response from the Watsonx.ai server

The response from the server, including the generated text or any relevant information, is stored in the response variable for further handling and analysis.


![Response](img/result.png)


## 2. Chat Summarization using Watsonx.ai




In this blog, we'll take you through the incredible summarization powers of Watsonx.ai using two  methods: the Prompt Lab and the API. Think of the Prompt Lab as your play space to explore and get to know Watsonx.ai's summarization abilities. Now, if you're eager to bring summarization into your real-world applications, the API method is your go-to, offering a seamless way to integrate this solution for instant use. These approaches highlight how Watsonx.ai caters to both exploration and real-world implementation, making summarization a breeze. For a deeper understanding of Watsonx.ai and its features, please refer to the official [documentation](https://dataplatform.cloud.ibm.com/docs/content/wsj/getting-started/overview-wx.html?context=wx&audience=wdp) provided. 




