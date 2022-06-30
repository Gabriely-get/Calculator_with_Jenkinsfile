# Running Karyon calculator with Jenkin's job

Project using the Calculator repository and running in 03 Jenkins jobs. <br> Job 01: The artifact of the calculator project is uploaded on Jfrog Artifactory <br> Job 02: The artifact is downloaded and a packer image of this artifact is built and uploaded to a Docker Hub repository <br> Job 03: The calculator image is pulled from Docker Hub and run

- [Required](#required)
- [Configuration](#configuration)
  - [Jfrog](#jfrog)
  - [Jenkins](#jenkins)
- [Running with Jenkinsfile](#running-with-jenkinsfiles)
- [Endpoints](#endpoints)
- [Extra](#extra-references)

<hr>

## Required

- Java 8 / Java 11
- Install and configure Docker 
- Create a jfrog account: https://jfrog.com/start-free/#hosted
- Create a Docker Hub account: https://hub.docker.com/signup

## Configuration
### Jfrog

1. Pull Jfrog Artifactory docker image: 
   1.     docker pull releases-docker.jfrog.io/jfrog/artifactory-pro:latest
2. Create a Jfrog volume: 
   1.     docker volume create artifactory-data
3. Run the Artifactory Docker container: 
   1.     docker run -d --name artifactory -p 8082:8082 -p 8081:8081 -v artifactory-data:/var/opt/jfrog/artifactory releases-docker.jfrog.io/jfrog/artifactory-pro:latest
4. Access Jfrog: http://localhost:8082/
   ###### When logging in for the first time, the _username_ is: admin and the _password_ is: password
5. Create a Repository:
   1. Click on *Create a Repository*
   2. Click on *Create Local Repository*
   3. Paste your license which was sent to your registered email account and save
   4. On *Administration* tab: Click on `User Management` -> `Users` -> `admin` -> Put an email and change your password
   5. On *Administration* tab: Click on `Repositories` -> `Add Repositories` -> `Local Repository` -> `Generic` -> Name your repository, E.g.: `generic-calculator-build` 


### Jenkins
1. Pull Jenkins docker image:
   1.     docker pull jenkins/jenkins:lts
2. Create a Jenkins volume:
   1.     docker volume create jenkins_home
3. Run the Jenkins container:
   1.     docker run --name jenkins -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -v $(which docker):$(which docker) jenkins/jenkins:lts
4. Give permission so that the container terminal can run docker commands
   1. Access the container terminal
      1.     docker exec -u root -it jenkins bash
   2. Give permission
      1.     chmod 666 /var/run/docker.sock
5. Access Jenkins: http://localhost:8082/
   ###### The password will be shown in the terminal
6. Install suggested plugins -> Skip and continue as admin -> Verify the URL, finish and start
7. Click on *admin* in the top of the page -> *Configure* -> Change the password -> Go back to *Dashboard*
8. Create credentials: Go to `Manage Jenkins` -> `Manage Credentials` -> `Stores scoped to Jenkins` -> Jenkins -> Global credentials -> *Add Credentials*
   1. Select kind *Username with password* and fill with you Docker Hub credentials
   2. Click *Add Credentials* again. Select kind *Secret text* and write your Docker Hub repository name
9. Install plugins: Go to `Manage Jenkins` -> `Manage Plugins` -> `Available` -> Search and Install *Packer* and *Artifactory*
10. Configuring plugins:
    1. **Gradle**
       1. Go to `Manage Jenkins` -> `Global Tool Configuration` -> Gradle -> `Add Gradle`
       2. Name E.g.: *gradle7* -> *Install automatically* -> Choose a version -> Save <br> <br>
    2. **Artifactory**
       1. Go to `Manage Jenkins` -> `Configure System` -> Jfrog -> `Add Jfrog Platform Instance`
       2. Fill in **Instance ID** as *Calculator Artifactory*
       3. Open terminal -> Type *hostname -I* -> Copy first address
       4. Fill in **JFrog Platform URL** with the Jfrog URL and replace *localhost* with the IP copied
          <br> E.g.: `http://192.168.15.99:8082/`
       5. Fill in **Default Deployer Credentials** with the Jfrog credential and **Test Connection** 
          <br> Now you should see the message: *Found JFrog Artifactory* <br> <br>
    3. **Packer**
       1. Go to `Manage Jenkins` -> `Global Tool Configuration` -> Packer -> `Add Packer`
       2. Name as, E.g.: *packer_1.8.2_linuxamd64* -> *Install automatically* -> Choose a version compatible with your S.O. _E.g.: linux (amd64)_ -> Save 

## Running with Jenkinsfiles
> Running with Jenkinsfiles, these following files won't be needed in Calculator project: build_image.pkr.hcl, common.yml and install-ansible.sh

### Creating jobs
1. On *Dashboard* click in `New Item`
2. Set a name, choose `Pipeline` and click `OK`
   >Choose appropriated names. E.g.: *Automating_Calculator_Pipeline*; *Build_Calculator_Pipeline*.
3. Go to `Pipeline` -> `Definition` -> *Pipeline script from SCM* -> `SCM` -> Git -> Paste https://github.com/Gabriely-get/Calculator_with_Jenkinsfile.git
4. Change the branch **/master* to **/main*
5. Choose the Jenkinsfile path. The steps to create the 03 jobs are the same, the only thing that will change is the path:
   1. ./job-01/Jenkinsfile
   2. ./job-02/Jenkinsfile
   3. ./job-03/Jenkinsfile

## Endpoints
The endpoints for the calculator is set by Get method and available on *http://localhost:8080/calculate*


For calculate is necessary provide values for: *value1*, *value2* and *operation*

- SUM

` http://localhost:8888/calculate?value1=<value>&value2=<value>&operation=SUM`

- SUB

`http://localhost:8888/calculate?value1=<value>&value2=<value>&operation=SUB`

- DIVISION

`http://localhost:8888/calculate?value1=<value>&value2=<value>&operation=DIVISION`

- MULTIPLY

`http://localhost:8888/calculate?value1=<value>&value2=<value>&operation=MULTIPLY`

- POW

`http://localhost:8888/calculate?value1=<value>&value2=<value>&operation=POW`

## Extra References

- https://devopscube.com/run-docker-in-docker/
- https://docs.docker.com/storage/bind-mounts/