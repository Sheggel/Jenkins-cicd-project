
# End-to-End DevSecOps Pipeline for a Three-Tier Kubernetes Application using Terraform, AWS EKS, ArgoCD, Prometheus, Grafana, and Jenkins.

![alt text](https://github.com/Sheggel/Jenkins-cicd-project/blob/0b5c8dfaed07b24ac0c73ee2e43185091925e8a0/assets/Jenkins%20Project%20Diagram.png).

## Project Overview 

Designed and implemented a fully automated, production-grade DevOps pipeline to deploy a containerized MERN stack application on Amazon EKS. The solution emphasizes security, observability, and GitOps-driven continuous delivery using industry-standard cloud-native tools.

## Key Architecture & Implementation

### 1. Infrastructure as Code
Provisioned all cloud infrastructure using Terraform, including a private Amazon VPC, Amazon EKS cluster, and a bastion/jump server for secure cluster administration. All infrastructure changes are version-controlled and applied via Jenkins pipelines for repeatability and auditability.

### 2. CI/CD with Jenkins
Configured a Jenkins server on EC2 with AWS CLI, Docker, Terraform, SonarQube, and Trivy. GitHub webhooks trigger automated builds on every commit or pull request.  

### CI Pipeline Stages:
 - Source Checkout: Fetch latest code from GitHub
   - Security & Quality: OWASP Dependency-Check and SonarQube static code analysis
   - File System Scanning: Vulnerability scan of source files
   - Containerization: Build Docker image for the MERN application
   - Image Scanning: Trivy scan for CVEs before deployment
   - Registry Push: Push verified images to Amazon ECR
   - Manifest Update: Automatically bump image tags in Kubernetes manifests
### 3. GitOps Continuous Delivery
Integrated Argo CD for GitOps-based deployment. Argo CD monitors the manifest repository and synchronizes changes to the EKS cluster, ensuring declarative, auditable, and rollback-friendly deployments.
### 4. Ingress & Traffic Management
Deployed AWS Load Balancer Controller to manage Application Load Balancers. ALB serves as the Kubernetes Ingress, routing external traffic to application services with SSL termination and path-based routing.
### 5. Monitoring & Observability
 Deployed Prometheus on EKS for metrics collection and Grafana for visualization. Set up dashboards and alerts to monitor cluster health, application performance, and resource utilization in real time.
### Key Technologies:
 AWS EKS, EC2, VPC, ECR, ALB, Terraform, Jenkins, Docker, Kubernetes, Argo CD, SonarQube, Trivy, OWASP Dependency-Check, Prometheus, Grafana, GitHub
### Impact  
 This project establishes a secure, scalable, and fully automated deployment workflow. It enforces security scanning at multiple stages, reduces manual intervention to zero, enables rapid rollbacks, and provides complete visibility into infrastructure and application health.
### Prerequisites:
 Before starting the project, ensure you have the following prerequisites:
 - An AWS account with the necessary permissions to create resources.
- Terraform and AWS CLI are installed on your local machine.
- Basic familiarity with Kubernetes, Docker, Jenkins, and DevOps principles.
## Step 1: We need to create an IAM user and generate the AWS Access key
Create a new IAM User on AWS and give it AdministratorAccess for testing purposes (not recommended for your organisation's Projects)

Go to the AWS IAM Service and click on **Users.**

![External Image](https://miro.medium.com/v2/resize:fit:720/format:webp/0*V9SD-s0nd8UGjjol).

Click on **Create user**

![App Screenshot](https://miro.medium.com/v2/resize:fit:750/format:webp/0*YkmJMmPJLp_C3cPZ).

Provide the name to your user and click on **Next**

![App Screenshot](https://miro.medium.com/v2/resize:fit:750/format:webp/0*JHAaZKv7GxGK_nLk).

Select the **Attach policies directly** option and search for **AdministratorAccess**, then select it.

Click on **Next**.

![App Screenshot](https://miro.medium.com/v2/resize:fit:750/format:webp/0*WkJnqN_wwmSwaaaC).

Click on Create **user**

![App Screenshot](https://miro.medium.com/v2/resize:fit:750/format:webp/0*rqG8tMLvYrebO2FE).

Now, select your created user, then click on **Security credentials** and generate an access key by clicking on **Create access key**.

![App Screenshot](https://miro.medium.com/v2/resize:fit:750/format:webp/0*2fc2AxuLIV7jmbOR).

Select the **Command Line Interface (CLI)**, then select the check mark for the confirmation and click on **Next**.

![App Screenshot](https://miro.medium.com/v2/resize:fit:750/format:webp/0*aZMrjWaoxKsyQ7Xm).

Provide the **Description** and click on the **Create access key**.

![App Screenshot](https://miro.medium.com/v2/resize:fit:750/format:webp/0*sHZC9cDdCEyiHSLS).

Here, you will see that you got the credentials, and you can also download the CSV file for the future.

![App Screenshot](https://miro.medium.com/v2/resize:fit:750/format:webp/0*fOjOS5briseP-tVx).

## Step 2: We will install Terraform & AWS CLI to deploy our Jenkins Server(EC2) on AWS.
Install & Configure Terraform and AWS CLI on your local machine to create a Jenkins Server on AWS Cloud

### Terraform Installation Script

```bash
  wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg - dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update
sudo apt install terraform -y
```
### AWSCLI Installation Script

```bash
  curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip -y
unzip awscliv2.zip
sudo ./aws/install
```
### Configure AWS CLI
Run the below command, and add your keys
```bash
  aws configure
```

## Step 3: Deploy the Jenkins Server(EC2) using Terraform
Clone the Git repository- https://github.com/Sheggel/Jenkins-cicd-project

Navigate to the **Jenkins-Server-TF**

Do some modifications to the backend.tf file, such as changing the **bucket** name and **dynamodb** table(make sure you have created both manually on AWS Cloud). Also, you have to replace the **PEM file** name as you have some other name for your PEM file.

Initialise the backend by running the command below
```bash
  terraform init
```
Run the command below to check the syntax error
```bash
  terraform validate
```
Run the below command to get the blueprint of what kind of AWS services will be created.
```bash
  terraform plan -var-file=variables.tfvars
```
Now, run the below command to create the infrastructure on AWS Cloud, which will take 3 to 4 minutes maximum
```bash
terraform apply -var-file=variables.tfvars --auto-approve
```

We can now connect to your Jenkins server by clicking on **Connect**.

![App Screenshot](https://miro.medium.com/v2/resize:fit:750/format:webp/0*ezFgBNWFgdB7RLKZ).

Copy the **SSH** command and paste it on your local machine.

![App Screenshot](https://miro.medium.com/v2/resize:fit:750/format:webp/0*zmfCKcSKbshUWVxw).

## Step 4: Configure the Jenkins
Now, we logged into our Jenkins server. We have installed some services such as Jenkins, Docker, Sonarqube, Terraform, Kubectl, AWS CLI, and Trivy.

Let’s validate whether all our installed or not.
```bash
jenkins --version
docker --version
docker ps
terraform --version
kubectl version
aws --version
trivy --version
eksctl --version
```
Now, we have to configure Jenkins. So, copy the public IP of your Jenkins Server and paste it into your favourite browser on port 8080.

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*5TbuZn5IlK0B4DQi).

Click on **Install suggested plugins**

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*jf6h7sIpMTvoCIoh).

After installing the plugins, continue as admin

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*xnzj3AqswIMou3Y1).

Click on **Save and Finish**

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*S7lmx0AJUQWeoqRU).

Click on **Start using Jenkins**

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*JqsKBq71DGO-lKEV).

The Jenkins Dashboard will look like the snippet below

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*Cmu3Xv8wYgwPmo1O).

## Step 5: We will deploy the EKS Cluster using the eksctl commands
Now, go back to your Jenkins Server **terminal** and configure the AWS.

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*Fz7D7ZED7AS-f83m).

Go to **Manage Jenkins**

Click on **Plugins**

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*00Y0vchDDh-QOYoe).

Select the **Available plugins**, install the following plugins and click on **Install**

**AWS Credentials**

**Pipeline: AWS Steps**

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*QWxwPmI2YdIeNu-s).

Once both **plugins** are installed, restart your Jenkins service by checking the **Restart Jenkins** option.

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*fRBQ5VF78r04M8y-).

Log in to your Jenkins Server Again

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*tvpY1Iv0DbCu8xsh).

Now, we have to set our **AWS credentials** on Jenkins

Go to **Manage Plugins** and click on **Credentials**

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*xSjY4Gp3hEv3xxIU).

Click on **global**.

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*ykCdz9NneQsOFD3i).

