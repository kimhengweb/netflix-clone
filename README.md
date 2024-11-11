# netflix-clone-on-kubernetes
![Screenshot 2024-03-18 220830](https://github.com/Eric-Kay/netflix-clone-on-kubernetes/assets/126447235/a37b6ccf-17cf-4505-affc-6c6bd399513b)

Step 1 — Launch an Ubuntu(22.04) T2 Large Instance

Step 2 — Install Jenkins, Docker and Trivy. Create a Sonarqube Container using Docker.

Step 3 — Create a TMDB API Key.

Step 4 — Install Prometheus and Grafana On the new Server.

Step 5 — Install the Prometheus Plugin and Integrate it with the Prometheus server.

Step 6 — Email Integration With Jenkins and Plugin setup.

Step 7 — Install Plugins like JDK, Sonarqube Scanner, Nodejs, and OWASP Dependency Check.

Step 8 — Create a Pipeline Project in Jenkins using a Declarative Pipeline

Step 9 — Install OWASP Dependency Check Plugins

Step 10 — Docker Image Build and Push

Step 11 — Deploy the image using Docker

Step 12 — Kubernetes master and slave setup on Ubuntu (20.04)

Step 13 — Access the Netflix app on the Browser.

Step 14 — Terminate the AWS EC2 Instances.

_Now, let’s get started and dig deeper into each of these steps:_

## STEP1:Launch an Ubuntu(22.04) T2 Large Instance
Launch an AWS T2 Large Instance. Use the image as Ubuntu. You can create a new key pair or use an existing one. Enable HTTP and HTTPS settings in the Security Group and open all ports (not best case to open all ports but just for learning purposes it’s okay).
![Screenshot 2024-03-18 231127](https://github.com/Eric-Kay/netflix-clone-on-kubernetes/assets/126447235/08de7879-94a5-42cf-aa67-0e3496fb0001)

## Step 2 — Install Jenkins, Docker and Trivy

2A — To Install Jenkins

Connect to your console, and enter these commands to Install Jenkins
```bash
vi jenkins.sh #make sure run in Root (or) add at userdata while ec2 launch
```
```bash
#!/bin/bash
sudo apt update -y
#sudo apt upgrade -y
wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc
echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list
sudo apt update -y
sudo apt install temurin-17-jdk -y
/usr/bin/java --version
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
                  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
                  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
                              /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update -y
sudo apt-get install jenkins -y
sudo systemctl start jenkins
sudo systemctl status jenkins
```

```bash
sudo chmod 777 jenkins.sh
./jenkins.sh    # this will installl jenkins
```

Once Jenkins is installed, you will need to go to your AWS EC2 Security Group and open Inbound Port 8080, since Jenkins works on Port 8080.

Now, grab your Public IP Address
```bash
<EC2 Public IP Address:8080>
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

![Screenshot 2024-03-18 232236](https://github.com/Eric-Kay/netflix-clone-on-kubernetes/assets/126447235/37441fee-47da-43b3-8591-0c86eb096624)

Unlock Jenkins using an administrative password and install the suggested plugins.

2B — Install Docker

```bash
sudo apt-get update
sudo apt-get install docker.io -y
sudo usermod -aG docker $USER   #my case is ubuntu
newgrp docker
sudo chmod 777 /var/run/docker.sock
```
After the docker installation, we create a sonarqube container (Remember to add 9000 ports in the security group).
```bash
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```
![Screenshot 2024-03-18 233551](https://github.com/Eric-Kay/netflix-clone-on-kubernetes/assets/126447235/e0f4ec44-8e4e-44b0-a22b-6fd7e841ce3b)

Enter username and password, click on login and change password

```bash
username admin
password admin
```

2C — Install Trivy
```bash
vi trivy.sh
```

Copy and paste on your vi editor

```bash
sudo apt-get install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy -y
```

## Step 3: Create a TMDB API Key

+ Open a web browser and navigate to TMDB (The Movie Database) website.
+ Click on "Login" and create an account.
+ Once logged in, go to your profile and select "Settings."
+ Click on "API" from the left-side panel.
+ Create a new API key by clicking "Create" and accepting the terms and conditions.
+ Provide the required basic details and click "Submit."
+ You will receive your TMDB API key.

## Step 4 — Install Prometheus and Grafana On the new Server

First of all, let’s create a dedicated Linux user sometimes called a system account for Prometheus. Having individual users for each service serves two main purposes:

It is a security measure to reduce the impact in case of an incident with the service.

It simplifies administration as it becomes easier to track down what resources belong to which service.

To create a system user or system account, run the following command:

```bash
sudo useradd \
    --system \
    --no-create-home \
    --shell /bin/false prometheus
```

You can use the curl or wget command to download Prometheus.
```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.47.1/prometheus-2.47.1.linux-amd64.tar.gz
```
Then, we need to extract all Prometheus files from the archive.
```bash
tar -xvf prometheus-2.47.1.linux-amd64.tar.gz
```
simply create a /data directory. Also, you need a folder for Prometheus configuration files.
```bash
sudo mkdir -p /data /etc/prometheus
```
Now, let’s change the directory to Prometheus and move some files.
```bash
cd prometheus-2.47.1.linux-amd64/
```
let’s move the Prometheus binary and a promtool to the /usr/local/bin/. promtool is used to check configuration files and Prometheus rules.
```bash
sudo mv prometheus promtool /usr/local/bin/
```
Move console libraries to the Prometheus configuration directory.
```bash
sudo mv consoles/ console_libraries/ /etc/prometheus/
```
Finally, let’s move the example of the main Prometheus configuration file.
```bash
sudo mv prometheus.yml /etc/prometheus/prometheus.yml
```

To avoid permission issues, you need to set the correct ownership for the /etc/prometheus/ and data directory.

```bash
sudo chown -R prometheus:prometheus /etc/prometheus/ /data/
```
You can delete the archive and a Prometheus folder when you are done.
```bash
cd
rm -rf prometheus-2.47.1.linux-amd64.tar.gz
```

Verify that you can execute the Prometheus binary by running the following command:
```bash
prometheus --version
```
We’re going to use Systemd, which is a system and service manager for Linux operating systems. For that, we need to create a Systemd unit configuration file.
```bash
sudo vim /etc/systemd/system/prometheus.service
```
Insert the Prometheus.service into your vi editor
```bash
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target
StartLimitIntervalSec=500
StartLimitBurst=5
[Service]
User=prometheus
Group=prometheus
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/data \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090 \
  --web.enable-lifecycle
[Install]
WantedBy=multi-user.target
```

To automatically start the Prometheus after reboot, run enable.
```bash
sudo systemctl enable prometheus
```
Then just start the Prometheus.
```bash
sudo systemctl start prometheus
```
To check the status of Prometheus run the following command:
```bash
sudo systemctl status prometheus
```

Now we can try to access it via the browser. I’m going to be using the IP address of the Ubuntu server. You need to append port 9090 to the IP

```bash
<public-ip:9090>
```

![Screenshot 2024-03-19 001637](https://github.com/Eric-Kay/netflix-clone-on-kubernetes/assets/126447235/9a29a6a6-7862-48d6-84ff-1cdd7beac2fe)

Install Node Exporter on Ubuntu 22.04
Next, we’re going to set up and configure Node Exporter to collect Linux system metrics like CPU load and disk I/O. Node Exporter will expose these as Prometheus-style metrics. Since the installation process is very similar, I’m not going to cover as deep as Prometheus.

First, let’s create a system user for Node Exporter by running the following command:

```bash
sudo useradd \
    --system \
    --no-create-home \
    --shell /bin/false node_exporter
```

Use the wget command to download the binar
```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
```

Extract the node exporter from the archive.
```bash
tar -xvf node_exporter-1.6.1.linux-amd64.tar.gz
```
Move binary to the /usr/local/bin.
```bash
sudo mv \
  node_exporter-1.6.1.linux-amd64/node_exporter \
  /usr/local/bin/
```

Clean up, and delete node_exporter archive and a folder.
```bash
rm -rf node_exporter*
```
Verify that you can run the binary.
```bash
node_exporter --version
```

Next, create a similar systemd unit file.
```bash
sudo vim /etc/systemd/system/node_exporter.service
```
Insert the node_exporter.service into your vi editor
```bash
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target
StartLimitIntervalSec=500
StartLimitBurst=5
[Service]
User=node_exporter
Group=node_exporter
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/node_exporter \
    --collector.logind
[Install]
WantedBy=multi-user.target
```
To automatically start the Node Exporter after reboot, enable the service.
```bash
sudo systemctl enable node_exporter
```
Then start the Node Exporter.
```bash
sudo systemctl start node_exporter
```

Check the status of Node Exporter with the following command:
```bash
sudo systemctl status node_exporter
```
To create a static target, you need to add job_name with static_configs.
```bash
sudo vim /etc/prometheus/prometheus.yml
```
Insert the below configuration in prometheus.yml 
```bash
- job_name: node_export
    static_configs:
      - targets: ["localhost:9100"]
```
![Screenshot 2024-03-19 064500](https://github.com/Eric-Kay/netflix-clone-on-kubernetes/assets/126447235/b8c8000f-2bab-4c5e-aa7c-aec36fd1d43f)

By default, Node Exporter will be exposed on port 9100.

Since we enabled lifecycle management via API calls, we can reload the Prometheus config without restarting the service and causing downtime.

Before, restarting check if the config is valid.
```bash
promtool check config /etc/prometheus/prometheus.yml
```

POST request to reload the config.
```bash
curl -X POST http://localhost:9090/-/reload
```
 Go to __STATUS__ --> __TARGETS__ to Check the targets on prometheus browser
```bash
http://<ip>:9090/targets
```
Install Grafana on Ubuntu 22.04
To visualize metrics we can use Grafana. There are many different data sources that Grafana supports, one of them is Prometheus.

First, let’s make sure that all the dependencies are installed.
```bash
sudo apt-get install -y apt-transport-https software-properties-common
```
Next, add the GPG key.
```bash
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
```
Add this repository for stable releases.
```bash
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```
After you add the repository, update and install Garafana.
```bash
sudo apt-get update
```
```bash
sudo apt-get -y install grafana
```
To automatically start the Grafana after reboot, enable the service.
```bash
sudo systemctl enable grafana-server
```
Then start the Grafana.
```bash
sudo systemctl start grafana-server
```
To check the status of Grafana, run the following command:
```bash
sudo systemctl status grafana-server
```

Go to http://<ip>:3000 and log in to the Grafana using default credentials. The username is admin, and the password is admin as well.
```bash
username admin
password admin
```

![Screenshot 2024-03-19 070641](https://github.com/Eric-Kay/netflix-clone-on-kubernetes/assets/126447235/f4e21c44-192a-4458-b107-1105f9dd5e82)
When you log in for the first time, you get the option to change the password

+ To visualize metrics, you need to add a data source first.
+ Click Add data source and select Prometheus.
+ For the URL, enter localhost:9090 and click Save and test. You can see Data source is working.
+ Click on Save and Test.
+ Add Dashboard for a better view.
+ Click on Import Dashboard paste this code 1860 and click on load.
+ Select the Datasource and click on Import.

## Step 5 — Install the Prometheus Plugin and Integrate it with the Prometheus server

Let’s Monitor JENKINS SYSTEM

Need Jenkins up and running machine

Goto Manage Jenkins –> Plugins –> Available Plugins

Search for Prometheus and install it

To create a static target, you need to add job_name with static_configs. go to Prometheus server.
```bash
sudo vim /etc/prometheus/prometheus.yml
```
Paste below code
```bash
- job_name: 'jenkins'
    metrics_path: '/prometheus'
    static_configs:
      - targets: ['<jenkins-ip>:8080']
```
![Screenshot 2024-03-19 071645](https://github.com/Eric-Kay/netflix-clone-on-kubernetes/assets/126447235/e5968064-619e-433d-a5b7-2bb1e9726e3f)

Before, restarting check if the config is valid.
```bash
promtool check config /etc/prometheus/prometheus.yml
```
POST request to reload the config.
```bash
curl -X POST http://localhost:9090/-/reload
```

Let’s add Dashboard for a better view in Grafana

Click On __Dashboard__ –> + __symbol__ –> Import Dashboard

Use Id 9964 and click on load

+ Select the data source and click on Import to see the Detailed overview of Jenkins

## Step 6 — Email Integration With Jenkins and Plugin Setup

+ Install Email Extension Plugin in Jenkins.
+ Go to your Gmail and click on your profile.
+ Then click on Manage Your Google Account –> click on the security tab on the left side panel you will get this page(provide mail password).
+ 2-step verification should be enabled.
+ Search for the app in the search bar you will get app passwords.
+ Click on other and provide your name and click on Generate and copy the password.
+ Once the plugin is installed in Jenkins, click on manage Jenkins –> configure system there under the E-mail Notification section for configuration
+ Click on Manage Jenkins–> credentials and add your mail username and generated password

## Step 7 — Install Plugins like JDK, Sonarqube Scanner, NodeJs, OWASP Dependency Check

### 7A — Install Plugin

Goto Manage Jenkins →Plugins → Available Plugins →

Install below plugins

1 → Eclipse Temurin Installer (Install without restart)

2 → SonarQube Scanner (Install without restart)

3 → NodeJs Plugin (Install Without restart)

### 7B — Configure Java and Nodejs in Global Tool Configuration

Goto Manage Jenkins → Tools → Install JDK(17) and NodeJs(16)→ Click on Apply and Save

### 7C — Create a Job

## Step 8 — Configure Sonar Server in Manage Jenkins

Grab the Public IP Address of your EC2 Instance, Sonarqube works on Port 9000, so <Public IP>:9000. Goto your Sonarqube Server. Click on Administration → Security → Users → Click on Tokens and Update Token → Give it a name → and click on Generate Token

+ Goto Jenkins Dashboard → Manage Jenkins → Credentials → Add Secret Text.
+ In the Sonarqube Dashboard add a quality gate also Administration–> Configuration–>Webhooks

Add details
```bash
#in url section of quality gate
<http://jenkins-public-ip:8080>/sonarqube-webhook/
```
Let’s go to our jenkins Pipeline and add the script in our Pipeline Script.
```bash
pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
    }
    post {
     always {
        emailext attachLog: true,
            subject: "'${currentBuild.result}'",
            body: "Project: ${env.JOB_NAME}<br/>" +
                "Build Number: ${env.BUILD_NUMBER}<br/>" +
                "URL: ${env.BUILD_URL}<br/>",
            to: 'avworho.eric@gmail.com',
            attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
        }
    }
}

