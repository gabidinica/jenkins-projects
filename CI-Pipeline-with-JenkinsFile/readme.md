# Demo Project

## Goal
Create a CI Pipeline with `Jenkinsfile` using:
- Freestyle
- Pipeline
- Multibranch Pipeline

## Technologies Used
- Jenkins  
- Docker  
- Linux  
- Git  
- Java  
- Maven  

## Project Description
CI Pipeline for a Java Maven application to build and push to the repository.

### Steps:
1. Install build tools (Maven, Node) in Jenkins  
2. Make Docker available on the Jenkins server  
3. Create Jenkins credentials for the Git repository  
4. Create different Jenkins job types for the Java Maven project using a `Jenkinsfile`:
   - Freestyle  
   - Pipeline  
   - Multibranch Pipeline  

Each job should:
- a. Connect to the application’s Git repository  
- b. Build the JAR file  
- c. Build the Docker image  
- d. Push the Docker image to a private DockerHub repository  

## I. Install Build Tools (Maven, Node) in Jenkins

There are two options for installing build tools:

#### Option 1: Install via Jenkins UI

##### 1. Configure Plugin for Maven
- In the browser, on Jenkins, click on **Tools**  
- Scroll down to **Maven** and click **Add Maven**  
- Name it `maven-3.9` and select version `3.9.2`, then click **Save**  
- Click again on **Tools** and verify that Maven is installed under **Maven installations**

#### Option 2: Install Directly on Jenkins Server

You can also install tools like Maven and Node directly on the server where Jenkins is running.  
This involves using the system package manager (e.g., `apt`, `yum`, `brew`) or downloading and configuring the tools manually.

Example for Maven on Ubuntu:

```bash
sudo apt update
sudo apt install maven
```

### Install npm and Node in Jenkins Container

1. Go to terminal and SSH into your droplet as root:

```bash
ssh root@DROPLET_IP
```

2. To install Node.js and npm inside the Jenkins container, execute as root using the `-u 0` flag:
```bash
docker exec -u 0 -it <container_id> bash
```

If successful, you’ll see you're logged in as the root user inside the container.

3. To find the Linux distribution running in the container, run:
```bash
cat /etc/issue
```

Example output: `Debian GNU/Linux 12 \n \l`

4. Update the package list:
```bash
apt update
```

5. Check if `curl` is installed: `curl`
If `curl` is not found, install it:
```bash
apt install curl
```

6. Download and run the Node.js installation script (this script will set up the Node.js repository and install Node.js + npm):
```bash
curl -sL https://deb.nodesource.com/setup_20.x -o nodesource_setup.sh
```

> Type the following to list files and verify the script is downloaded: `ls` and you should see the `nodesource_setup.sh` script displayed.

7. To execute the script, run:
```bash
bash nodesource_setup.sh
```

8. Install Node.js:
```bash
apt install nodejs
```

> Verify the Node.js installation by checking the version: `node -v`
> Check the installed npm version: `npm -v`

#### Install Stage View Plugin

The Stage View plugin shows the progress of each pipeline stage.

1. In your browser, open your Jenkins dashboard.  
2. Click on **Manage Jenkins** > **Manage Plugins**.  
3. Go to the **Available** tab.  
4. Search for **Stage View**.  
5. Select the plugin and click **Install without restart** (or **Install and restart Jenkins**).  
6. Wait for the installation to complete and restart Jenkins if required.

##### Managing Jenkins Docker Container and Verifying Plugin Installation

1. In the terminal, check running containers:
 ```bash
   docker ps
```

> If the Jenkins container is **not running**, you won’t see it listed.

2. To see all containers (running or stopped), type: `docker ps -a`
3. Start the Jenkins container: 
```bash
docker start <container_id>
```

4. Go to your browser and refresh the Jenkins dashboard.
5. Navigate to Manage Jenkins > Manage Plugins.
6. Click on the Installed tab, and you should see the plugin listed as installed.


# Freestyle JOB

1. In your browser, go to `http://DROPLET_IP:8080`.  
2. To create a new job, click on **New Item**.  
3. Name the job `my_job` and select **Freestyle project**.  
4. In the configuration window, scroll down to **Build Steps**.  
5. Click **Add Build Step** and select **Execute Shell**.  
6. In the command box, type: `npm --version`
7. Since Maven is configured, add another build step and select **Invoke top-level Maven targets**.  
8. For **Maven version**, select `maven-3.9`.  
9. In **Goals**, enter any Maven command needed; for example: `--version`
10. Click **Save**.
11. Click on the **Jenkins main view** — here you’ll see the list of jobs.  
12. Click on the job name you want to build.  
13. On the right side, click **Build Now**.  
14. After the build completes, you’ll see the results under **Builds**.  
15. Click on the build number.  
16. From the left side menu, click **Console Output** to check the build details and logs.

17. Duplicate the Jenkins window.  
18. Go to **Manage Jenkins** -> **Manage Plugins** -> **Available** tab.  
19. Search for **nodejs** and install the plugin.  
20. After installation, go to **Manage Jenkins** -> **Global Tool Configuration** (or **Tools**).  
21. You will see a **NodeJS** section there.  
22. Configure NodeJS by:  
   - Naming it `my-nodejs`  
   - Selecting version `20.2.0`  
23. Click **Save**.

### A. Connect to the Application’s Git Repository

1. Go to your job (`my_job`) -> **Configuration**.  
2. Under **Source Code Management**, select **Git**.  
3. In the **Repository URL**, add the path to your project repository (e.g., where your `java-maven-app` is located).  
4. For **Credentials**, click **Add** -> **Jenkins**.  
5. Fill in your GitHub/GitLab username and password.  
6. For **ID**, type: `GitHub-credentials`.  
7. Click **Add** and then select these credentials.  
8. For **Branch**, leave it as `master`.  
9. Click **Save**.

##### Access Jenkins Jobs in Docker Container

1. SSH into your droplet:
```bash
ssh root@droplet_IP
```

2. List running Docker containers: `docker ps`
3. Enter the Jenkins container shell: 
```bash
docker exec -it <container_id> bash
```

4. List the contents of Jenkins home directory to see jobs: `ls /var/jenkins_home/`
5. Specifically check the jobs directory: `ls /var/jenkins_home/jobs`

##### Edit Job Configuration

1. Edit the job configuration.  
2. Change the branch from `master` to `ci-pipeline-with-jenkins`
3. In the first build step, remove: npm --version
4. Add the following commands instead:

```bash
chmod +x ./freestyle-build.sh
./freestyle-build.sh
```

### B. Run Tests and Build Jar File

1. Create a new Freestyle project named: `java-maven-build`.  
2. Under **Source Code Management**, select **Git**.  
3. Add the Git repository URL and choose your GitHub credentials.  
4. Select the branch: `ci-pipeline-with-jenkins`.  

5. In the **Build** section, add a **Maven** build step.  
   - For **Goals**, enter: `test`.  

6. Add another **Maven** build step.  
   - For **Goals**, enter: `package`.  

7. Click **Save**.  
8. Run the job.  
9. Click on the latest **Build**, then go to **Console Output** to check the test results.


