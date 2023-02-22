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