Select **AWS Credentials** as **Kind** and add **the ID** same as shown in the below snippet, except for your AWS Access Key & Secret Access key, and click on **Create**.

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*xmqbCBS8Jl0CY0Bx).

The Credentials will look like the snippet below.

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*VFmP6yGuiY0MvJuc).

Now, we need to add GitHub credentials as well if your repository is Private.

So, add the username and personal access token of your GitHub account.

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*EYXSJ5dvm8CjFPgx).


Create an **eks cluster** using the commands below.
```bash
eksctl create cluster --name Three-Tier-K8s-EKS-Cluster --region us-east-1 --node-type t2.medium --nodes-min 2 --nodes-max 2
aws eks update-kubeconfig --region us-east-1 --name Three-Tier-K8s-EKS-Cluster
```
Once your cluster is created, you can validate whether your nodes are ready or not by using the following command
```bash
kubectl get nodes
```
## Step 6: Now, we will configure the Load Balancer on our EKS because our application will have an ingress controller.
Download the policy for the LoadBalancer prerequisite.
```bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
```
Create the IAM policy using the command below
```bash
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json
```
Create OIDC Provider
```bash
eksctl utils associate-iam-oidc-provider --region=us-east-1 --cluster=Three-Tier-K8s-EKS-Cluster --approve
```
Create a Service Account by using the below command and replace your account ID with your one
```bash
eksctl create iamserviceaccount --cluster=Three-Tier-K8s-EKS-Cluster --namespace=kube-system --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::<your_account_id>:policy/AWSLoadBalancerControllerIAMPolicy --approve --region=us-east-1
```
Run the below command to deploy the AWS Load Balancer Controller
```bash
sudo snap install helm --classic
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=my-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
```
After 2 minutes, run the command below to check whether your pods are running or not.
```bash
kubectl get deployment -n kube-system aws-load-balancer-controller
```
If the pods are getting Error or CrashLoopBackOff, then use the below command
```bash
helm upgrade -i aws-load-balancer-controller eks/aws-load-balancer-controller \
  --set clusterName=<cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=us-west-1 --set vpcId=<vpc#> -n kube-system
```
## Step 7: We need to create Amazon ECR Private Repositories for both Tiers (Frontend & Backend)
Click on **Create repository**

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*i2Y0E-wwltsEDbs_).

