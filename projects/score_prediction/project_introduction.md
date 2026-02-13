**Project Introduction**

In this project I want to explore the different stages of the MLOps lifeycle and the tools we will use to take a model all the way through those.

The MLOps lifecyle stages can be broken down into:

- Ingestion of Raw Data and Data Engineering
- ML Experimentation
- Model Training
- Deployment of the Model
- Model Serving
- Monitoring

The whole project will be ran on minikube as it will be the easiest way to test and develop without incurring any costs.  In a future iteration it could be that I deploy the whole solution in a cloud environment to explore components using IAC tools and cloud native services.

What we plan to use:

- gitlab - code repo + CICD
- gitlab runners 
- mlflow 
- postgresdb
- seaweedfs
- flux

All helm charts will be deployed and maintained via gitops throuhg flux.  The plan is to bootstrap flux when deploying minikube and then let flux handle the rest via values files for each component.

All secrets will be stored in gitlab encrypted variables and then held within K8s secret objects


![Alt text](images/ec2_image_1.png)

**EC2 Change instance type**