```

## Step 9 — Install OWASP Dependency Check Plugins

+ GotoDashboard → Manage Jenkins → Plugins → OWASP Dependency-Check. Click on it and install it without restart.
+ Configure the Tool Goto Dashboard → Manage Jenkins → Tools →

Now go configure → Pipeline and add this stage to your pipeline and build.
```bash
stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
```

## Step 10 — Docker Image Build and Push

We need to install the Docker tool in our system, Goto Dashboard → Manage Plugins → Available plugins → Search for Docker and install these plugins

+ Docker
+ Docker Commons
+ Docker Pipeline
+ Docker API
+ docker-build-step

Add this stage to Pipeline Script.
```bash
stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       sh "docker build --build-arg TMDB_V3_API_KEY=ff0b559baaea5add32ada10b92749108 -t netflix ."
                       sh "docker tag netflix erickay/netflix:latest "
                       sh "docker push erickay/netflix:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image erickay/netflix:latest > trivyimage.txt"
            }
        }
```

When you log in to your Dockerhub, you will see a new image is created.

![Screenshot 2024-03-19 080043](https://github.com/Eric-Kay/netflix-clone-on-kubernetes/assets/126447235/a97cad01-da37-4e04-adf1-c96a68a6b9cf)



## Step 11 — Kuberenetes Setup

### Take-Two Ubuntu 20.04 instances one for k8s master and the other one for worker.

Install Kubectl on Jenkins machine also.
```bash
sudo apt update
sudo apt install curl
curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client
```


Part 1 ———-Master Node————
```bash
sudo hostnamectl set-hostname K8s-Master
```
———-Worker Node————
```bash
sudo hostnamectl set-hostname K8s-Worker
```

Part 2 ————Both Master & Node ————
```bash
sudo apt-get update
sudo apt-get install -y docker.io
sudo usermod -aG docker $USER 
newgrp docker
sudo chmod 777 /var/run/docker.sock
```

```bash
sudo apt-get update
```

```bash
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
```

```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