Select the Private option to provide the repository and click on **Save**.

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*3FoUALXSU9qjbsYC).

Do the same for the backend repository and click on **Save**

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*LysvSFnlJrbiwfJR).

Now, we have set up our ECR Private Repository

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*kMUuFZTZ5eZPWi6M).

Now, we need to configure ECR locally because we have to upload our images to Amazon ECR.

Copy the **1st command** for login and run the copied command on your **Jenkins Server**.

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*jZatwfI7hXZA8BoH).

Let us create a secret for our ECR Repo by using the below command and then, we will add this secret to the deployment file.
```bash
kubectl create namespace three-tier
```
```bash
kubectl create secret generic ecr-registry-secret \
  --from-file=.dockerconfigjson=${HOME}/.docker/config.json \
  --type=kubernetes.io/dockerconfigjson --namespace three-tier
kubectl get secrets -n three-tier
```
## Step 8: Install & Configure ArgoCD
To do that, create a separate namespace for it and apply the argocd configuration for installation.
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.4.7/manifests/install.yaml
```
All pods must be running. To validate, run the command below
```bash
kubectl get pods -n argocd
```
Now, expose the argoCD server as a LoadBalancer using the below command
```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```
You can validate whether the Load Balancer is created or not by going to the AWS Console

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*0i1RWu1jP9iNQEY5).

To access the argoCD, copy the LoadBalancer DNS and hit it on your favourite browser.

You will get a warning like the snippet below.

Click on **Advanced**.

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*XFz4syg6yPbraayd).

Click on the link below, which is appearing under **Hide advanced**

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*QH2XTU2oCp81Ir7o).

Now, we need to get the password for our argoCD server to perform the deployment.

To do that, we have a prerequisite, which is **jq.** Install it by the command below.
```bash
sudo apt install jq -y
```
```bash
export ARGOCD_SERVER=$(kubectl get svc argocd-server -n argocd -o json | jq -r '.status.loadBalancer.ingress[0].hostname')
export ARGO_PWD=$(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d)
```
Enter the username and password in argoCD and click on **SIGN IN.**

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*Cih_eJ5o_4qpyUNj).

Here is our ArgoCD **Dashboard.**

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*dAa4bJlR_pWCCtmP).

## Step 9: Now, we have to configure SonarQube for our DevSecOps Pipeline
To do that, copy your Jenkins Server public IP and paste it into your favourite browser with a 9000 port

The username and password will be **admin**

Click on **Log In.**

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*VLQi4ARw2u2sAyMp).

Update the **password**

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*Y_Sk-S4usHdFuwCJ).

Click on **Administration**, then **Security**, and select **Users**

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*BJBUJhn7kdWLfINO).

Click on **Update tokens**

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*rNJI6nC-Bl4D_ZT6).

Click on **Generate**

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*qT2RYD0tJgI5MU9X).

Copy the **token**, keep it somewhere safe and click on **Done.**

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*bt8XAF4MROuWki_u).

Now, we have to configure **webhooks** for quality checks.

Click on **Administration**, then Configuration, and select **Webhooks**

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*bQ9cOkumcl4ZDh5L).

Click on **Create**

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*qNxFFvdZLeKqGws6).

Provide the name of your project and in the URL, provide the Jenkins server public IP with port 8080, add sonarqube-webhook in the suffix, and click on Create.

http://<jenkins-server-public-ip>:8080/sonarqube-webhook/

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*mwNvzfs2bgMS-eRG).

Here, you can see the **webhook.**

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*-KfhYHTFcevfWsFx).

Now, we have to create a Project for the frontend code.

Click on **Manually.**

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*cYQHP_vVN9KuSmPq).

Provide the display name to your **Project** and click on **Setup**

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*6TmbHaWHpD5kKUN2).

Click on **Locally.**

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*u4i-WFHilywDXusS).

Select the **Use existing token** and click on **Continue.**

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*07c-DyJf_aBZMrfA).

Select **Other** and **Linux** as **OS.**

After performing the above steps, you will get the command, which you can see in the snippet below.

Now, use the command in the Jenkins Frontend Pipeline where Code Quality Analysis will be performed.

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*J4cL6zQNpLpO4xpy).

Now, we have to create a Project for the backend code.

Click on **Create Project.**

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*90X-HvJMVGufXUws).

Provide the name of your project and click on **Set up.**

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*678-zmRHMUwibqGS).

Click on **Locally.**

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*2Ym3JEFOAiiOrjBT).

Select the **Use existing token** and click on **Continue.**

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*dtXMJxBjpmWRE_J1).

Select **Other** and **Linux** as **OS.**

After performing the above steps, you will get the command, which you can see in the snippet below.

Now, use the command in the Jenkins Backend Pipeline where Code Quality Analysis will be performed.

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*7ncu2YvpCOrSsmZS).

Now, we have to store the sonar credentials.

Go to **Dashboard** -> **Manage Jenkins** -> **Credentials**

Select the kind as **Secret text**, paste your token in **Secret** and keep other things as it is.

Click on **Create**

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*16qKLACxpe53NHhX).

Now, we have to store the GitHub Personal access token to push the deployment file, which will be modified in the pipeline itself for the ECR image.

**Add GitHub credentials**

Select the kind as **Secret text** and paste your GitHub Personal access token(not password) in Secret, and keep other things as it is.

Click on **Create**

**Note:** If you haven’t generated your token, you generate it first, then paste it into the Jenkins

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*PmFud39CH5O2peL0).

Now, according to our Pipeline, we need to add an Account ID in the Jenkins credentials because of the ECR repo URI.

Select the kind as **Secret text**, paste your AWS Account ID in Secret and keep other things as it is.

Click on **Create**

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*TI-h4azZp1BxYDPT).

Now, we need to provide our ECR image name for the frontend, which is **frontend** only.

Select the kind as **Secret text**, paste your frontend repo name in Secret and keep other things as it is.

Click on **Create**

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*xQ3RN8HFk6LRQ3xP).

Now, we need to provide our ECR image name for the backend, which is **backend** only.

Select the kind as **Secret text**, paste your backend repo name in Secret, and keep other things as it is.

Click on **Create**

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*Od9oUAzatYi7pf2I).

## Step 10: Install the required plugins and configure the plugins to deploy our Three-Tier Application
Install the following plugins by going to **Dashboard** -> **Manage Jenkins** -> **Plugins** -> **Available Plugins**
```bash
Docker
Docker Commons
Docker Pipeline
Docker API
docker-build-step
Eclipse Temurin installer
NodeJS
OWASP Dependency-Check
SonarQube Scanner
```

Now, we have to configure the installed plugins.

Go to **Dashboard** -> **Manage Jenkins** -> **Tools**

We are configuring **JDK**

Search for **JDK** and provide the configuration like the snippet below.

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*7RsMr58tNsVzJG9o).

Now, we will configure the SonarQube scanner

Search for the SonarQube scanner and provide the configuration like the snippet below.

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*_mU6-Vd06Lz83g_b).

Now, we will configure **nodejs**

Search for the **node** and provide the configuration, like the snippet below.

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*b2z_rnTavmEfQul5).

Now, we will configure the OWASP Dependency Check

Search for **Dependency-Check** and provide the configuration like the snippet below.

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*MZoquLi-h9BopM-f).

Now, we will configure the **Docker**

Search for **Docker** and provide the configuration like the snippet below.

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*-UKXqzEodSOl0KfC).

Now, we have to set the path for **SonarQube** in **Jenkins**

Go to **Dashboard** -> **Manage Jenkins** -> **System**

Search for **SonarQube installations**

Provide the name as it is, then in the Server URL, copy the SonarQube public IP (same as Jenkins) with port 9000, select the Sonar token that we have added recently, and click on Apply & Save.

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*xVxL6x-iO_2E6PJY).

Now, we are ready to create our Jenkins Pipeline to deploy our Backend Code.

Go to Jenkins **Dashboard**

Click on **New Item**

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*ZBFtbNPtlyCanrmr).

Provide the name of your **Pipeline** and click on **OK**.

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*livXCTbX_MR1yIbq).

This is the Jenkins file to deploy the Backend Code on **EKS**.

Copy and paste it into the **Jenkins**

https://github.com/Sheggel/Jenkins-cicd-project/blob/main/Jenkins-Pipeline-Code/Jenkinsfile-Backend

Click **Apply & Save**.

![App Screenshot](https://miro.medium.com/v2/resize:fit:720/format:webp/0*NuBmt1dUWJsObdZj).

Now, click on the **build**.

Our **pipeline** was **successful** after addressing a few common mistakes.

**Note**: Do the changes in the Pipeline according to your project.

![External Image](https://miro.medium.com/v2/resize:fit:720/format:webp/0*asHSELq69aMhiS2c).

Now, we are ready to create our Jenkins Pipeline to deploy our Frontend Code.

Go to Jenkins **Dashboard**

Click on **New Item**

Provide the name of your **Pipeline** and click on **OK**.

![External Image](https://miro.medium.com/v2/resize:fit:720/format:webp/0*-e4NX_4iBeF-qyYf).

This is the Jenkins file to deploy the Frontend Code on **EKS**.

Copy and paste it into the **Jenkins**

https://github.com/Sheggel/Jenkins-cicd-project/blob/main/Jenkins-Pipeline-Code/Jenkinsfile-Frontend

Click **Apply & Save**.

![External Image](https://miro.medium.com/v2/resize:fit:720/format:webp/0*-wGcfCCl4ZGH8Ghn).

Now, click on the **build**.

Our **pipeline** was **successful** after a few common mistakes.

**Note**: Do the changes in the Pipeline according to your project.

![External Image](https://miro.medium.com/v2/resize:fit:720/format:webp/0*5zV2Mm-0Dy-3tv01).

### Setup 10: We will set up the Monitoring for our EKS Cluster. We can monitor the Cluster Specifications and other necessary things.
We will achieve the monitoring using Helm

Add the Prometheus repo by using the command below
```bash
helm repo add stable https://charts.helm.sh/stable
```
Install the Prometheus
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/prometheus
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm install grafana grafana/grafana
```
Now, check the service by the command below
```bash
kubectl get svc
```
Now, we need to access our Prometheus and Grafana consoles from outside of the cluster.

