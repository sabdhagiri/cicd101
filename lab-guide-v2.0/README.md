<h1>CI/CD lab guide</h1>

<h1>Index</h1>

1. [Introduction](#introduction)
2. [Demo Application overview](#demo-application-overview)
3. [Gerrit Configuration](#gerrit-configuration)
4. [Jenkins Configuration](#jenkins-configuration)
5. [Setup Gerrit - Jenkins integration](#setup-gerrit---jenkins-integration)
6. [Setup local development tree](#setup-local-development-tree)
7. [Setup Gerrit for code reviews](#setup-gerrit-for-code-reviews)
8. [Setup Continuous Integration job in Jenkins](#setup-continuous-integration-job-in-jenkins)
9. [Setup Continuous Deployment job in Jenkins](#setup-continuous-deployment-job-in-jenkins)
10. [Verify and Validate CI/CD pipeline for incremental changes](#)


--------------------------------------



<h2>Introduction</h2>

High level overview of different components of the lab and what students will be doing in each section are as follows:
 
1. Login to pre-installled Gerrit instance running in the VM and complete the setup.
2. Login to pre-installed Jenkins instance running in the VM and complete the setup.
3. Configure Gerrit and Jenkins to enable integration between the two.
4. Configure the VM to setup local development tree and use it for submitting patches in Gerrit.
5. Login to Github and fork the demo application code.
6. Login to the VM and clone the application code to the VM.
7. Use the pre-installed docker runtime on the VM to build the application image.
8. Run the application as a docker container and use if to get a feel for the sample application.
9. Create a job in Jenkins to verify the patch set submitted to Gerrit.
10. Make a change to the code and verify the CI job execution.
11. Create a job in Jenkins to build and deploy container image when a change is merged to Gerrit master.
12. Manual code-review and merge the change to master in Gerrit and verify the CD job execution.
13. Make incremental changes and verify automatic deployment of the application.


For this class, one VM is being used as an all-in-one host. Students will use this for the following:

1. Make development changes to the application code
2. Run Gerit
3. Run Docker runtime
4. Run Jenkins


**[Back to top](#)**

----------------------------------

<h2>Demo Application overview</h2>


Simple todo application:
* Written in Python using Flask framework
* UI templates uses Bootstrap


<h5>Current version:</h5>

![](screenshots/latest/intro/intro-1.png)

<h5>After CI/CD:</h5>

![](screenshots/latest/intro/intro-2.png)

**[Back to top](#)**

----------------------------------

<h2>Gerrit Configuration</h2>

<h3>Gerrit Instance details</h3>

URL: `http://<your-pod-ip>/gerrit/`  
  
  
admin username: `admin`  
admin password: `devops`  
  
  

jenkins username: `jenkins`  
jenkins password: `devops`  


1. From your RDP machine, open Chrome and navigate to Gerrit URL.

	![](screenshots/latest/gerrit-config/gerrit-config-1.png)

2. Login with the `admin` user account and password.
	
	![](screenshots/latest/gerrit-config/gerrit-config-2.png)

3. Once you login, you will see a screen like below image

	![](screenshots/latest/gerrit-config/gerrit-config-3.png)

4. Click on the `Register New Email ...` button

	![](screenshots/latest/gerrit-config/gerrit-config-4.png)
	
5. In the screen that follows type in a valid email-id and click on register, Gerrit will send a verification email, so this has to be a vaid email id to verify the email.

	![](screenshots/latest/gerrit-config/gerrit-config-5.png)
	
6. Check the inbox of your email id which you used in previous step and you should see verification email from Gerrit like the one below. 

	![](screenshots/latest/gerrit-config/gerrit-config-6.png)
	
7. Copy the verification link and open a new tab in the Chrome browser in the RDP machine and paste it in the address bar and hit enter/return.
	
	![](screenshots/latest/gerrit-config/gerrit-config-7.png)
	
8. Your id will be verified and you will redirected to a page like in the below image.

	![](screenshots/latest/gerrit-config/gerrit-config-8.png)
	
9. Click on **`Projects`** on the top navigation bar and click on *`List`*

	![](screenshots/latest/gerrit-config/gerrit-config-9.png)
	
10. From the list of projects that show up on the next screen click on **`All-Projects`**

	![](screenshots/latest/gerrit-config/gerrit-config-10.png)

11. In the screen that follows, click on **`Edit Config`** button on the botton section of the screen.

	![](screenshots/latest/gerrit-config/gerrit-config-11.png)
	
12. You will see some thing like the below image, 

	![](screenshots/latest/gerrit-config/gerrit-config-12.png)
	
	scroll to the bottom of the file and add the following code
	
	```
	[label "Verified"]
    		function = MaxWithBlock
    		value = -1 Fails
    		value = 0 No score
    		value = +1 Verified
    		copyAllScoresIfNoCodeChange = true
	```
	
	and click on **`Save`**
	
	![](screenshots/latest/gerrit-config/gerrit-config-13.png)
	
13. Now click on **`Close`** buttin next to `Save`
	
	![](screenshots/latest/gerrit-config/gerrit-config-14.png)
	
14. You will see a screen like the below image, click on **`Publish Edit`**
	
	![](screenshots/latest/gerrit-config/gerrit-config-15.png)
	
15. Click on **`Publish`**
	
	![](screenshots/latest/gerrit-config/gerrit-config-16.png)
	
16. Click on **`Code-Review+2`**
	
	![](screenshots/latest/gerrit-config/gerrit-config-17.png)
	
17. Click on **`Submit`**

	![](screenshots/latest/gerrit-config/gerrit-config-18.png)

18. This will merge the change that has been made to the project config to the *`master`* branch of **`All-Projects`** project.
	
	![](screenshots/latest/gerrit-config/gerrit-config-19.png)
			
	**Congratulations, on reviewing your first change in Gerrit**

**[Back to top](#)**

----------------------------------

<h2>Jenkins Configuration</h2>

The Ubuntu VM runs `Gerrit`, `Docker` and `Jenkins`.

Access details to the VM:

IP: `<your-pod-ip>`  
username: `devops`  
password: ``  

Login to the VM using the PuTTY client installed on your RDP machine with the above details.

Jenkins runs as a docker container in the VM and it needs to be configured to be able to setup continuous integration and deployment jobs.

Jenkins URL: `http://<your-pod-ip>:8080`

1. Open Jenkins URL in Chrome browser from your RDP machine

	![](screenshots/latest/jenkins-config/jenkins-config-1.png)
	
2. Jenkins will display the following page.

	![](screenshots/latest/jenkins-config/jenkins-config-2.png)
	
3. Jenkins will have default admin user provisioned while setup and a random password will be generated for this user and stored at `/var/jenkins_home/secrets/initialAdminPassword`.
4. Since, Jenkins runs as a container this password can obtained by executing the below command on the VM

	~~~bash
	docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
	~~~
	
5. Copy the password and paste it in the next screen and click on **`Continue`**

	![](screenshots/latest/jenkins-config/jenkins-config-3.png)
	

6. Click on **`Install suggested Plugins`** in the next screen and the plugins will be installed.

	![](screenshots/latest/jenkins-config/jenkins-config-4.png)
	
	![](screenshots/latest/jenkins-config/jenkins-config-5.png)
	
7. Once plugins are installed create the first admin user and make sure you remember the username and password that you create here, it will be used in upcoming sections. Click on **`Save and Continue`**.
	
	![](screenshots/latest/jenkins-config/jenkins-config-6.png)
	
8. Click on **`Save and Finish`**
	
	![](screenshots/latest/jenkins-config/jenkins-config-7.png)
	
9. Click on **`Start using Jenkins`**

	![](screenshots/latest/jenkins-config/jenkins-config-8.png)
	
	![](screenshots/latest/jenkins-config/jenkins-config-9.png)


**[Back to top](#)**

-----------------------------------

<h2>Setup Gerrit - Jenkins integration</h2>

Jenkins connects to Gerrit using the service account which has been pre-provisioned for this class. Authentication between Jenkins and Gerrti is facilitated with the help of SSH keys. Jenkins will use the SSH keypair which will be created in the Jenkins container. follow the steps to create the SSH keypair for the Jenkins user.

1. Create a SSH keypair for the **`jenkins`** user in the jenkins container.
2. Login to the VM as **`devops`** user and execute the following command

	~~~bash
	docker exec jenkins mkdir /var/jenkins_home/.ssh
	
	docker exec jenkins ssh-keygen -t rsa -b 2048 -f /var/jenkins_home/.ssh/id_rsa -N ''

	~~~
	
	This will generate a SSH private-public key pair for jenkins user and store it under `/var/jenkins_home/.ssh` in two files `id_rsa` - private key and `id_rsa.pub` - public key. Gerrit offers a different kexAlgorithm. so, the SSH client on the docker container needs to know about the KeyExchange Algorithm Gerrit will offer. We will put this in a config file. execute the following command on the VM
	
	~~~bash
	docker exec -it jenkins bash
	~~~
	
	This will login to jenkins container as `jenkins` user, now execute the following command to edit the `config` file.
	
	~~~bash
	vim /var/jenkins_home/.ssh/config
	~~~
	
	and paste the following content in the file, be sure to replace with your pod information.
	
	~~~bash
	Host <your-pod-ip>
		Hostname <your-pod-ip>
    	Port 29418
    	IdentityFile /var/jenkins_home/.ssh/id_rsa
    	KexAlgorithms +diffie-hellman-group1-sha1	~~~
    
   save and close the file by typing `esc`+`:wq`+`enter` and exit out of the container by typing `Ctrl`+`d`
	
	
	
3. Now make sure you are in the VM as `devops` user and copy the public key of the keypair and update the profile information for `jenkins` user in Gerrit. execute the following command on the VM and copy the output returned.
	
	~~~bash
	docker exec jenkins cat /var/jenkins_home/.ssh/id_rsa.pub
	~~~
	
4. Now Chrome browser open a new incognito window and navigate to the Gerrit URL: `http://<your-pod-ip>/gerrit/`

	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-1.png)
	
5. Login as `jenkins` user, with password `devops`
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-2.png)
	
6. In the screen that follows paste the SSH key which copied from step 3 and click on **`Add`**
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-3.png)
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-4.png)
	
7. Click on **`Continue`** and you will taken to the changes page.
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-5.png)
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-6.png)
	
8. Verify access by trying to SSH to Gerrit host from the jenkins container.

9. switch to the VM and execute the following command:

	~~~bash
	docker exec -it jenkins ssh jenkins@<your-pod-ip> -p 29418
	~~~
	
	and you should see something like this
	
~~~
devops@CICDLAB:~$ docker exec -it jenkins ssh jenkins@172.16.10.8 -p 29418
The authenticity of host '[172.16.10.8]:29418 ([172.16.10.8]:29418)' can't be established.
RSA key fingerprint is SHA256:AQNoEvc1zM9Ft9Rpd3cpc0Rp38YN1S0VnxKVIAMZ0cY.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[172.16.10.8]:29418' (RSA) to the list of known hosts.

  ****    Welcome to Gerrit Code Review    ****

  Hi jenkins, you have successfully connected over SSH.

  Unfortunately, interactive shells are disabled.
  To clone a hosted Git repository, use:

  git clone ssh://jenkins@172.16.10.8:29418/REPOSITORY_NAME.git

Connection to 172.16.10.8 closed.
~~~

10. Open Chrome browser in RDP host and navigate to Gerrit URL and login as `admin` user

	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-7.png)

11. Configure group membership for the `jenkins` user by clicking on **`People`** tab in the top navigation bar and select **`List Groups`**, select **`Non-Interactive Users`** and add **`jenkins`** to that group by following the below steps

	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-8.png)
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-9.png)
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-10.png)
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-11.png)
	
	

12. Now that, `jenkins` is added to **`Non-Interactive Users`** group, this group should be configured with proper access rights to Stream Gerrit events to know when new events happen in Gerrit, this is the key feature in Gerrit which Jenkins leverage to trigger build jobs. Also, we need access rights to the `jenkins` user to submit score for **`Label: Verified`**. Access rights can be configured by following the steps below
13. Click on **`Projects`** on top navigation menu and select **`List`** to see available projects

	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-12.png)

	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-16.png)
	
14. Click on `All-Projects` and then select `Access` from the top navigation bar.

	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-13.png)
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-14.png)
	
