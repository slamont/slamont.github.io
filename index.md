# Personal Notes

## Today I Learned

### 2018-07-06

Today I learned about the command `run-parts` while looking for information on [hooks with certbot](https://stackoverflow.com/questions/42300579/letsencrypt-certbot-multiple-renew-hooks).

Its simple description is:
> run-parts - run scripts or programs in a directory

Very useful when you want to add the *dir.d* concept in one of your scripts.

It was not obvious to me right away when reading the manpage, but the scripts that you place in the directory that you will point run-parts to, should **not** have extensions.

i.e:
```
# /scripts
#  a.sh
#  b.sh
#  c.sh
run-parts /scripts
# Will not execute anything
#
# But with /scripts
#  a
#  b
#  c
run-parts /scripts
# Will execute the 3 scripts a,b and c
```

[run-parts manpage](http://manpages.ubuntu.com/manpages/bionic/man8/run-parts.8.html)



### 2018-07-10

Today I learned a cool trick to extract a value from a maven project.

As an example, to extract the project name and the version from maven, here is what I got:

```

project_name=$(./mvnw -Dexec.executable='echo' -Dexec.args='${project.name}' --non-recursive exec:exec -q)
project_version=$(./mvnw -Dexec.executable='echo' -Dexec.args='${project.version}' --non-recursive exec:exec -q)

```

I was looking to use it in a Jenkinsfile in preparation for a validation step that I'm building in another project. It got me to learned how to grab the output of a command and set it as a variable inside the Jenkinsfile.

```

pipeline {
  agent any
  environment {
    PATH = '/opt/python/active/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin'
  }

  // The options directive is for configuration that applies to the whole job.
  options {
    timestamps()
    skipDefaultCheckout()
    buildDiscarder(logRotator(numToKeepStr:'10'))
    timeout(time: 60, unit: 'MINUTES')
    ansiColor('xterm')
    disableConcurrentBuilds()
  }

  stages {
    stage('Building and testing') {
      steps('Checking out code and verifying') {
        checkout scm
        sshagent(['2cafeb85-dead-beaf-cafe-8beaffed5786']) {
          sh "env"
          sh "echo '-------------------------------'"
          sh "#./mvnw -U -B clean verify"
          script {
            env.project_name = sh(script: "./mvnw -Dexec.executable='echo' -Dexec.args='\${project.name}' --non-recursive exec:exec -q", returnStdout: true).trim()
            env.project_version = sh(script: "./mvnw -Dexec.executable='echo' -Dexec.args='\${project.version}' --non-recursive exec:exec -q | sed 's/-SNAPSHOT//g'", returnStdout: true).trim()
          }
          sh "env"
          sh "echo '-------------------------------'"
        }
      }
    }
  }

  post {
    always {
      //This is there as an example
      junit(allowEmptyResults: true, testResults: '**/target/surefire-reports/TEST-*.xml')
    }

    failure {
      slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
    }

    success {
      echo "Build Successful"
      echo "Cleaning up workspace"
      deleteDir()
    }

    unstable {
      slackSend (color: '#FFFF00', message: "UNSTABLE: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
    }

    changed {
      echo "The state of the Pipeline has changed"
    }
  }
}

```

The trick here was the returnStdout parameter to the sh step

* [returnStdout from stackoverflow](https://stackoverflow.com/questions/36547680/how-to-do-i-get-the-output-of-a-shell-command-executed-using-into-a-variable-fro)
* [Official sh description](https://jenkins.io/doc/pipeline/steps/workflow-durable-task-step/#sh-shell-script)
* [Comment on stackoverflow for extracting value of Maven](https://stackoverflow.com/a/36572816/1828564)


### 2018-09-13

Today I learned a very simple trick to get a mostly accurate date and time on a machine which doesn't have any ntp packages installed. With only `cat` and some socket tricks in Linux, it is possible to get the time from the legacy TIME protocol still available from NIST.

```
cat </dev/tcp/time.nist.gov/13
```
Will return something like the following:
```
58374 18-09-13 17:35:57 50 0 0  97.7 UTC(NIST) *  
```

It is also possible to do it using netcat:
```
nc time.nist.gov 13
```

---

Funny stuff, Star Wars in ascii !
```
cat </dev/tcp/towel.blinkenlights.nl/23
```
More telnet fun:
```
telnet telehack.com
```

* [Taken from a comment on superuser.com](https://superuser.com/a/635039)


### 2018-09-14

Today I learned about the command `fold`.
This command is used to wrap the input into a maximum number of columns.
See the following examples to better understand:

No fold
```
echo "Bacon ipsum dolor amet tri-tip porchetta flank pork loin andouille shoulder chicken sirloin shankle. T-bone hamburger turkey rump boudin ham hock corned beef. Porchetta rump tenderloin shank, meatloaf meatball capicola jowl prosciutto landjaeger brisket alcatra short ribs bacon. Chicken filet mignon biltong shoulder."
Bacon ipsum dolor amet tri-tip porchetta flank pork loin andouille shoulder chicken sirloin shankle. T-bone hamburger turkey rump boudin ham hock corned beef. Porchetta rump tenderloin shank, meatloaf meatball capicola jowl prosciutto landjaeger brisket alcatra short ribs bacon. Chicken filet mignon biltong shoulder.
```

fold with default 80 columns
```
echo "Bacon ipsum dolor amet tri-tip porchetta flank pork loin andouille shoulder chicken sirloin shankle. T-bone hamburger turkey rump boudin ham hock corned beef. Porchetta rump tenderloin shank, meatloaf meatball capicola jowl prosciutto landjaeger brisket alcatra short ribs bacon. Chicken filet mignon biltong shoulder." | fold
Bacon ipsum dolor amet tri-tip porchetta flank pork loin andouille shoulder chic
ken sirloin shankle. T-bone hamburger turkey rump boudin ham hock corned beef. P
orchetta rump tenderloin shank, meatloaf meatball capicola jowl prosciutto landj
aeger brisket alcatra short ribs bacon. Chicken filet mignon biltong shoulder.
```

fold with 160 columns
```
echo "Bacon ipsum dolor amet tri-tip porchetta flank pork loin andouille shoulder chicken sirloin shankle. T-bone hamburger turkey rump boudin ham hock corned beef. Porchetta rump tenderloin shank, meatloaf meatball capicola jowl prosciutto landjaeger brisket alcatra short ribs bacon. Chicken filet mignon biltong shoulder." | fold -w 160
Bacon ipsum dolor amet tri-tip porchetta flank pork loin andouille shoulder chicken sirloin shankle. T-bone hamburger turkey rump boudin ham hock corned beef. P
orchetta rump tenderloin shank, meatloaf meatball capicola jowl prosciutto landjaeger brisket alcatra short ribs bacon. Chicken filet mignon biltong shoulder.
```

It could be really useful when you want to quickly format a logfile containing very long lines... It never hurts to learn something new.

