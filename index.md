# Personal Notes

## Today I Learned

### 2019-01-24

Today I learned about Ansible's `block`. I was trying to group some task together and to avoid executing them with a `when` statement.

I came across [this post](https://stackoverflow.com/a/41561885) on stackoverflow which was exactly what I wanted to do.

```
---

- name: Check for already install app
  stat:
    path: /opt/app/bin/MyApp
  register: app_installed

- block:
    - name: Fetch and extract App
      unarchive:
        src: http://someplace.com/getApp/app.zip
        dest: /opt/
        remote_src: yes

    - name: Install Application
      shell: /opt/app/installer.sh
      args:
        executable: /bin/bash

    - name: Application Install Summary
      vars:
        msg: |
          The application is now installed. This is fake, but it would print only once.

          The block can use a when to disable those tasks.

      debug:
        msg: "{{ msg.split('\n') }}"

  when: app_installed.stat.exists == False
```

### 2018-11-14

Today I learned about the command `bsdiff` and its counterpart `bspatch`.
I already knew the concept of binary diffs and that it could be used to reduce the size of certain updates, but I never took the time try it.

I decided to try it this morning, so here it goes!

Lets create an example:

#### Initial state
You have two computer with a binary at version x.y.a and you get a newer version x.y.b on the first computer. But transferring the whole binary would take to much bandwidth. So you want to transfer only the differences between x.y.a and x.y.b to the second computer.

I simulated this with 2 directories, computer_a and computer_b. The binaries are simply 2 version of a simple application I used from time to time.

```
$ tree
.
├── computer_a
│   ├── freemind-bin-max-1.0.0.zip
│   └── freemind-bin-max-1.0.1.zip
└── computer_b
    └── freemind-bin-max-1.0.0.zip

2 directories, 3 files

```

#### Generating Binary diff on computer_a

The command `bsdiff` take 3 arguments, oldfile, newfile and the name of the resulting patch file.
```
bsdiff computer_a/freemind-bin-max-1.0.0.zip computer_a/freemind-bin-max-1.0.1.zip computer_a/freemind_patch_from_1.0.0_to_1.0.1.bsdiff
```
We then have the following tree.
```
.
├── computer_a
│   ├── freemind-bin-max-1.0.0.zip
│   ├── freemind-bin-max-1.0.1.zip
│   └── freemind_patch_from_1.0.0_to_1.0.1.bsdiff
└── computer_b
    └── freemind-bin-max-1.0.0.zip

2 directories, 4 files

```
Lets look at the size of the files.
```
$ ls -lh computer_a/
total 75M
drwxrwxr-x 2 lams lams 4.0K Nov 14 14:41 .
drwxrwxr-x 4 lams lams 4.0K Nov 14 11:17 ..
-rw-rw-r-- 1 lams lams  36M Nov 14 14:18 freemind-bin-max-1.0.0.zip
-rw-rw-r-- 1 lams lams  36M Nov 14 14:17 freemind-bin-max-1.0.1.zip
-rw-rw-r-- 1 lams lams 3.3M Nov 14 14:41 freemind_patch_from_1.0.0_to_1.0.1.bsdiff
```
See, both binaries are 36MB each but the resulting binary patch is only 3.3MB

#### Transfer the patch file to computer_b

We simulate the transfer of the 3.3 MB patch file to computer_b by simply copying it to the computer_b directory.
```
.
├── computer_a
│   ├── freemind-bin-max-1.0.0.zip
│   ├── freemind-bin-max-1.0.1.zip
│   └── freemind_patch_from_1.0.0_to_1.0.1.bsdiff
└── computer_b
    ├── freemind-bin-max-1.0.0.zip
    └── freemind_patch_from_1.0.0_to_1.0.1.bsdiff

2 directories, 5 files
```

#### Creating new version from the old file and the patch file on computer_b

On the computer_b, we use the command `bspatch` with the old file and the patch file to construct the new file.
The command use the same arguments order as `bsdiff`.
```
bspatch computer_b/freemind-bin-max-1.0.0.zip computer_b/freemind-bin-max-1.0.1.zip computer_b/freemind_patch_from_1.0.0_to_1.0.1.bsdiff
```  
We then have the following tree.
```
.
├── computer_a
│   ├── freemind-bin-max-1.0.0.zip
│   ├── freemind-bin-max-1.0.1.zip
│   └── freemind_patch_from_1.0.0_to_1.0.1.bsdiff
└── computer_b
    ├── freemind-bin-max-1.0.0.zip
    ├── freemind-bin-max-1.0.1.zip
    └── freemind_patch_from_1.0.0_to_1.0.1.bsdiff

2 directories, 6 files
```
Same sizes
```
ls -lah computer_a/* computer_b/*
-rw-rw-r-- 1 lams lams  36M Nov 14 14:18 computer_a/freemind-bin-max-1.0.0.zip
-rw-rw-r-- 1 lams lams  36M Nov 14 14:17 computer_a/freemind-bin-max-1.0.1.zip
-rw-rw-r-- 1 lams lams 3.3M Nov 14 14:41 computer_a/freemind_patch_from_1.0.0_to_1.0.1.bsdiff
-rw-rw-r-- 1 lams lams  36M Nov 14 14:21 computer_b/freemind-bin-max-1.0.0.zip
-rw-rw-r-- 1 lams lams  36M Nov 14 15:00 computer_b/freemind-bin-max-1.0.1.zip
-rw-rw-r-- 1 lams lams 3.3M Nov 14 14:54 computer_b/freemind_patch_from_1.0.0_to_1.0.1.bsdiff
```
Same sum
```
md5sum computer_a/* computer_b/*
3d15122b99d5c830eb9c35034b66d525  computer_a/freemind-bin-max-1.0.0.zip
bb217c2566e1476f11f1a68ff88a5669  computer_a/freemind-bin-max-1.0.1.zip
55d838e290e01c40daedc7443aec69e4  computer_a/freemind_patch_from_1.0.0_to_1.0.1.bsdiff
3d15122b99d5c830eb9c35034b66d525  computer_b/freemind-bin-max-1.0.0.zip
bb217c2566e1476f11f1a68ff88a5669  computer_b/freemind-bin-max-1.0.1.zip
55d838e290e01c40daedc7443aec69e4  computer_b/freemind_patch_from_1.0.0_to_1.0.1.bsdiff
```
All this by transferring only 3.3MB to computer_b instead of 36MB.
Off course, those are simple files with sizes that are insignificant for today's internet and computers. But if you have in mind embedded computers that requires big updates on low bandwidth networks, then I think it should be something to consider.

* [Good resources for bsdiff/bspatch](http://www.daemonology.net/bsdiff/)
* [Man pages for bsdiff](http://manpages.ubuntu.com/manpages/bionic/man1/bsdiff.1.html)
* [Man pages for bspatch](http://manpages.ubuntu.com/manpages/bionic/man1/bspatch.1.html)


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