15. Once in the Access page click on `Edit` button and you will be given option to add permissions to different contexts of the project and allow permissions to groups.

	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-15.png)
	
16. Stream Events permission is allowed for `Non-Interactive Users` group by default in Gerrit. Just select `Add Permission` dropdown menu under section `References: refs/*`

	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-16.png)
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-17.png)
	
17. In the dropdown menu select `Label Verified` and then add the `Non-Interactive Users` group and `Save Changes` by clicking on the button in the bottom.

	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-18.png)
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-19.png)
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-20.png)
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-21.png)
	
		
18. This will solve one piece of the puzzle, now Jenkins needs some mechanism to connect to Gerrit in order to listen to the event stream. Jenkins uses a plugin to connect to Gerrit and trigger jobs based on the events it listen from Gerrit.

19. Login to Jenkins via Chrome browser in the RDP machine using the Jenkins URL.

	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-22.png)
	
20. Click on **`Credentials`** on the left navigation pane.
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-23.png)
	
21. Click on **`Jenkins`** in the next screen.
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-24.png)
	
22. Click on **`Global credentials`** on the next screen.
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-25.png)
	
23. Click on **`Add Credentials`** on the next screen.
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-26.png)
	
24. In the next screen select **`SSH Username with private key`** from the dropdown and fill in the details as follows and click on **`OK`**
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-27.png)
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-28.png)
	
