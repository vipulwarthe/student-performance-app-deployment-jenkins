# student-performance-app-deployment-with-jenkins-CICD-pipeline

First create IAM Role Name "JenkinsAWSRole" with the following managed policies:

* AmazonEC2ContainerRegistryFullAccess
  
* AmazonECS_FullAccess
  
* AmazonS3ReadOnlyAccess
  
* CloudWatchLogsFullAccess

Create one EC2 instance with Name "Jenkins-Pipeline" /UBUNTU 22.04/t2.medium/all-traffic/20gb and attach created IAM Role to the instance and install below services.

sudo apt update

Install AWS CLI    

aws configure     (secret key and access key and region)

Install Docker

Install Jenkins

To execute the above pipeline successfully, you will need to install the following Jenkins plugins: Pipeline / Git / GitHub / Docker / Docker Pipeline / Credentials Binding / AWS CLI / JUnit / Publish Over SSH (optional)/ Blue Ocean / Email Extension / Slack Notification


  
