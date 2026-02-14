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
   --capabilities CAPABILITY_IAM \
   --template-body file://eks-cluster-and-vpc.yaml \
   --parameters ParameterKey=ClusterName,ParameterValue=my-eks-cluster
   ```

Commands 3 and 4 clone this GitHub repo and deploy the CloudFormation template which 1) creates a new VPC with 2 private and 2 public subnets, a NAT Gateway and an Internet Gateway, and then creates an EKS Cluster with 2 t3.medium nodes.

5. Step 4 should take about 15 minutes to complete. While it's deploying, create a new ECR Repository. You can do this manually, via the CLI, or using IaC, as we did in step 6. However, here is the command to do this via the AWS CLI:
   ```
   aws ecr create-repository --repository-name detection-container-repository
   ```
7. We'll soon push an image to ECR. First we'll authenticate the docker CLI in our CloudShell instance to ECR. Copy the "repositoryUri" which was logged after command 6. Then run:
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

9. Once the EKS cluster is compelte and it's nodes are ready, we'll need to authetnicate the Kubeneretes CLI (Kubectl) to the cluster in order to interact with it and deploy the DetectionContianer. Try running the following command in CloudShell.
    ```
    aws eks update-kubeconfig --name my-eks-cluster
    ```
    If the command returns a message like "the cluster is in a CREATING state" then it's not ready yet. Try re-running the command in 5 minutes or so, and continue until it gives you a message like "Updated context arn..."


10. Once you're authenticated, it's time time to deploy the DetectionContainer to the cluster. We'll first download the Kubernetes manfiest for the DetectionContainer. We'll also use the sed command to update the image reference to that of our copy of the image in ECR by re-using the RepositoryUri from earlier. 
    ```
    wget https://raw.githubusercontent.com/CrowdStrike/detection-container/refs/heads/main/detections.example.yaml
    sed -i 's|image: quay.io/crowdstrike/detection-container|image:  YOUR-ECR-REPO-URI:latest|g' detections.example.yaml
    ```
    Make sure to replate YOUR-ECR-REPO-URI with the repositoryUri value, without double quotes ("")
    
12. Finally, use kubectl to deploy the DetectionContainer as a Kubernetes Deployment to the EKS cluster.
    ```
    kubectl apply -f detections.example.yaml
    ```


## Installing XDR and the K8s Connector

### Installing the Kubernetes XDR agent
1. If you're installing Cortex XDR, first create an agent installation package in the UI or via the API. Then download the agent installation package and upload it to your cloudshell instnace by selecting the "Actions" button and then the "Upload File" option. This will upload the installer to the directory "/home/cloudshell-user". 

2. The commands that you use to deploy the agent will vary depending on the method that you used to create the installation package.

   For this guide, please download the Kubernetes installer as a yaml file, then run `kubectl apply -f NAME-OF-YOUR-XDR-INSTALLER.yaml`.

3. Once installed, you should see the XDR agent listed in the endpoints tab. You'll also see the detections start to populate over the next hour. 

See Documentation: https://docs-cortex.paloaltonetworks.com/r/Cortex-XDR/9.1/Cortex-XDR-Agent-Administrator-Guide/Install-the-Cortex-XDR-Agent-for-Kubernetes-Hosts 

#### Installing the K8s Connector

  1.  The Kubernetes Connector uses Helm, which you can install by running the following commands in CloudShell: 
   ```
   curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4
   chmod +x get_helm.sh
   ./get_helm.sh
   ```
   Once Helm is installed, you can then deploy the XDR Connector: 
   
   2. You can leverage the official documentation for instructions on how to generate the Kubernetes Connector deployment files: https://docs-cortex.paloaltonetworks.com/r/Cortex-CLOUD/KSPM-Documentation/Onboard-the-Kubernetes-Connector 

   3. Once you create Kubernetes installer, you'll have a YAML file to download and re-upload to CloudShell. Then you can run: 
   ```
   helm repo add cortex https://paloaltonetworks.github.io/cortex-cloud --force-update
   helm upgrade --install konnector cortex/konnector --wait-for-jobs --create-namespace --namespace panw --values K8s-security-profile-panw-SOME-DATE-TIME.values.yaml
   ```
   Make sure to replace the --values flag with the name of your values file that you downloaded from the Kubenretes Connector onboarding page. 

With that you've onboarded the Kubernetes Connector to your cluster!


## Create XDR Profiles and Policies
I suggest that you create an agent settings profile and that you enable "XDR Pro". Then, you can create a custom policy and assign it to your XDR agents. If you added an endpoint tag when you created the XDR agent installation package, this can be a nice way to select your specific XDR agents. 

Follow the instructions to create an Agent Settings profile: 
https://docs-cortex.paloaltonetworks.com/r/Cortex-CLOUD/Cortex-Cloud-Runtime-Security-Documentation/Set-up-agent-settings-profiles 

Follow these instructions to create a policy: 
https://docs-cortex.paloaltonetworks.com/r/Cortex-CLOUD/Cortex-Cloud-Runtime-Security-Documentation/Apply-profiles-to-endpoints 
