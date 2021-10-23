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
         environment { NAME = 'Douglas Db'}
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
5. Set Douglas Db as the environment variable NAME.
6. Only in the case of the true input parameter
   - Print Hello from Douglas Db
   - Print Testing the chrome browser
   - Print Testing the firefox browser
7. Print I will always say Hello again, no matter if there are any errors during the executions

## Sections

> - Stages: This defines a series of one or more stage directives
> - Steps: This defines a series of one or more step instructions
> - Post: This defines a series of one or more step instructions that are run at the end of pipeline marked to run as (always, success or failure)

## Directives

> - **Agent**: This specifies where the execution takes place and can define the
>   label to match the equally labeled agents or docker to specify a container that
>   is dynamically provisioned to provide an environment for the pipeline
>   execution
> - **Triggers**: This defines automated ways to trigger the pipeline and can use
>   cron to set the time-based scheduling or pollScm to check the repository for
>   changes (we will cover this in detail in the Triggers and notifications
>   section)
> - **Options**: This specifies pipeline-specific options, for example, timeout
>   (maximum time of pipeline run) or retry (number of times the pipeline
>   should be rerun after failure)
> - **Environment**: This defines a set of key values used as environment
>   variables during the build
> - **Parameters**: This defines a list of user-input parameters
> - **Stage**: This allows for logical grouping of steps
> - **When**: This determines whether the stage should be executed depending on
>   the given condition
> - **Complete List** [link](https://www.jenkins.io/doc/pipeline/steps/)

## Strategies

- **Trunk-based workflow:** implies constantly struggling against the broken pipeline. If everyone commits to the main codebase, then the pipeline often fails. In this case, the old Continuous Integration rule says, "If the build is broken, then the development team stops whatever they are doing and fixes the problem immediately
- **Branching workflow:** solves the broken trunk issue but introduces another one: if everyone develops in their own branches, then where is the integration? A feature usually takes weeks or months to develop, and for all this time, the branch is not integrated into the main code, therefore it cannot be really called "continuous" integration; not to mention that there is a constant need for merging and resolving conflicts.
- **Forking workflow:** implies managing the Continuous Integration process by every repository owner, which isn't usually a problem. It shares however, the same issues as the branching workflow
- **Application configuration:** This involves software properties that decide how the system works, which are usually expressed in the form of flags or properties files passed to the application, for example, the database address, the maximum chunk size for file processing, or the logging level. They can be applied during different development phases: build, package, deploy, or
run.
- **Infrastructure configuration:** This involves server infrastructure and environment configuration, which takes care of the deployment process. It defines what dependencies should be installed on each server and specifies the way applications are orchestrated (which application is run on which
server and in how many instances)