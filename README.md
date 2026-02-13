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
   --template-body file://eks-nodegroup-step2.yaml \
   --parameters ParameterKey=ClusterName,ParameterValue=my-eks-cluster
   ```

Commands 3 and 4 clone this GitHub repo and deploy the CloudFormation template which 1) creates a new VPC with 2 private and 2 public subnets, a NAT Gateway and an Internet Gateway, and then creates an EKS Cluster with 2 t3.medium nodes.




