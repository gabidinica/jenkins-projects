# Demo Project:  
**Dynamically Increment Application Version in Jenkins Pipeline**

## Technologies Used
- Jenkins  
- Docker  
- GitLab  
- Git  
- Java  
- Maven  

## Project Description
- [Increment patch version](#increment-patch-version)  
- [Build Java application and clean old artifacts](#build-java-application-and-clean-old-artifacts)  
- Build image with dynamic Docker image tag  
- Push image to private DockerHub repository  
- [Commit version update of Jenkins back to Git repository](#commit-version-update-of-jenkins-back-to-git-repository)  
- [Avoid commit loop by skipping CI trigger](#avoid-commit-loop-by-skipping-ci-trigger)  

---

## Increment Patch Version

In the terminal, navigate to the `java-maven-app` project directory.

Execute the following Maven command to automatically update the version defined in `pom.xml`:

```bash
mvn build-helper:parse-version versions:set \
  -DnewVersion=${parsedVersion.majorVersion}.${parsedVersion.minorVersion}.${parsedVersion.nextIncrementalVersion} \
  versions:commit
```

*Explanation*:
`build-helper:parse-version`: Locates the pom.xml file and parses the version into three components: major, minor, and incremental.

`versions:set with -DnewVersion=...`: Constructs the new version by incrementing the patch (incremental) part.

`versions:commit`: Applies the version change and updates the pom.xml file.

>⚠️ Note:
>Every time you run this command, the patch version will increment again.
>You can verify the change in the pom.xml file using Visual Studio Code or any text editor.


1. Open the `Jenkinsfile` in Visual Studio Code.
2. In the `build app` stage, the command used is: `mvn package`.
This builds a .jar file using the version specified in the `pom.xml`.
3. To verify the build locally, open the terminal in Visual Studio Code.
Run: `mvn package`
- Navigate to the target/test-classes directory to check that the .jar file has been created.
4. You need to add a new stage named `increment version` in the Jenkinsfile, and ensure it is executed before the mvn package step:
```bash
stage('increment version') {
    steps {
        script {
            echo 'Incrementing app version ...'
            
            sh '''
                mvn build-helper:parse-version versions:set \
                -DnewVersion=\\${parsedVersion.majorVersion}.\\${parsedVersion.minorVersion}.\\${parsedVersion.nextIncrementalVersion} \
                versions:commit
            '''
            def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
            def version = matcher[0][1]
            
            // Set IMAGE_NAME environment variable for Docker tagging
            env.IMAGE_NAME = "$version-$BUILD_NUMBER"
        }
    }
}
```


## Build Java Application and Clean Old Artifacts

We use the **application version** (from `pom.xml`) as the Docker image version.

Update the `build image` stage in your `Jenkinsfile` to use the `IMAGE_NAME` environment variable:

```groovy
stage('build image') {
    steps {
        script {
            sh "docker build -t docker_repo:${IMAGE_NAME} ."
            sh 'echo $PASS | docker login -u $USER --password-stdin'
            sh "docker push docker_repo:${IMAGE_NAME}"
        }
    }
}
```

Update your `Dockerfile` to dynamically copy any `.jar` file from the `target` folder:
```bash
FROM amazoncorretto:8-alpine3.17-jre

EXPOSE 8080

COPY ./target/java-maven-app-*.jar /usr/app/
WORKDIR /usr/app

CMD java -jar java-maven-*.jar
```

To ensure the `target` folder is cleared of old `.jar` files before building the new one, update the `build app` stage in your `Jenkinsfile` as follows:

```groovy
stage("build app") {
    steps {
        script {
            echo 'Building the application...'
            sh 'mvn clean package'
        }
    }
}
```

Commit and push the changes to `jenkins-branch` to test it in Jenkins after that.
Go to Jenkins to java-maven-app and build the pipeline.

## Commit Version Update of Jenkins Back to Git Repository

Add a new stage in your `Jenkinsfile` to commit the updated application version back to the Git repository.
```bash
stage("commit version update"){
            steps{
                script{
                    withCredentials([usernamePassword(credentialsId: '0e1b9c50-4389-48d2-bc25-7efeb014364b', passwordVariable: 'PASS', usernameVariable: 'USER')]){
                 sh 'git config --global user.email "jenkins@example.com"'
                        sh 'git config --global user.name "jenkins"'
                     sh 'git status'
                        sh 'git branch'
                        sh 'git config --list'
                 sh "git remote set-url origin https://${USER}:${PASS}@github.com/gabidinica/java-maven-app.git" 
                 

                        sh 'git add .'
                        sh 'git commit -m "ci: version bump "'
                        sh 'git push origin HEAD:jenkins-branch'
                    }
                }
            }
        }  
```


## Avoid Commit Loop by Skipping CI Trigger

To avoid triggering the pipeline when Jenkins commits a version update (e.g., after incrementing the version), configure the **Ignore Committer Strategy** plugin:

### 🔧 Steps

1. **Install the Plugin:**
   - Go to **Jenkins Dashboard** → **Manage Jenkins** → **Plugins**.
   - Search for `Ignore Committer Strategy`.
   - Click **Install without restart**.

2. **Configure Multibranch Pipeline:**
   - Go to your **Multibranch Pipeline** project in Jenkins.
   - Click **Configure**.
   - Under **Branch Sources**, scroll to **Build Strategies**.
   - Click **Add** → **Ignore Committer Strategy**.

3. **Set the Jenkins Commit Email:**
   - Enter the email you configured in your `Jenkinsfile`, e.g.:
     ```
     jenkins@example.com
     ```

4. **Enable Conditional Builds:**
   - Check the box:
     ```
     Allow builds when a changeset contains non-ignored author(s)
     ```

5. **Save the Configuration.**

---

### ✅ Outcome
With this configuration, Jenkins will **not trigger a build** if the only commit was made by the Jenkins user itself (e.g., version update commit). This prevents unnecessary **build loops**.

To test, make some changes locally and push them, for example in .gitignore file add target folder.
Check in Jenkins that no pipeline is triggered after our commit, also check in GitHub. 
