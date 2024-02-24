                           REGISTRATION APPLICATION
Requirements:
--> Jenkins
--> Maven
--> Sonarqube
--> Trivy
--> Argocd
--> Prometheus & Grafana
--> 3 servers ,t2.large,ubuntu


                              Jenkins Installations and Configuring Jenkins-Master and Jenkins slave
         
## Install Java
$ sudo apt update
$ sudo apt upgrade
$ sudo nano /etc/hostname                // used to change the hostname //
$ sudo init 6
$ sudo apt install openjdk-17-jre
$ java -version

## Install Jenkins
Refer--https://www.jenkins.io/doc/book/installing/linux/
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins

$ sudo systemctl enable jenkins       //Enable the Jenkins service to start at boot
$ sudo systemctl start jenkins        //Start Jenkins as a service
$ systemctl status jenkins
$ sudo nano /etc/ssh/sshd_config
$ sudo service sshd reload
$ ssh-keygen 
$ cd .ssh
========================================================================================================================

**Real Time Master slave architecture:**

In real time we are not using JNLP jars for this! , So Follow the steps.

Ssh setup:
1.Take 3 server, master, 2 slaves
2.In each machine open the file 
   # cd vim /etc/ssh/sshd_config
   --> uncoment it: PubkeyAuthentication yes
   --> uncoment it: AuthorizedKeysFile  
3.go to master machine and generate keys #ssh-keygen,
 after go to .ssh folder vim id_rsa.pub “ copy the key”
4.go to slaves, paste code into ./ssh/authoriged_keys
#connections are established.



Step1: 

managejenkins—>nodes—->built-in node—>configure—>no of executors=0—>save.

Step2:

Mananage jenkins—>Nodes—>new node—>Name for node(jenkins-agent)--->[clickon]permanent agent—-create.

—>  no of executors  [2]
—-> remote root directory [/home/ubuntu]
—-> Labels  [jenkins-agent]
—-> usage [use this node as much as possible]
—-> Launch method  [launch agent via ssh]
—-> host  [private ip of agent]
—-> credentials —-> add jenkins—>kind —>[ssh username with privatekey]
—-> ID [jenkins-Agent]
—-> username [ubuntu]
    Private key[clickon] enter, directly [paste private key of master]
—>  add
—>   Host key verification strategy?
    [Non verifying verification strategy]--->save

    ========================================================================================================================
                            **On next server installations**
    
---> This server is used for installtion of monitoring tools and also installing scaning tools
---> prometheus & Grafana
---> sonarqube
Step:1
======
**sonarqube installtions**
==================================================
## Update Package Repository and Upgrade Packages
    $ sudo apt update
    $ sudo apt upgrade
## Add PostgresSQL repository
    $ sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
    $ wget -qO- https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo tee /etc/apt/trusted.gpg.d/pgdg.asc &>/dev/null
## Install PostgreSQL
    $ sudo apt update
    $ sudo apt-get -y install postgresql postgresql-contrib
    $ sudo systemctl enable postgresql
## Create Database for Sonarqube
    $ sudo passwd postgres
    $ su - postgres
    $ createuser sonar
    $ psql 
    $ ALTER USER sonar WITH ENCRYPTED password 'sonar';
    $ CREATE DATABASE sonarqube OWNER sonar;
    $ grant all privileges on DATABASE sonarqube to sonar;
    $ \q
    $ exit
## Add Adoptium repository
    $ sudo bash
    $ wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc
    $ echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list
 ## Install Java 17
    $ apt update
    $ apt install temurin-17-jdk
    $ update-alternatives --config java
    $ /usr/bin/java --version
    $ exit 
## Linux Kernel Tuning
   # Increase Limits
    $ sudo vim /etc/security/limits.conf
    //Paste the below values at the bottom of the file
    sonarqube   -   nofile   65536
    sonarqube   -   nproc    4096

    # Increase Mapped Memory Regions
    sudo vim /etc/sysctl.conf
    //Paste the below values at the bottom of the file
    vm.max_map_count = 262144

