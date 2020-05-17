About Task -2 
1.	Create container image that’s has Jenkins installed using docker file 

2.	When we launch this image, it should automatically start the Jenkins service in the container.

3.	Create a job chain of job1, job2, job3 and job4 using build pipeline plugin in Jenkins 

4.	 Job1: Pull the Github repo automatically when some developers push the repo to Github.

5.	 Job2 : By looking at the code or program file, Jenkins should automatically start the respective language interpreter install image container to deploy code ( eg. If code is of PHP, then Jenkins should start the container that has PHP already installed ).

6.	Job3 : Test your app if it is working or not.

7.	Job4: if app is not working , then send email to developer with error messages.

8.	Create One extra job job5 for monitor : If container where app is running. fails due to any reson then this job should automatically start the container again.   
CODE for the Whole Automated Process  ----
DockerFile - 
    FROM centos
    RUN yum install wget -y
    RUN yum install net-tools -y
    RUN wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat/jenkins.repo
    RUN rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key    RUN yum install java-11-openjdk.x86_64 -y
    RUN yum install jenkins -y
    RUN yum install git -y
    RUN yum install sudo -y
    RUN echo "jenkins ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
    USER jenkins
    ENV USER jenkins
    CMD ["java" , "-jar", "/usr/lib/jenkins/jenkins.war"]
   
   Command to build docker image- docker build -t jenkins:v1 .
  
  JOB 1 
  cp -rf * /web

  JOB 2 
  #!/bin/bash 
sudo chroot /web /bin/bash <<"EOT"
fullfilename="/web/index.html"
filename=$(basename "$fullfilename")
ext="${filename##*.}"
echo $ext 
if [ $ext == html ] 
then
	if sudo docker ps | grep html
    then
    	echo "already running"
    else    
		sudo docker run -dit -p 81:80 -v /web:/usr/local/apache2/htdocs --name html httpd
	fi
elif [ $ext == php ]
    then
      if sudo docker ps | grep php
      then 
      echo "already running"
      else
      sudo docker run -dit -p 81:80 -v /web:/var/www/html --name php vimal13/apache-webserver-php
      fi
else
	echo "everything working"
    
 fi
EOT
  JOB 3
     status=$(curl -o /dev/null -s -w "%{http_code}" http://172.20.10.3:81)
      if [ $status == 200 ]
      then
      exit 0
      else 
      exit 1
      fi
      
  JOB 4
 sudo chroot /web /bin/bash <<"EOT"

if sudo docker ps  | grep php
then
echo "it is working fine"
else
  sudo docker rm -f html
   sudo docker run -dit -p 81:80 -v /web:/usr/local/apache2/htdocs --name html httpd
fi
EOT
    
