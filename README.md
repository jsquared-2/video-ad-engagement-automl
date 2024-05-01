<h1 align="center">
    Video Ad Engagement AutoML
</h1>

<br>

<div align="center">
    <img src="https://img.shields.io/badge/License-MIT-green">
</div>

## Introduction
This repository focuses on training a machine learning model using Google's Vertex AI's AutoML feature for tabular data in order to produce a model that predicts how long a user watched a particular ad for. The data source for this project is [Video Ads Engagement Dataset](https://www.kaggle.com/datasets/karnikakapoor/video-ads-engagement-dataset/) on Kaggle.

## Table of Contents
1. [Prerequisites](#prerequisites)
   1. [Docker Desktop](#docker-desktop)
   2. [Google Cloud Platform CLI](#google-cloud-platform-cli)
2. [Getting Started](#getting-started)
   1. [Downloading Repository](#downloading-repository)
   2. [Setting Up Google Cloud Platform Environment](#setting-up-google-cloud-platform-environment)
      1. [Creating the Project](#creating-the-project)
      2. [Enabling Billing](#enabling-billing)
      3. [Creating a Cloud Storage Bucket & Uploading Data](#creating-a-cloud-storage-bucket--uploading-data)
      4. [Creating BigQuery Dataset](#creating-bigquery-dataset)
      5. [Instantiating Jupyter Notebook Environment](#instantiating-jupyter-notebook-environment)
   3. [Running the Notebook](#running-the-notebook)
   4. [Exporting the Model](#exporting-the-model)
      1. [Moving Model Artifact to Cloud Storage](#moving-model-artifact-to-cloud-storage)
      2. [Downloading Model Artifact to Local Machine](#downloading-model-artifact-to-local-machine)
   5. [Making Predictions](#making-predictions)
3. [Cleaning Up](#cleaning-up)
4. [License](#license)

## Prerequisites
In order for this project to work, there are some prerequisite technologies, and environments that need to be set up.

### Docker Desktop
In order to view the exported model from this project [Docker Desktop](https://www.docker.com/products/docker-desktop/) needs to be installed. When exporting models from Google Cloud Platform (GCP) for use on non-cloud machines, the resulting artifact is a model intended to run in a pre-configured Docker container hosted by GCP.

### Google Cloud Platform CLI
As this project is centered around using GCP products, it is recommended to use the [Google Command Line Interface (gcloud CLI)](https://cloud.google.com/cli?hl=en) to interface with the platform from your host machine. Alternatively, the Console (UI) can be used instead. The documentation of this project will focus on use of the CLI.

> **Note:**
> 
> After installing the CLI, it is important to make sure that the contents of the `bin` directory have been added to the `PATH` environment variable. This ensures that the tools can be called within any terminal session without having to work from the installation directory.

<div align="right">
    <a href="#table-of-contents">↑ Back to Top ↑</a>
</div>

## Getting Started
### Downloading Repository
In order to access the contents of the repository locally, you can clone the repository with the following command:

```bash
git clone https://github.com/jsquared-2/video-ad-engagement-automl.git
```

From there `cd` into the directory:
```bash
cd video-ad-engagement-automl
```

<div align="right">
    <a href="#table-of-contents">↑ Back to Top ↑</a>
</div>

### Setting Up Google Cloud Platform Environment
The majority of the resources needed by the AutoML feature will be created when running the cells in the notebook. Other resources such as the project, the BigQuery data source, and the Cloud Storage bucket can be created via the gcloud CLI or the Console.

#### Creating the Project
In order to create a new project called `video-ad-automl`, use the following command:

```bash
gcloud projects create video-ad-automl
```
By default, GCP allows for a maximum of 10 active projects. You can use the following command to see a list of projects under your account:

```bash
gcloud projects list
```

To ensure you are within the correct active project, use the following command:

```bash
gcloud config set project video-ad-automl
```

<div align="right">
    <a href="#table-of-contents">↑ Back to Top ↑</a>
</div>

#### Enabling Billing
Before the remaining steps can be executed, a billing account needs to linked to the project.

To view a list of billing accounts, use the following command:

```bash
gcloud billing accounts list
```

<div align="right">
    <a href="#table-of-contents">↑ Back to Top ↑</a>
</div>

Using the project ID and the billing account ID, run the following command to link the two:

```bash
gcloud billing projects link video-ad-automl --billing-account <account-id>
```

#### Creating a Cloud Storage Bucket & Uploading Data
The data that the model will use needs to initially be stored in a Cloud Storage bucket due to its size. 

The bucket created needs to be created within the same region as where the model is. In this project, it is `us-central1`:

```bash
gcloud storage buckets create gs://video-ad-automl-bucket  --location=us-central1 --default-storage-class=nearline
```

As the data file is larger than the 100MB limit allowed in the Console, the only way to upload the data is via the gcloud CLI.

To upload the file, the `gsutil` CLI tool needs to be used. The following command copies the file from your local file system, and up to the bucket:

```bash
gsutil cp ./data/ad_df.csv gs://video-ad-automl-bucket/data/ad_df.csv
```

<div align="right">
    <a href="#table-of-contents">↑ Back to Top ↑</a>
</div>

#### Creating BigQuery Dataset
While the data can be used from the Cloud Storage bucket, it is a better idea to load it into BigQuery, as it allows for exploring the data.

The first step to getting the data connected, is to create the dataset with the following command:

```bash
bq.cmd mk video_ad_engagement
```

To then load the data from the bucket into BigQuery, use the following command:

```bash
bq.cmd load --autodetect --source_format=CSV video_ad_engagement.ad_engagement gs://video-ad-automl-bucket/data/ad_df.csv
```

To view the properties of the loaded table, you can use the following command:

```bash
bq.cmd show --format=prettyjson video_ad_engagement.ad_engagement
```

<div align="right">
    <a href="#table-of-contents">↑ Back to Top ↑</a>
</div>

#### Instantiating Jupyter Notebook Environment
In order to run the contents of the Jupyter Notebook, an instance of JupyterLab needs to be provisioned. As interacting with the notebook is best done via the Console, this step can be completed via the gcloud CLI or the Console.

The first step in either case is enabling the Notebook API. To do so via the gcloud CLI, use:

```bash
gcloud services enable notebooks.googleapis.com
```

The second service to enable is the Vertex AI Platform API:

```bash
gcloud services enable aiplatform.googleapis.com
```

In order to provision a minimal environment within the us-central region, use the following command:

```bash
gcloud notebooks instances create video-ad-automl-lab --location=us-central1-b --machine-type=n1-standard-2
```

<div align="right">
    <a href="#table-of-contents">↑ Back to Top ↑</a>
</div>

### Running the Notebook
After the instance has been provisioned and is up and running, click on the `OPEN JUPYTERLAB` button within the Console, after navigating to the Vertex AI Workbench product.

> **Note:**
>
> If the instance was created through the CLI tool, then it will show up under the **User-Managed Notebooks** view. If made through the Console, it will show up under the **Instances** view.

Once the notebook instance has loaded, you can use the Git extension on the left-side to clone this repository using the following link:

```md
https://github.com/jsquared-2/video-ad-engagement-automl.git
```

Once the repository has been cloned, navigate to the notebook. If you have used different names, the constants need to be updated. After reviewing that everything looks good, all cells can be run.

After the training job is complete, a model will be available in the Model Registry.

<div align="right">
    <a href="#table-of-contents">↑ Back to Top ↑</a>
</div>

### Exporting the Model
#### Moving Model Artifact to Cloud Storage
To move the model from the Registry to a bucket, visit the Model Registry section in Vertex AI through the Console. Click on the model named `video_ad_engagement_model_v1`, then click on `Export`. This will open up a side-menu asking to specify a location in a storage bucket. Select the bucket named `video-ad-automl-bucket`.

This will kick off a job that will create the model artifact within that bucket. Once the job is complete, you can review the bucket via the Cloud Storage Bucket product in the Console.

<div align="right">
    <a href="#table-of-contents">↑ Back to Top ↑</a>
</div>

#### Downloading Model Artifact to Local Machine
To ensure compatibility with the local file system, it is good idea to rename the directories containing the model information.

The following command renames the top-level directory, and the directory with the timestamp to a more readable format:

```bash
gsutil mv gs://video-ad-automl-bucket/model-<id>/tf-saved-model/<timestamp>/ gs://video-ad-automl-bucket/model-v1/tf-saved-model/properties/
```

After renaming the directories, the files can be downloaded recursively to your local machine using the following command:

```bash
gsutil cp -r gs://video-ad-automl-bucket/model-v1/tf-saved-model/properties ./models
```

<div align="right">
    <a href="#table-of-contents">↑ Back to Top ↑</a>
</div>

### Making Predictions
With the model downloaded to the specified path, running the following command will start up a Docker container that allows for querying the model for predictions via a REST API:

```bash
docker-compose up -d
```

The model will be available for making predictions on `localhost:8080/predict`.

Below is a screenshot of a sample POST request to the endpoint via [Postman](https://www.postman.com/). 

![postman_demo](/assets/images/postman_demo.png)

The JSON payload used was:

```json
{
    "instances": [
        {
            "auction_id": "0008b046-b675-4f51-8ad6-fe06e5d81f8e",
            "timestamp": "1517334694",
            "creative_duration": "25",
            "creative_id": "198280",
            "campaign_id": "210671",
            "advertiser_id": 7109,
            "placement_id": "47216",
            "placement_language": "fr",
            "website_id": "31838",
            "referer_deep_three": "de/golf/publish",
            "ua_country": "ch",
            "ua_os": "Windows",
            "ua_browser": "Microsoft Edge",
            "ua_browser_version": 16,
            "ua_device": "PersonalComputer",
            "user_average_seconds_played": 2
        }
    ]
}
```

With the returned response being:

```json
{
    "predictions": [
        {
            "value": 3.995027780532837,
            "lower_bound": 0.025828728452324867,
            "upper_bound": 24.361164093017578
        }
    ]
}
```

<div align="right">
    <a href="#table-of-contents">↑ Back to Top ↑</a>
</div>

## Cleaning Up
The simplest way to clean up all resources related to a project is to delete the project. The command to delete the project is as follows:

```bash
gcloud projects delete video-ad-automl
```

<div align="right">
    <a href="#table-of-contents">↑ Back to Top ↑</a>
</div>

## License
MIT