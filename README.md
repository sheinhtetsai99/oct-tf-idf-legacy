# Deployment Guide on AI Core

## Deploy models on AI-Core (legacy and non-legacy)
1. Ensure that you are logged into internal docker registry on CLI
    ```sh
    docker login sti.common.repositories.cloud.sap
    ```
    username: usually your I number\
    password: create an access token at https://common.repositories.cloud.sap/ui/login/

1. Build and push docker images to docker registry
    1. for legacy model:
        ```sh
        docker build --target=legacy-serving -t sti.common.repositories.cloud.sap/text-classifier-inference-legacy .
        docker push sti.common.repositories.cloud.sap/text-classifier-inference-legacy:latest
        ```

    1. for non-legacy model:
        ```sh
        docker build --target=serving -t sti.common.repositories.cloud.sap/text-classifier-inference .
        docker push sti.common.repositories.cloud.sap/text-classifier-inference:latest
        ```

1. Configure individual yaml templates
    1. Ensure that docker secret with specified name has been created in postman
    1. Ensure that image name has been updated to the image pushed to registry in step 2

1. Initialise git respository, create a 'templates' folder and store all templates configured in step 3 in it

1. Onboard created git repository in postman with the below as the body
    ```sh
    {
    "path": "templates", # name of the folder you created on github to store templates
    "repositoryUrl": "xxx", # change link to that of your github repo
    "revision": "HEAD",
    "applicationName": "xxx" # change name accordingly
    }
    ```

1. Create an application in postman with the below as the body
    ```sh
    {
    "path": "templates", # name of the folder you created in github to store templates
    "repositoryUrl": "xxx", # change link to that of your github repo
    "revision": "HEAD",
    "applicationName": "xxx" # change name accordingly
    }
    ```

1. Create docker secret registry with the below as the body
    ```sh
    {
        "data": {
            ".dockerconfigjson": "{\"auths\":{\"sti.common.repositories.cloud.sap\":{\"username\":\"xxx\", \"password\":\"xxx\"}}}" # username and password is the same as in step 1 
        },
        "name": "xxx" # change name according to imagepullsecrets name as written in yaml template
    }
    ````

1. Create a S3 bucket on AWS - https://aws.amazon.com/console/

1. Create create folders and upload files according to requirement on S3

1. Create object store secret on postman with body as below
    ```sh
    {
        "name": "xxx", # change name accordingly
        "data": {
            "AWS_ACCESS_KEY_ID": "xxx", # change aws object store id accordingly
            "AWS_SECRET_ACCESS_KEY": "xxx" # change aws access key accordingly
        },
        "type": "S3",
        "bucket": "xxx", # change name of bucket accordingly
        "endpoint": "xxx", # change end point, usually s3.ap-southeast-1.amazonaws.com
        "region": "xxx",# change region accordingly, usually ap-southeast-1
        "pathPrefix": "xxx" # change according to folder structure made in previous step
    }
    ```

1. Register artifact on postman with body as below
    ```sh
        {
            "kind": "model",
            "name": "xxx", # change name accordingly
            "scenarioId": "xxx", # change scenario accordingly
            "url": "ai://<<name of object store secret name in previous step>>/xxx", # change according to folder structure of object store
            "labels": [
                {
                    "key": "ext.ai.sap.com/xxx", # change accordingly
                    "value": "xxx" # change accordingly
                }
            ],
            "description": "xxx" # change accordingly
        }
    ```

1. Create configuration in postman with artifact id from previous step with body as below
    ```sh
    {
        "name": "xxx", # change name accordingly
        "scenarioId": "xxx", # change scenario name accordingly
        "executableId": "xxx", # change executableID acordingly
        "inputArtifactBindings": [
            {
                "key": "modelArtifactId", # ensure that name of artifact binding matches with that in yaml template
                "artifactId": "xxx" # replace with artifact ID from previous step
            }
        ]
    }
    ```





