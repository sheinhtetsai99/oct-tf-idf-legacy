# Deployment Guide on AI Core for TF-IDF classifier

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
    "name": "xxx", # change name of repo accordingly
    "password": "xxx", # personal access token from github
    "url": "xxx", # git repo url
    "username": "xxx" # github username
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

1. Create deployment in postman with configuration id from previous step as parameters

1. Once deployed, copy the deployment url and make a new post request
    1. Endpoint: <<'deployment url'>>/v1/models/tf-idf-classifier:predict
    1. Body: 
    ```sh
    {
    "texts": [
        "The GeForce RTX™ 2060 is powered by the NVIDIA Turing™ architecture, bringing incredible performance and the power of real-time ray tracing and AI to the latest games and to every gamer. RTX. It’s On.",
        "NVIDIA® TITAN RTX™ is the fastest PC graphics card ever built. It’s powered by the award-winning Turing™ architecture, bringing 130 Tensor TFLOPs of performance, 576 tensor cores, and 24 GB of ultra-fast GDDR6 memory to your PC.",
        "NVIDIA’s newest flagship graphics card is a revolution in gaming realism and performance. Its powerful NVIDIA Turing™ GPU architecture, breakthrough technologies, and 11 GB of next-gen, ultra-fast GDDR6 memory make it the world’s ultimate gaming GPU. RTX. It’s On.",
        "The GeForce® GTX 1660 Ti and 1660 are built with the breakthrough graphics performance of the award-winning NVIDIA Turing™ architecture. Easily upgrade your PC and get performance that rivals the GeForce GTX 1070*, a 16 Series GPU is a blazing-fast supercharger for today’s most popular games, and even faster with modern titles.",
        "Take on today's most challenging, graphics-intensive games without missing a beat. The GeForce GTX 1070 Ti and GeForce GTX 1070 graphics cards deliver the incredible speed and power of NVIDIA Pascal™—the most advanced gaming GPU architecture ever created. This is the ultimate gaming platform. #GameReady.",
        "NVIDIA Jetson Nano enables the development of millions of new small, low-power AI systems. It opens new worlds of embedded IoT applications, including entry-level Network Video Recorders (NVRs), home robots, and intelligent gateways with full analytics capabilities.",
        "In case of multiple occurrences of the maximum values, the indices corresponding to the first occurrence are returned.",
        "I am sure some bashers of Pens fans are pretty confused about the lack of any kind of posts about the recent Pens massacre of the Devils."
    ],
    "options-keys": ["whatever"],
    "options-values": ["whatever"]
    }
    ```
    1. Add JWT token under 'Headers' section in the following format: 'Bearer <<'jwt token'>>'

1. Send request and you should be able to geta response
