# Demo Project

## Install Jenkins on DigitalOcean

**Technologies used:**  
Jenkins, Docker, DigitalOcean, Linux

## Project Description

- Create an Ubuntu server on DigitalOcean
- Set up and run Jenkins

-----

## Steps

1. Go to [DigitalOcean](https://www.digitalocean.com/) and create a new droplet.
2. Choose the **Regular - 4 GB** option for the droplet size.
3. After the droplet is created, change its name to **Jenkins-server**.
4. Go to **Networking → Firewalls**, click **Edit Firewalls**, and assign a firewall to your droplet.
5. Create a new firewall named **Jenkins-firewall**:
   - Leave **port 22** open to allow SSH access to the server.
   - Add a custom rule to open **port 8080** (this will allow access to Jenkins’ web interface).
6. Scroll down to the bottom of the page, assign the firewall to **Jenkins-server**, then click **Create Firewall**.

### Connect to the Server

7. Open your terminal and connect to the droplet via SSH:
```bash
 ssh root@IP_DROPLET
```

8. Update the package lists: `apt update`

### Install Docker

9. Check if Docker is installed by typing: `docker`
If Docker is not installed, install it with: `apt install docker.io`
10. Verify Docker installation by typing: `docker` 
(This should display available Docker commands.)

### Run Jenkins in Docker

11. Start a Jenkins container:
```bash
docker run -p 8080:8080 -p 50000:50000 -d -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts
```

- **First port (`-p 8080:8080`)**:  
  This is the port we use for browser access. We bind port 8080 on our host (the droplet) so we can access the Jenkins UI from a browser.

- **`-p 50000:50000`**:  
  This port is used for communication between the Jenkins master and worker nodes. It allows Jenkins to build and start jobs in distributed or clustered environments.

- **`-v jenkins_home:/var/jenkins_home`**:  
  This option creates a named Docker volume to persist data stored in the `/var/jenkins_home` directory inside the Jenkins container. It ensures Jenkins data (jobs, configs, plugins) are not lost when the container restarts or is recreated.

12. Check that Jenkins is running by typing: `docker ps` 
- You should see a container with port 8080 exposed.

### Access Jenkins

Open your browser and navigate to: **http://IP_DROPLET:8080**
Replace *IP_DROPLET* with your droplet’s public IP address.

### Initialize Jenkins

13. To access the running Jenkins container, first get the container ID by running:
```bash
docker ps
```

Then open a shell inside the container (replace `CONTAINER_ID` with your actual container ID):
```bash
docker exec -it CONTAINER_ID bash
```

14. Inside the container, get the initial admin password by typing:
```bash
cat /var/jenkins_home/secrets/initialAdminPassword
```

- Copy the password and paste it into the Jenkins setup page in your browser.

15. Exit the container shell: `exit`

### Check Jenkins Data Folder

To check the folder on the server where Jenkins data is stored, type:
```bash
docker volume inspect jenkins_home
```

This command will display details about the jenkins_home Docker volume, including the folder path on the server.

**Complete Jenkins Setup**

Click to Install suggested plugins when prompted.

After the plugins are installed, create your first admin user.

Jenkins is now ready to be used!

