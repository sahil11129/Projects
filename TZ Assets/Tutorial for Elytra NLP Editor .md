# Tutorial 

This tutorial explains the basic concepts in the NLP editor for generating the custom rules for detecting Driving License Number.

# Setup the Elyra Visual NLP Editor using Docker container.

You will find below instructions for running the NLP Editor frontend and the AQL processing SystemT backend together inside an all-in-one docker container.

## Prerequsites:

* Docker 
* All-in-one docker container [Download](https://ibm.box.com/s/sw901pmlq27i0bqkb7aaiejolcgflt8q) 

1. Follow steps below to **Run the editor locally**


2. Extract `01-ibm_watson_discovery_nlp_web_tool_frontend_backend-<date>.tar.gz` into a folder of your choice, say `watson_nlp_web_tool`

3. Build the container image
   ```
   cd watson_nlp_web_tool
   docker build -t watson_nlp_web_tool:1.0 .
   ``` 

4. Run the container image with volumes mapped. Note that `/path/to/nlp-editor` is the absolute path to the `nlp-editor` repository (from Step 1).

   ```
   docker run -d -p 8080:8080 --name watson_nlp_web_tool watson_nlp_web_tool:1.0
   ```

5. Open http://localhost:8080 in a web browser. 

<img width="1335" alt="image" src="https://user-images.githubusercontent.com/112084296/220576702-259167c4-cbcf-48d2-9a0c-86f9c2c075e0.png">


# Generate the custom rules for Driving License Number 

## 1. Set up the input document
Expand the Extractors, drag and drop Input Documents on the canvas. Configure with document [Driving License Data](https://ibm.box.com/s/bb9emr7kocc5v705ltohoobh9n4rayei). Click Upload, then Close.

<img width="1337" alt="image" src="https://user-images.githubusercontent.com/112084296/220580308-d5af9213-a16f-4c1c-b5c1-6b694c159b60.png">

## 2. Create a dictionary of Driving License Tag 

Under **Extractors**, drag **Dictionary** on the canvas. Connect its input to the output of **Input Documents**. 
Rename the node to `DLicence` and enter the terms: `Driving` and `Licence`, check the IGNORE CASE option BELOW. Click **Save**.
(Skip the test if you want to detect direct Driving Licence number without any context)

<img width="1344" alt="image" src="https://user-images.githubusercontent.com/112084296/220590601-9636208f-1da1-489c-ab03-5ad315a8677c.png">


## 3. Run the dictionary and see results highlighted

Select the `DLicence` node, and click **Run**.

<img width="1341" alt="image" src="https://user-images.githubusercontent.com/112084296/220591216-fc19fa6a-e25d-44b2-ba81-bf27bf45d91d.png">


## 4. Create a regular expression to capture Driving Licence Number 

In the United States, the driver's license number pattern varies across states, with each state having its own unique set of regulations and formats for assigning the license numbers, this demonstration includes 8 United State's regular expression for captures Driving Licence Number.


|State|Driving License Pattern|
|-----|-----------------------|
|California,Texas & Hawaii| [a-zA-Z]{1}[0-9]{7}|
|Arkansas & South Carolina|[0-9]{9}|
|Alaska & Alabama|[0-9]{7}|
|North Carolina|[0-9]{12}|
|Colorado|[0-9]{2}(-)[0-9]{3}(-)[0-9]{4}|
|New York|[0-9]{3}(-)[0-9]{3}(-)[0-9]{3}|

Combined regular expression: `\b[a-zA-Z]{1}[0-9]{8}(\b)|\b[0-9]{9}($|\b)|\b[0-9]{7}($|\b)|[0-9]{12}($|\b)|[0-9]{2}(-| |)[0-9]{3}(-| |)[0-9]{4}|[0-9]{3}(-| |)[0-9]{3}(-| |)[0-9]{3}`


Under Extractors, drag ReGex to the canvas. Name it `Driving_Licence_Number` and specify the regular expression as explain above. Click Save, then Run. The regular expression captures mentions of Driving Licence Number.

<img width="1337" alt="image" src="https://user-images.githubusercontent.com/112084296/220600986-05fc88bd-5cd9-4427-b060-05538865425b.png">

<img width="1338" alt="image" src="https://user-images.githubusercontent.com/112084296/220601192-87d561b4-8079-49f3-a02e-0121b591a5f8.png">


## 5. Create a sequence to combine the Driving Licence key words and Driving License Number 

Create a sequence called `Driving_Licence_Detect` and specify the pattern as `(<DLicence .DLicence >)<Token>{1,2}(<Driving_Licence_Number.Driving_Licence_Number>)`. Ensure the name of the first attribute is also `RevenueByDivision`, Click **Save** and **Run**.

(<DLicence .DLicence >)<Token>{1,2}(<Driving_Licence_Number.Driving_Licence_Number>)
   


