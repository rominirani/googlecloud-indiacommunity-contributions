
### Google Cloud Vertex AI and dotnet : HCIH App

How can I help — HCIH app

## Overview

The primary objective of this app is to analyze comments and images received through a contact form, create labels from the comments, assign them to the correct categories, and redirect them to the appropriate department. The ultimate goal is to generate engaging tweet that tag high-profile individuals in the relevant area, thereby increasing the outreach and awareness of the issue at hand.

![](https://cdn-images-1.medium.com/max/2000/1*dpBzDS2SI8R81TJPEdESkQ.gif)

## Tech Stack

* ASP.Net Core MVC Project using .Net6 Framework.

* Vertext AI — PaLM2 for text using text-bison model, classification and extraction prompts.

* Google Cloud Run.

## Setup

* Login in to Google Console.

* Locate this dropdown on google console.

![](https://cdn-images-1.medium.com/max/2000/1*p0RYR8JequML8eFQPKwkvA.png)

* Create a new project using plus icon.

![](https://cdn-images-1.medium.com/max/2590/0*Zqf78x-s9zUUTrqE)

* Once the project is created, click into it and locate the create credentials by searching “credentials” in the search bar.

![](https://cdn-images-1.medium.com/max/2000/0*Yx0ROC7XRF7dbaIZ)

* Once you are in the credential section, click “Create Credentials” and view the different options it provides.

![](https://cdn-images-1.medium.com/max/2000/0*I7-xNhIM8NE5F124)

* We will need an API key for working with the Vision API using dotNet as we will be making REST calls using this key within code, so go ahead and click it and follow the prompt to create the API Key.

* Next step is to create a service account. A service account is like a middleman between our application and google cloud account, it will be used to make authorized API calls from our application to google cloud. When we create a service account in google console, we get a unique emailId for that account and a unique key.

## Creating a new Service Account

* Follow the steps discussed in detail here to create a service account [https://cloud.google.com/iam/docs/service-accounts-create](https://cloud.google.com/iam/docs/service-accounts-create)

![](https://cdn-images-1.medium.com/max/2068/1*uHhFQ56NReJriFFAdszUCQ.png)

* Once the service account is created, click into it and locate that email address or account alias you provided and add roles to that service account. This is an important step that will be used at the time of deployment of this project. For this app you will need your service account user to have roles as owner, app engine deployer, vertex AI user.

* Next step is to create an OAuth Client ID as we will need this in the vertex AI API calls from dotnet code. The OAuth Client Id can be located from “create credentials” section as shown below:-

![](https://cdn-images-1.medium.com/max/2000/0*I7-xNhIM8NE5F124)

* When you click the OAuth Client ID from above step, you will see a screen like this, follow the prompt to create a OAuth clientId.

![](https://cdn-images-1.medium.com/max/2000/1*ikRAihi1DTh8oWtwRPrSYQ.png)

* Once you create the OAuth Client, click into it and navigate this section and the download button, to download the JSON file with all the settings in it.

![](https://cdn-images-1.medium.com/max/2000/0*GQUYjOIGoMpkas-Q)

* Once the json file is downloaded to local drive, copy it to your project directory. You can rename the file to be anything, I called it GoogleSettings.json and placed it under the wwwroot folder of my project.

* Make sure to open this file and replace the service account key, email or client Id if needed. If everything looks good, simply save it.

* Finally the last step to do within google cloud console would be to enable the APIs that we will need for this project, so search the name API here and once you locate the API, enable it. For this project we will need Vertex AI and Vision API.

![](https://cdn-images-1.medium.com/max/2154/0*27SHxzie7Ot0EIsM)

## **Technical Nuances**

* In dotnet framework if you are using Vertex AI using OAuth, you will need a bearer token. The tokens provided for OAuth do not live infinitely therefore you will need to build a code to generate a token in real time. For that you can install Google Cloud Auth API into your project and utilize the googlesettings.json file to create tokens.

![](https://cdn-images-1.medium.com/max/2000/0*y0rc5UMmxvbjKk30)

* For using Vision API you need to convert the image to its base64 string and then form a request out of it, you will also need the vision API Key that we have created in step 6 and you can send a request something like this.

![](https://cdn-images-1.medium.com/max/2000/0*2V4OecL0y35GT9Ds)

* Once we get the hang of these calls, we can use and customize them in any way we want. For more specifics please refer to the github demo project located here. Please note that this is a sample repo, just to get you started and give some examples, you will have to adjust the settings to run the project [**https://github.com/mayurilad6/GoogleCloudDemo](https://github.com/mayurilad6/GoogleCloudDemo)**

## **Publish and Deploy**

* We will be publishing this project using the Google cloud run and upload a docker image to it. Create a docker file. The docker file should look something similar like the below screenshot.

![](https://cdn-images-1.medium.com/max/2000/0*oKcJIG03Zvf9XLWa)

* Start the Google Cloud Shell, a remote shell environment in cloud.

![](https://cdn-images-1.medium.com/max/2000/1*VnkmxhiSspNRaUSCVLvrKA.png)

* Go to root directory and clone the git repository. ( Assumption is you have already uploaded the repo to github)

    cd ~/

    git clone https://github.com/mayurilad6/GoogleCloudDemo

We will be using Google Artifact Registry to build and store our Docker Images. Cloud Run is used to deploy the containers in Serverless Architecture.

Create an Artifact Registry Repository. Open the Cloud Shell (use the same terminal opened in previous section).

    gcloud artifacts repositories create vertex-repo --repository-format=docker --location=${REGION} 

Create an environment variable for Artifact Registry repository URL. (replace <projectkey> with your project key.)

    export DOCKER_URL=${REGION}-docker.pkg.dev/${PROJECT_ID}/vertex-repo/hcihelp-image

Build the docker container and tag it with Artifact Registry Repository location. Tagging the Docker image with a repository name configures the docker push command to push the image to a specific location.

    docker build . -t ${DOCKER_URL}

Push the image to Artifact Registry.

    docker push ${DOCKER_URL}

Deploy docker container to Cloud Run.

    gcloud run deploy vertex-summarizer --allow-unauthenticated --platform=managed --region=${REGION} --image=${DOCKER_URL}

## **Final Check**

* Once this is done, you can search for cloud run in search bar and navigate to the cloud run to see the progress of your deployment, when the deployment is done, it will generate a public URL for your web app to be accessed by everyone. Your web app using Vertex AI and Vision API is up and running.

![](https://cdn-images-1.medium.com/max/2000/0*vLDiQ86LLL6xt97J)

![](https://cdn-images-1.medium.com/max/2000/1*dpBzDS2SI8R81TJPEdESkQ.gif)
### [Blog Link](https://medium.com/google-cloud/google-cloud-vertex-ai-and-net-a79abaa4f8c9)
