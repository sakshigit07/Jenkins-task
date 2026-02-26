# Readme.md

[Jenkins_task.pdf](attachment:1dd9a150-76b9-41d2-afb8-896c1cfe0f30:Jenkins_task.pdf)

# Jenkins task

## MCQs Questions

---

1.  **What is Jenkins mainly used for?**
    
    B) Continuous Integration and Continuous Delivery
    
2. **Which type of job allows you to define build steps using code
in Jenkins?**
    
    B) Pipeline Project
    
3. **Which file is used to define a pipeline in Jenkins?**
    
    C) Jenkinsfile
    
4. **What is the purpose of a Jenkins Agent (Node)?**
    
    B) To execute jobs assigned by the Jenkins controller
    
5. **Which plugin is required to connect Jenkins with GitHub?**
    
    B) Git Plugin
    
6. **What is the purpose of a Webhook in Jenkins CI/CD?**
    
    B) To trigger build automatically on code push
    
7. **Which command is used inside Jenkins Pipeline to execute
shell commands?**
    
    C) sh
    
8. **What is the purpose of `post` block in Jenkins Pipeline?**
    
    B) Execute steps after pipeline stages
    

1. W**hat is the use of `sshagent` in Jenkins Pipeline?**
    
    C) Use stored SSH credentials during execution
    

1. **What happens if a stage fails in Jenkins Pipeline (by default)?**
    
    B) The pipeline stops execution
    
    ---
    

# Scenario Based questions

You are hired as a **Junior DevOps Engineer** in a startup fitness company called
**FitLife Gym**.
The development team has created a static HTML website for the gym.
Your manager has assigned you the following tasks:
1. Set up CI/CD infrastructure.
2. Host the website on a target server.
3. Ensure automatic deployment when code is pushed to GitHub.
4. Troubleshoot and fix pipeline issues if deployment fails.
You are responsible for complete automation.

---

## Task 1: Infrastructure Setup

### Create two Linux servers:

1. **Jenkins Server**
    - Install Java
    - Install Jenkins
    - Configure required plugins
    - Configure SSH credentials
2. **Target Server**
    - Install nginx
    - Ensure port 80 is open
    - Website should be accessible via browser
    
    ![Screenshot (276).png](attachment:11336853-479b-43ec-8455-45d1cada3243:Screenshot_(276).png)
    
    ![Screenshot (282).png](attachment:c068e9b3-6834-47b4-ac62-7c3e987ecf7f:c40bf4dc-8f89-4aa4-98a0-f312b2281cc0.png)
    
    ---
    
    ### Task 2: Create GitHub Repository
    
    1. Create a new public GitHub repository:
    

```bash
gym-static-website
```

![Screenshot (277).png](attachment:f5d753c9-9b9a-4635-9737-0249ec5fcf6b:Screenshot_(277).png)

1. Add the following HTML code:
    
     **Simple Gym HTML Website Code**
    

```html
<!DOCTYPE html>
<html>
<head>
    <title>FitLife Gym</title>
    <style>
        body {
            font-family: Arial;
            text-align: center;
            background-color: #111;
            color: white;
            }
        .container {
                margin-top: 100px;
            }
        h1 {
                color: #00ff99;
            }
        button {    
                padding: 10px 20px;
                background-color: #00ff99;
                border: none;
                cursor: pointer;
                }
    </style>
</head>
<body>
    <div class="container">
        <h1>Hello from FitLife Gym</h1>
        <p>Transform Your Body. Transform Your Life.</p>
        <button>Join Now</button>
    </div>
</body>
</html>
```

Push this code to GitHub.

```bash
git add . 
git commit -m "added files"
git remote add origin https://github.com/sakshigit07/gym-static-website.git
git push -u origin main
```

![Screenshot (278).png](attachment:23e6cf97-3b3c-480d-bf68-45e940a67d7d:Screenshot_(278).png)

---

### Task 3: Make Required Change

Modify the website:

Change:

```bash
Hello from FitLife Gym
```

To

```bash
Hi from FitLife Gym
```

Push the changes to GitHub.

![Screenshot (279).png](attachment:ec93007b-1a69-461b-b7a3-3bffefc39040:Screenshot_(279).png)

---

### Task 4: Configure Webhook

- Configure GitHub webhook
- Trigger Jenkins job automatically on push

![Screenshot (280).png](attachment:b22d662d-7b4c-428d-b780-aa5e23bc096e:Screenshot_(280).png)

---

### Task 5: Jenkins Pipeline

Create a Pipeline Job and add the following Jenkinsfile:

### Jenkinsfile

```bash
pipeline {
    agent any

    environment {
        SERVER_IP      = '172.31.31.203'
        SSH_CREDENTIAL = 'gym-key'
        REPO_URL       = 'https://github.com/sakshigit07/gym-static-website.git'
        BRANCH         = 'main'
        REMOTE_USER    = 'ubuntu'
        REMOTE_PATH    = '/var/www/html'
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: "${BRANCH}", url: "${REPO_URL}"
            }
        }

        stage('Deploy to Target Server') {
            steps {
                sshagent(credentials: ["${SSH_CREDENTIAL}"]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${SERVER_IP} 'sudo mkdir -p ${REMOTE_PATH}'
                        scp -o StrictHostKeyChecking=no -r * ${REMOTE_USER}@${SERVER_IP}:${REMOTE_PATH}
                        """
                }
            }
        }

        stage('Restart Nginx') {
            steps {
                sshagent(credentials: ["${SSH_CREDENTIAL}"]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${SERVER_IP} '
                        sudo systemctl restart nginx
                        '
                        """
                }
            }
        }
    }
    post {
        success {
            echo 'Deployment Successful'
        }
        failure {
            echo 'Deployment Failed'
        }
    }
}
```

![Screenshot (281).png](attachment:ff6aafff-e8ff-4f58-af6a-cd7440073df8:Screenshot_(281).png)

## Deployment Issue: Permission Denied Error

While deploying the project using Jenkins, the following error occurred

```bash
+ ssh -o StrictHostKeyChecking=no ubuntu@172.31.31.203 mkdir -p /var/www/html 
+ scp -o StrictHostKeyChecking=no -r index.html ubuntu@172.31.31.203:/var/www/html 
scp: dest open "/var/www/html/index.html": Permission denied scp: failed to upload file index.html to /var/www/html
```

### Root Cause

The directory `/var/www/html` is owned by the `root` user by default.

The `ubuntu` user (used for SSH deployment) did not have write permission to this directory.

Since `scp` does not support `sudo` on the remote destination, the file upload failed.

### ✅ Solution

Changed ownership of the web root directory to the `ubuntu` user:

```bash
sudo chown -R ubuntu:ubuntu /var/www/html
```
