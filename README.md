# main_devops_project
**Step1**
Launch 3 ubuntu instances 
Sg: All traffic
name it as below 
    1. Jenkins_server
    2. Staging_server
    3. Prod_server
Install jenkins setup on jenkins server 

**Step2**
On staging and Prod server do below steps 
#sudo su
#passwd root
#Vijay22@
#vi /etc/ssh/sshd_config
make here permit login uncommented and root login to yes 
#systemctl restart sshd 

**Step3 **
Now, we have to connect our nodes i.e staging server and prod server to jenkins node 
Manage jenkins > manage nodes and clouds > new node 
Name - Staging_server 
Remote root directory - /home/ubuntu/jenkins
Usage - use this node as much as possible 
Launch method - launch agents via ssh 
       host - <private ip of staging_server>
       credentials - add - jenkins - username and passwords - giver here uname as root and pw as Vijay22@
Host Key Verification Strategy - manually trusted key verification statergy     
leave everything as default 
##Do the above steps for prod server also 

**Step3**
in github create repo name as main_devops_project
ssh into jenkins server 
#sudo su 
#mkdir main_devops_project
#cd main_devops_project
# vi index.html
#git init 
#git add .
#git commit -m "adding file into git"
#git remote add origin <repo url>
#git push origin master
(it will push the index.html file to github)

**Step4**
Now, create 1st freestyle job which will load code from git to staging_server only 
Name - Git_job
Restrict where this project can be run - label expression = Staging_server
Git repo - <add repo url here>
branch - master
save the job 

**Step5**
install docker onto Staging_server and Prod_server
#apt install docker.io

**Step6**
Creation of docker file - so that it will copy index file from jenkins server to staging into docker container 
ssh into jenkins server 
#sudo su 
#cd main_devops_project
#vi Dockerfile
push this dockerfile to git 

**Step7**
Now, create build job which will only build your code 
Name - build-website (freestyle)
Restrict where this project can be run - label expression = Staging_server
build--execute shell -- 
#sudo docker rm -f $(sudo docker ps -a -q)
#sudo docker build /home/ubuntu/jenkins/workspace/Git_job/ -t website
#sudo docker run -it -p 82:80 -d website

Run Git_job first and then build-website job 
Now, check website is deployed on container or not 
<public_ip of staging_server>:82
Result: You will see your website is deployed on port 82

**Step8**
We have run 1 1 job manually 
Now , we have to integrate those job and create cicd pipeline
Go to Git_job 
add post build action > build other projects
project to build - build-website 
trigger only if the build is stable 
Save it 

**Step9**
integrate jenkins with github with the help of webhook
so that if you made any change in code , new job is triggered 
Test: 
made some changes in index.html , add file , commit it 
Result: you will see new job is triggered automatically 

**Step10**
Now, we have to finally deploy our website to prod_server 
Create new job name as Prod_job (freestyle)
Github project - <repo url>
Restrict where this project can be run - label expression = Prod_server
scm: mention <repo url > here also 
build--
    execute shell 
    #sudo docker rm -f $(sudo docker ps -a -q)
    #sudo docker build /home/ubuntu/jenkins/workspace/Prod_job/ -t website
    #sudo docker run -it -p 80:80 -d website (we are running it on port 80 coz we finally deployed it on prod server)

go to build-website job and add post build action --add here prod_job

on prod server run random container 
#sudo docker run -it -d ubuntu
so that our pipeline works (coz we mention any privious container running should delete first)
Check it on brower 
<public_ip of Prod_server>:80
