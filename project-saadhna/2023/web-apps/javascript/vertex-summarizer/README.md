
# Google Cloud Vertex AI and Svelte Kit: Vertex Summarizer

## Overview

Vertex Summarizer is a web app that I created to allow users to summarize articles, text, and other forms of content using Google Cloud Vertex AI.

Google Cloud Services:

-   Vertex AI PaLM 2 API.
-   Cloud Functions.
-   Cloud Build.

![](https://cdn-images-1.medium.com/max/880/1*6GfaMmSsqZm34Fhwz_6vKg.gif)

## Setup

#### Create new Google Cloud Project:

Sign in or sign up to Google Cloud Console. Store the project ID for further use cases.

Start the Google Cloud Shell, a remote shell environment in cloud.

Create Environment variables for REGION and PROJECT ID.
```bash
export PROJECT_ID=<your project id>
```
```bash
export REGION=asia-south1
```
Enabling APIs
```bash
gcloud services enable cloudbuild.googleapis.com \  
run.googleapis.com \  
cloudfunctions.googleapis.com \  
aiplatform.googleapis.com
```
## Vertex AI Cloud Function

### Creating a new Service Account

Create a new service account. (use the same terminal opened in previous section)
```bash
gcloud iam service-accounts create vertex-service-acc
```
To provide access to your project and your resources, grant a role to the service account.
```bash
gcloud projects add-iam-policy-binding ${PROJECT_ID} --member="serviceAccount:vertex-service-acc@${PROJECT_ID}.iam.gserviceaccount.com" --role=roles/ml.developer
```
Grant your Google Account a role that lets you use the service account’s roles and attach the service account to other resources.

Replace USER_EMAIL with your Google Account Email ID.
```bash
gcloud iam service-accounts add-iam-policy-binding vertex-service-acc@${PROJECT_ID}.iam.gserviceaccount.com --member="user:USER_EMAIL" --role=roles/iam.serviceAccountUser
```
### Creating Cloud Functions

A Python Function is used to access the summarize text using PaLM API.

The PaLM 2 for text is ideal for tasks that can be completed with one API response, without the need for continuous conversation.

Create a new directory using cloud shell and navigate to it. (use the same terminal opened in previous section)
```bash
mkdir vertex-ai-functions
```
```bash
cd vertex-ai-functions
```
Create a main.py file for writing cloud function and requirements.txt file for storing dependencies.
```bash
touch main.py requirements.txt
```
This Python file defines a simple HTTP Cloud Function that uses a Vertex AI Text Generation Model to generate short summaries of text inputs. The function takes a text input as a parameter and returns a short summary of the input. The function uses a variety of parameters to control the generation process, such as the creativity, diversity, and fluency of the generated text. The HTTP Cloud Function accepts a request object and returns the model’s summary as the response.

Open Google Cloud Editor and replace main.py contents with this code.

[main.py : Python Code for Vertex AI Cloud Function](https://github.com/bhaaratkrishnan/vertex-summarizer-svelte/blob/master/cloud_functions/main.py)

Overall, this file provides a concise way to generate short summaries of text inputs using Vertex AI.

The requirements.txt file have package dependencies:

-   functions-framework==3.*: Ensures that the function uses the latest features and bug fixes of the Functions Framework.
-   google-cloud-aiplatform: Required to use the Vertex AI Text Generation Model.

Add this to `requirements.txt` file.

[requirements.txt : Python Dependencies File](https://github.com/bhaaratkrishnan/vertex-summarizer-svelte/blob/master/cloud_functions/requirements.txt)

Deploy to Google Cloud Functions.
```bash
gcloud functions deploy vertex-ai-function \  
--gen2 \  
--runtime=python311 \  
--region=${REGION} \  
--source=. \  
--entry-point=hello_vertex \  
--trigger-http \  
--allow-unauthenticated \  
--max-instances=30
```
Use the search bar and go to Cloud Functions Panel.

The function panel will contain the `vertex-ai-function` cloud function and the public URL will be given in the function page. We use this to connect our Frontend and Vertex AI API. Store this URL.

## Vertex Summarizer Web App

The Vertex Summarizer provides a frontend interface to interact with our Vertex AI API through Google Cloud Functions.

### Clone Repository and Setup Dockerfile

Go to root directory and clone the git repository.
```bash
cd ~/
```
```bash
git clone https://github.com/bhaaratkrishnan/vertex-summarizer-svelte.git
```
```bash
cd  vertex-summarizer-svelte
```
To run this application, you need to add the `PUBLIC_FUNCTION_URL` environment variable in docker file. This URL is the Cloud Function URL created and stored in previous section.

Open Cloud Editor and edit contents of Dockerfile. Replace the `PUBLIC_FUNCTION_URL` variable with your Cloud Function URL.

[Dockerfile: Docker Container File for Web Frontend](https://github.com/bhaaratkrishnan/vertex-summarizer-svelte/blob/master/Dockerfile)

### Deploy Frontend to Cloud Run

We will be using Google Artifact Registry to build and store our Docker Images. Cloud Run is used to deploy the containers in Serverless Architecture.

Create an Artifact Registry Repository. Open the Cloud Shell (use the same terminal opened in previous section).
```bash
gcloud artifacts repositories create vertex-repo --repository-format=docker --location=${REGION} 
```
Create an environment variable for Artifact Registry repository URL.
```bash
export DOCKER_URL=${REGION}-docker.pkg.dev/${PROJECT_ID}/vertex-repo/vertex-summarizer-image
```
Build the docker container and tag it with Artifact Registry Repository location. Tagging the Docker image with a repository name configures the `docker push` command to push the image to a specific location.
```bash
docker build . -t ${DOCKER_URL}
```
Push the image to Artifact Registry.
```bash
docker push ${DOCKER_URL}
```
Deploy docker container to Cloud Run.
```bash
gcloud run deploy vertex-summarizer --allow-unauthenticated --platform=managed --region=${REGION} --image=${DOCKER_URL}
```
Yaay !! vertex summarizer is up and running. The URL will be shown in Cloud Shell, so Explore and Enjoy Vertex AI🤖.
