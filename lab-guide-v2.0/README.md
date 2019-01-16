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
11. [Q&A](#)


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
	docker exec jenkins cat /etc/jenkins_home/initialAdminPassword
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

1. 

![](screenshots/gerrit-jenkins/gerrit-jenkins-1.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-2.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-3.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-4.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-5.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-6.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-7.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-8.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-9.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-10.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-11.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-12.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-13.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-14.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-15.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-16.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-17.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-18.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-19.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-20.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-21.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-22.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-23.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-24.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-25.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-26.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-27.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-28.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-29.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-30.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-31.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-32.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-33.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-34.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-35.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-36.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-37.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-38.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-39.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-40.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-41.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-42.png)

![](screenshots/gerrit-jenkins/gerrit-jenkins-43.png)


**[Back to top](#)**

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

<h2>Process Overview</h2>

The objective of this course is to automate the above steps using tools like Gerrit, Jenkins, Docker and Github by using integration plugins and process pipelines.

When a new change is made to the development tree, it needs to be integrated to the master as and when it is tested and validated.

The typical flow will be as follows:

![CI/CD workflow](flow.png)



<h2>Setup Gerrit for code reviews</h2>

Setup the Gerrit repository to enable code review

~~~bash
ssh-keygen -t rsa -b 2048 
cat ~/.ssh/id_rsa.pub
cd demo-app/
git remote -v
git remote set-url origin ssh://admin@192.168.56.103:29418/demo-app

git rev-parse HEAD

sudo su gerrit

ssh-keygen -t rsa -b 2048

ssh git@github.com

[remote "github"]
 url = git@github.com:sabs6488/${name}.git
 push = +refs/heads/*:refs/heads/*
 push = +refs/tags/*:refs/tags/*
 timeout = 5
 replicationDelay = 0
 authGroup = Replication
 
./review_site/bin/gerrit.sh stop
./review_site/bin/gerrit.sh start


~~~


![](screenshots/setup-code/setup-code-1.png)

![](screenshots/setup-code/setup-code-2.png)

![](screenshots/setup-code/setup-code-3.png)

![](screenshots/setup-code/setup-code-4.png)

![](screenshots/setup-code/setup-code-5.png)

![](screenshots/setup-code/setup-code-6.png)

![](screenshots/setup-code/setup-code-7.png)

![](screenshots/setup-code/setup-code-8.png)

![](screenshots/setup-code/setup-code-9.png)

![](screenshots/setup-code/setup-code-10.png)

![](screenshots/setup-code/setup-code-11.png)

![](screenshots/setup-code/setup-code-12.png)

![](screenshots/setup-code/setup-code-13.png)

![](screenshots/setup-code/setup-code-14.png)

![](screenshots/setup-code/setup-code-15.png)

![](screenshots/setup-code/setup-code-16.png)

![](screenshots/setup-code/setup-code-17.png)

![](screenshots/setup-code/setup-code-18.png)

![](screenshots/setup-code/setup-code-19.png)

![](screenshots/setup-code/setup-code-20.png)

![](screenshots/setup-code/setup-code-21.png)

![](screenshots/setup-code/setup-code-22.png)

![](screenshots/setup-code/setup-code-23.png)

![](screenshots/setup-code/setup-code-24.png)

![](screenshots/setup-code/setup-code-25.png)


**[Back to top](#)**

<h2>Setup Continuous Integration job in Jenkins</h2>

1. Create a verification job in Jenkins to trigger a job on new patch set created event in Gerrit.
2. Make a change to the source code from the VM and submit the code for review using the `git review` option. 
3. Validate the build job to see if it is getting triggered on a new patch set created event.
4. Add a new .gitignore file and check if the Jenkins build job is getting triggered.


![](screenshots/setup-ci/setup-ci-1.png)

![](screenshots/setup-ci/setup-ci-2.png)

![](screenshots/setup-ci/setup-ci-3.png)

![](screenshots/setup-ci/setup-ci-4.png)

![](screenshots/setup-ci/setup-ci-5.png)

![](screenshots/setup-ci/setup-ci-6.png)

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

![](screenshots/setup-ci/setup-ci-26.png)

![](screenshots/setup-ci/setup-ci-27.png)

![](screenshots/setup-ci/setup-ci-28.png)

![](screenshots/setup-ci/setup-ci-29.png)

![](screenshots/setup-ci/setup-ci-30.png)

![](screenshots/setup-ci/setup-ci-31.png)

![](screenshots/setup-ci/setup-ci-32.png)

![](screenshots/setup-ci/setup-ci-33.png)

![](screenshots/setup-ci/setup-ci-34.png)

![](screenshots/setup-ci/setup-ci-35.png)

![](screenshots/setup-ci/setup-ci-36.png)

**[Back to top](#)**

<h2>Setup Continuous deployment job in Jenkins</h2>

1. This section will show how to build a new docker container image when change is reviewed and merged to master branch in gerrit and also it will trigger the deployment job which will deploy the updated image to docker environment.
2. In Jenkins, create a new freestyle project called `demo-build` build the image on new Gerrit change merged event.
3. In order to trigger this job, a new patch set needs to be created, verified and merged in Gerrit.
4. For demo purposes, in the code there are three files which can be renamed to change the look and feel of the top header.
5. cd into the working directory
	
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


 **[Back to top](#)**


<h2>Verify and Validate CI/CD pipeline for incremental changes</h2>



<h2>Q&A</h2>