#### Sonarqube Installation ####
## Download and Extract
    $ sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.9.0.65466.zip
    $ sudo apt install unzip
    $ sudo unzip sonarqube-9.9.0.65466.zip -d /opt
    $ sudo mv /opt/sonarqube-9.9.0.65466 /opt/sonarqube
## Create user and set permissions
     $ sudo groupadd sonar
     $ sudo useradd -c "user to run SonarQube" -d /opt/sonarqube -g sonar sonar
     $ sudo chown sonar:sonar /opt/sonarqube -R
## Update Sonarqube properties with DB credentials
     $ sudo vim /opt/sonarqube/conf/sonar.properties
     //Find and replace the below values, you might need to add the sonar.jdbc.url
     sonar.jdbc.username=sonar
     sonar.jdbc.password=sonar
     sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube
## Create service for Sonarqube
$ sudo vim /etc/systemd/system/sonar.service
//Paste the below into the file
     [Unit]
     Description=SonarQube service
     After=syslog.target network.target

     [Service]
     Type=forking

     ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
     ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop

     User=sonar
     Group=sonar
     Restart=always

     LimitNOFILE=65536
     LimitNPROC=4096

     [Install]
     WantedBy=multi-user.target

## Start Sonarqube and Enable service
     $ sudo systemctl start sonar
     $ sudo systemctl enable sonar
     $ sudo systemctl status sonar

## Watch log files and monitor for startup
     $ sudo tail -f /opt/sonarqube/logs/sonar.log

                **           or

      create sonarqube container using docker 
      ===========================================
      $ docker run --name sonar_name -d -p 9000:9000 sonarqube:latest

    -----> After installing sonarqube, configure with jenkins as below
        1. Access the sonarqube with browser & login 
           username: admin
           password: admin
        2.click on 
          MyAccounts-->Security-->Generate Tokens and save that token in notepad

        3. Now go into jenkins and configure and add pluggins of sonarqubescanner and qualitygates
        4. Go inside tools and configure sonarqube
        5.Go insde system and add token of sonarqube with kind as **secrettext**
        =============================================================================================================
      
       ** installtions of prometheus and grafana**
       ===========================================
       

Install Prometheus and Grafana:
======================================================================================================================================
Set up Prometheus and Grafana to monitor your application.

Installing Prometheus:

First, create a dedicated Linux user for Prometheus and download Prometheus:

sudo useradd --system --no-create-home --shell /bin/false prometheus
wget https://github.com/prometheus/prometheus/releases/download/v2.47.1/prometheus-2.47.1.linux-amd64.tar.gz
Extract Prometheus files, move them, and create directories:

tar -xvf prometheus-2.47.1.linux-amd64.tar.gz
cd prometheus-2.47.1.linux-amd64/
sudo mkdir -p /data /etc/prometheus
sudo mv prometheus promtool /usr/local/bin/
sudo mv consoles/ console_libraries/ /etc/prometheus/
sudo mv prometheus.yml /etc/prometheus/prometheus.yml
Set ownership for directories:

sudo chown -R prometheus:prometheus /etc/prometheus/ /data/
Create a systemd unit configuration file for Prometheus:

sudo nano /etc/systemd/system/prometheus.service
Add the following content to the prometheus.service file:

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
Here's a brief explanation of the key parts in this prometheus.service file:

User and Group specify the Linux user and group under which Prometheus will run.

ExecStart is where you specify the Prometheus binary path, the location of the configuration file (prometheus.yml), the storage directory, and other settings.

web.listen-address configures Prometheus to listen on all network interfaces on port 9090.

web.enable-lifecycle allows for management of Prometheus through API calls.

Enable and start Prometheus:

sudo systemctl enable prometheus
sudo systemctl start prometheus
Verify Prometheus's status:

sudo systemctl status prometheus
You can access Prometheus in a web browser using your server's IP and port 9090:

http://<your-server-ip>:9090

Installing Node Exporter:

