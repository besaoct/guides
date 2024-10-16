# DigitalOcean Deployment Workflow with GitHub Actions

### Overview:
This YAML file is a **GitHub Actions** workflow configuration designed to automate the deployment of a web application named **Project** to a **DigitalOcean** server. It defines a set of instructions (jobs and steps) that are triggered when specific events occur, such as pushing code to the `main` branch or manually triggering the workflow. This workflow facilitates continuous deployment (CD) of the application to a remote server, ensuring seamless and automated updates.

### Language:
The file is written in **YAML** (Yet Another Markup Language), a human-readable data serialization format commonly used for defining configurations and workflows in GitHub Actions.

### Where It's Used:
This file is utilized within **GitHub Actions**, a feature provided by GitHub for automating software development workflows like testing, building, and deploying code. Specifically, it resides inside a repository in the `.github/workflows/` directory. The file outlines the procedure for deploying code from the GitHub repository to a server hosted on **DigitalOcean**.

### YAML Configuration:

```yaml
name: DigitalOcean 

on:
  push:
    branches: 
      - main  # Trigger on push to the main branch
  workflow_dispatch:  # Allow manual triggering

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Install sshpass
      run: sudo apt-get install sshpass

    - name: Deploy to DigitalOcean
      run: |
        sshpass -p "${{ secrets.PROJECT_DO_PASSWORD }}" ssh -o StrictHostKeyChecking=no root@${{ secrets.PROJECT_DO_IP }} << 'EOF'
          # Lock file to prevent concurrent deployments
          LOCK_FILE="/var/www/tmp/deployment.lock"
          APP_DIR="/var/www/project"
          NEW_BUILD_DIR="/var/www/projectNew"
          BACKUP_DIR="/var/www/backup/app_backup"
          
          # Check if a deployment is already in progress
          if [ -f "$LOCK_FILE" ]; then
            echo "A deployment is already in progress. Terminating the ongoing deployment."
            rm -rf $NEW_BUILD_DIR  # Clean up any existing build
            pm2 stop all           # Stop the running process (optional, based on preference)
          fi
          
          # Create the lock file
          mkdir -p /var/www/tmp
          touch "$LOCK_FILE"
          
          echo "Cleaning up any previous deployment artifacts..."
          rm -rf $NEW_BUILD_DIR    # Ensure the new build directory is clean
          
          echo "Resetting local changes and cleaning up untracked files..."
          if [ -d "$APP_DIR/.git" ]; then
            git -C $APP_DIR reset --hard origin/main  # Reset local changes to match the main branch
            git -C $APP_DIR clean -fd                # Clean untracked files
          else
            echo "No git repository found. Exiting."
            rm "$LOCK_FILE"
            exit 1
          fi
          
          echo "Pulling the latest code from the main branch..."
          git -C $APP_DIR fetch origin || { echo "Git fetch failed. Exiting."; rm "$LOCK_FILE"; exit 1; }
          git -C $APP_DIR checkout main || { echo "Git checkout failed. Exiting."; rm "$LOCK_FILE"; exit 1; }
          git -C $APP_DIR pull origin main || { echo "Git pull failed. Exiting."; rm "$LOCK_FILE"; exit 1; }
          
          echo "Creating a new build directory..."
          cp -R $APP_DIR $NEW_BUILD_DIR  # Copy current app to a new directory to prepare the build
          
          cd $NEW_BUILD_DIR
          echo "Installing dependencies..."
          npm install  # Install any new dependencies
          
          echo "Building the new Next.js app..."
          if NODE_ENV=production npm run build; then  # Build the new version in production mode
            if [ -d "$NEW_BUILD_DIR/.next" ]; then
              echo "Build succeeded and .next folder found. Preparing to switch..."
              
              # Ensure the backup directory exists
              mkdir -p /var/www/backup
              
              # Backup old version and switch directories
              mv $APP_DIR $BACKUP_DIR  # Backup old build
              mv $NEW_BUILD_DIR $APP_DIR
              
              echo "Restarting the application with PM2..."
              cd $APP_DIR  # Change to the application directory
              pm2 restart project || pm2 start npm --name "project" -- start  # Restart or start the new version
              
              echo "New version deployed successfully."
              rm -rf $BACKUP_DIR  # Clean up backup after successful deployment
            else
              echo "Build succeeded but .next folder not found. Reverting to previous build."
              rm -rf $NEW_BUILD_DIR  # Clean up the failed build
              mv $BACKUP_DIR $APP_DIR  # Restore the previous version
            fi
          else
            echo "Build failed. Keeping the old version running."
            rm -rf $NEW_BUILD_DIR  # Clean up the failed build
          fi
          
          # Remove the lock file to allow future deployments
          rm "$LOCK_FILE"
        EOF
```

### How It Works:
1. **Events that Trigger the Workflow:**
   - **Push to `main` Branch:** The workflow is automatically triggered whenever code is pushed to the `main` branch.
   - **Manual Trigger:** Users can manually initiate the workflow using the **`workflow_dispatch`** event.

