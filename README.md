An elegant, professionally formatted README is provided below, concisely explaining the steps to create the end-to-end Jenkins pipeline from your repository. It also includes the tech stack used.

markdown
# End-to-End CI/CD Pipeline with Jenkins, SonarQube, Docker, and Kubernetes

This repository outlines the steps to build and automate a complete end-to-end CI/CD pipeline. The pipeline fetches code from a GitHub repository, builds it, performs static code analysis, containerizes the application using Docker, and finally deploys it to a Kubernetes cluster managed by Argo CD.

---

## Tech Stack

* **Cloud Provider:** Amazon Web Services (AWS)
* **CI/CD:** Jenkins
* **Code Quality:** SonarQube
* **Containerization:** Docker
* **Orchestration:** Kubernetes (Minikube)
* **GitOps:** Argo CD

---

## Pipeline Setup Guide

### Step 1: Configure AWS EC2 Instance

First, we need a server to host Jenkins and SonarQube.

1.  **Launch an EC2 Instance:** It's recommended to use a `t3.large` instance or equivalent to handle the workload.
2.  **Generate a PEM Key:** Create and download a key pair to securely SSH into your instance.
3.  **Configure Security Group:** Open the following inbound ports in your instance's security group:
    * **Port 8080:** For Jenkins
    * **Port 9000:** For SonarQube
    * **Port 22:** For SSH access

### Step 2: Install Jenkins

Next, SSH into your EC2 instance and install Jenkins.

```bash
# SSH into your EC2 instance
ssh -i "your-key.pem" ubuntu@<your-ec2-public-ip>
````

# Install Java Development Kit (JDK)
```bash
sudo apt update
sudo apt install openjdk-17-jdk -y
````

# Install Jenkins
```bash
curl -fsSL [https://pkg.jenkins.io/debian/jenkins.io-2023.key](https://pkg.jenkins.io/debian/jenkins.io-2023.key) | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  [https://pkg.jenkins.io/debian](https://pkg.jenkins.io/debian) binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins -y
````

### Step 3: Configure Jenkins

Now, let's set up Jenkins and create our pipeline job.

1.  **Access Jenkins:** Open your web browser and navigate to `http://<your-ec2-public-ip>:8080`.
2.  **Initial Setup:** Follow the on-screen instructions to set up the admin user.
3.  **Install Plugins:** Go to **Manage Jenkins** \> **Plugins** and install:
      * Docker Pipeline
      * SonarQube Scanner
4.  **Create Pipeline Job:**
      * Click on **New Item**, enter a name, and select **Pipeline**.
      * In the **Pipeline** section, select **Pipeline script from SCM**.
      * Set **SCM** to **Git** and provide your GitHub repository URL.
      * Specify the **Script Path** as `Jenkinsfile`.
      * Enable **Lightweight checkout**.
      * Click **Save**.

### Step 4: Set Up SonarQube

SonarQube is used for continuous code quality inspection.

```bash
# Switch to the root user
sudo su -

# Create a dedicated user for SonarQube
adduser sonarqube

# Download and set up SonarQube
wget [https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.4.1.88267.zip](https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.4.1.88267.zip)
unzip *.zip
chown -R sonarqube:sonarqube /home/sonarqube/sonarqube-10.4.1.88267
chmod -R 775 /home/sonarqube/sonarqube-10.4.1.88267

# Start the SonarQube server
cd /home/sonarqube/sonarqube-10.4.1.88267/bin/linux-x86-64
./sonar.sh start
```

### Step 5: Integrate Jenkins with SonarQube

1.  **Access SonarQube:** Go to `http://<your-ec2-public-ip>:9000` and log in with `admin` for both username and password.
2.  **Generate a Token:** In SonarQube, go to **My Account** \> **Security** and generate a new token.
3.  **Add SonarQube Credentials to Jenkins:**
      * In Jenkins, navigate to **Manage Jenkins** \> **Credentials**.
      * Click on **(global)** under **System** and then **Add Credentials**.
      * Select **Secret text** as the kind.
      * Paste the SonarQube token into the **Secret** field.
      * Enter `sonarqube` as the **ID**.
      * Click **Create**.

### Step 6: Install Docker

Docker will be used to containerize our application.

```bash
# Install Docker
sudo apt install docker.io -y

# Add Jenkins and ubuntu users to the docker group
sudo usermod -aG docker jenkins
sudo usermod -aG docker ubuntu

# Restart services to apply changes
sudo systemctl restart docker
sudo systemctl restart jenkins
```

### Step 7: Set Up Kubernetes and Argo CD

For deployment, we'll set up a local Kubernetes cluster using Minikube and install Argo CD. This can be done on the same EC2 instance or a separate machine.

1.  **Install `kubectl` and `minikube`**.
2.  **Start Minikube:**
    ```bash
    minikube start --memory=4098 --driver=docker
    ```
3.  **Install Argo CD:** Follow the instructions at [operatorhub.io](https://operatorhub.io/operator/argocd-operator) to install the Argo CD operator.

### Step 8: Final Jenkins Configuration

1.  **Add Credentials:**
      * **Docker Hub:** In Jenkins, add your Docker Hub credentials (**Username with password**).
      * **GitHub:** Add a GitHub personal access token (**Secret text**) for private repositories if needed.
2.  **Update `Jenkinsfile`:** Ensure the SonarQube server URL in your `Jenkinsfile` is correctly set.
3.  **Build Now:** Go to your Jenkins job and click **Build Now**.

### Step 9 & 10: Configure Argo CD

1.  **Apply Argo CD Configuration:**
    ```bash
    kubectl apply -f argo-cd-basic.yaml
    ```
2.  **Expose Argo CD Server:**
    ```bash
    kubectl edit svc argocd-server -n argocd
    # Change type from ClusterIP to NodePort
    ```
3.  **Access Argo CD UI:**
    ```bash
    minikube service argocd-server -n argocd
    ```
    This command will provide a URL to access the Argo CD dashboard.

### Step 11: Deploy Application with Argo CD

1.  **Log in to Argo CD:**
      * The username is `admin`.
      * To get the password, run:
        ```bash
        echo $(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d)
        ```
2.  **Create an Application:**
      * Click on **New App**.
      * **Application Name:** Choose a name.
      * **Project:** `default`.
      * **Sync Policy:** `Automatic`.
      * **Source:** Your GitHub repository URL and the path to your Kubernetes deployment manifests.
      * **Destination:** `https://kubernetes.default.svc`.
      * **Namespace:** `default`.
      * Click **Create**.

### Step 12: Verify Deployment

Check that your application has been successfully deployed.

```bash
# Check deployments and pods in your Kubernetes cluster
kubectl get deployments
kubectl get pods
```

Congratulations\! You have successfully created and automated an end-to-end CI/CD pipeline.

```
```
