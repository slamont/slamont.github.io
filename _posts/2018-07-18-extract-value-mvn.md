---
---
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
