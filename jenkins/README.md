# Build
```bash
docker login registry.$DEMO_SERVER_HOSTNAME -u admin
docker-compose build
docker-compose push

docker stack deploy jenkins -c docker-compose.yml
```
Jenkins will become available on `http://jenkins.$DEMO_SERVER_HOSTNAME` .

# Configure jenkins

First, get the initial install tool password from the docker log:
```bash
docker service logs jenkins_jenkins
```

Now, visit http://jenkins.$DEMO_SERVER_HOSTNAME to walk through the Jenkins installation

During the initial installation, install the following plugins
* Credentials Binding
* Pipeline
* Git
* GitHub

After the initial installation, 
* install the `Docker` plugin. Allow jenkins to reboot
* Add the label 'docker' to Jenkins

Add credentials, so Jenkins can use the docker engine and reg: 

### Add credentials for the docker registry

| Field    | Value                        |
| -------- | ---------------------------- |
| Kind     | Username with password       |
| username | admin                        |
| password | admin                        |
| id       | sample-docker-registry-login |

### Add credentails for the docker demo server

The credentials for the docker demo server are stored on your harddrive, within the directory echo $DOCKER_CERT_PATH. Copy the contents of those files into a Jenkins credential.

| Field              | Value                                   |
| ------------------ | --------------------------------------- |
| type               | Docker host certificate authentication  |
| Client Key         | Contents of $DOCKER_CERT_PATH/key.pem   |
| Client Certificate | Contents of $DOCKER_CERT_PATH/cert.pem  |
| Server CA          | Contents of $DOCKER_CERT_PATH/ca.pem    |                                
| id                 | sample-docker-demo-login                |

### Build environment variables

Finally, add a file with build environment variables for the TYPO3 project. 

Sample file: https://raw.githubusercontent.com/SomeBdyElse/typo3_jenkins_sample/master/sample-jenkins-environment.groovy.sample
Modify the sample file to contain the right servernames (Replace dockerdemo.example.com by the value of $DEMO_SERVER_HOSTNAME) and add it to Jenkins as secret file with id `sample-jenkins-environment`. 

### Configure Jenkins to deploy the project

Add a new item of type multibranch pipeline. Configure pipeline to use source from `https://github.com/SomeBdyElse/typo3_jenkins_sample.git`.
