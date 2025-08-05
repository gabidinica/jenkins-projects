# Demo Project

## Create a Jenkins Shared Library

### Technologies Used
- Jenkins  
- Groovy  
- Docker  
- Git  
- Java  
- Maven  

### Project Description
Create a Jenkins Shared Library to extract common build logic:

- Create a separate Git repository for the Jenkins Shared Library project  
- Create functions in the JSL to use in the Jenkins pipeline  
- Integrate and use the JSL in Jenkins Pipeline:  
  - Globally  
  - For a specific project via `Jenkinsfile`  

---

## I. Create Separate Git Repository for Jenkins Shared Library Project

In the `vars` folder you can find all the functions that are common.

1. Clone locally the repo:  
   `https://github.com/gabidinica/jenkins-shared-library-project.git`

2. In GitHub, create a new repository where you’ll push the locally copied one.

3. To make the Shared Library globally available in Jenkins:  
   - In Jenkins UI, go to **Manage Jenkins** → **Configure System**  
   - Scroll to the **Global Pipeline Libraries** section

4. Click on **Add**.  
   - In **Name**, type: `jenkins-shared-library`  
   - For **Default version**, add: `main`

5. To add the GitHub repo:  
   - Select **Modern SCM** → **Git**

6. In the **Project Repository** field, add the HTTPS address of your GitHub repo.  
   - For **Credentials**, add your GitHub credentials

7. Click **Save**

## II. Use Shared Library in Jenkinsfile

1. Go to the `java-maven-app` repository and create a new branch where you’ll use the new functions, called:  
   `jenkins-shared-lib`

2. Move locally to **Visual Studio Code**, checkout the new branch and open the project folder to edit the `Jenkinsfile`.

3. Update `Jenkinsfile` with the following changes, commit and then push:
```bash
 #!/user/bin/env groovy
@Library('jenkins-shared-library')
def gv

pipeline {   
    agent any
    tools {
        maven 'maven-3.9'
    }
    stages {
        stage("init") {
            steps {
                script {
                    gv = load "script.groovy"
                }
            }
        }
        stage("build jar") {
            steps {
                script {
                    buildJar()
                }
            }
        }

        stage("build image") {
            steps {
                script {
                    buildImage('dockerhubName:jma3.0')
                    dockerLogin();
                    dockerPush 'dockerhubName:jma3.0'
                }
            }
        }

        stage("deploy") {
            steps {
                script {
                    gv.deployApp()
                }
            }
        }               
    }
} 
```

- replace dockerhubName with your Docker Hub Repository name

## III. Project Scoped Shared Library

If the Shared Library is only needed for your pipeline or project (e.g., one or two projects), it's best to use a **project-scoped** library instead of configuring it globally.

### Steps:

1. In Jenkins UI, go to:  
   **Manage Jenkins** → **Configure System** → **Global Pipeline Libraries**  
   Remove the global library you previously added.

2. Click **Save**.

3. On your local machine:  
   - Open the `java-maven-app` project in **Visual Studio Code**
   - Edit the `Jenkinsfile` to reference the shared library repository directly:
```bash
#!/user/bin/env groovy

library identifier: 'jenkins-shared-library@main', retriever: modernSCM(
    [$class: 'GitSCMScource',
     remote: 'https://github.com/YOUR_REPO/jenkins-shared-library-project.git',
     credentialsID: 'jenkins-shared-lib-new-pat-classic'])
```

   - replace with this before the `def gv` 


