# Amazon EKS CI/CD with AWS CodeBuild

This project helps you build a complete Amazon EKS cluster with nodegroup and CI/CD pipeline with CodeBuild 100% in **AWS CDK**.

![](images/eks-cicd-codebuild.png)

# Resource List

This stack provisions the following resources with **AWS CDK**

- [x] **Amazon EKS cluster**
- [x] **Amazon EKS nodegroup**
- [x] **AWS CodeBuild Project for Amazon EKS CI/CD**
- [x] **AWS CodeCommit as a sample source repo**
- [x] **Amazon ECR repository**



# Usage

Just deploy the stack with AWS CDK

```bash
$ git clone https://github.com/pahud/eks-cicd-codebuild.git
$ cd cdk
# install required packages defined in package.json
$ npm i
# compile typescript to js
$ npm run build 
# deploy the complete stack
$ cdk deploy
```



# Walkthrough

When you complete the `cdk deploy`, an empty **CodeCommit** repository will be created(check `Resource List` above to see all resource being created)

```
Outputs:
CdkStack.ClusterClusterNameEB26049E = cluster-e262edb4-e3f4-4384-82f3-366ea3b341de
CdkStack.ClusterConfigCommand43AAE40F = aws eks update-kubeconfig --name cluster-e262edb4-e3f4-4384-82f3-366ea3b341de --region us-west-2 --role-arn arn:aws:iam::112233445566:role/CdkStack-AdminRole38563C57-1US2EG9014AO1
CdkStack.CodeCommitRepoArn = arn:aws:codecommit:us-west-2:112233445566:CdkStack-repo
CdkStack.CodeCommitRepoName = CdkStack-repo
CdkStack.ClusterGetTokenCommand06AE992E = aws eks get-token --cluster-name cluster-e262edb4-e3f4-4384-82f3-366ea3b341de --region us-west-2 --role-arn arn:aws:iam::112233445566:role/CdkStack-AdminRole38563C57-1US2EG9014AO1
CdkStack.CodeCommitCloneUrlSsh = ssh://git-codecommit.us-west-2.amazonaws.com/v1/repos/CdkStack-repo
CdkStack.CodeCommitCloneUrlHttp = https://git-codecommit.us-west-2.amazonaws.com/v1/repos/CdkStack-repo
```

Verify the Amazon EKS is running **kubectl**

```bash
# copy the 'aws eks update-kubeconfig' command string and run it in the terminal to generate/update the kubeconfig
$ aws eks update-kubeconfig --name cluster-e262edb4-e3f4-4384-82f3-366ea3b341de --region us-west-2 --role-arn arn:aws:iam::112233445566:role/CdkStack-AdminRole38563C57-1US2EG9014AO1
# list the nodes with kubectl
$ kubectl get no
# deploy the initial flask sample service
$ kubectl apply -f flask-docker-app/k8s/flask.yaml
# list the service and deployment
$ kubectl get svc,deploy
```

Copy the ELB dns name from the **EXTERNAL-IP** column and open it in browser.

You will see the **Flask-demo** homepage.

![](images/flask01.png)



```bash
# copy the ELB dns name from the EXTERNAL-IP column and open it in browser.
# You will see the Flask-demo homepage
# set codecommit as another upstream 
$ git remote add codecommit ssh://git-codecommit.us-west-2.amazonaws.com/v1/repos/CdkStack-repo
# push all current repo to codecommit. This will trigger CodeBuild for CI/CD.
$ git push -u codecommit master
```

Check the **CodeBuild** console to see the build status.

On build complete, reload the browser and see the **Flask-demo** homepage again. You will see the running platform string has changed from **Amazon Web Services** to **Amazon EKS**.

![](images/flask02.png)

