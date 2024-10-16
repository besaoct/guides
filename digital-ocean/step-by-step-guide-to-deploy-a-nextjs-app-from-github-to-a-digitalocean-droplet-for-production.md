# Step-by-Step Guide to Deploy a Next.js App from GitHub to a DigitalOcean Droplet for Production

This beginner-friendly guide will walk you through deploying a Next.js app from GitHub to a DigitalOcean Droplet, setting up a reverse proxy with Nginx, using a process manager (PM2), configuring environment variables, connecting a GoDaddy domain, and setting up SSL for a production environment.

---

### **Prerequisites:**
- A **DigitalOcean** account.
- A **GitHub** repository with your Next.js project.
- A **DigitalOcean Droplet** (Ubuntu 22.04 is recommended).
- A domain name (optional but recommended, e.g., from GoDaddy).

---

### **Step 1: Create a Droplet on DigitalOcean**

1. **Log in to DigitalOcean** and go to the Droplets page.
2. Click **Create Droplet**.
3. Choose an **Ubuntu 22.04** image (or any similar version).
4. Select a Droplet size (for small apps, at least 1GB of RAM is recommended).
5. Choose a data center region close to your users.
6. Under **Authentication**, use an **SSH key** (recommended) or a **password**.
7. Click **Create Droplet**.

---

### **Step 2: Set Up Your Droplet**

1. **SSH into your Droplet** using your terminal:
   ```bash
   ssh root@your_droplet_ip
   ```
   Replace `your_droplet_ip` with the IP address of your droplet.

2. **Update your system** to ensure everything is up to date:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

3. **Install Node.js and npm:**
   Run the following commands to install Node.js and npm:
   ```bash
   curl -fsSL https://deb.nodesource.com/setup_20.x | sudo bash -
   sudo apt-get install -y nodejs
   ```
   Verify the installations:
   ```bash
   node -v
   npm -v
   ```

