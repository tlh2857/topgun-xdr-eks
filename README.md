# Create an EKS Cluster and VPC with CloudFormation
## Plus, deploy CrowdStrike's DetectionContainer and defend it with Cortex XDR 


### Steps: 
1. Create a Free AWS Account if you haven't already
2. Create a cloudshell instance in the region of your choosing
3. Run the following commands:
   ```
   aws cloudformation create-stack \
  --stack-name eks-nodegroup-stack \
  --template-body file://eks-nodegroup-step2.yaml \
  --parameters ParameterKey=ClusterName,ParameterValue=my-eks-cluster
   ```