2. **Jobs and Steps:**
   The workflow defines a single job named `deploy`, which runs on the **`ubuntu-latest`** virtual machine provided by GitHub. The job comprises several steps:

   - **Checkout Code:**
     Utilizes `actions/checkout@v2` to pull the latest version of the code from the GitHub repository, ensuring that the most recent code is available for deployment.

   - **Install `sshpass`:**
     Installs the `sshpass` utility on the runner machine. `sshpass` facilitates password-based SSH connections, enabling secure access to the DigitalOcean server.

   - **Deploy to DigitalOcean:**
     Establishes an SSH connection to the DigitalOcean server using the `sshpass` tool. The SSH credentials (IP address and password) are securely stored in GitHub Secrets (`PROJECT_DO_IP` and `PROJECT_DO_PASSWORD`). Once connected, a series of shell commands execute to deploy the updated application.

### Detailed Breakdown of the Deployment Process:
Upon establishing an SSH connection to the server, the deployment process follows these steps:

1. **Locking the Deployment:**
   - **Lock File Creation:** A lock file (`/var/www/tmp/deployment.lock`) is created to prevent multiple deployments from occurring simultaneously.
   - **Check for Ongoing Deployments:** If a lock file already exists, indicating an ongoing deployment, the script terminates the current deployment, cleans up any existing build artifacts in the new build directory, and optionally stops all running PM2 processes to avoid conflicts.

2. **Resetting the Code:**
   - **Verify Git Repository:** The script checks if the application directory (`/var/www/project`) contains a Git repository.
   - **Reset Local Changes:** If a Git repository is present, it resets any local changes to match the `main` branch and cleans up untracked files.
   - **Error Handling:** If no Git repository is found, the script exits with an error after removing the lock file.

3. **Pulling the Latest Code:**
   - **Fetch Updates:** Executes `git fetch origin` to retrieve the latest updates from the remote repository.
   - **Checkout `main` Branch:** Switches to the `main` branch using `git checkout main`.
   - **Pull Latest Changes:** Updates the local repository with the latest commits from the `main` branch using `git pull origin main`.
   - **Error Handling:** If any Git command fails, the script exits early after removing the lock file.

4. **Preparing the New Build:**
   - **Create New Build Directory:** Copies the current application directory to a new directory (`/var/www/projectNew`) to prepare for building the updated version.
   - **Install Dependencies:** Navigates to the new build directory and runs `npm install` to install any new or updated dependencies.

5. **Building the Application:**
   - **Production Build:** Executes `npm run build` in production mode (`NODE_ENV=production`) to build the Next.js application. This step generates a `.next` folder containing the build artifacts.
   - **Verify Build Success:** Checks for the existence of the `.next` folder to confirm a successful build.

6. **Switching to the New Build:**
   - **Backup Current Build:** Moves the existing application directory to a backup location (`/var/www/backup/app_backup`).
   - **Deploy New Build:** Moves the new build directory (`/var/www/projectNew`) to replace the old application directory (`/var/www/project`).
   - **Restart Application with PM2:** Restarts the application using **PM2**, a process manager for Node.js applications. If the PM2 process named `project` is already running, it restarts it; otherwise, it starts a new PM2 process named `project`.
   - **Cleanup:** Removes the backup directory after a successful deployment.

7. **Handling Build Failures:**
   - **Revert to Previous Build:** If the build fails or the `.next` folder is not found, the script cleans up the failed build directory and restores the previous version of the application from the backup.
   - **Error Logging:** Outputs relevant error messages to aid in troubleshooting.

8. **Final Cleanup:**
   - **Remove Lock File:** Deletes the lock file (`/var/www/tmp/deployment.lock`) to allow future deployments.

### Key Elements:
- **SSH Credentials:** The SSH password and server IP address are securely stored using GitHub Secrets (`PROJECT_DO_PASSWORD` and `PROJECT_DO_IP`), ensuring they are not exposed in the codebase.
- **PM2 Process Manager:** PM2 manages the running Node.js application, ensuring it is properly restarted or started after deployment.
- **Git Operations:** The workflow leverages Git commands (`git reset`, `git fetch`, `git checkout`, `git pull`) to synchronize the server's codebase with the latest commits from the `main` branch.
- **Backup Mechanism:** Implements a backup system to restore the previous version of the application in case of deployment failures, enhancing reliability and reducing downtime.

### Workflow Features:
- **Locking Mechanism:** Prevents overlapping deployments by using a lock file, thereby avoiding potential conflicts or corruption during simultaneous deployment attempts.
- **Error Handling:** Incorporates robust error handling at each critical step, ensuring that the deployment process can gracefully handle failures without leaving the system in an inconsistent state.
- **Secure Secrets Management:** Utilizes GitHub Secrets to manage sensitive information like SSH credentials, enhancing security by preventing unauthorized access.
- **Automated Deployment:** Facilitates continuous deployment by automatically deploying the latest code changes to the server upon pushes to the `main` branch or manual triggers.

### Advantages:
- **Continuous Deployment:** Automates the deployment pipeline, ensuring that the latest code changes are promptly and reliably deployed to the production environment.
- **Reliability:** The use of lock files, backup mechanisms, and comprehensive error handling enhances the reliability of the deployment process.
- **Scalability:** Can be adapted to deploy multiple projects by adjusting configurations and secrets accordingly.
- **Maintainability:** Written in YAML, the configuration is easy to read, understand, and modify, making maintenance straightforward.

### Conclusion:
This GitHub Actions workflow provides an efficient and automated solution for deploying the **Project** application to a DigitalOcean server. It manages the entire deployment lifecycle, from checking out the latest code to installing dependencies, building the application, and deploying it while maintaining a backup system for recovery in case of failures. This setup is ideal for projects that require frequent deployments and a streamlined, automated delivery pipeline, ensuring that updates are deployed consistently and reliably.