```bash
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo snap install kube-apiserver
```

Part 3 ————— Master —————
```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
# in case your in root exit from it and run below commands
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

———-Worker Node————
```bash
sudo kubeadm join <master-node-ip>:<master-node-port> --token <token> --discovery-token-ca-cert-hash <hash>
```

Copy the config file to Jenkins master or the local file manager and save it.

![Screenshot 2024-03-19 082349](https://github.com/Eric-Kay/netflix-clone-on-kubernetes/assets/126447235/f6bec923-7b50-4a87-a46e-d9af315ab471)

+ Copy it and save it in documents or another folder save it as secret-file.txt
+ Create a secret-file.txt in your file explorer save the config in it and use this at the kubernetes credential section.
+ Install Kubernetes Plugin, Once it’s installed successfully
+ Goto manage Jenkins –> manage credentials –> Click on Jenkins global –> add credentials

### Install Node_exporter on both master and worker

create a system user for Node Exporter by running the following command:
```bash
sudo useradd \
    --system \
    --no-create-home \
    --shell /bin/false node_exporter
```

Use the wget command to download the binary.
```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
```

Extract the node exporter from the archive.
```bash
tar -xvf node_exporter-1.6.1.linux-amd64.tar.gz
```

Move binary to the /usr/local/bin.
```bash
sudo mv \
  node_exporter-1.6.1.linux-amd64/node_exporter \
  /usr/local/bin/
