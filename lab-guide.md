#CI/CD Lab</h1>

##Table of contents

1. **Setup gerrit**
2. **Setup docker runtime**
3. **Setup jenkins**
4. **Setup gerrit/jenkins integration**
5. **Setup CI job to verify the patchset**
6. **Setup CD job to build and publish docker image**
7. **Setup CD job to deploy the services**

###Setup gerrit - Lab I

#####update systemand install pre-requisites

~~~bash
ubuntu@cicd-lab:~$ sudo apt-get -y update
ubuntu@cicd-lab:~$ sudo apt-get install git default-jdk wget -y
~~~

#####create gerrit user

~~~bash
ubuntu@cicd-lab:~$ sudo useradd -m gerrit

ubuntu@cicd-lab:~$ sudo su gerrit

gerrit@cicd-lab:/home/ubuntu$ cd

gerrit@cicd-lab:~$ pwd
/home/gerrit

~~~

#####download gerrit war file and verify java install

~~~bash
gerrit@cicd-lab:~$ wget https://www.gerritcodereview.com/download/gerrit-2.12.8.war -O gerrit.war

gerrit@cicd-lab:~$ java -version
openjdk version "1.8.0_171"
OpenJDK Runtime Environment (build 1.8.0_171-8u171-b11-0ubuntu0.16.04.1-b11)
OpenJDK 64-Bit Server VM (build 25.171-b11, mixed mode)
~~~

#####Initialize gerrit code review site

~~~bash
gerrit@cicd-lab:~$ java -jar gerrit.war init -d review_site

*** Gerrit Code Review 2.12.8
*** 

Create '/home/gerrit/review_site' [Y/n]? Y

*** Git Repositories
*** 

Location of Git repositories   [git]:

*** SQL Database
*** 

Database server type           [h2]: 

*** Index
*** 

Type                           [LUCENE/?]:

*** User Authentication
*** 

Authentication method          [OPENID/?]: 
Enable signed push support     [y/N]?


*** Review Labels
*** 

Install Verified label         [y/N]? y

*** Email Delivery
*** 

SMTP server hostname           [localhost]: 
SMTP server port               [(default)]: 
SMTP encryption                [NONE/?]: 
SMTP username                  : 

*** Container Process
*** 

Run as                         [gerrit]: 
Java runtime                   [/usr/lib/jvm/java-8-openjdk-amd64/jre]: 
Copy gerrit.war to review_site/bin/gerrit.war [Y/n]? 
Copying gerrit.war to review_site/bin/gerrit.war

*** SSH Daemon
*** 

Listen on address              [*]: 
Listen on port                 [29418]: 

Gerrit Code Review is not shipped with Bouncy Castle Crypto SSL v152
  If available, Gerrit can take advantage of features
  in the library, but will also function without it.
Download and install it now [Y/n]? n
Generating SSH host key ... rsa(simple)... done

*** HTTP Daemon
*** 

Behind reverse proxy           [y/N]? 
Use SSL (https://)             [y/N]? 
Listen on address              [*]: 
Listen on port                 [8080]: 
Canonical URL                  [http://localhost:8080/]: http://172.16.101.23:8080/

*** Plugins
*** 

Installing plugins.
Install plugin reviewnotes version v2.12.8 [y/N]? y
Install plugin replication version v2.12.8 [y/N]? y
Install plugin commit-message-length-validator version v2.12.8 [y/N]? y
Install plugin download-commands version v2.12.8 [y/N]? y
Install plugin singleusergroup version v2.12.8 [y/N]? y
Initializing plugins.
No plugins found with init steps.

Initialized /home/gerrit/review_site
Executing /home/gerrit/review_site/bin/gerrit.sh start
Starting Gerrit Code Review: OK
Waiting for server on 172.16.101.23:8080 ... OK
Opening http://172.16.101.23:8080/#/admin/projects/ ...FAILED
Open Gerrit with a JavaScript capable browser:
  http://172.16.101.23:8080/#/admin/projects/

~~~

#####Register initial admin user


![Alt image text](images/labs/gerrit-setup/1.png)

