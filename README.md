# Simple CI/CD building a docker image and uploading it to the Docker hub

architecture
<img width="1663" alt="Screenshot 2022-09-01 at 21 17 41" src="https://user-images.githubusercontent.com/106388100/188001972-c473570a-9c2d-4334-a01d-0739eb2ab7e7.png">

We build a docker image from the Dockerfile located in Github. And we upload the resulting image to Docker Hub\
docker_build.jenkins

```groovy
#!groovy
// Run docker build
properties([disableConcurrentBuilds()])

pipeline {
    agent any
    environment {
      DOCKERHUB_CREDENTIALS=credentials('docker-hub')
    }
    triggers { pollSCM('* * * * *') }
    options {
      buildDiscarder(logRotator(numToKeepStr: '10', artifactNumToKeepStr: '10'))
      timestamps()
    }
    stages {
      stage("Set description") {
          steps {
                script {
                        currentBuild.displayName = "Do ${env.JOB_NAME} in Jenkins pipeline [#${BUILD_NUMBER}]"
              }
          }
      }
      stage ("docker login") {
          steps {
            echo "=================== docker login ==========================="
				     sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
              }
          }
      stage("create docker image") {
          steps {
            echo "==================== start build image ====================="
            dir ('docker/toolbox') {
                    sh 'docker build -t okorsi/docker_build:latest . '
            }
         }
      }
      stage("docker push") {
        steps {
          echo "====================== start pushing image ==================="
          sh '''
          docker push okorsi/docker_build:latest
          '''
        }
      }
    }
    post {
        always {
            cleanWs()
      }
   }
}
````

![Screenshot 2022-09-01 at 22 00 26](https://user-images.githubusercontent.com/106388100/188002817-4d4ac0a3-71ce-4c2a-a9db-d3b232acdd32.png)

<img width="1564" alt="Screenshot 2022-09-01 at 22 01 39" src="https://user-images.githubusercontent.com/106388100/188002856-adfcefae-61ac-4714-8127-2c03fc1cc8fa.png">

```bash
jenkins@client2:~$ docker image ls
REPOSITORY            TAG       IMAGE ID       CREATED          SIZE
okorsi/docker_build   latest    f119b02b0147   27 minutes ago   160MB
hello-world           latest    46331d942d63   5 months ago     9.14kB
jenkins@client2:~$ docker image rm f119b02b0147
Untagged: okorsi/docker_build:latest
Untagged: okorsi/docker_build@sha256:a872d5003473f942fa1190bb246601a24dba9f6d051942120c8cf333df686e31
Deleted: sha256:f119b02b01475f589e3cd9d3a7345d21b463766f2f6185a475938fd5cffd8c2c
Deleted: sha256:3f459ab2131e056bb2fff3c907863f4e3ce38ca9436bed909a88aa3a2d74ded9
Deleted: sha256:5d3e392a13a0fdfbf8806cb4a5e4b0a92b5021103a146249d8a2c999f06a9772
jenkins@client2:~$ docker image ls
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
hello-world   latest    46331d942d63   5 months ago   9.14kB
````


download the docker image from the docker hub to the local server\
docker_pull.jenkins
```groovy
#!groovy
// Run docker build
properties([disableConcurrentBuilds()])

pipeline {
    agent any
    environment {
      DOCKERHUB_CREDENTIALS=credentials('docker-hub')
    }
    triggers { pollSCM('* * * * *') }
    options {
      buildDiscarder(logRotator(numToKeepStr: '10', artifactNumToKeepStr: '10'))
      timestamps()
    }
    stages {
      stage("Set description") {
          steps {
                script {
                        currentBuild.displayName = "Do ${env.JOB_NAME} in Jenkins pipeline [#${BUILD_NUMBER}]"
              }
          }
      }
      stage ("docker login") {
          steps {
            echo "=================== docker login ==========================="
				     sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
              }
          }

      stage("docker pull") {
        steps {
          echo "====================== start pulling image ==================="
          sh '''
          docker pull okorsi/docker_build:latest
          '''
        }
      }
    }
    post {
        always {
            cleanWs()
      }
   }
}
````

```bash
jenkins@client2:~$ docker image ls
REPOSITORY            TAG       IMAGE ID       CREATED          SIZE
okorsi/docker_build   latest    f119b02b0147   29 minutes ago   160MB
hello-world           latest    46331d942d63   5 months ago     9.14kB
jenkins@client2:~$
````


Dockerfile
````
FROM alpine
RUN apk add --no-cache curl wget busybox-extras netcat-openbsd python3 py-pip && pip install awscli && apk --purge -v del py-pip
CMD tail -f /dev/null
```