Create a system user for Node Exporter and download Node Exporter:

sudo useradd --system --no-create-home --shell /bin/false node_exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
Extract Node Exporter files, move the binary, and clean up:

tar -xvf node_exporter-1.6.1.linux-amd64.tar.gz
sudo mv node_exporter-1.6.1.linux-amd64/node_exporter /usr/local/bin/
rm -rf node_exporter*
Create a systemd unit configuration file for Node Exporter:

sudo nano /etc/systemd/system/node_exporter.service
Add the following content to the node_exporter.service file:

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
ExecStart=/usr/local/bin/node_exporter --collector.logind

[Install]
WantedBy=multi-user.target
Replace --collector.logind with any additional flags as needed.

Enable and start Node Exporter:

sudo systemctl enable node_exporter
sudo systemctl start node_exporter
Verify the Node Exporter's status:

sudo systemctl status node_exporter
You can access Node Exporter metrics in Prometheus.

Configure Prometheus Plugin Integration:

Integrate Jenkins with Prometheus to monitor the CI/CD pipeline.

Prometheus Configuration:

To configure Prometheus to scrape metrics from Node Exporter and Jenkins, you need to modify the prometheus.yml file. Here is an example prometheus.yml configuration for your setup:

global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']

  - job_name: 'jenkins'
    metrics_path: '/prometheus'
    static_configs:
      - targets: ['<your-jenkins-ip>:<your-jenkins-port>']
Make sure to replace <your-jenkins-ip> and <your-jenkins-port> with the appropriate values for your Jenkins setup.

Check the validity of the configuration file:

promtool check config /etc/prometheus/prometheus.yml
Reload the Prometheus configuration without restarting:

curl -X POST http://localhost:9090/-/reload
You can access Prometheus targets at:

http://<your-prometheus-ip>:9090/targets

####Grafana
======================================================================================================================================

Install Grafana on Ubuntu 22.04 and Set it up to Work with Prometheus

Step 1: Install Dependencies:

First, ensure that all necessary dependencies are installed:

sudo apt-get update
sudo apt-get install -y apt-transport-https software-properties-common
Step 2: Add the GPG Key:

Add the GPG key for Grafana:

wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
Step 3: Add Grafana Repository:

Add the repository for Grafana stable releases:

echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
Step 4: Update and Install Grafana:

Update the package list and install Grafana:

sudo apt-get update
sudo apt-get -y install grafana
Step 5: Enable and Start Grafana Service:

To automatically start Grafana after a reboot, enable the service:

sudo systemctl enable grafana-server
Then, start Grafana:

sudo systemctl start grafana-server
Step 6: Check Grafana Status:

Verify the status of the Grafana service to ensure it's running correctly:

sudo systemctl status grafana-server
Step 7: Access Grafana Web Interface:

Open a web browser and navigate to Grafana using your server's IP address. The default port for Grafana is 3000. For example:

http://<your-server-ip>:3000

You'll be prompted to log in to Grafana. The default username is "admin," and the default password is also "admin."

Step 8: Change the Default Password:

When you log in for the first time, Grafana will prompt you to change the default password for security reasons. Follow the prompts to set a new password.

Step 9: Add Prometheus Data Source:

To visualize metrics, you need to add a data source. Follow these steps:

Click on the gear icon (⚙️) in the left sidebar to open the "Configuration" menu.

Select "Data Sources."

Click on the "Add data source" button.

Choose "Prometheus" as the data source type.

In the "HTTP" section:

Set the "URL" to http://localhost:9090 (assuming Prometheus is running on the same server).
Click the "Save & Test" button to ensure the data source is working.
Step 10: Import a Dashboard:

To make it easier to view metrics, you can import a pre-configured dashboard. Follow these steps:

Click on the "+" (plus) icon in the left sidebar to open the "Create" menu.

Select "Dashboard."

Click on the "Import" dashboard option.

Enter the dashboard code you want to import (e.g., code 1860).

Click the "Load" button.

Select the data source you added (Prometheus) from the dropdown.

