# 🐧 Deploying a Simple Website on CentOS Using Vagrant and HTTPD

Ever wanted to spin up a website on your Linux box faster than you can say _“systemctl restart httpd”_?  
In this tutorial, you’ll learn how to set up a simple web server on **CentOS** using **Vagrant**, deploy a ready-made HTML template, and get it running in your browser like a pro.

---

## ✅ Quick Checklist

Before diving in, here’s your roadmap:

1. **Set up a CentOS VM using Vagrant**
    
2. **Install required packages** (`httpd`, `wget`, `vim`, `unzip`, etc.)
    
3. **Start and enable the HTTPD service**
    
4. **Deploy a simple HTML page manually**
    
5. **Download and deploy a professional HTML template**
    
6. **Validate and troubleshoot using `systemctl` and firewall checks**
    
7. **Destroy the VM when done**
    

---

## 🧱 Step 1: Create a CentOS Vagrant VM

1. Check if you have existing VMs running:
    
    `vagrant global-status`
    
    If any are active, remove them to avoid IP conflicts:
    
    `vagrant destroy`
    
2. Create a project folder (for example, on your `F:` drive):
    
    `mkdir F:\vagrant-vms\finance cd F:\vagrant-vms\finance`
    
3. Initialize a CentOS box (e.g., `boxen/centos-stream-9`):
    
    `vagrant init eurolinux/centos-stream-9`
    
4. Open the generated `Vagrantfile` with your favorite editor (Notepad, Vim, VS Code...whatever gets you typing).  
    Add or modify:
    
    `config.vm.network "private_network", ip: "192.168.56.127"`
    `config.vm.network "public_network"`
    `config.vm.provider "virtualbox" do |vb`
    `vb.memory = "1024"
     end`

    Save an quit. 
    ⚠️ Make sure this IP doesn’t collide with other machines on your network.
    
6. Start your VM:
    
    `vagrant up`
    
7. Log in and switch to root:
    
    `vagrant ssh`
    `sudo -i`
    

---

## 🖥️ Step 2: (Optional) Change Hostname

This step is purely cosmetic, but hey, who doesn’t like a personalized hostname?

`echo "finance" > /etc/hostname hostname`
`hostname finance`

Then logout and login again.

---

## 📦 Step 3: Install HTTPD and Dependencies

Update and install all the packages you’ll need for this setup:
`sudo yum update && upgrade -y`
`yum install -y httpd wget vim unzip zip`

**Why these packages?**

- `httpd`: The web server
    
- `wget`: To download our website template
    
- `vim`: For quick file editing
    
- `unzip`: To unpack our template
    
- `zip`: Optional, but nice to have
    

Start and enable the service:

`systemctl start httpd systemctl enable httpd`

---

## 🌐 Step 4: Verify the Service

Check your VM’s IP:

`ip addr show`

Now, open your browser and visit:

`http://<your_vm_ip>`

You should see the default Apache test page — a good sign that HTTPD is running fine!

---

## ✏️ Step 5: Create a Basic HTML Page

Let’s test by creating a simple HTML file.

`cd /var/www/html vim index.html`

Add this inside:

`<h1>This is my first website setup!</h1> <p>Yes, it’s basic. No, it’s not ugly—it’s *minimalist* 😎</p>`

Restart the service to load your changes:

`systemctl restart httpd`

Now refresh your browser — your _masterpiece_ should appear.

---

## 🧩 Step 6: Download and Deploy a Template

1. Visit [https://www.tooplate.com](https://www.tooplate.com) using **Brave Browser** (recommended to avoid annoying popups).  
    Pick any template you like — for this tutorial, we’ll use **“Mini Finance”**.
    
2. To find the real download link:
    
    - Open DevTools with `F12`
        
    - Go to the **Network** tab
        
    - Click **Download**
        
    - Look for a `.zip` file and copy its full download URL from **Headers**
        
3. Back in your VM, navigate to a temporary directory:
    
    `cd /tmp`
    
4. Use `wget` to download the zip file:
    
    `wget <paste_download_link_here>`
    
5. Unzip it:
    
    `unzip <file_name>.zip cd <unzipped_folder>`
    
6. Copy the contents to your web root:
    
    `cp -r * /var/www/html/`
    
    If asked to overwrite `index.html`, say **yes**.
    
7. Restart the service:
    
    `systemctl restart httpd`
    
8. Refresh your browser — your shiny new template is live 🎉
    

---

## 🧰 Step 7: Validate and Troubleshoot

Check the status of your services:

`systemctl status httpd systemctl status firewalld`

- If `firewalld` is active and blocking access, you can temporarily disable it:
    
    `systemctl stop firewalld systemctl disable firewalld`
    

_(Note: this is fine for learning — not for production!)_

Confirm everything is running and accessible from your browser via your VM’s IP.

---

## 🧹 Step 8: Clean Up

Once you’re done experimenting, shut down and remove the VM:

`exit vagrant destroy`

Congratulations! You’ve successfully:

- Set up a CentOS VM
    
- Installed and configured an HTTP server
    
- Deployed a live HTML website
    

That’s no small feat — you’re officially the admin of your own tiny corner of the internet 🌍
