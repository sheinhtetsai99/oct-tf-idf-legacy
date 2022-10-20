# Deployment Guide on AI Core

## Test serving templates (legacy and non-legacy)
1. Ensure that you are logged into internal docker registry on CLI
    ```sh
    docker login sti.common.repositories.cloud.sap
    ```
    username: usually your I number\
    password: create an access token at https://common.repositories.cloud.sap/ui/login/

1. Build and push docker images to docker registry\
    for legacy model:
    ```sh
    docker build --target=legacy-serving -t sti.common.repositories.cloud.sap/text-classifier-inference-legacy .
    docker push sti.common.repositories.cloud.sap/text-classifier-inference-legacy:latest
    ```

    for non-legacy model:
    ```sh
    docker build --target=legacy-serving -t sti.common.repositories.cloud.sap/text-classifier-inference-legacy .
    docker push sti.common.repositories.cloud.sap/text-classifier-inference:latest
    ```

1. Configure yaml templates
    1. Ensure that docker secret with specified name has been created in postman
    1. Ensure that image name has been updated to the image pushed to registry in step 2

1. Initialise git respository, create a `templates` folder, store all templates configured in step 3 in it