25. In the screen that follows click on **`Jenkins`** on the top navigation menu
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-29.png)
	
26. In the home page click on **`Manage Jenkins`** on the left navigation pane.
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-30.png)
	
27. In the next screen click on **`Manage Plugins`**, Jenkins needs plugins to be able to connect to Gerrit. Jenkins use `Gerrit Trigger` plugin to connect to Gerrit and `ShiningPanda` plugin to run test cases in a python virtualenv. 
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-31.png)
	
28. Click on **`Available`** tab on the navigation panel and search for `Gerrit` in the Filter search box and select the **`Gerrit Trigger`** plugin
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-32.png)
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-33.png)
	
29. Search for **`Shining`** in the filter search box and select **`ShiningPanda`** plugin and click on **`Download now and Install after restart`**.
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-34.png)
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-35.png)

30. Check the **`Restart Jenkins when installation is complete and no jobs are running`** and wait for some time and click on the **`Jenkins`** link on the top navigation menu.
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-36.png)
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-37.png)
	
31. Login to Jenkins with your jenkins admin username and password.
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-38.png)
	
32. Click on **`Manage Jenkins`** on the left navigation pane. Scroll down n the next page and you will see **`Gerrit Trigger`** section, click on it.
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-39.png)
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-40.png)
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-41.png)
	
33. Click on **`Add New Server`** in the next screen.
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-42.png)
	
34. Give your Gerrit server any name and check the **`Gerrit Server with Default Configurations`** radio button and click **`OK`**
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-43.png)
	