```

Clean up, and delete node_exporter archive and a folder.
```bash
rm -rf node_exporter*
```

Next, create a similar systemd unit file.
```bash
sudo vim /etc/systemd/system/node_exporter.service
```

Insert the node_exporter.service configuration into your vi editor
```bash
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target
StartLimitIntervalSec=500
StartLimitBurst=5
[Service]
User=node_exporter
Group=node_exporter
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/node_exporter \
    --collector.logind
[Install]
WantedBy=multi-user.target
```

To automatically start the Node Exporter after reboot, enable the service.
```bash
sudo systemctl enable node_exporter
```

Then start the Node Exporter.
```bash
sudo systemctl start node_exporter
```

Check the status of Node Exporter with the following command:
```bash
sudo systemctl status node_exporter
```

To create a static target, you need to add job_name with static_configs. Go to Prometheus server.
```bash
sudo vim /etc/prometheus/prometheus.yml
```

Insert the below code into the prometheus.yml.
```bash
- job_name: node_export_masterk8s
    static_configs:
      - targets: ["<master-ip>:9100"]
  - job_name: node_export_workerk8s
    static_configs:
      - targets: ["<worker-ip>:9100"]
```

By default, Node Exporter will be exposed on port 9100.
![Screenshot 2024-03-19 085923](https://github.com/Eric-Kay/netflix-clone-on-kubernetes/assets/126447235/2067e2cd-a11f-4d75-85a2-c986843bbeb2)

Before, restarting check if the config is valid.
```bash
promtool check config /etc/prometheus/prometheus.yml
```

POST request to reload the config.
```bash
curl -X POST http://localhost:9090/-/reload
```

Now Run the container by adding the below stage
```bash
stage('Deploy to container'){
            steps{
                sh 'docker run -d --name netflix -p 8081:80 sevenajay/netflix:latest'
            }
        }
```
View your output on your browser wit <Jenkins-public-ip:8081>

final step to deploy on the Kubernetes cluster.
```bash
stage('Deploy to kubernets'){
            steps{
                script{
                    dir('Kubernetes') {
                        withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                                sh 'kubectl apply -f deployment.yml'
                                sh 'kubectl apply -f service.yml'
                        }
                    }
                }
            }
        }
```

## STEP 12:Access from a Web browser with
<public-ip-of-slave:service port>

## Step 13: Terminate instances.
