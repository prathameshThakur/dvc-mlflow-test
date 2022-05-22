# Data versioning use cases

-  **Data engineering** during data cleaning and dataset preparation

- Test data in **software engineering** to assure the functionality of the software

- Development and testing of database applications

- Ontology versioning for Semantic Web

  

## Why is the exact version of a dataset important?

- Data quality management

- Reproducible training and re-training.

- Automation of testing or deployment

- Use in applications and web services

- Audit

  

# What is [DVC](https://dvc.org/) ?

  

- Tracks datasets and ML projects

- Works with many types of storages(cloud, local, HDFS, HTTP ...)

- Runs on top of a Git repository

- Supports building and running pipelines

  

Our focus will be on dataset tracking!

  

# What is [MLflow](https://mlflow.org/) ?

- MLflow is an open source platform to manage the ML lifecycle, including experimentation, reproducibility, deployment, and a central model registry.

- MLflow currently offers four components:
	
	- MLflow tracking

	- MLflow models

	- MLflow projects

	- MLflow registry

  

# Let's put them together!!

  Here we will use a basic ML project for the demo, you can find one from the examples in [mlflow](https://github.com/mlflow/mlflow) repo..

### 1. Setup

Checkout the docs of [DVC](https://dvc.org/) and [MLflow](https://mlflow.org/) for installation on your machine.

  

The basic dependencies are:

```bash

pip install scikit-learn
	mlflow
	pandas
	dvc
```

### 2. Initialize the repo and dvc

```bash

git init

```

```bash

dvc init

```

### 3. Set the dvc remote

The same way as GitHub provides storage hosting for Git repositories, DVC remotes provide a location to store and share data and models. You can pull data assets created by colleagues from DVC remotes without spending time and resources to build or process them locally. Remote storage can also save space on your local environment – DVC can fetch into the cache directory only the data you need for a specific branch/commit.

DVC supports several types of remote storage: local file system, SSH, Amazon S3, Google Cloud Storage, HTTP, HDFS, among others. Refer to [`dvc remote add`](https://dvc.org/doc/command-reference/remote/add) for more details.

#### [Example: Add a default local remote](https://dvc.org/doc/command-reference/remote#example-add-a-default-local-remote)

We use the  `-d`  (`--default`) option of  [`dvc remote add`](https://dvc.org/doc/command-reference/remote/add)  for this:

```dvc
$ dvc remote add -d myremote /path/to/remote
```

The  project's config file should now look like this:

```ini
['remote "myremote"']
url = /path/to/remote
[core]
remote = myremote
```
You can also add the remote as your cloud storage bucket, then all the data will be dumped there!

### 4.  Add the data to dvc
Once your initial setup is ready and the required files are in place, the next step is to add the data you need to track into dvc, the functionality is very similar to git..
```bash
dvc add <file to track>
```
once you execute the above step, then a `<file name>.dvc` file is created which have some key and identification paramaters
**Eg:**
```yaml
outs:
- md5: a5ae3bb252be359cd87a685f3cdee5e5
  size: 269127
  path: wine-quality.csv
```
### 5. Commit the .dvc file with a tag
```bash
git add <file name>.dvc
```
```bash
git commit -m "message!"
```
The simplest way to switch between the different versions of your data will be by assigning `tags` to the respective commits of the data update at variour stages of the project lifecycle.
```bash
git tag -a <tag name> -m "tag description!"
```
### 6. Push the updates
after the .dvc file is commited now its ready to be pushed 
```bash
dvc push
```
This will create new directory in the remote dvc location;
and now push the changes into github repo 
```bash
git push
```
also we need to push the individual tag
```bash
git push origin main <tag name>
```

#### You can repeate the step 4, 5 & 6 for every change you make in the tracked data!

## Code space
In the `train.py` we have a small code snippet for getting the url of the actual data tracked by DVC by using `get_url` fn in dvc package, it will simply return the url for the remote storage/path in which the file/data is  stored.
```py
import  dvc.api

path = 'wine-quality.csv' # acutal name of the file
repo= 'https://github.com/prathameshThakur/dvc-mlflow-test' # path to the repo
rev = 'data-v1' # version: it can be any tag/ commit id of data to use

data_url = dvc.api.get_url(path, repo, rev)
```
Then use this `data_url` to fetch the data!

We have variour parameters logged in the `mlflow` which will later provide us with a UI for our experiments. 

### Run!
```bash
python train.py {alpha} {l1_ratio}
```

You can switch between the various versions of the data that you tracked with help of DVC, and simply update the `rev` variable in `train.py` with the commit/tag, and then experiment the impact on performance with change in data ; -)

Later the results can be viewed on mlflow by executing:
```bash
mlflow ui
```

## Results

All the experiments/training we make are logged in the mlflow dashboard

![ml-dashboard](https://user-images.githubusercontent.com/56729180/169716335-c88569db-bc46-4985-9953-3839285f3319.png)

Clicking at individual test you will find a detailed view for Description, Parameters
Metrics, Tags & Artifacts

https://user-images.githubusercontent.com/56729180/169716227-dd570822-03a0-47ee-bdb6-e90081222983.mp4

Later we can compare all our experiments together to make conclusions in mlflow!

https://user-images.githubusercontent.com/56729180/169716274-1dd2383f-08b3-4012-9bab-6a9283308213.mp4