35. Fill the form with details as seen in the screenshot, be sure to replace the IP address with your `<your-pod-ip>` and make the form look like the below image and click on **`Test Connection`**
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-44.png)
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-45.png)

36. The connection should succeed and you should see a message like below image.
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-46.png)
	
37. Scroll down and click on the **`Advanced`** button.
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-47.png)
	
38. In the next screen remove **`--tag autogenerated:jenkins-gerrit-trigger`** from all the text boxes as indicated in the below screenshots and click on **`Save`** **(Don't forget to svae the changes)**
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-48.png)
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-49.png)
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-50.png)

39. In the Gerrit Trigger plugin page that follows you will see the new server entry added under Gerrit Servers section. Click on the Red status icon and see it turn to blue upon successful connection as shown in below screenshots.
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-51.png)
	
	![](screenshots/latest/gerrit-jenkins/gerrit-jenkins-52.png)	
	
**[Back to top](#)**

---------------------------------

<h2>Setup local development tree</h2>  



<h3> Fork the demo application from Github </h3>  



1. Navigate to `https://github.com` from your browser and Login with your account.

	![](screenshots/latest/setup-code/setup-code-1.png)
	
	![](screenshots/latest/setup-code/setup-code-2.png)
	
2. Search for the `sabdhagiri/demo-app` repository

	![](screenshots/latest/setup-code/setup-code-3.png)
	
	![](screenshots/latest/setup-code/setup-code-4.png)

3. fork the repository into your github account

	![](screenshots/latest/setup-code/setup-code-5.png)
	
	![](screenshots/latest/setup-code/setup-code-6.png)
	
	![](screenshots/latest/setup-code/setup-code-7.png)

<h3> Clone the application from Github to local workstation </h3>  


1. Login to the VM as `devops` user using PuTTY.

2. Goto github and get the URL of the repository which you will be cloning here. Copy it to clipboard by clicking on the copy-to-clipboard  button near clone or download option.

	![](screenshots/latest/setup-code/setup-code-8.png)
	
3. On your vm execute the following command to clone the code repo, be sure to replace URL with the repo URL you copied in the previous step.

	~~~bash
	git clone https://github.com/sabs6488/demo-app.git
	~~~
	
4. This should create a local working tree under the directory from which you cloned the code and you should be able to see the `demo-app` directory.  


	
<h3> Build and run application </h3>  


1. This section is to get yourself familiar with some of the docker commands that will be used for the rest of the labs.
2. Login to your vm if you are not already in it.
3. execute the following command to check if you have access to the docker client
	
	~~~bash
	docker info
	~~~
	
4. This will show the system-wide information for the docker runtime.
5. build the docker image for the application using the Dockerfile shipped with the code repo using the following commands:

	~~~bash
	cd demo-app
	docker build -t demo-app:1.0 .
	~~~
	
6. This should build a new docker container image for the application and tag it as `demo-app:1.0`
7. This image can be seen when the following command is executed

	~~~bash
	docker images
	~~~
8. Now this image can be used to run an instance of the application by using the following command:

	~~~bash
	docker run -d --name demo-app -p 9000:9000 demo-app:1.0
	~~~
9. Now the application is running in a container and serving requests at `http://<your-pod-ip>:9000`

10. Once you verified the application make sure to delete the container you created.

	~~~bash
	docker stop demo-app
	docker rm -f demo-app
	~~~





<h2>Setup Gerrit for code reviews</h2>

1. Gerrit needs to configured so that the repo cloneed from Github can be loaded into Gerrit and Gerrit can be used to review and Jenkins to validate the changes before it gets pushed to Github.
2. The process is as follows 
		
	The objective of this course is to automate the above steps using tools like Gerrit, Jenkins, Docker and Github by using integration plugins and process pipelines.

	When a new change is made to the development tree, it needs to be integrated to the master as and when it is tested and validated.

	The typical flow will be as follows:

	![CI/CD workflow](flow.png)
		
3. Login to Gerrit as `admin` user and create a new project called `demo-app`

	![](screenshots/latest/setup-review/setup-review-1.png)
	
	Click on **`Projects`** and click on **`Create New Project`**
	
	![](screenshots/latest/setup-review/setup-review-2.png)

	![](screenshots/latest/setup-review/setup-review-3.png)
	
	Type in `demo-app` in the Project Name field and click on **`Create Project`**. The project name should match the repo name of you repo in Github as this will be used to setup replication later.
	
	![](screenshots/latest/setup-review/setup-review-4.png)
	
	![](screenshots/latest/setup-review/setup-review-5.png)

	
4. Login to the VM and cd into the `demo-app` directory, which will be the root of the repo you cloned earlier.
5. You can check the git remote by typing in the following command

	~~~bash
	cd demo-app
	~~~
	
	~~~bash
	git remote -v
	~~~
	
	![](screenshots/latest/setup-review/setup-review-6.png)
	
6. Install the `git-review` utility which can be used to submit patch sets to gerrit for code reviews

	~~~bash
	sudo apt-get install git-review
	~~~

7. Now, configure a new remote to your git repo and set the project url of the project which you created in Gerrit for `demo-app`.

	Get the project url by logging into Gerrit and clicking on **`All-Projects`** **>** **`List`** **>** **`demo-app`** and you will find the project URL under clone section select **`ssh`** on the right corener of the clone section get the ssh URL for the `demo-app` project.
	
	![](screenshots/latest/setup-review/setup-review-7.png)
	
8. In your VM terminal configure the git remote with following command: use the URL which you copied in the previous step.

	~~~bash
	git remote add gerrit ssh://admin@172.16.10.8:29418/demo-app
	~~~
	
	![](screenshots/latest/setup-review/setup-review-8.png)
	
9. Now that your local development tree knows where your Gerrit is, it is time to push the code to Gerrit to enable code reviews. In order to do that, Gerrit needs to autheticate you to be able to, allow you to push the code. Gerrit allows two ways to push the code, one is over `http/s` and other is over `ssh`. In order to use SSH the user needs to authenticate themselves using a SSH key.

10. Generate a SSH key pair in your VM so that it can be used by the `admin` user to authenticate against Gerrit whenever you push code or submit patch set.

	~~~bash
	ssh-keygen -t rsa -b 2048
	~~~
	
	accept default values by hitting `enter` on the prompts.
	
11. Now login to Gerrit as `admin` user and add this SSH public key. Click on **`admin`** username on the top right of the screen and click on **`Settings`**

	![](screenshots/latest/setup-review/setup-review-9.png)
	
12. Login to the VM and get the SSH public key by executing the following command
	
	~~~bash
	cat ~/.ssh/id_rsa.pub
	~~~
	
	![](screenshots/latest/setup-review/setup-review-10.png)
		
	and add it to Gerrit SSH keys section for `admin` user
	
13. Goto the Gerrit UI and click on **`SSH Public Keys`** section and paste the copied key.
	
	![](screenshots/latest/setup-review/setup-review-11.png)

	![](screenshots/latest/setup-review/setup-review-12.png)
	
	![](screenshots/latest/setup-review/setup-review-13.png)
	
	![](screenshots/latest/setup-review/setup-review-14.png)
	
14. Gerrit uses a different key exchange algorithm for SSH and this needs to be informed to the SSH client via a config file. (make sure to use the IP address of your Gerrit instance)

	~~~bash
	vim ~/.ssh/config
	~~~
	
	In the file paste the following contents. ( be sure to replace the Host and Hostname values with `<your-pod-ip>`)
	
	~~~bash
	Host 172.16.10.8
		Hostname 172.16.10.8
    	Port 29418
    	IdentityFile ~/.ssh/id_rsa
    	KexAlgorithms +diffie-hellman-group1-sha1 
	~~~	
	
	and save the file by typing **`esc`** **`+`** **`:wq`** and hit `enter`
	
	
15. Now, if you try to SSH into the Gerrit server using following command you should see a message.(replace IP with `<your-pod-ip>`)

	~~~bash
	ssh admin@172.16.10.8 -p 29418
	~~~
	
	![](screenshots/latest/setup-review/setup-review-15.png)
	
16. The user account is now configured to push code and we can upload the code for `demo-app` by pushing it to *`master`* branch in Gerrit. You can push the code by executing the following command:

	~~~bash
	git push gerrit master
	~~~
	
	and this will push the code that we clone from Github into Gerrit.
	
	![](screenshots/latest/setup-review/setup-review-16.png)
	
	![](screenshots/latest/setup-review/setup-review-17.png)
	
	![](screenshots/latest/setup-review/setup-review-18.png)
	
	![](screenshots/latest/setup-review/setup-review-19.png)
	
	![](screenshots/latest/setup-review/setup-review-20.png)
	

17. **Note of Gerrit access control**

	* The overall idea is to have Github as upstream code repo used only for `fetch/pull`
	
	* Any new change to the code will be submited in Gerrit as a patch set, gets validated, tested, reviewed and then accepted.
	
	* Once the change is accepted and merged, Jenkins will Deploy the build(in the case of automated deployments).
	
	* Only the Gerrit user will have *write* access in Github and all other developers will have only *read* access enabling them to fetch updates but not push directly bypassing Gerrit.
	
	* On Gerrit, the access is controlled with the help of Groups, each project can inherit its access control policies from other projects or manage their own.
	
	* Users can be added to multiple groups, by default the first user who logs into the system after initializing the Gerrit site will become `admin`
	
	* The `admin` user as the name implies can administer Projects, Groups, People, Access, Plugins etc.,
	
	* All other users will be part of `Registered Users` groups and they will have standard access. The access is managed using distinct `Permissions` getting assigned to different `Groups` for different `References` or as a `Global Capability`

18. As mentioned earlier, if Gerrit needs to write the changes to Github, we need some sort of Replication mechanism to sync the code changes that is available in Gerrit. Gerrit offers a built-in `Replication` plugin to do that.

19. This `Replication` plugin needs to be configured in Gerrit and SSH keys are needed for the Gerrit user to *push* changes to Github via SSH.

20. Login to VM and become the `gerrit` user by typing 

	~~~bash
	sudo su gerrit
	~~~
	
21. Generate a SSH keypair by running the following command and accept default values at the prompt.

	~~~bash
	ssh-keygen -t rsa -b 2048
	~~~
	
22. Copy the SSH public key with

	~~~bash
	cat ~/.ssh/id_rsa.pub
	~~~
	
	![](screenshots/latest/setup-review/setup-review-22.png)
	
23. Login to your Github account and add this SSH key under **`Settings`** **>** **`SSH and GPG keys`** and **`New SSH key`** and paste your key and save it by clicking **`Add SSH key`**
	
	![](screenshots/latest/setup-review/setup-review-23.png)
	
	![](screenshots/latest/setup-review/setup-review-24.png)

	![](screenshots/latest/setup-review/setup-review-25.png)
	
	![](screenshots/latest/setup-review/setup-review-26.png)
	
	![](screenshots/latest/setup-review/setup-review-27.png)
	
	
	
24. Switch back to your VM and as `gerrit` user validate the access by trying to SSH into github.com

	~~~bash
	ssh git@github.com
	~~~
	
	you should be greeted with a message from Github.com without any shell access.
	
	![](screenshots/latest/setup-review/setup-review-28.png)
	
25. Now that, SSH key has been added for the `gerrit` user to login to Github with SSH, the same account can be used to configure the `Replication` plugin in Gerrit.

26. Create a file called `replication.config` in `/home/gerrit/review_site/etc/` directory and add the following content, be sure to replace the github account according to your use.

	~~~bash
	vim /home/gerrit/review_site/etc/replication.config
	~~~

	and paste the following content into the file and save it.

	~~~bash
	[remote "github"]
 	 url = git@github.com:sabs6488/${name}.git
 	 push = +refs/heads/*:refs/heads/*
 	 push = +refs/tags/*:refs/tags/*
 	 timeout = 5
 	 replicationDelay = 0
 	 authGroup = Replication
	~~~
	
27. Login to Gerrit as `admin` user and create and configure **`Replication`** group and add the `admin` user to that group.

	![](screenshots/latest/setup-review/setup-review-29.png)
	
	![](screenshots/latest/setup-review/setup-review-30.png)
	
	![](screenshots/latest/setup-review/setup-review-31.png)
	
28. Also, the projects should allow the `Replication` group members to`Start Replication` and `Read` from all refs.
	
	![](screenshots/latest/setup-review/setup-review-32.png)
	
	![](screenshots/latest/setup-review/setup-review-33.png)
	
	![](screenshots/latest/setup-review/setup-review-34.png)
	
	![](screenshots/latest/setup-review/setup-review-35.png)
	
	![](screenshots/latest/setup-review/setup-review-36.png)
	
	![](screenshots/latest/setup-review/setup-review-37.png)
	
	![](screenshots/latest/setup-review/setup-review-38.png)
	
	![](screenshots/latest/setup-review/setup-review-39.png)
	
	![](screenshots/latest/setup-review/setup-review-40.png)
	
	![](screenshots/latest/setup-review/setup-review-41.png)
	
	![](screenshots/latest/setup-review/setup-review-42.png)

28. Now from the VM restart the gerrit service to make the changes effective.

	~~~bash
	/home/gerrit/review_site/bin/gerrit.sh stop
	/home/gerrit/review_site/bin/gerrit.sh start
	~~~
	
29. Now exit the session for `gerrit` user in the VM by typing

	~~~bash
	exit
	~~~
	
	This will logout the `gerrit` user if already logged in. make sure you are logged into the VM as `devops` user by executing `whoami` and the execute the following command to start Gerrit replication.

	~~~bash
	ssh admin@172.16.10.8 -p 29418 replication start
	~~~


**[Back to top](#)**

------------------------------

<h2>Setup Continuous Integration job in Jenkins</h2>

1. Login to Jenkins as the admin user you created earlier during the setup.

	![](screenshots/latest/setup-ci/setup-ci-1.png)

2. Create a build job by clicking in **`New Item`** on the left navigation pane.

	![](screenshots/latest/setup-ci/setup-ci-2.png)

3. Create a new **`Freestyle project`** by following the steps below:

	![](screenshots/latest/setup-ci/setup-ci-3.png)
	
	![](screenshots/latest/setup-ci/setup-ci-4.png)
	
	![](screenshots/latest/setup-ci/setup-ci-5.png)
	
	![](screenshots/latest/setup-ci/setup-ci-6.png)
	
	![](screenshots/latest/setup-ci/setup-ci-7.png)
	
	![](screenshots/latest/setup-ci/setup-ci-8.png)
	
	![](screenshots/latest/setup-ci/setup-ci-9.png)
	
	![](screenshots/latest/setup-ci/setup-ci-10.png)
	
	![](screenshots/latest/setup-ci/setup-ci-11.png)
	
	![](screenshots/latest/setup-ci/setup-ci-12.png)
	
	![](screenshots/latest/setup-ci/setup-ci-13.png)
	
	![](screenshots/latest/setup-ci/setup-ci-14.png)
	
	![](screenshots/latest/setup-ci/setup-ci-15.png)
	
	![](screenshots/latest/setup-ci/setup-ci-16.png)
	
	![](screenshots/latest/setup-ci/setup-ci-17.png)
	
	![](screenshots/latest/setup-ci/setup-ci-18.png)
	
	![](screenshots/latest/setup-ci/setup-ci-19.png)
	
	![](screenshots/latest/setup-ci/setup-ci-20.png)
	
	![](screenshots/latest/setup-ci/setup-ci-21.png)
	
	![](screenshots/latest/setup-ci/setup-ci-22.png)
	
	![](screenshots/latest/setup-ci/setup-ci-23.png)
	
	![](screenshots/latest/setup-ci/setup-ci-24.png)
	
	![](screenshots/latest/setup-ci/setup-ci-25.png)
	
	![](screenshots/latest/setup-ci/setup-ci-26.png)
	
	![](screenshots/latest/setup-ci/setup-ci-27.png)
	
	![](screenshots/latest/setup-ci/setup-ci-28.png)


4. Validate the build job to see if it is getting triggered on a new patch set created event. Add a new `.gitignore` file and check if the Jenkins build job is getting triggered.

	Login to the VM as `devops` user and cd into the `demo-app` directory.
	
	Create a file called `.gitreview` and add the following content
	
	~~~bash
	vim .gitreview
	~~~
	
	and copy the content into the file.(replace the host value with `<your-pod-ip>` and save the file
	
	~~~bash
	[gerrit] 
	host=172.16.10.8 
	port=29418 
	project=demo-app  
	~~~
	
	make sure there are no additional spaces or tabs on the file and the file look like
	
	![](screenshots/latest/setup-ci/setup-ci-29.png)
	
	Similairly, Create a file called `.gitignore` 
	
	~~~bash
	vim .gitignore
	~~~
	
	and add the following content and save the file
	
	~~~bash
	*.pyc
	env
	flaskr.db
	.gitreview 
	~~~
	
	make sure there are no additional spaces or tabs on the file and the file look like
	
	![](screenshots/latest/setup-ci/setup-ci-30.png)
	
5. add and commit the changes to the local development work tree. then submit this changes for review to see if the Jenkins verification job is getting triggered or not.

	~~~bash
	git status
	~~~
	
	![](screenshots/latest/setup-ci/setup-ci-31.png)	
	
	~~~bash
	git add .gitignore
	~~~
	
	Before the code can be commited, git always wants to know the author of the change, so it will be easy to contact them regarding any clarifications. run the following commands to configure git
	
	~~~bash
	git config --global user.email "sabdhagiri@gmail.com"
	git config --global user.name "sabdhagiri"
	~~~
	
	~~~bash
	git commit -a -m "added .gitignore file to project"
	git status
	~~~
	
	![](screenshots/latest/setup-ci/setup-ci-32.png)
	
	Now `.gitreview` file shouldn't be listed in untracked files section as `.gitignore` instructs git not to track it.
	
	Submit this change for review
	
	~~~bash
	git review
	~~~
	
	![](screenshots/latest/setup-ci/setup-ci-33.png)
	
	This will create a new change in open changes section in Gerrit UI

	![](screenshots/latest/setup-ci/setup-ci-34.png)

	Jenkins `demo-verify` will be triggered due to the matching `PatchSet created` event and Jenkins will send `+1` score for Verified

	![](screenshots/latest/setup-ci/setup-ci-35.png)

	![](screenshots/latest/setup-ci/setup-ci-36.png)

	![](screenshots/latest/setup-ci/setup-ci-37.png)

	![](screenshots/latest/setup-ci/setup-ci-38.png)

	![](screenshots/latest/setup-ci/setup-ci-39.png)

	![](screenshots/latest/setup-ci/setup-ci-40.png)


	This will enable Jenkins to set the `Label: Verified` score whenever a new change is commited and a patch set created for that. 

5. Once Gerrit sees a +1 score for Label: Verified and +2 for Label: Code-Review it will allow the change to be submitted and merged to master.

	![](screenshots/setup-ci/setup-ci-44.png)

	![](screenshots/setup-ci/setup-ci-45.png)

	![](screenshots/setup-ci/setup-ci-46.png)

	![](screenshots/setup-ci/setup-ci-47.png)

	
	
6. Since, Gerrit is configured with `Replication` plugin it will replicate the changes to the upstream repo in `Github` enabling Continuous Integration of development changes.

	![](screenshots/setup-ci/setup-ci-50.png)

	![](screenshots/setup-ci/setup-ci-51.png)
	
	![](screenshots/setup-ci/setup-ci-48.png)
	
	![](screenshots/setup-ci/setup-ci-49.png)

	![](screenshots/setup-ci/setup-ci-52.png)

**[Back to top](#)**

------------------------------

<h2>Setup Continuous deployment job in Jenkins</h2>

1. This section will show how to build a new docker container image when change is reviewed and merged to master branch in gerrit and also it will trigger the deployment job which will deploy the updated image to docker environment.
2. In Jenkins, create a new **`Freestyle project`** called **`demo-build`** to build the image on new Gerrit change merged event and deploy the changes

	![](screenshots/latest/setup-cd/setup-cd-1.png)
	
	![](screenshots/latest/setup-cd/setup-cd-2.png)
	
	![](screenshots/latest/setup-cd/setup-cd-3.png)
	
	![](screenshots/latest/setup-cd/setup-cd-4.png)
	
	![](screenshots/latest/setup-cd/setup-cd-5.png)
	
	![](screenshots/latest/setup-cd/setup-cd-6.png)
	
	![](screenshots/latest/setup-cd/setup-cd-7.png)
	
	![](screenshots/latest/setup-cd/setup-cd-8.png)
	
	![](screenshots/latest/setup-cd/setup-cd-9.png)
	
	![](screenshots/latest/setup-cd/setup-cd-10.png)
	
	![](screenshots/latest/setup-cd/setup-cd-11.png)
	
	![](screenshots/latest/setup-cd/setup-cd-12.png)
	
	![](screenshots/latest/setup-cd/setup-cd-13.png)
	
	![](screenshots/latest/setup-cd/setup-cd-14.png)
	
	![](screenshots/latest/setup-cd/setup-cd-15.png)
	
	In the build section select `execute shell` from the dropdown and copy paste the following snippet in the shell box in the build section
	
	~~~bash
	docker build -t demo-app:1.0.$BUILD_NUMBER .

	sed -i 's/sabdhagiri\/todo\:latest/demo-app:1.0.'"$BUILD_NUMBER"'/' docker-compose.yml

	sed -i 's/6488/9000/g' docker-compose.yml

	docker-compose down

	docker-compose up -d
	~~~
	
	![](screenshots/latest/setup-cd/setup-cd-16.png)
	
	![](screenshots/latest/setup-cd/setup-cd-17.png)
	
	![](screenshots/latest/setup-cd/setup-cd-18.png)
	
	![](screenshots/latest/setup-cd/setup-cd-19.png)
	
	

3. In order to trigger this job, the patch set which was created in last section can be reviewed and merged. Once Gerrit sees a +1 score for Label: Verified and +2 for Label: Code-Review it will allow the change to be submitted and merged to master.

	![](screenshots/latest/setup-cd/setup-cd-20.png)
	
	![](screenshots/latest/setup-cd/setup-cd-21.png)
	
	![](screenshots/latest/setup-cd/setup-cd-22.png)
	
	![](screenshots/latest/setup-cd/setup-cd-23.png)
	
	![](screenshots/latest/setup-cd/setup-cd-24.png)
	
	![](screenshots/latest/setup-cd/setup-cd-25.png)
	
	![](screenshots/latest/setup-cd/setup-cd-26.png)
	
	![](screenshots/latest/setup-cd/setup-cd-27.png)
	
4. Since, Gerrit is configured with `Replication` plugin it will replicate the changes to the upstream repo in `Github` enabling Continuous Integration of development changes.

	![](screenshots/latest/setup-cd/setup-cd-28.png)
	
	![](screenshots/latest/setup-cd/setup-cd-29.png)
	
	![](screenshots/latest/setup-cd/setup-cd-30.png)
		

 **[Back to top](#)**
 
----------------------

<h2>Verify and Validate CI/CD pipeline for incremental changes</h2>

1. Make additional changes and submit the patch set the CI job and once the code is merged the CD job should be triggered automatically.

	Login to the VM as `devops` user and cd into the working directory and execute the following commands
	
	~~~bash
	cd demo-app
	
	git fetch
	
	mv templates/login_v1.html templates/login.html
	
	mv templates/index_v1.html templates/index.html
	
	mv static/style_v1.css static/style.css
	
	git add templates/login.html templates/index.html static/style.css
	
	git commit -a -m "Updating the style of the Application header"
	
	git fetch
	
	git rebase origin master
	
	git review
	
	~~~
	
2. Login to Gerrit UI as `admin` and check the status of the patch and follow steps as in the CI and CD lab to validate the pipeline.

	![](screenshots/latest/validate/validate-1.png)
	
	![](screenshots/latest/validate/validate-2.png)
	
	![](screenshots/latest/validate/validate-3.png)

	![](screenshots/latest/validate/validate-4.png)
	
	![](screenshots/latest/validate/validate-5.png)
	
	![](screenshots/latest/validate/validate-6.png)
	
	![](screenshots/latest/validate/validate-7.png)
	
	![](screenshots/latest/validate/validate-8.png)
	
	![](screenshots/latest/validate/validate-9.png)
	
	![](screenshots/latest/validate/validate-10.png)
	
	![](screenshots/latest/validate/validate-11.png)
	
	![](screenshots/latest/validate/validate-12.png)

3. This concludes the basics of CI/CD with Github, Gerrit, Jenkins and Docker.

4. This model can be scaled to various scenarios and there could be multiple use cases where Jenkins can deploy to a remote docker cluster, or even a Kubernetes cluster based on the requirements.

5. Jenkins is extensible, thanks to the amazing plugin ecosystem available for Jenkins to make it integrate with most of the Dev and Ops systems available in the industry.

 **[Back to top](#)**
 
----------------------
