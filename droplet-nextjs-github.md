Here's a step-by-step guide to deploy a Next.js app from GitHub to a DigitalOcean Droplet for production:

### Prerequisites:
- You have a DigitalOcean account.
- A GitHub repository with your Next.js project.
- A DigitalOcean Droplet (preferably Ubuntu 22.04 or similar).
- A domain name (optional but recommended for production).

---

### Step 1: Create a Droplet on DigitalOcean
1. **Log in to DigitalOcean** and go to the [Droplets page](https://cloud.digitalocean.com/droplets).
2. Click **Create Droplet**.
3. Choose an **Ubuntu** image (22.04 is recommended).
4. Select the appropriate **Droplet size** (at least 1GB RAM for small apps, more for larger).
5. Choose a **datacenter region** close to your users.
6. Under **Authentication**, use an SSH key (recommended) or password.
7. Finally, click **Create Droplet**.

---

### Step 2: Set Up Your Droplet
1. **SSH into your Droplet** from your terminal:

   ```bash
   ssh root@your_droplet_ip
   ```

2. **Update your Droplet**:

   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

3. **Install Node.js and npm**:

```bash
 curl -fsSL https://deb.nodesource.com/setup_20.x | sudo bash -
 sudo apt-get install -y nodejs
```

   Verify the installation:

   ```bash
   node -v
   npm -v
   ```

4. **Install Git** (if not already installed):

   ```bash
   sudo apt install git -y
   ```

---

## Step 3: Clone Your Next.js App from GitHub

1. Navigate to the directory where you'd like to store the app:

   ```bash
   cd /var/www
   ```

2. Clone the GitHub repository:

   ```bash
   git clone https://github.com/yourusername/your-nextjs-app.git
   ```

   To clone a private GitHub repository using SSH, follow these steps:

### Step 1: Set Up an SSH Key (if you haven't already)
1. **Check for existing SSH keys**:

   ```bash
   ls -al ~/.ssh
   ```

   If you see `id_rsa` and `id_rsa.pub`, you already have an SSH key pair. Otherwise, you'll need to generate one.

2. **Generate a new SSH key** (if necessary):

   ```bash
   ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
   ```

   Press `Enter` to accept the default location (`/home/your_user/.ssh/id_rsa`), and optionally, provide a passphrase.

3. **Start the SSH agent**:

   ```bash
   eval "$(ssh-agent -s)"
   ```

4. **Add your SSH private key to the SSH agent**:

   ```bash
   ssh-add ~/.ssh/id_rsa
   ```

### Step 2: Add SSH Key to Your GitHub Account
1. **Copy your SSH public key**:

   ```bash
   cat ~/.ssh/id_rsa.pub
   ```

   Copy the output (the entire key) to your clipboard.

2. **Go to GitHub**:
   - Go to your GitHub account.
   - Navigate to **Settings** > **SSH and GPG keys**.
   - Click **New SSH key**, and paste your SSH public key there.
   - Give it a title (e.g., "DigitalOcean Droplet Key"), then click **Add SSH key**.

### Step 3: Test SSH Connection to GitHub
To ensure that your SSH key works, run:

```bash
ssh -T git@github.com
```

If everything is set up correctly, you should see a message like:

```
Hi username! You've successfully authenticated, but GitHub does not provide shell access.
```

## Step 4: Clone the Private Repository
Now that the SSH key is set up and added to GitHub, you can clone your private repository using the SSH URL.

1. **Clone the repo**:

   ```bash
   git clone git@github.com:your-username/private-repo.git
   ```

Replace `your-username` with your GitHub username and `private-repo` with the name of your private repository.

Your private GitHub repo should now be cloned successfully using SSH!

4. Change to the project directory:

   ```bash
   cd your-nextjs-app
   ```

---

## Step 4: Install Dependencies and Build the App
1. Install the dependencies:

   ```bash
   npm install
   ```

2. Build the Next.js app:

   ```bash
   npm run build
   ```

3. Test it locally by running:

   ```bash
   npm start
   ```

   Ensure everything is working before proceeding.

---

## Step 5: Set Up Environment Variables (Optional)
If your project uses environment variables, create a `.env.production` file in the root of your project, and add the necessary variables:

```bash
touch .env.production
nano .env.production
```

Add your variables in the following format:

```
NEXT_PUBLIC_API_URL=https://your-api-url.com
NEXT_PUBLIC_API_KEY=your-api-key
```

---

### Step 6: Set Up a Process Manager
Using a process manager like **PM2** ensures your app stays up and restarts automatically if it crashes.

1. Install PM2 globally:

   ```bash
   npm install -g pm2
   ```

2. Start the app using PM2:

   ```bash
   pm2 start npm --name "app-folder-name" -- start
   ```

3. Set PM2 to start on system boot:

   ```bash
   pm2 startup
   pm2 save
   ```

---

## Step 7: Set Up a Reverse Proxy with Nginx
For production, it's best to use **Nginx** to serve your Next.js app and handle requests.

1. Install Nginx:

   ```bash
   sudo apt install nginx
   ```

2. Configure Nginx for your app:

   ```bash
   sudo nano /etc/nginx/sites-available/default
   ```

   Replace the contents with the following configuration:

   ```nginx
   server {
       listen 80;
       server_name yourdomain.com; # Or your Droplet's IP

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

3. Test the Nginx configuration:

   ```bash
   sudo nginx -t
   ```

4. Restart Nginx:

   ```bash
   sudo systemctl restart nginx
   ```

---

### Step 8: (Optional) Set Up a Domain Name and SSL
1. If you have a domain name, point it to your DigitalOcean Droplet’s IP.
2. Install **Certbot** to get an SSL certificate:

   ```bash
   sudo apt install certbot python3-certbot-nginx
   ```

3. Run the following command to set up SSL:

   ```bash
   sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
   ```

   Certbot will handle the SSL certificate installation and Nginx configuration.

---

### Step 9: Monitor the App
To check the status of your app:

```bash
pm2 list
```

You can view logs with:

```bash
pm2 logs nextjs-app
```

---

### Your Next.js app should now be running in production on your DigitalOcean Droplet!