For that, we need to change the Service type from ClusterType to **LoadBalancer**

Edit the **stable-kube-prometheus-sta-prometheus** service
```bash
kubectl edit svc stable-kube-prometheus-sta-prometheus
```
Modification in the 48th line from ClusterType to **LoadBalancer**

![External Image](https://miro.medium.com/v2/resize:fit:720/format:webp/0*n6WrahD4FpcKqUnS).

Edit the **stable-grafana** service
```bash
kubectl edit svc stable-grafana
```
Modification in the 39th line from ClusterType to **LoadBalancer**

![External Image](https://miro.medium.com/v2/resize:fit:720/format:webp/0*DJHsWa0GAKt17iyM).

Now, if you list the service again, then you will see the LoadBalancers' DNS names
```bash
kubectl get svc
```
![External Image](https://miro.medium.com/v2/resize:fit:720/format:webp/0*8FQJWV5UGE-SfOWs).

You can also validate from your console.

![External Image](https://miro.medium.com/v2/resize:fit:720/format:webp/0*n__Cg9-txhe3vo1b).

Now, access your Prometheus Dashboard

Paste the <Prometheus-LB-DNS>:9090 in your favourite browser, and you will see it like this

![External Image](https://miro.medium.com/v2/resize:fit:720/format:webp/0*65gDRAqoz2VsgvhB).

Click on **Status** and select **Target**.

You will see a lot of Targets

![External Image](https://miro.medium.com/v2/resize:fit:720/format:webp/0*wNgdePSYlFcbsU1J).

Now, access your **Grafana Dashboard**

Copy the ALB DNS of Grafana and paste it into your favourite browser.

The username will be **admin**, and the password will be **prom-operator** for your Grafana LogIn.

![External Image](https://miro.medium.com/v2/resize:fit:720/format:webp/0*v-aKBEgp1HjwZbpK).

Now, click on **Data Source**

![External Image](https://miro.medium.com/v2/resize:fit:720/format:webp/0*w5sjCh7_X8WNm5hd).

Select **Prometheus**

![External Image](https://miro.medium.com/v2/resize:fit:720/format:webp/0*R7E0N_Fbf5y2We20).

In the **Connection**, paste your <Prometheus-LB-DNS>:9090.

![External Image](https://miro.medium.com/v2/resize:fit:720/format:webp/0*Z2MqEMYQyIlIRHpj).

If the URL is correct, then you will see a green notification/

Click on **Save** & **test**.

![External Image](https://miro.medium.com/v2/resize:fit:720/format:webp/0*zd4mMa4NkIQPGW0f).

Now, we will create a dashboard to visualise our Kubernetes Cluster Logs.

Click on **Dashboard**.

![External Image](https://miro.medium.com/v2/resize:fit:720/format:webp/0*Dd-95JdO7f7xrg3i).

Once you click on **Dashboard**. You will see a lot of Kubernetes components being monitored.

![External Image](https://miro.medium.com/v2/resize:fit:720/format:webp/0*xjfOA2P0_v8tJps1).

Let’s try to import a type of Kubernetes Dashboard.

Click on **New** and select **Import**

![External Image](https://miro.medium.com/v2/resize:fit:720/format:webp/0*fT_tSwtXLohNWxuG).

Provide **6417** ID and click on **Load**

**Note**: 6417 is a unique ID from Grafana, which is used to monitor and visualise Kubernetes Data

![External Image](https://miro.medium.com/v2/resize:fit:720/format:webp/0*0jCNbY5ZiIOxkoAb).

Select the **data source** that you have created earlier and click on **Import**.

![External Image](https://miro.medium.com/v2/resize:fit:720/format:webp/0*3ZLxhJR-IzihlRG5).

Here, you go.

You can view your Kubernetes Cluster Data.

Feel free to explore the other details of the Kubernetes Cluster.

![External Image](https://miro.medium.com/v2/resize:fit:720/format:webp/0*QtmraIKAuiOOKysG).

## Step 11: We will deploy our Three-Tier Application using ArgoCD.
As our repository is private. So, we need to configure the Private Repository in ArgoCD.

Click on **Settings** and select **Repositories**

![External Image](https://miro.medium.com/v2/resize:fit:720/format:webp/0*iiANkZQawTciUtjd).

Click on **CONNECT REPO USING HTTPS**

![External Image](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*ZZIbX0Qa28KmApre).

Now, provide the repository name where your Manifest files are present.

Provide the username and GitHub Personal Access token and click on **CONNECT**.

If your **Connection Status** is **Successful**, it means the repository connected successfully.

Now, we will create our first application, which will be a database.

Click on **CREATE APPLICATION**.

![External Image](https://miro.medium.com/v2/resize:fit:720/format:webp/0*NRGUVQ8_9Xvg7kCd).

Provide the details as it is provided in the snippet below and scroll down.

![External Image](https://miro.medium.com/v2/resize:fit:720/format:webp/0*7WABYpxZc6zQv9mz).

Select the same repository that you configured in the earlier step.

In the **Path**, provide the location where you have your database Manifest files are presented and provide other details.

Click on **CREATE**.

While your database Application is starting to deploy, we will create an application for the backend.

Select the same repository that you configured in the earlier step.

In the **Path**, provide the location where you have your backend Manifest files are presented and provide other details.

Click on **CREATE**.

While your backend Application is starting to deploy, we will create an application for the frontend.

Select the same repository that you configured in the earlier step.

In the **Path**, provide the location where you have your frontend Manifest files are presented and provide other details.

Click on **CREATE**.

While your frontend Application is starting to deploy, we will create an application for the ingress.

Select the same repository that you configured in the earlier step.

In the **Path**, provide the location where you have your ingress Manifest files are presented and provide other details.

Click on **CREATE**.

Once your Ingress application is deployed. It will create an **Application Load Balancer**

You can check out the load balancer named k8s-three.

![External Image](https://miro.medium.com/v2/resize:fit:720/format:webp/0*3AmniUgnLLoFwbx_).

Now, copy the ALB-DNS and go to your **Route53** and set up public hosted zone for your domain. Point **Alias** to the ALB-DNS

You can see all 4 application deployments in the snippet below.

![External Image](https://miro.medium.com/v2/resize:fit:720/format:webp/0*u7B1vbstPC6fSj-k).
