# Demo Project

### Create a CI Pipeline with `Jenkinsfile` using:
- [Freestyle](#freestyle)
- [Pipeline](#pipeline)
- [Multibranch Pipeline](#multibranch-pipeline)


#### Technologies Used
- Jenkins  
- Docker  
- Linux  
- Git  
- Java  
- Maven  

#### Project Description
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

### Install Build Tools (Maven, Node) in Jenkins

There are two options for installing build tools:

##### Option 1: Install via Jenkins UI

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

#### Install npm and Node in Jenkins Container

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

##### Install Stage View Plugin

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


## Freestyle

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

#### B. Run Tests and Build Jar File

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

### 🐳Docker in Jenkins

1. **Stop your Docker container:**
```bash
   docker ps
   docker stop <container_id>
```

2. Check that the Jenkins volume still exists: `docker volume ls`.
3. Start a new Jenkins container with volumes attached (for Jenkins data and Docker socket). Second volume is a docker volume on the host (droplet) and we’ll mount it in the container under the same destination:
```bash
docker run -p 8080:8080 -p 50000:50000 -d \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkins/jenkins:lts
```

4. Verify the container is running: `docker ps`.
5. Open your browser and go to `http://<your-droplet-ip>:8080`
Jenkins should be running with the previous configuration. Log in as usual.
6. Access the Jenkins container (as root):
```bash
docker exec -u 0 -it <container_id> bash
```

7. Enable Docker inside the Jenkins container:
```bash
curl https://get.docker.com/ > dockerinstall && chmod 777 dockerinstall && ./dockerinstall
```

Jenkins will fetch the latest Docker version from the official site, install it inside the container, and configure it.
8. Check permissions on the Docker socket file: `ls -l /var/run/docker.sock`.
You should see something like: **srw-rw---- 1 root 111 0 Jul 31 11:22 /var/run/docker.sock**
9. Update permissions to allow all users to access Docker: `chmod 666 /var/run/docker.sock`
✅ Test that Jenkins can now run Docker commands:

Log in to the container as the jenkins user: `docker exec -it <container_id> bash`
- Then try: docker pull redis
If the image pulls successfully, your setup is working correctly!

#### 🛠️ Build Docker Image from Jenkins

1. **Edit the `java-maven-build` job in the browser:**
   - Open Jenkins and go to the `java-maven-build` job.
   - Click **Configure**.
   - **Remove** the existing test step (if any).
   - **Add a new build step**:
     - Select: **Execute shell**
     - Command:
       ```bash
       docker build -t java-maven-app:1.0 .
       ```
  
   - Click **Save**.

2. **Build the project:**
   - Click **Build Now** in the job.
   - Open the build log to verify that the Docker image was built successfully.

3. **Verify the image exists (in your terminal):**
   ```bash
   docker images
  ```

You should see something like:
REPOSITORY         TAG       IMAGE ID       CREATED         SIZE
java-maven-app     1.0       abc123456789   X minutes ago   XYZMB

## 📦 Push Docker Image to Docker Hub

### 🔐 1. Create a Docker Hub account and repository
- Go to [https://hub.docker.com/](https://hub.docker.com/)
- Create a new **repository** named: `demo-app`
- Set it to **Private**
- Click **Create**

---

#### 🔑 2. Add Docker Hub credentials in Jenkins

- Navigate to:  
  **Jenkins Dashboard → Manage Jenkins → Security → Credentials**
- Under **Stores scoped to Jenkins**, click on **System**
- Click: **Global credentials (unrestricted)**  
  URL path should be:  
  `/manage/credentials/store/system/domain/_/`

- Click **Add Credentials** and fill in:
  - **Kind:** Username with password
  - **Username:** your Docker Hub username
  - **Password:** your Docker Hub password
  - **ID:** `docker-hub`
  - **Description:** `Docker Hub`

---

#### ⚙️ 3. Configure `java-maven-build` job to push the image

- Go to your **`java-maven-build`** job in Jenkins
- Click **Configure**

---

#### 🔐 4. Bind Docker Hub credentials securely

- Scroll to **Build Environment**
- Check: **Use secret text(s) or file(s)**
- Under **Bindings**, click **Add**
  - Select: **Username and password (separated)**
  - Username variable: `USERNAME`
  - Password variable: `PASSWORD`
  - Credentials: select the one labeled `Docker Hub`

---
 
#### 🛠️ 5. Add build step to tag and push the Docker image

- Scroll to **Build** → Add **Execute shell** step with the following script:
```bash
  docker build -t dockerhubrepo:1.0 .
  echo $PASSWORD | docker login -u $USERNAME --password-stdin
  docker push dockerhubrepo:1.0
```

- Replace dockerhubrepo with your actual Docker Hub repo.
- Make sure to use environment variables or credentials binding to securely access the Docker Hub username and password.
- Click Save.

#### ▶️ 6. Build the project
- Click Build Now
- Check the console output to verify:
- Docker login was successful

#### 🌐 7. Verify the image on Docker Hub
Go to [https://hub.docker.com/](https://hub.docker.com/). 
Open your demo-app repository.
You should see the image tag listed.

##### 🚢 Push Docker Image to Nexus Repository

1. **Add insecure registries to Docker daemon** on the droplet:  
   Edit `/etc/docker/daemon.json` using `vim` or your preferred editor:
   ```json
   {
     "insecure-registries": ["IP_ADDRESS:8083"]
   }
```

2. Restart Docker: systemctl restart docker
3. List all containers (including stopped ones): docker ps -a
4. Run the container: docker start <container_id>
5. Access the container as root and update permissions on Docker socket
```bash
docker exec -u 0 -it <CONTAINER_ID> bash
chmod 666 /var/run/docker.sock
exit
```

6. Exit the container: *exit*
7. **Create Nexus credentials in Jenkins**  
   - Similar to Docker Hub credentials, go to:  
     **Jenkins Dashboard → Manage Jenkins → Security → Credentials**  
   - Add new credentials for Nexus and name them `nexus-docker-repo`.

8. **Configure `java-maven-build` to use Nexus credentials**  
   - In the job configuration, replace Docker Hub credentials with `nexus-docker-repo`.

9. **Build and push Docker image to Nexus repo**  

```bash
   docker build -t CONTAINER_ID:8083/java-maven-app:1.1 .
   echo $PASSWORD | docker login -u $USERNAME --password-stdin CONTAINER_ID:8083
   docker push CONTAINER_ID:8083/java-maven-app:1.1
```

10. Build the project and check the console output
Ensure that the image is pushed without errors.
11. Verify the image in Nexus
Log in to your Nexus repository manager and confirm the image is listed.

## Pipeline

All the changes are done in the `Jenkinsfile`. You can update it with your private Docker Hub repo name.

1. In the browser, go to **New Item**.
2. Name the job: `my-pipeline` and select **Pipeline** as the type.
3. In the **Pipeline** section:
   - Choose **Pipeline script from SCM**.
   - Select **Git** as the SCM.
   - Paste the URL of the `java-maven-app` repository.
   - Select the same credentials used in the previous job.
4. Specify the branch to build, e.g., `jenkins-branch`.
5. Verify the **Script Path** is set to `Jenkinsfile`.
6. Click **Save** and then **Build Now**.

> **OBS:**  
> If your droplet was stopped and you restarted the container,  
> you need to re-add read/write permissions to `docker.sock`:

1. Access the container as root:
   ```bash
   docker exec -u 0 -it <container_ID> bash
   ```

2. Add the permissions: `chmod 666 /var/run/docker.sock`
3. Exit the container: `exit`

##  Multibranch Pipeline

1. In the browser, go to **New Item**.
2. Name the job: `my-multibranch-pipeline` and select **Multibranch Pipeline** as the option.
3. In **Branch Sources**, configure Git:
   - Add a Git source with the HTTPS URL of your GitHub repository.
4. Under **Discover branches**, select **Filter by name (with regular expression)** and use `.*` to match all branches.
5. Click **Save**.
6. **OBS:** Whenever a Jenkinsfile is found in a branch, Jenkins will build that branch. If no Jenkinsfile is present, the branch is skipped.  
   (You have one Jenkinsfile that is shared across branches.)
7. When you click inside `my-multibranch-pipeline`, all branches will be displayed.
8. To rerun branch scans, click on **Scan Multibranch Pipeline Now**.
 