Click on the "Import" button.

You should now have a Grafana dashboard set up to visualize metrics from Prometheus.

Grafana is a powerful tool for creating visualizations and dashboards, and you can further customize it to suit your specific monitoring needs.

That's it! You've successfully installed and set up Grafana to work with Prometheus for monitoring and visualization.
======================================================================================================================
-->  **Install Trivy on agent**

sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy        
======================================================================================================================

**create eks cluster   **
=========================
## Install AWS Cli on the above EC2
Refer--https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html
$ sudo su
$ curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
$ apt install unzip,   $ unzip awscliv2.zip
$ sudo ./aws/install
         OR
$ sudo yum remove -y aws-cli
$ pip3 install --user awscli
$ sudo ln -s $HOME/.local/bin/aws /usr/bin/aws
$ aws --version

## Installing kubectl
Refer--https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html
$ sudo su
$ curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.27.1/2023-04-19/bin/linux/amd64/kubectl
$ ll , $ chmod +x ./kubectl  //Gave executable permisions
$ mv kubectl /bin   //Because all our executable files are in /bin
$ kubectl version --output=yaml

## Installing  eksctl
Refer---https://github.com/eksctl-io/eksctl/blob/main/README.md#installation
$ curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
$ cd /tmp
$ ll
$ sudo mv /tmp/eksctl /bin
$ eksctl version

## Setup Kubernetes using eksctl
Refer--https://github.com/aws-samples/eks-workshop/issues/734
$ eksctl create cluster --name virtualtechbox-cluster \
--region ap-south-1 \
--node-type t2.small \
--nodes 3 \

$ kubectl get nodes

======================================================================================================================
**Argocd Installtions on the cluster**

1 ) First, create a namespace
    $ kubectl create namespace argocd

2 ) Next, let's apply the yaml configuration files for ArgoCd
    $ kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

3 ) Now we can view the pods created in the ArgoCD namespace.
    $ kubectl get pods -n argocd

4 ) To interact with the API Server we need to deploy the CLI:
    $ curl --silent --location -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/download/v2.4.7/argocd-linux-amd64
    $ chmod +x /usr/local/bin/argocd

5 ) Expose argocd-server
    $ kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

6 ) Wait about 2 minutes for the LoadBalancer creation
    $ kubectl get svc -n argocd

7 ) Get pasword and decode it.
    $ kubectl get secret argocd-initial-admin-secret -n argocd -o yaml
    $ echo WXVpLUg2LWxoWjRkSHFmSA== | base64 --decode

## Add EKS Cluster to ArgoCD
9 ) login to ArgoCD from CLI
    $ argocd login a2255bb2bb33f438d9addf8840d294c5-785887595.ap-south-1.elb.amazonaws.com --username admin

10 ) 
     $ argocd cluster list

11 ) Below command will show the EKS cluster
     $ kubectl config get-contexts

12 ) Add above EKS cluster to ArgoCD with below command
     $ argocd cluster add i-08b9d0ff0409f48e7@virtualtechbox-cluster.ap-south-1.eksctl.io --name virtualtechbox-eks-cluster

13 ) $ kubectl get svc
======================================================================================================================

--> Access the argocd with endpoint of loadbalancer
    username: admin
    password: get the password from the argocd-initial-admin pod which is in the secrets pod on argocd namespace
      $ kubectl get secret -n argocd
      $ kubectl edit argocd-initial-admin from that pod copy password and decode it as below
      $ echo password | base64 --decode
--> Take that decoded password and use it for login of argocd 
--> change the password by clicking on** userinfo**
--> Then click on settings and add your github repo url
--> Then click on create application 
====================================================================================================================
using scm in jenkins trigger the pipeline, it will build and deploy into your cluster
clone : https://github.com/sudarshansw7/registration-app.git
clone : https://github.com/sudarshansw7/gitOps-registration-app.git

**Note: after cloning remove .git from it and push into your github and add to your pipeline with scm and execute the jobs 
**

         
