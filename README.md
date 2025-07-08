Project 1: End-to-end Jenkins Pipeline

Step 1: Create AWS EC2 instance (preferably t2.large)
- Generate .pem key to ssh connect it through your terminal
- Launch it in your nearest server
- Configure inbound rules for future use

Step 2: Open Local Terminal:
- ssh into your instance with pem key
- Install open idk 17+ :
- Sudo apt install openjdk-17-jdk
- Install jenkins 
- curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins

Step 3: Open Jenkins
- Open any browser
- Provide url http://(Public IP of Ec2):8080/
- Setup user as admin
- Create Job
- Use Pipeline
- Add description (if required)
- Use pipeline script from SCM
- Provide Github repo link
- Add JenkinsFile path
- Select Lightweight checkout
- Save
- Go to manage plugins
- Install docker pipeline plugin
- Install sonarqube scanner plugin

Step 4: Setup SonarQube
* sudo su -
* adduser sonarqube
* wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.4.1.88267.zip
* unzip *
* chown -R sonarqube:sonarqube /home/sonarqube/sonarqube-10.4.1.88267
* chmod -R 775 /home/sonarqube/sonarqube-10.4.1.88267
* cd /home/sonarqube/sonarqube-10.4.1.88267/bin/linux-x86-64
* ./sonar.sh start

Step 5: Open SonarQube
- Open any browser
- Provide url http://(Public IP of Ec2):9000/
- Login using user : admin, pass : admin
- 
