# Create an EKS Cluster and VPC with CloudFormation
## Plus, deploy CrowdStrike's DetectionContainer and defend it with Cortex XDR 


### Steps: 
1. Create a Free AWS Account if you haven't already
2. Create a cloudshell instance in the region of your choosing
3. In CloudShell, run:
   ```
   git clone https://github.com/tlh2857/topgun-xdr-eks.git
   cd topgun-xdr-eks
4. Run the following commands:
   ```
   aws cloudformation create-stack \
   --stack-name eks-nodegroup-stack \
   --template-body file://eks-cluster-and-vpc.yaml \
   --parameters ParameterKey=ClusterName,ParameterValue=my-eks-cluster
   ```

Commands 3 and 4 clone this GitHub repo and deploy the CloudFormation template which 1) creates a new VPC with 2 private and 2 public subnets, a NAT Gateway and an Internet Gateway, and then creates an EKS Cluster with 2 t3.medium nodes.

5. Step 4 should take about 15 minutes to complete. While it's deploying, create a new ECR Repository. You can do this manually, via the CLI, or using IaC, as we did in step 4. However, here is the command to do this via the AWS CLI:
  ```
   aws ecr create-repository --repository-name detection-container-repository
  ```
6. We'll soon push an image to ECR. First we'll authenticate the docker CLI in our CloudShell instance to ECR. Copy the "repositoryUri" which was logged after command 6. Then run:
   ```
   aws ecr get-login-password | docker login --username AWS --password-stdin YOUR-ECR-REPO-URI
   ```
   Make sure to replate YOUR-ECR-REPO-URI with the repositoryUri value, without double quotes ("")
8. Let's pull the DetectionContainer image, tag it, and upload it to the ECR repository that we just created. In CloudShell run:
   ```
   docker pull quay.io/crowdstrike/detection-container
   docker tag quay.io/crowdstrike/detection-container YOUR-ECR-REPO-URI:latest
   docker push YOUR-ECR-REPO-URI:latest
   ```
   Make sure to replate YOUR-ECR-REPO-URI with the repositoryUri value, without double quotes ("")
   
   BTW, You could also build it from scratch using the Dockefile that's included with the GitHub repository for DetectionContainer. However, for simplicity, I chose to let you just build it and push it. Feel free to do otherwise. https://github.com/CrowdStrike/detection-container




