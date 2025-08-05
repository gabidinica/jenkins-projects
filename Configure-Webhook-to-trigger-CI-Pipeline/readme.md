# Demo Project

## Configure Webhook to Trigger CI Pipeline Automatically on Every Change

### Technologies Used
- Jenkins  
- GitHub  
- Git  
- Docker  
- Java  
- Maven  

---

## Summary

- [Install GitHub Plugin in Jenkins](#1-install-github-plugin-in-jenkins)  
- [Configure Github Access Token and Connection](#2-configure-github-access-token-and-connection)  
- [Configure Jenkins to Trigger CI on Github Push](#3-configure-jenkins-to-trigger-ci-on-github-push)

---

## 1. Install Github Plugin in Jenkins


- Go to **Manage Jenkins** → **Plugins**
- Search for **GitHub Branch Source**
- Make sure the **GitHub Plugin** is also installed

## 2. Configure Github Access Token and Connection


- Go to **Manage Jenkins** → **Configure System**
- Scroll to the **GitHub** section and ensure a GitHub server is listed
- Click **Add GitHub Server**
- From the dropdown, select **GitHub Server**
- Fill in the fields:
  - **Name**: `GitHub-conn`
  - **API URL**: leave it as default (`https://api.github.com`)
  - **Credentials**: click **Add**, then select **Jenkins** to add your GitHub credentials
- Go to **GitHub**:  
  `Settings` → `Developer settings` → `Personal access tokens` → `Tokens (classic)` → **Generate new token**

- Steps:
  - Give the token a descriptive name (e.g., `Jenkins PAT`)
  - Select the following scopes:
    - `repo` (allows access to your repositories)
    - `admin:repo_hook` *(optional but recommended – allows Jenkins to manage GitHub webhooks)*
  - Click **Generate Token**
  - **Copy the token immediately** – you won’t be able to see it again later

- In **Jenkins**, go to:  
  **Manage Jenkins** → **Credentials** → *(select domain, usually `(global)`)*
  
- Click **Add Credentials**

- Complete the following fields:
  - **Kind**: `Secret text`
  - **Secret**: *Paste your GitHub PAT here*
  - **ID**: `github-pat-token` *(or any unique ID — this is how you'll reference it in Jenkins jobs)*
  - **Description**: *(optional)* `GitHub PAT for triggering builds`

- Click **OK** or **Save**
- Now Save the configuration.

## 3. Configure Jenkins to Trigger CI on Github Push

## 6. Configure Jenkins Job and GitHub Webhook

### In Jenkins:

- Go to your Jenkins dashboard
- Click on the job (e.g., `my-pipeline`) and select **Configure**
- Scroll to the **Build Triggers** section
- Check the option:  
  `GitHub hook trigger for GITScm polling`
- Click **Save**

### In GitHub:

- Go to your GitHub repository (e.g., `java-maven-app`)
- Navigate to:  
  `Settings` → `Webhooks` → **Add webhook**

- Fill in the webhook details:
  - **Payload URL**:  
    `http://<jenkins-url>/github-webhook/`  
    > In our case: `http://Droplet_id:8080/github-webhook/`
  - **Content type**: `application/json`
  - **Events**: Select **Just the push event**
  - Click **Add webhook** / **Save**

#### Test the Integration

- Open the project locally
- Copy again the `echo` command (or any test command) into the appropriate file
- Commit and push the changes to the repository
- Verify in Jenkins that the build is triggered automatically

### Configure for Multibranch Pipeline

1. Go to **Manage Jenkins** → **Plugins** and search for `Multibranch Scan Webhook Trigger`. Install the plugin.

2. Go to Jenkins **Dashboard** and open your Multibranch Pipeline job.

3. Click **Configure**.

4. In the **Scan Multibranch Pipeline Triggers** section, check **Scan by webhook**.

5. For **Trigger token**, enter: `githubtoken`

6. Click **Save**.

7. Go back to GitHub repository:

   - Navigate to **Settings** → **Webhooks**

   - Edit the existing webhook URL to:  
     `http://Droplet_id:8080/multibranch-webhook-trigger/invoke?token=githubtoken`

   - Save the webhook.

#### Test the Integration

- Open the project locally
- Remove the `echo` command (or any test command) into the appropriate file
- Commit and push the changes to the repository
- Verify in Jenkins that the build is triggered automatically

