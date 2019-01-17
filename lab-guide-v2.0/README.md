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

1. Login to Github and fork the demo application code.
2. Login to the VM and clone the application code to the VM.
3. Use the pre-installed docker runtime on the VM to build the application image.
4. Run the application as a docker container and use if to get a feel for the sample application.
5. Login to pre-installled Gerrit instance running in the VM and complete the setup.
6. Login to pre-installed Jenkins instance running in the VM and complete the setup.
7. Configure Gerrit and Jenkins to enable integration between the two.
8. Configure the VM to setup local development tree and use it for submitting patches in Gerrit.
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

The application is a simple todo application written in python using the Flask framework with templating using html and javascript, there is no persistent storage and sqlite database is used for demo purposes.

The application will look like this

![](screenshots/labII/todo-1.png)

Once we make changes the application will look like this

![](screenshots/labII/todo-6.png)

**Before changes**   |    | **After changes**
---------------------|----|-----------------------------------
![](screenshots/labII/todo-1.png) |  ![](https://cdn3.iconfinder.com/data/icons/faticons/32/arrow-right-01-512.png)  | ![](screenshots/labII/todo-6.png)


**[Back to top](#)**

----------------------------------

<h2>Gerrit Configuration</h2>

1. Navigate to gerrit and login with the admin user account and password.

	for this lab we have two user accounts pre-provisioned `admin/devops` and `jenkins/devops`

![](screenshots/gerrit/gerrit-boot-1.png)

2. Configure admin user email

	* Once you login click on the top right corner on the username and click on settings to open the user profile for user `admin`
	* On the profile page which opens next click on the contact information section in the menu which appears on the left.
	* On the Contact information screen click on **Register New Email..** button and type in the email id on which you want to receive emails. You can enter a real email id because Gerrit needs to verify and validate the user before it could let them make changes.
	* Once an email id has been registered, you can check your inbox and click on the verification link.

	
![](screenshots/gerrit/gerrit-boot-2.png)

![](screenshots/gerrit/gerrit-boot-3.png)

![](screenshots/gerrit/gerrit-boot-4.png)

![](screenshots/gerrit/gerrit-boot-5.png)

![](screenshots/gerrit/gerrit-boot-6.png)

![](screenshots/gerrit/gerrit-boot-7.png)



3. enable Verified label for all projects to be used by Jenkins

	* Once your email id is verified, you will have access to make changes to the project config.
	* In Gerrit you have to enable `Label: Verified` so that Jenkins can update the score after executing the verification job as part of the CI lab which will follow after this.
	* Click on `projects` on the top navigation bar
	
		![](screenshots/gerrit/gerrit-boot-8.png)
		
	* Click on `List` and select `All-Projects` from the list that shows up.
	
		![](screenshots/gerrit/gerrit-boot-10.png)
		
	* Click on `Edit config` button on the bottom and you will see something like below

		 ![](screenshots/gerrit/gerrit-boot-11.png)
		 
	* Add the following code near the end of the file:
	
	  ```
	  [label "Verified"]
      function = MaxWithBlock
      value = -1 Fails
      value = 0 No score
      value = +1 Verified
      copyAllScoresIfNoCodeChange = true
	  ```

	* Now the file will look like this:

		![](screenshots/gerrit/gerrit-boot-12.png)
	* Click on `Save` on the top of the file

		![](screenshots/gerrit/gerrit-boot-13.png)
		
	* You will see something like the below, Gerrit actually tracks this change in the `git` repository created for `All-Projects` and you have to `Publish` the change before it could become effective

		![](screenshots/gerrit/gerrit-boot-14.png)
		
		click on the `Publish Edit` button and then `Publish` button

		![](screenshots/gerrit/gerrit-boot-15.png)
		
	* Now you will be shown `Code-Review+2` button. 
	
		![](screenshots/gerrit/gerrit-boot-16.png)
		
	* When you accept the change by clicking on `Code-Review+2` button you are automatically giving a +2 score for `Label: Code-Review` and qualifying the change to be merged to `master` branch.  
	
		![](screenshots/gerrit/gerrit-boot-17.png)
		  	
	* Once you click on the `Submit` button this is how you typically accept the change and merge it to master  
	
		![](screenshots/gerrit/gerrit-boot-18.png)
		
	**Congratulations, on reviewing your first change in Gerrit**


**[Back to top](#)**

----------------------------------

<h2>Jenkins Configuration</h2>

1. Jenkins runs as a docker container in the VM.
2. Jenkins needs to be intialized and configured for use.
3. Jenkins will have default admin user provisioned while setup and a random password will be generated for this user and stored at `/var/jenkins_home/secrets/initialAdminPassword`.
4. Since, Jenkins runs as a container this password can obtained by executing the below command on the VM

	~~~bash
	docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
	~~~
	
5. Copy the password and paste it in the next screen

	![](screenshots/jenkins/boot-jenkins-1.png)

	Get the initailAdminPassword from the jenkins container by executing the following command in the VM

	![](screenshots/jenkins/boot-jenkins-2.png)
	
6. Install suggested plugins by clicking on the suggested plugins button
	
	![](screenshots/jenkins/boot-jenkins-3.png)
	
	![](screenshots/jenkins/boot-jenkins-4.png)
	
7. Once the plugins are installed create the first admin user and in the screen that follows click on `Save and Finish` to complete the initial setup

	![](screenshots/jenkins/boot-jenkins-5.png)

	![](screenshots/jenkins/boot-jenkins-6.png)

	![](screenshots/jenkins/boot-jenkins-7.png)

**[Back to top](#)**

-----------------------------------

<h2>Setup Gerrit - Jenkins integration</h2>

1. Jenkins connects to Gerrit using the service account which has been pre-provisioned for this class.
2. Authentication between Jenkins and Gerrti is facilitated with the help of SSH keys. 
3. Jenkins will use the SSH keypair which will be created in the Jenkins container. follow the steps to create the SSH keypair for the Jenkins user.

	~~~bash
	docker exec jenkins mkdir /var/jenkins_home/.ssh
	
	docker exec jenkins ssh-keygen -t rsa -b 2048 -f /var/jenkins_home/.ssh/id_rsa -N ''

	~~~
	
	Gerrit offers a different kexAlgorithm so the SSH client on the docker container needs to know about the KeyExchange Algorith Gerrit will offer. We will puth this in a config file.
	
	~~~bash
	docker exec -it jenkins bash
	
	echo "Host 192.168.56.103
    Hostname 192.168.56.103
    Port 29418
    IdentityFile /var/jenkins_home/.ssh/id_rsa
    KexAlgorithms +diffie-hellman-group1-sha1" >> /var/jenkins_home/.ssh/config
	
	~~~
	
	The above code will create a directory to hold the SSH keypair and generate a private-public keypair signed using rsa algorithm.
4. Now copy the public key of the keypair and update the profile information for `jenkins` user in Gerrit
	
	~~~bash
	docker exec jenkins cat /var/jenkins_home/.ssh/id_rsa.pub
	~~~
5. Login to Gerrit as user `jenkins` with password `devops` and follow similar steps to setup email id as you did for `admin` user


6. Once that is done, paste the SSH `id_rsa.pub` public key which you obtained from the Jenkins container in step `4` and save your information by clicking on `add` and then click on `continue`

![](screenshots/gerrit-jenkins/gerrit-jenkins-1.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-3.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-5.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-6.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-7.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-8.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-9.png)

7. Login as Gerrit `admin` user in another incognito window or private browsing session.
8. Configure group membership for the `jenkins` user by clickingg on `People` tab in the top navigation bar and select `List Groups`, select `Non-Interactive Users` and add `jenkins` to that group by following the below steps

![](screenshots/gerrit-jenkins/gerrit-jenkins-10.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-11.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-12.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-13.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-14.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-15.png)

9. Now that, `jenkins` is added to `Non-Interactive Users` group, this group should be configured with proper access rights to Stream Gerrit events to know when new events happen in Gerrit, this is the key feature in Gerrit which Jenkins leverage to trigger build jobs. Also, we need access rights to the `jenkins` user to submit score for `Label: Verified`. Access rights can be configured by following the steps below
10. Click on `Projects` on top navigation menu and select `List` to see available projects

	![](screenshots/gerrit-jenkins/gerrit-jenkins-16.png)
	
11. Click on `All-Projects` and then select `Access` from the top navigation bar.

	![](screenshots/gerrit-jenkins/gerrit-jenkins-17.png)
	
	![](screenshots/gerrit-jenkins/gerrit-jenkins-18.png)
	
12. Once in the Access page click on `Edit` button and you will be given option to add permissions to different contexts of the project and allow permissions to groups.

	![](screenshots/gerrit-jenkins/gerrit-jenkins-19.png)
	
13. Stream Events permission is allowed for `Non-Interactive Users` group by default in Gerrit.
14. Just select `Add Permission` dropdown menu under section `References: refs/*`
15. In the dropdown menu select `Label Verified` and then add the `Non-Interactive Users` group and `Save Changes` by clicking on the button in the bottom.

	![](screenshots/gerrit-jenkins/gerrit-jenkins-20.png)

	![](screenshots/gerrit-jenkins/gerrit-jenkins-21.png)

	![](screenshots/gerrit-jenkins/gerrit-jenkins-22.png)
	
16. This will solve one piece of the puzzle, now Jenkins needs some mechanism to connect to Gerrit in order to listen to the event stream. Jenkins uses a plugin to connect to Gerrit and trigger jobs based on the events it listen from Gerrit.
17. Login to Jenkins, and click on `Manage Jenkins` option on the left navigation pane.

	![](screenshots/gerrit-jenkins/gerrit-jenkins-24.png)
	
	![](screenshots/gerrit-jenkins/gerrit-jenkins-25.png)
	
	![](screenshots/gerrit-jenkins/gerrit-jenkins-26.png)
	
18. In the screen that follows, click on `Manage Plugins`

	![](screenshots/gerrit-jenkins/gerrit-jenkins-27.png)
	
	
19. Select `Available` tab in the next screen and click on the `Filter` on the right above the tab box and type in `gerrit`

	![](screenshots/gerrit-jenkins/gerrit-jenkins-28.png)
	
	
	
20. You will see a filtered list of Gerrit related plugins which can be installed to Jenkins to extend the functionality.

	![](screenshots/gerrit-jenkins/gerrit-jenkins-29.png)

21. Select `Gerrit Trigger` plugin and others are optional. Other Gerrit plugins are not used for this class.

	![](screenshots/gerrit-jenkins/gerrit-jenkins-30.png)
	
	Jenkins also need another plugin to execute the verification job which will be setup later. Search and install plugin called `ShiningPanda` in Jenkins to execute the tests in a virtualenv.

	![](screenshots/gerrit-jenkins/gerrit-jenkins-45.png)
	
22. Once the selection is done, click on `Download now and install after restart` and you will see Jenkins will be downloading the plugins.

	![](screenshots/gerrit-jenkins/gerrit-jenkins-31.png)
	
		
23. Select the `Restart Jenkins when installation is complete and no jobs are running` this will ensure all plugins will be installed and the restart wouldn't interupt any running jobs.

	![](screenshots/gerrit-jenkins/gerrit-jenkins-32.png)

24. Once Jenkins is restarted click on `Manage Jenkins` option on the left navigation pane.

	![](screenshots/gerrit-jenkins/gerrit-jenkins-33.png)

25. You will a new option called `Gerrit Trigger` in the next screen, click on it

	![](screenshots/gerrit-jenkins/gerrit-jenkins-34.png)

26. In the screen that follows, click on `Add New Server` and fill in the form as shown in the screenshots, be sure to use the IP addresses of the VMs assigned to you for `Hostname` and `Frontend URL` fields on the form

	![](screenshots/gerrit-jenkins/gerrit-jenkins-35.png)

	![](screenshots/gerrit-jenkins/gerrit-jenkins-36.png)

	![](screenshots/gerrit-jenkins/gerrit-jenkins-37.png)

27. Test the connection to the server by clicking on the `Test Connection` button.

![](screenshots/gerrit-jenkins/gerrit-jenkins-38.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-39.png)

28. Save the config by clicking on the `Save` button at the bottom of the screen.

![](screenshots/gerrit-jenkins/gerrit-jenkins-41.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-42.png)

29. Click on the `Status` red icon to see if turning to blue upon successful connection, this indicates that there are no connection issues between Gerrit and Jenkins.

![](screenshots/gerrit-jenkins/gerrit-jenkins-43.png)



**[Back to top](#)**

---------------------------------

<h2>Setup local development tree</h2>


<h4> Fork the demo application from Github </h4>

1. Login to Github with your github account
2. Search for the `sabdhagiri/demo-app` repository
3. fork the repository into your github account

![](screenshots/labI/git-1.png)

![](screenshots/labI/git-2.png)

![](screenshots/labI/git-3.png)

![](screenshots/labI/git-4.png)

![](screenshots/labI/git-5.png)

![](screenshots/labI/git-6.png)

![](screenshots/labI/git-7.png)


<h4> Clone the application from Github to local workstation </h4>

1. Login to the VM assigned to you, details will be in the POD details page.
2. Goto github and get the URL of the repository which you will be cloning here. Copy it to clipboard by clicking on the copy-to-clipboard  button near clone or download option.
3. On your vm execute the following command to clone the code repo, be sure to replace `sabs6488` with `<your-github-handle>`

	~~~bash
	git clone https://github.com/sabs6488/demo-app.git
	~~~
	
4. This should create a local working tree under the directory from which you cloned the code and you should be able to see the `demo-app` directory

<h4> Build and run application </h4>

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

1. Gerrit needs to configured so that the repo cloneed from Github can be loaded Gerrit and Gerrit can be used to review and Jenkins to validate the changes before it gets pushed to Github.
2. The process is as follows 
		
	The objective of this course is to automate the above steps using tools like Gerrit, Jenkins, Docker and Github by using integration plugins and process pipelines.

	When a new change is made to the development tree, it needs to be integrated to the master as and when it is tested and validated.

	The typical flow will be as follows:

	![CI/CD workflow](flow.png)
		
3. Login to Gerrit as `admin` user and create a new project called `demo-app`

	![](screenshots/setup-code/setup-code-1.png)

	![](screenshots/setup-code/setup-code-2.png)

	![](screenshots/setup-code/setup-code-3.png)
	
4. Login to the VM and cd into the `demo-app` directory, which will be the root of the repo you cloned earlier.
5. You can check the git remote by typing in the following command

	~~~bash
	git remote -v
	~~~
	
	![](screenshots/setup-code/setup-code-4.png)
	
6. Install the `git-review` utility which can be used to submit patch sets to gerrit for code reviews

	~~~bash
	sudo apt-get install git-review
	~~~

7. Now, configure a new remote to your git repo and set the project url of the project which you created in Gerrit for `demo-app`.

	Get the project url by logging into Gerrit and clicking on `All-Projects` > `List` > `demo-app` and you will find the project URL under clone section select `ssh` on the right corener of the clone section get the ssh URL for the `demo-app` project.
	
	![](screenshots/setup-code/setup-code-5.png)
	
8. In you VM terminal configure the git remote with following command:

	~~~bash
	git remote add gerrit ssh://admin@192.168.56.103:29418/demo-app
	~~~
	
	![](screenshots/setup-code/setup-code-6.png)
	
9. Now that your local development tree knows where your Gerrit is, it is time to push the code to Gerrit to enable code reviews. In order to do that, Gerrit needs to autheticate you to be able to, allow you to push the code. Gerrit allows two ways to push the code, one is over `http/s` and other is over `ssh`. In order to use SSH the user needs to authenticate themselves using a SSH key.

10. Generate a SSH key pair in your VM so that it can be used by the `admin` user to authenticate against Gerrit whenever you push code or submit patch set.

	~~~bash
	ssh-keygen -t rsa -b 2048
	~~~
	
11. Now login to Gerrit as `admin` user and add this SSH public key.

	copy the public key from VM by executing the command 
	
	~~~bash
	cat ~/.ssh/id_rsa.pub
	~~~
	
	![](screenshots/setup-code/setup-code-7.png)

	and add it to Gerrit SSH keys section for `admin` user
	
	![](screenshots/setup-code/setup-code-8.png)
	
	![](screenshots/setup-code/setup-code-9.png)
	
12. Gerrit uses a different key exchange algorithm for SSH and this needs to be informed to the SSH client via a config file. (make sure to use the IP address of your Gerrit instance)

	~~~bash
	echo "Host 192.168.56.103
    Hostname 192.168.56.103
    Port 29418
    IdentityFile ~/.ssh/id_rsa
    KexAlgorithms +diffie-hellman-group1-sha1" >> ~/.ssh/config 
	~~~	
	
13. Now, if you try to SSH into the Gerrit server using following command you should see a message.

	~~~bash
	ssh admin@192.168.56.103 -p 29418
	~~~
	
	![](screenshots/setup-code/setup-code-10.png)
	
14. The user account is now configured to push code and we can upload the code for `demo-app` by pushing it to *`master`* branch in Gerrit. You can push the code by executing the following command:

	~~~bash
	git push gerrit master
	~~~
	
	and this will push the code that we clone from Github into Gerrit.
	
	![](screenshots/setup-code/setup-code-11.png)
	![](screenshots/setup-code/setup-code-12.png)
	

15. **Note of Gerrit access control**

	* The overall idea is to have Github as upstream code repo used only for `fetch/pull`
	
	* Any new change to the code will be submited in Gerrit as a patch set, gets validated, tested, reviewed and then accepted.
	
	* Once the change is accepted and merged, Jenkins will Deploy the build(in the case of automated deployments).
	
	* Only the Gerrit user will have *write* access in Github and all other developers will have only *read* access enabling them to fetch updates but not push directly bypassing Gerrit.
	
	* On Gerrit, the access is controlled with the help of Groups, each project can inherit its access control policies from other projects or manage their own.
	
	* Users can be added to multiple groups, by default the first user who logs into the system after initializing the Gerrit site will become `admin`
	
	* The `admin` user as the name implies can administer Projects, Groups, People, Access, Plugins etc.,
	
	* All other users will be part of `Registered Users` groups and they will have standard access. The access is managed using distinct `Permissions` getting assigned to different `Groups` for different `References` or as a `Global Capability`

16. As mentioned earlier, if Gerrit needs to write the changes to Github, we need some sort of Replication mechanism to sync the code changes that is available in Gerrit. Gerrit offers a built-in `Replication` plugin to do that.

17. This `Replication` plugin needs to be configured in Gerrit and SSH keys are needed for the Gerrit user to *push* changes to Github via SSH.

18. Login to VM and become the `gerrit` user by typing 

	~~~bash
	sudo su gerrit
	~~~
	
19. Generate a SSH keypair by running

	~~~bash
	ssh-keygen -t rsa -b 2048
	~~~
	
20. Copy the SSH public key with

	~~~bash
	cat ~/.ssh/id_rsa.pub
	~~~
	
21. Login to your Github account and add this SSH key under `Settings` > `SSH and GPG keys` and `New SSH key` and paste your key and save it by clicking `Add SSH key`

	![](screenshots/setup-code/setup-code-13.png)
	![](screenshots/setup-code/setup-code-14.png)
	![](screenshots/setup-code/setup-code-15.png)
	![](screenshots/setup-code/setup-code-16.png)
	

22. Switch back to your VM and as `gerrit` user validate the access by trying to SSH into github.com

	~~~bash
	sudo su gerrit
	
	ssh git@github.com
	~~~
	
	you should be greeted with a message from Github.com without any shell access.
	
	![](screenshots/setup-code/setup-code-17.png)
	
23. Now that, SSH key has been added for the `gerrit` user to login to Github with SSH, the same account can be used to configure the `Replication` plugin in Gerrit.

24. Create a file called `replication.config` in `/home/gerrit/review_site/etc/` directory and add the following content, be sure to replace the github account according to your use.

	~~~bash
	[remote "github"]
 	 url = git@github.com:sabs6488/${name}.git
 	 push = +refs/heads/*:refs/heads/*
 	 push = +refs/tags/*:refs/tags/*
 	 timeout = 5
 	 replicationDelay = 0
 	 authGroup = Replication
	~~~
	
25. Create a group in Gerrit called `Replication` and add the admin user to it.

	![](screenshots/setup-code/setup-code-18.png)
	![](screenshots/setup-code/setup-code-19.png)
	![](screenshots/setup-code/setup-code-20.png)

26. Also, the projects should allow the `Replication` group members to`Start Replication` and `Read` from all refs.

	![](screenshots/setup-code/setup-code-21.png)
	![](screenshots/setup-code/setup-code-22.png)
	![](screenshots/setup-code/setup-code-23.png)
	![](screenshots/setup-code/setup-code-24.png)
	![](screenshots/setup-code/setup-code-25.png)

27. Now from the VM restart the gerrit service to make the changes effective.

	~~~bash
	./review_site/bin/gerrit.sh stop
	./review_site/bin/gerrit.sh start
	~~~
	
28. Now login to the VM as `devops` user and execute the following command to start `Replication`

	~~~bash
	ssh admin@192.168.56.103 -p 29418 replication start
	~~~


**[Back to top](#)**

------------------------------

<h2>Setup Continuous Integration job in Jenkins</h2>

1. Create a verification job in Jenkins to trigger a job on new patch set created event in Gerrit.

	* Configure Jenkins credentials
	
	![](screenshots/setup-ci/setup-ci-1.png)

	![](screenshots/setup-ci/setup-ci-2.png)

	![](screenshots/setup-ci/setup-ci-3.png)

	![](screenshots/setup-ci/setup-ci-4.png)

	![](screenshots/setup-ci/setup-ci-5.png)

	![](screenshots/setup-ci/setup-ci-6.png)
	
	* Setup `demo-verify` job in Jenkins to verify the patch set by executing the test cases.
	
	![](screenshots/setup-ci/setup-ci-7.png)
	
	![](screenshots/setup-ci/setup-ci-8.png)
	
	![](screenshots/setup-ci/setup-ci-9.png)
	
	![](screenshots/setup-ci/setup-ci-10.png)
	
	![](screenshots/setup-ci/setup-ci-11.png)
	
	![](screenshots/setup-ci/setup-ci-12.png)
	
	![](screenshots/setup-ci/setup-ci-13.png)
	
	![](screenshots/setup-ci/setup-ci-14.png)
	
	![](screenshots/setup-ci/setup-ci-15.png)
	
	![](screenshots/setup-ci/setup-ci-16.png)
	
	![](screenshots/setup-ci/setup-ci-17.png)
	
	![](screenshots/setup-ci/setup-ci-18.png)
	
	![](screenshots/setup-ci/setup-ci-19.png)
	
	![](screenshots/setup-ci/setup-ci-20.png)
	
	![](screenshots/setup-ci/setup-ci-21.png)
	
	![](screenshots/setup-ci/setup-ci-22.png)
	
	![](screenshots/setup-ci/setup-ci-23.png)
	
	![](screenshots/setup-ci/setup-ci-24.png)
	
	![](screenshots/setup-ci/setup-ci-25.png)

	* Configure Gerrit Trigger plugin for version compatibility

	![](screenshots/setup-ci/setup-ci-26.png)

	![](screenshots/setup-ci/setup-ci-27.png)
	
	![](screenshots/setup-ci/setup-ci-28.png)
	
	![](screenshots/setup-ci/setup-ci-29.png)
	
	![](screenshots/setup-ci/setup-ci-30.png)
	
	![](screenshots/setup-ci/setup-ci-31.png)
	
	![](screenshots/setup-ci/setup-ci-32.png)



2. Validate the build job to see if it is getting triggered on a new patch set created event.

3. Add a new .gitignore file and check if the Jenkins build job is getting triggered.

	Login to the VM as `devops` user and cd into the `demo-app` directory.
	
	Create a file called `.gitreview` and add the following content
	
	~~~bash
	echo "[gerrit]
host=192.168.56.103
port=29418
project=demo-app" >> .gitreview
	~~~
	
	Create a file called `.gitignore` and add the following content
	
	~~~bash
	echo "*.pyc
env
flaskr.db
.gitreview" >> .gitignore 
	~~~
	
4. add and commit the changes to the local development work tree. then submit this changes for review to see if the Jenkins verification job is getting triggered or not.

	~~~bash
	git status
	~~~
	
	![](screenshots/setup-ci/setup-ci-33.png)	
	
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
	
	![](screenshots/setup-ci/setup-ci-35.png)
	
	Now `.gitreview` file shouldn't be listed in untracked files section as `.gitignore` instructs git not to track it.
	
	Check Gerrit web UI to see for any open changes
	
	![](screenshots/setup-ci/setup-ci-36.png)
	
	Submit this change for review
	
	~~~bash
	git review
	~~~
	
	This command will prompt for the username to connect to Gerrit, this is one time and you can use the `admin` user as we have already setup connectivity using SSH keys.
	
	![](screenshots/setup-ci/setup-ci-37.png)
	
	This will create a new change in open changes section in Gerrit UI

	![](screenshots/setup-ci/setup-ci-38.png)

	Jenkins `demo-verify` will be triggered due to the matching `PatchSet created` event and Jenkins will send `+1` score for Verified

	![](screenshots/setup-ci/setup-ci-39.png)

	![](screenshots/setup-ci/setup-ci-40.png)

	![](screenshots/setup-ci/setup-ci-41.png)

	![](screenshots/setup-ci/setup-ci-42.png)

	![](screenshots/setup-ci/setup-ci-43.png)


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
2. In Jenkins, create a new freestyle project called `demo-build` build the image on new Gerrit change merged event.

	![](screenshots/setup-cd/setup-cd-1.png)
	
	![](screenshots/setup-cd/setup-cd-2.png)
		
	![](screenshots/setup-cd/setup-cd-3.png)
		
	![](screenshots/setup-cd/setup-cd-4.png)
		
	![](screenshots/setup-cd/setup-cd-5.png)
		
	![](screenshots/setup-cd/setup-cd-6.png)
		
	![](screenshots/setup-cd/setup-cd-7.png)
		
	![](screenshots/setup-cd/setup-cd-8.png)
		
	![](screenshots/setup-cd/setup-cd-9.png)
		
	![](screenshots/setup-cd/setup-cd-10.png)
	
	![](screenshots/setup-cd/setup-cd-11.png)
	
	![](screenshots/setup-cd/setup-cd-12.png)
	
	![](screenshots/setup-cd/setup-cd-13.png)

3. In order to trigger this job, a new patch set needs to be created, verified and merged in Gerrit.
	
	cd into the working directory
	
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
	
4. This will create a new patch set in Gerrit and will trigger the `demo-verify` job just like in the previous lab.

	![](screenshots/setup-cd/setup-cd-14.png)
	
	![](screenshots/setup-cd/setup-cd-15.png)
	
5. Once the change has been reviewed and submitted the `demo-build` job is suppose to trigger.

	![](screenshots/setup-cd/setup-cd-18.png)
	
	![](screenshots/setup-cd/setup-cd-19.png)
	
	![](screenshots/setup-cd/setup-cd-20.png)
	
	![](screenshots/setup-cd/setup-cd-21.png)
	
	![](screenshots/setup-cd/setup-cd-22.png)
	
	![](screenshots/setup-cd/setup-cd-23.png)
	
6. Now, `demo-build` would have deployed the changes as new container in the docker host. You can verify the changes by visiting `http://192.168.56.103:9000` (replcae with your IP).
	
	![](screenshots/setup-cd/setup-cd-24.png)
	
7. This should've also replicated the changes to Github
	
	![](screenshots/setup-cd/setup-cd-25.png)
	
	![](screenshots/setup-cd/setup-cd-26.png)


 **[Back to top](#)**
 
----------------------

<h2>Verify and Validate CI/CD pipeline for incremental changes</h2>

1. Make additional changes and submit the patch set the CI job and once the code is merged the CD job should be triggered automatically.
2. This concludes the basics of CI/CD with Github, Gerrit, Jenkins and Docker.
3. This model can be scaled to various scenarios and there could be multiple use cases where Jenkins can deploy to a remote docker cluster, or even a Kubernetes cluster based on the requirements.
4. Jenkins is extensible, thanks to the amazing plugin ecosystem available for Jenkins to make it integrate with most of the Dev and Ops systems available in the industry.

 **[Back to top](#)**
 
----------------------
