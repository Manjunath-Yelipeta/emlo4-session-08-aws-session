## EMLOV4-Session-08 Assignment - AWS Crash Course - (Auto Github ECR push, CML to trigger EC2 spot, DVC Repro S3 storage using github actions)

**Abstract: Once github workflow is triggered it develops a docker image with github code content and pushes the image to ECR after it cml is used to trigger EC2 instance and docker image is fetched inside EC2 and used for training, evaluation, inferencing and checkpoint is stored in AWS S3 storage. Also both EC2 instance and spot request are turned off after run**

### Contents

- [Requirements](#requirements)
- [Development Method](#development-method)
    - [DVC Integration with AWS S3](#dvc-integration-with-aws-s3)
    - [Run AWS works manually for testing](#run-aws-works-manually-for-testing)
    - [Building ECR image for development](#building-ecr-image-for-development)
    - [Using CML to trigger EC2 spot instance](#using-cml-to-trigger-ec2-spot-instance)
- [Learnings](#learnings)
- [Results Screenshots](#results-screenshots)

### Requirements

- Build the Docker Image and push to ECR
    - You’ll be using this ECR image for training the model on AWS GPU
    - Make sure you use GPU version of PyTorch
- Connect DVC to use S3 remote
- Train model on EC2 g4dn.xlarge
- Test the model accuracy
- Push the trained model to s3
    - use specific folder and commit id

### Development Method

#### Build Command

**GPU Usage**

- Pass cuda parameter to trainer so that i trains with GPU
- You need to pass `--gpus=all` to docker run command so that it uses host GPU

**Debug Commands for development**

- Since GPU is used training is faster at inital stage you may commit the dataset also with docker file so that you can debug the workflow faster and run `dvc repro -f` command to verify the pipeline. Even for 70 MB it takes 3 minutes so if you use this method you can debug work for cml triggering EC2 spot instance faster. 
- Also i noted that GPU allocated when instance triggered through CML is T4 cuda 11.4 instance. But when I trigger manually throguh AWS UI I am getting T4 cuda 12.1 instance. Only few packages had a facility to launch with ami-id.
- Developed with `uv package` and `--system` in docker.

**Pull data from cloud**

```dvc pull -r myremote```

**Trigger workflow**

```dvc repro```


### DVC Integration with AWS S3

- Set environment variables in docker container and set the S3 bucket path

    ```
    export AWS_ACCESS_KEY_ID='myid'
    export AWS_SECRET_ACCESS_KEY='mysecret'
    dvc remote add -d myremote s3://<bucket>/<key>
    ```

**Reference**

- [Github Blog](https://github.com/ajithvcoder/dvc-gdrive-workflow-setup)
- [Medium blog](https://medium.com/@ajithkumarv/setting-up-a-workflow-with-dvc-google-drive-and-github-actions-f3775de4bf63)