4. **Install Git** (if it's not already installed):
   ```bash
   sudo apt install git -y
   ```

---

### **Step 3: Clone Your Next.js App from GitHub**

1. Navigate to the directory where you want to store the app (e.g., `/var/www`):
   ```bash
   cd /var/www
   ```

2. **Clone your repository** from GitHub (replace the URL with your repo):
   ```bash
   git clone https://github.com/yourusername/your-nextjs-app.git
   ```

#### **Cloning a Private Repository via SSH**

If your GitHub repository is private, follow these steps:

1. **Set up SSH keys** (if you haven't done this already):
   Check for existing SSH keys:
   ```bash
   ls -al ~/.ssh
   ```
   If you don’t have one, generate an SSH key:
   ```bash
   ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
   ```
   Start the SSH agent:
   ```bash
   eval "$(ssh-agent -s)"
   ```
   Add the SSH private key to the SSH agent:
   ```bash
   ssh-add ~/.ssh/id_rsa
   ```

2. **Add the SSH key to GitHub**:
   Copy your public key:
   ```bash
   cat ~/.ssh/id_rsa.pub
   ```
   - Go to **GitHub > Settings > SSH and GPG Keys**, and add the SSH key.

3. **Clone the private repository** using SSH:
   ```bash
   git clone git@github.com:your-username/private-repo.git
   ```

---

### **Step 4: Install Dependencies and Build the App**

1. **Navigate into the project directory**:
   ```bash
   cd your-nextjs-app
   ```

2. **Install the project dependencies**:
   ```bash
   npm install
   ```

3. **Build the Next.js app** for production:
   ```bash
   npm run build
   ```

4. **Test the app locally** to ensure it works:
   ```bash
   npm start
   ```
   This will run the app on `localhost:3000`.

---

### **Step 5: Set Up Environment Variables**

If your project uses environment variables, you'll need to create a `.env.production` file to store them.

1. In your **project root** (the same directory as `package.json`), create a `.env.production` file:
   ```bash
   touch .env.production
   nano .env.production
   ```
2. Add your environment variables in the following format:
   ```bash
   NEXT_PUBLIC_API_URL=https://your-api-url.com
   NEXT_PUBLIC_API_KEY=your-api-key
   ```

   - Save the file and exit (press `CTRL+X`, then `Y` and `Enter`).

---

### **Step 6: Set Up a Process Manager with PM2**

PM2 will ensure your app stays running, restarts automatically if it crashes, and starts on system reboot.

1. **Install PM2 globally**:
   ```bash
   npm install -g pm2
   ```

2. **Start your app** using PM2:
   ```bash
   pm2 start npm --name "your-app-name" -- start
   ```

3. **Set PM2 to start on system boot**:
   ```bash
   pm2 startup
   pm2 save
   ```

4. **Basic PM2 Commands** for managing your app:
   - **Stop the app**:
     ```bash
     pm2 stop your-app-name
     ```
   - **Restart the app**:
     ```bash
     pm2 restart your-app-name
     ```
   - **Delete the app from PM2** (removes it from the PM2 list):
     ```bash
     pm2 delete your-app-name
     ```
   - **View app logs**:
     ```bash
     pm2 logs your-app-name
     ```

---

### **Step 7: Set Up a Reverse Proxy with Nginx**

To expose your app on the web, we’ll configure Nginx as a reverse proxy to forward requests to your Node.js server.

1. **Install Nginx**:
   ```bash
   sudo apt install nginx
   ```

2. **Configure Nginx for your Next.js app**:
   Open the default Nginx configuration file:
   ```bash
   sudo nano /etc/nginx/sites-available/default
   ```

   Replace the contents with the following:
   ```nginx
   server {
       listen 80;
       server_name yourdomain.com; # Replace with your domain or Droplet's IP

       location / {
           proxy_pass http://localhost:3000;
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection 'upgrade';
           proxy_set_header Host $host;
           proxy_cache_bypass $http_upgrade;
       }
   }
   ```

3. **Test the Nginx configuration** to ensure there are no errors:
   ```bash
   sudo nginx -t
   ```

4. **Restart Nginx** to apply the changes:
   ```bash
   sudo systemctl restart nginx
   ```

---

### **Step 8: Connect a GoDaddy Domain to DigitalOcean Droplet**

1. **Log in to your GoDaddy account** and go to the domain settings.
2. Under **DNS Management**, find the **A Record**.
3. **Update the A Record** to point to your DigitalOcean Droplet’s IP address.
4. Optionally, **create a CNAME record** to point `www` to `yourdomain.com`.
5. Save the changes. It may take up to 24 hours for DNS changes to propagate.

---

### **Step 9: Set Up SSL with Let’s Encrypt**

1. **Install Certbot** for Nginx:
   ```bash
   sudo apt install certbot python3-certbot-nginx
   ```

2. **Obtain an SSL certificate**:
   ```bash
   sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
   ```

3. Certbot will automatically handle the SSL certificate installation and update your Nginx configuration.

---

### **Step 10: Monitor and Manage Your App**

1. **Check the status of your app**:
   ```bash
   pm2 list
   ```

2. **View real-time logs** to monitor your app’s behavior:
   ```bash
   pm2 logs your-app-name
   ```

3. **Handle exceptions**:
   PM2 automatically restarts the app if it crashes. If your app crashes frequently, check the logs using `pm2 logs` and identify the root cause, such as unhandled exceptions in your code.

---

### **Tips for Production Stability:**

- **Error Handling**: Ensure your Next.js app has proper error handling (try-catch blocks, error pages, etc.).
- **Environment Variables**: Use environment-specific variables for production, such as database URLs, API keys, etc.
- **Scaling**: For large apps, consider DigitalOcean's load balancers or scaling services.

---

This guide provides all the essential steps to deploy your Next.js app on a DigitalOcean Droplet, set up Nginx, and connect a domain with SSL, making your application production-ready!
