![](https://codebuild.us-west-2.amazonaws.com/badges?uuid=eyJlbmNyeXB0ZWREYXRhIjoiQStpdGJMVDZ6b3BWODRiOGYvanJhTFhsNnZCVnExS1VxcnRManFSeWNjVndrVGRpV1g0QktxNWZONXZsU05WL3luU1ZQbC9jdnh4TWFKbXJ3emQ2Z1BFPSIsIml2UGFyYW1ldGVyU3BlYyI6IjNmUk00TERiZGlDNisvOEsiLCJtYXRlcmlhbFNldFNlcmlhbCI6MX0%3D&branch=master)
# eks-kubectl-docker
**eks-kubectl-docker** is a docker image with `kubectl` and `aws-iam-authenticator` built in the image.

# Usage
get all the pods from the cluster name `eksdemo`
```bash
docker run -v $HOME/.aws:/root/.aws \
-e REGION=us-west-2 \
-e CLUSTER_NAME=eksdemo \
-ti pahud/eks-kubectl-docker:latest \
kubectl get po
```



# CodeBuild support

You can use `pahud/eks-kubectl-docker` as the custom image for your CodeBuild environment.



## kubectl get pod

Create an IAM Role for CodeBuild with a custom policies

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "eks:DescribeCluster",
            "Resource": "arn:aws:eks:*:*:cluster/*"
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": "eks:ListClusters",
            "Resource": "*"
        }
    ]
}
```



edit `aws-auth` ConfigMap and add the created role Arn in the `system:masters` group

```
kubectl edit -n kube-system cm/aws-auth
```

![](images/01.png)



In CodeBuild, create a project and specify `pahud/eks-kubectl-docker` as the custom image.

![](images/02.png)

Specify the IAM Service role you just created in the previous step

![](images/03.png)



Create `buildspec.yaml` or just specify `/root/bin/entrypoint.sh kubectl get po` as the build command



**Insert build commands** and click **Switch to editor**, enter the build commands as below:

![](images/04.png)



Specify the required environment variables. In this case, we specify our Amazon EKS cluster name as `eksdemo`. Set your correct Amazon EKS cluster name here.

![](images/05.png)



start the build and see the build logs. Check the output of your `kubectl get po` commands.

![](images/06.png)



## kubectl apply or create

You may also let CodeBuild work with the `buildspec.yml` in your github repository. Check [buildspec.yml](./buildspec.yml) in the root repository for your reference. In this sample, we `kubectl apply` a nginx deployment as well as service within CodeBuild environment.

![](images/08.png)

And you'll see the deployment, replicaset, service as well as the pods are all created and running.

![](images/09.png)

## Docker in Docker support

**pahud/eks-kubectl-docker** has docker in docker support, which means you can docker build|pull|tag|push in the docker container and optionally push to Amazon ECR or other git repository. Behind the scene, when you bring up **pahud/eks-kubectl-docker** with **CODEBUILD_BUILD_ID** environment variable available, which is by default available in AWS CodeBuild, it will start the dockerd for you.



Make sure you flag **Privileged** in your CodeBuild Environment setting and specify **pahud/eks-kubectl-docker:latest** as your custom image.

![](images/07.png)

# AWS Fargate Support
TBD


# FAQ

Q: Do I need to specify **REGION**  in my CodeBuild environment variables?

A: No. It will determine the running region from the built-in CodeBuild environment varialble **CODEBUILD_AGENT_ENV_CODEBUILD_REGION**, however, if you specify **REGION**, it will override **CODEBUILD_AGENT_ENV_CODEBUILD_REGION**.

