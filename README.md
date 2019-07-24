# A standalone Helm charts repository 
This solution allows you to maintain your own custom helm charts repository.

## Mission statement
A Helm Charts repository is constructed on a index.yaml file, and optional charts packaged in a `tgz` tar format. 
You can easily create those file using `helm repo index` command to create the `index.yaml` file and `helm package <chartname>` to package your chart files into a `tgz` file.

However, while developing a chart, it turns a real hassle to repeat this process agian and again.

This project is solving this problem, by automating the packging and deployment process, in a similar way [helm charts official repo](https://github.com/helm/charts) does.


## The How
The flow is as follows:
1. Develop and make changes to your Chart. Make sure to increase the chart vesion number.
2. Push changes to your git repoository. 
3. A CI process is triggered, which packages the desired chart and publish it to the Chart Repository.

## Technology stack
**The repository**

I used [Azure Blob Storage](https://azure.microsoft.com/en-us/services/storage/blobs/) for the repository persistancy. It allows to serve static files, which is exactly what we need for the repo. In addition it's very easy to upload files to it, and you can configure your repo to be private or public.

The following script will create a public blob:
```bash
# create a new resource group(unless you already have one)
az group create --name helmgroup --location "westeurope"

# create a storage account
az storage account create --resource-group helmgroup --name helmstorage --sku Standard_LRS

# create a container called 'repo' which grants public read access for blobs.
az storage container create --name repo --public-access blob

# generate a sas-token which will be used later to upload files.
az storage container generate-sas --name repo --expiry 2030-01-01 --permissions lrw
```

**Your Code**

This scripts expects to find a `charts` folder at the root of your project. If it's located in a different directory, simply chnage the path in the CI script.

Every time you will make changes to your Charts, the CI will automatically package and upload the chart to your chart repo.

**CI**

I used [Azure DevOps Pipelines](https://azure.microsoft.com/en-us/services/devops/pipelines/) to create a CI process which triggers on any changes to the `charts` folder. It can be modified very easily by changing the `triggers` or `path` sections. The CI file is [azure-pipelines.yml](azure-pipelines.yaml). Modify the file and add the variables needed or add the variables from the settings.

## Initialize your repo for the first time
Once your storage account is ready, generate an initial `index.yaml` file, and upload to the storage container:
```bash
cd charts
helm repo index .
az storage blob upload --container-name repo --file index.yaml --name index.yaml
```

Add your repo to helm. This is done only once.

`helm repo add helm-repo https://<storgae-account>.blob.core.windows.net/<container-name>/`

## Using your new repo
Every time a new verion is deployed, to any repo, including the public repo, you first need to perform:

`helm repo update`

And now you can search and install charts:

```bash
helm repo list
helm search <chart-name>
```
