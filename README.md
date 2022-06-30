# Running Karyon calculator with Jenkin's job

Project using the Calculator repository and running in 03 Jenkins jobs. <br> Job 01: The artifact of the calculator project is uploaded on Jfrog Artifactory <br> Job 02: The artifact is downloaded and a packer image of this artifact is built and uploaded to a Docker Hub repository <br> Job 03: The calculator image is pulled from Docker Hub and run

- [Required](#Required)
- [Configuration](#configuration)
  - [Jfrog](#jfrog)
  - [Jenkins](#jenkins)
- [Running with Jenkinsfile](#Running with Jenkinsfile)
- [Running without Jenkinsfile](#Running without Jenkinsfile)
  - [Jobs](#jobs)
    - [Job 01](#Creating job 01)
    - [Job 02](#Creating job 02)
    - [Job 03](#Creating job 03)
- [Endpoints](#endpoints)

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
       2. Name E.g.: *packer_job02* -> *Install automatically* -> Choose a version compatible with your S.O. _E.g.: linux (amd64)_ -> Save 

## Running without Jenkinsfile

### Jobs

#### Creating job 01

1. On *Dashboard* click in `New Item`
2. Set a name. E.g.: *Automating_Calculator_Artifactory*, choose `Freestyle Project` and click `OK`
3. Go to `Source Code Management` -> Git -> Paste *https://github.com/Gabriely-get/Calculator.git* -> On *Branches to build*  change **/master* to **/main*
4. Go to `Build Environment` -> `Gradle-Artifactory Integration` -> Choose the configured artifactory -> Click on *Refresh Repositories* and select the calculator artifactory
5. Go to `Build` -> `Add build step` -> `Invoke Gradle script` -> Choose the configured gradle on *Gradle Version* -> in *Tasks* type *test*
6. Repeat _step 4_ but type *build* int the new *Tasks*
<br> <br>
#### Creating job 02

1. On *Dashboard* click in `New Item`
2. Set a name. E.g.: *Download_Calculator-RxNetty*, choose `Freestyle Project` and click `OK`
3. Go to `Source Code Management` -> Git -> Paste *https://github.com/Gabriely-get/Calculator.git* -> On *Branches to build*  change **/master* to **/main*
4. Go to `Build Environment` -> `Use secret text(s) or file(s)` -> 
   1. Add *Secret text*. Name as *repository* and choose the *repository credential*
   2. Add *Username and password (separated)*. Choose the docker hub credential and name as *username* and *password* 
5. Go to `Build Environment` -> `Generic-Artifactory Integration` -> On *Download Details* choose the configured Artifactory, then go to *Download spec source* and choose *Job Configuration* and paste the Spec:
   <pre> 
      {
          "files": [
              {
                  "pattern": "generic-calculator-build/com.gabrielyget/Calculator/1.0/Calculator-shadow-1.0.tar",
                  "flat": "true",
                  "target": "exploded_calculator/",
                  "explode": "true"
              }
          ]
      }
   </pre>
6. On `Post-build Actions` -> Choose the *Packer installation* -> Select *Packer Template File* -> In *Additional Parameters* paste this code line and *Save*.
   1.     -var REPOSITORY=$repository  -var USERNAME=$username  -var PASSWORD=$password
<br>

#### Creating job 03
1. On *Dashboard* click in `New Item`
2. Set a name. E.g.: *Pull_and_Run_Calculator_Artifactory*, choose `Freestyle Project` and click `OK`
3. Go to `Build` -> `Execute Shell` -> Paste the code
   <pre>
   if [[ "$(docker images -q gabsss/calculator-rxnetty:latest)" == "" ]]; then
      docker pull gabsss/calculator-rxnetty:latest
      docker run --name calculator -p 8888:8888 gabsss/calculator-rxnetty:latest
   else
      docker rm -f calculator
      docker run --name calculator -p 8888:8888 gabsss/calculator-rxnetty:latest
   fi
   </pre>

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