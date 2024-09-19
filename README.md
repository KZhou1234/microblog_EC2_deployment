# CI/CD


---



## Purpose

The goal of this practice is to practice to use AWS EC2, GitHub, Jenkins, Promethus and Grafana to implement CI/CD.

## System Diagram

## Steps

1. Clone this repo to the GitHub account. IMPORTANT: Make sure that the repository name is "microblog_EC2_deployment". GitHub is used for version control. Jenkins will get the latest version of cade from GitHub to deploy.

2. Install and run Jenkins on AWS EC2 allows full control over the environment. The security group for this instance should be configured to open port 22 to allow SSH traffic, port 443 for HTTP traffic and port 8080 for Jenkins. 

3. Configure the server by installing 'python3.9',  'python3.9-venv', 'python3-pip', and 'nginx'. By creating and ruuning an virtual environment, it can isolate dependencies and avoid conflics for projects.

4. Clone the GH repository to the server, cd into the directory, create and activate a python virtual environment with: 

```
$python3.9 -m venv venv
$source venv/bin/activate
```

5. While in the python virtual environment, install the application dependencies and other packages by running:

```
$pip install -r requirements.txt
$pip install gunicorn pymysql cryptography
```

6. Set the ENVIRONMENTAL Variable:

```
export FLASK_APP=microblog.py
```
Question: What is this command doing?

7. Run the following commands: 

```
$flask translate compile
$flask db upgrade
```

8. Edit the NginX configuration file at "/etc/nginx/sites-enabled/default" so that "location" reads as below.

```
location / {
proxy_pass http://127.0.0.1:5000;
proxy_set_header Host $host;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
```
### Question 1: What is this config file/NginX responsible for?  

A: The above code block is to set up the Nginx as the reverse proxy server. The configuration forward traffic from public internet to the local host port 5000 where the application server running.

9. Run the following command and then put the servers public IP address into the browser address bar

```
gunicorn -b :5000 -w 4 microblog:app
```
### Question 2: What is this command doing? You should be able to see the application running in the browser but what is happening "behind the scenes" when the IP address is put into the browser address bar?  
A: The command above will run the application manually. Go to the public IP address of Jenkins server which hosts the application, I got the login page, meanning that the application is running. That is because we use the cammand `-b` to bind port 5000 allowing port 5000 to listen to any traffic go to the IP. `-w` specifies the number of workers worked independently process requests to port 5000, by doing this the performance can be improved.  
The Last part `microblog:app` is telling the Gunicorn which is the reverse proxy server which application should run.
img here


10. If all of the above works, stop the application by pressing ctrl+c.  Now it's time to automate the pipeline.  Modify the Jenkinsfile and fill in the commands for the build and deploy stages.

  a. The build stage contains all steps of creating vertual environment, activating venv, installing and updating dependencies and configuring FLASK_APP (environment vsriable) to be the Flask application in our workload which is microblog.py.  
  

  b. The test stage will run pytest.  Create a python script called test_app.py to run a unit test of the application source code. IMPORTANT: Put the script in a directory called "tests/unit/" of the GitHub repository. 

  ```
import pytest
from app import create_app

@pytest.fixture()
  ```
  First two lines of code import the requred modules will be used in the unit test. The last line of code anotates the fixture in pytest, which allows the following pieces of code to be resuable.  
  In my used test script, first create an instance of Flask application and enable the `Testing` mode. Then create an client by calling `app.test_client()`. Last step is test a route in main page. Assert an client to send a HTTP GET request to the server, check the reponse `response = client.get('/explore')`. Since we can not access the main page without login; therefore we expected to get a `Status code 302` to redirect.


  c. The deploy stage will run the commands required to deploy the application so that it is available to the internet. 

  d. There is also a 'clean' and an 'WASP FS SCAN' stage.  What are these for?
  
11. In Jenkins, install the "OWASP Dependency-Check" plug-in

### Question 3: What is this plugin for?  What is it doing?  When does it do it?  Why is it important?
OWASP Dependency-Check is a security and risk audit tool to analyze the dependencies and identify the vulnerabilities.  
In the Jenkinsfile we can see that the `SCAN` stage is after the test stage. It's usually run after dependnecies resolved, before the deploy stage, ensure known vulnerabilities can be detected deployment.


12. Create a MultiBranch Pipeline and run the build.  IMPORTANT: Make sure the name of the pipeline is: "workload_3".

      `Daemon process` allows the process run in backgroud that allows Jenkins move to other stages without blocking.

14. After the application has successfully deployed, create another EC2 (t3.micro) called "Monitoring".  Install Prometheus and Grafana and configure it to monitor the activity on the server running the application.  
  Grafana and Prometheus are two monitoring tools. In order to monitor the Jenkins (application) server in this workload, we installed `Node Exporter` on the applicaiton server that collect metrics on port 9100.   
  Then installed Prometheus in Monitoring server to retrive the data from `Node Expoter` By configuring the `.yml` file.  
  At the meantime, install and start the Grafana on the Monitoring server, which can help use to visualize the data retrived from Permetheus.  

  Following are the data collected from Jenkins Server. The monitoring step will help us to better understand the performance of the application, resource. Allow engineers act fast if there's any abnormal activities.


## ISSUES/TROUBLESHOOTING

1. Permission denied issue in building stage.  
  While push updated files to GitHub repository, I've push created `venv` to GieHub. That will cause while Jenkins run the command, it will first check and use all the code on GitHub, but the path of the virtual environment now does not match the path in Jenkinsfile. This has been solved by remove the venv in the GithHub.  

2. Password not found issue.
  For the deploy stage, we have used `sudo` command to have the root permission, which cannot get through the deploy stage. To solve this problem, edit the `visudo` to give the permission to Jenkins.


## CONCLUSION

In this workload, we have practice more about using Jenkins to implement fully CI/CD.