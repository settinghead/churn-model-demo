# Predictive Modeling Using Skafos

## Introduction

The purpose of this example is to highlight the utility of Skafos, Metis Machine's data science operationalization and delivery platform. In this example, we will: 

* Build and train a model predicting cell phone [churn](http://www.businessdictionary.com/definition/churn.html) with data that is available on a public S3 bucket. 
* Save this model using the Skafos data engine
* Score new customers using this model and save these scores.
* Access these scores via an API and S3. 

## Functional Architecture + Code

The figure below provides a functional architecture for this process.

![Functional Architecture](functional-architecture.png)

## Pre-requisites

1. [Sign up](https://dashboard.metismachine.io/sign-up) for a Skafos account. _If you do not have a Skafos account, you will not be able to complete this tutorial._
2. [Install skafos on your machine](https://docs.metismachine.io/docs/installation)
3. Authenticate your account via the `skafos auth` command.
4. A working knowledge of how to use git. 
5. In this tutorial, we take advantage of Amazon S3 cloud storage. For information about how to use Amazon S3 buckets, please use the following documentation: 

	* [Working with Amazon S3 buckets](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingBucket.html) 
	* [Creating AWS access keys](https://aws.amazon.com/premiumsupport/knowledge-center/create-access-key/)

## Input Data

The source data for this example is available in a public S3 bucket provided by Metis Machine. _In the steps below, we will describe how to access it. No code modifications are required to access the input data._

This data has been slightly modified from its source, which is freely available and can be found [here](https://www.ibm.com/communities/analytics/watson-analytics-blog/predictive-insights-in-the-telco-customer-churn-data-set/) or [here](https://www.kaggle.com/blastchar/telco-customer-churn/home). 

## Tutorial

In the following step-by-step guide, we will walk you through how to use the code in this repository to run a job on Skafos. Following completion of this tutorial, you should be able to: 

1. Run the existing code and access its output on S3.
2. Replace the provided data and model with your own data and model. 

### Step 1: Fork the repo 

1. [Fork](https://help.github.com/articles/fork-a-repo/) the [churn-model-demo](https://github.com/skafos/churn-model-demo) from Github. This code is freely available as part of the Skafos organization. Note that the README is a copy of these instructions. 
2. Clone the forked repo to your machine, and add an upstream remote to connect to the original repo, if desired.

### Step 2: Initialize your own Skafos project 

Once in top level of the working directory of this project, type: `skafos init` on the command line. This will generate a new `metis.config.yml` file that is tied to your Skafos account and organization. 

### Step 3: Edit the `metis.config.yml` file

Open up this config file and edit the job name and entrypoint to match `metis.config.yml.example` included in the repo. Specifically, the name and entrypoint should look like this: 

``` yaml
name: build-churn-model 
entrypoint: "build-churn-model.py"
```

**Note: Do _not_ edit the project token or job_ids in the .yml file. Otherwise, Skafos will not recognize and run your job.** 

### Step 4: Add a second job to your Skafos project and `metis.config.yml`

In `metis.config.yml.example`, you'll note that there are two jobs: one to build a model, and one to score new users. You will need to add a second job to your Skafos project via the following command on the command line: 

`skafos create job score-new-users`

This will output a job_id on the command line. Copy this job id to your `metis.config.yml` file, again using `metis.config.yml.example` as a template, and including the following:

``` yaml
language: python 
name: score-new-users
entrypoint: "score-new-users.py"
dependencies: ["<job-id for build-churn-model.py>"]
```

This dependency will ensure that new users are not scored until the churn model has been built. If `build-churn-model.py` does not complete, then `score-new-users.py` will not run. Note the quotations that are necessary around the job id on the dependencies line.  

### Step 5: Add `metis.config.yml` to your repo

Now that your `metis.config.yml` file has all the necessary components, add it to the repo, commit, and push.  

### Step 6: Add Skafos to the Github repo
In Steps 3 and 4 above, you initialized a Skafos project so you can run the cloned repo in Skafos. Now, you will need to [add the Skafos app](https://github.com/apps/skafos) to your Github repository. 

To do this, navigate to the Settings page for your organization, click on _Installed GitHub Apps_ to add the Skafos app to this repository. Alternatively, if this repo is not part of an organization, navigate to your _Settings_ page, click on _Applications_, and install the Skafos app. 

### Step 7: Modify the AWS Keys and Private S3 Bucket

In [`common/data.py`](https://github.com/skafos/churn-model-demo/blob/master/common/data.py), the following variables are defined to store the output data scores, model, and project token: 

``` python
# Output models and scores
S3_PRIVATE_BUCKET = "skafos.example.output.data"
CHURN_MODEL_SCORES = "TelcoChurnData/churn_model_scores/scores.csv"
AWS_ACCESS_KEY_ID = os.getenv("AWS_ACCESS_KEY_ID")
AWS_SECRET_ACCESS_KEY = os.getenv("AWS_SECRET_ACCESS_KEY")

# Project Token
KEYSPACE = "91633e3d419e23dc7a2da419"
```

**You will need to create your own private S3 bucket and AWS access keys** Amazon provides documentation for how do to this: 

* [Create an AWS S3 bucket](https://docs.aws.amazon.com/quickstarts/latest/s3backup/step-1-create-bucket.html)
* [AWS access keys](https://aws.amazon.com/premiumsupport/knowledge-center/create-access-key/)

Once you have created these things do the following: 

1. Replace `S3_PRIVATE_BUCKET` with the bucket you just created. 
2. Provide Skafos with your `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` [via the command line](https://docs.metismachine.io/docs/usage#section-setting-environment-variables). `skafos env AWS_ACCESS_KEY_ID --set <key>` and `skafos env AWS_SECRET_ACCESS_KEY --set <key>` will do this. 
3. Update the [`KEYSPACE`](https://github.com/skafos/churn-model-demo/blob/master/common/data.py#L26) to be the `project_token` that was generated with the `metis.config.yml` file. 

### Step 8: Commit and Push All Code Changes to the Github Repo

In step 7, you generated several changes to `common/data.py.` These changes now need to be pushed to Github. In doing so, the Skafos app will pick them up and run both the training and scoring jobs. 

### Step 9: Monitor your jobs

Navigate to [dashboard.metismachine.io](https://dashboard.metismachine.io/) to monitor the status of the job you just pushed. Additional documentation about how to use the dashboard can be found [here](https://docs.metismachine.io/docs/dashboard).

### Step 10: Verify model and scores in S3

Once your job has completed, you can verify that the predictive model itself (in the form of a `.pkl` file) and the scored users (in a `.csv` file) are in the private S3 bucket you specified in Step 7. 

### Step 11: Access scores via API

In addition to data that has been output to S3, this code uses the [Skafos SDK](https://docs.metismachine.io/docs/skafos-sdk) to store scored users in a [Cassandra](https://docs.metismachine.io/docs/skafos-sdk#section-using-cassandra) database. Specifically, the `save_scores` function, will write scored users to a table. 

The scored users in Cassandra can be easily accessed via an API call. Navigating to the root project directory on the command line, type `skafos fetch --table model_scores`. This will return both a list of scores and a cURL command that can be incorporated into applications in the usual fashion to retrieve this data. 

## Next Steps

Now that you have successfully built a predictive model on Skafos and scored new data, you can adapt this code to build your own models.  



 


