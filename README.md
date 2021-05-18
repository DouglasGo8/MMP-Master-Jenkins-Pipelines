# Jenkins Master Pipelines from Scratch

---

- Docker ubuntu permission denied

- Edit - /etc/systemd/system/docker.socket.d/override.conf [mkdir docker.socket.d if not exists] add follow text

> [Socket] SocketGroup=YourUser [touch override.conf if not exists]

- After execute

```shell
sudo systemctl daemon-reload
sudo systemctl restart docker.service
```

## Ubuntu (Must run before start Jenkins Container)

- Cannot write to /var/jenkins_home/copy_reference_file.log. Wrong volume permissions?

```shell
sudo chmod -R 777 /media/data/.jenkins
sudo chown 1000 /media/data/.jenkins
```

Windows
Apply Share volumes Docker again

## Jenkins over Docker

```shell
docker run -d --name jenkins --restart always -p 15666:8080 \
   -v /media/data/.jenkins/home:/var/jenkins_home \
   -v /media/data/.m2/local/repo:/var/.m2/local/repo \
   -v /media/dev/local-repo/vscode/jenkins:/tmp/vscode/repo \
   -v /var/run/docker.sock:/var/run/docker.sock jenkins/jenkins:lts

docker logs jenkins

docker exec -i -t -u root jenkins /bin/bash

apt-get update && \
apt-get -y install apt-transport-https \
     ca-certificates \
     curl \
     gnupg2 \
     software-properties-common && \
curl -fsSL https://download.docker.com/linux/$(./etc/os-release; echo "$ID")/gpg > /tmp/dkey; apt-key add /tmp/dkey && \
add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
   $(lsb_release -cs) \
   stable" && \
apt-get update && \
apt-get -y install docker-ce

// any permession when AccessDenied Exception
sudo chown -R jenkins:jenkins jobs

usermod -aG docker jenkins
usermod -aG root jenkins
chmod 777 /var/run/docker.sock

exit

docker ps
```

## Docker Compose

```html
sudo curl -L
"https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname
-s)-$(uname -m)" -o /usr/local/bin/docker-compose sudo chmod +x
/usr/local/bin/docker-compose
```

## Links

http://antigluk.blogspot.com/2015/03/caching-maven-local-repository-in-docker.html
https://github.com/carlossg/docker-maven/issues/63
https://getintodevops.com/blog/the-simple-way-to-run-docker-in-docker-for-ci

## Pipeline Syntax

```groovy
pipeline {
   agent any
   triggers {cron ('* * * * *')} // each minute
   options { timeout (time: 5)}
   parameters {
      booleanParam (name: 'DEBUG_BUILD', defaultValue: true, description: 'Is it the debug build?')
   }
   stages {
      stage ('Example') {
         environment { NAME = 'Rafal'}
         when { expression { return params.DEBUG_BUILD}}
         steps {
            echo 'Hello from $NAME'
            script {
               def browsers = ['chrome', 'firefox']
               for (int =0;i< browsers.size(); i++) {
                  echo 'Testing the ${browsers[i]} browser.'
               }
            }
         }
      }
   }
   post {
      always { echo 'I will always say Hello again'}
   }

}
```

1. Use any agent
2. Execute automatically every minute
3. Stop if the execution takes
4. Ask for boolean input parameter before starting
5. Set Rafal as the environment variable NAME.
6. Only in the case of the true input parameter
   - Print Hello from Rafal
   - Print Testing the chrome browser
   - Print Testing the firefox browser
7. Print I will always say Hello again, no matter if there are any errors during the executions

## Sections

> - Stages: This defines a series of one or more stage directives
> - Steps: This defines a series of one or more step instructions
> - Post: This defines a series of one or more step instructions that are run at the end of pipeline marked to run as (always, success or failure)

## Directives

> - **Agent**: This specifies where the execution takes place and can define the
label to match the equally labeled agents or docker to specify a container that
is dynamically provisioned to provide an environment for the pipeline
execution
> - **Triggers**: This defines automated ways to trigger the pipeline and can use
cron to set the time-based scheduling or pollScm to check the repository for
changes (we will cover this in detail in the Triggers and notifications
section)
> - **Options**: This specifies pipeline-specific options, for example, timeout
(maximum time of pipeline run) or retry (number of times the pipeline
should be rerun after failure)
> - **Environment**: This defines a set of key values used as environment
variables during the build
> - **Parameters**: This defines a list of user-input parameters
> - **Stage**: This allows for logical grouping of steps
> - **When**: This determines whether the stage should be executed depending on
the given condition
> - **Complete List** [link](https://www.jenkins.io/doc/pipeline/steps/)