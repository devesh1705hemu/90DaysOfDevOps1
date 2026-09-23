# Part 1: Launch EC2 Instance & SSH Access

## Step 1: Launch an EC2 Instance

Launched an **Amazon EC2 instance** with Ubuntu Linux and configured the required:

- Instance type
- Key pair
- Security group
- Storage
- Network settings
- Public IP address

## Step 2: Connect to EC2 via SSH

Connected to the Ubuntu EC2 instance using SSH.
```
chmod 400 your-key.pem
```

```bash
ssh -i your-key.pem ubuntu@<your-instance-ip>
```
### Challenges Faced

1.Ensured the correct EC2 public IP address was used.

2.Verified the correct SSH username for the Ubuntu EC2 instance.

3.Used the correct .pem private key for authentication.

4.Ensured SSH access was allowed through the EC2 Security Group on port 22.

### What I Learned

1.Learned how to launch an AWS EC2 instance with Ubuntu Linux

2.Learned how to connect to an EC2 instance remotely using SSH.

3.Understood the role of EC2 Security Groups in allowing SSH access.

4.Learned how SSH key pairs are used for secure authentication.

5.Learned how to verify a successful SSH connection using basic Linux commands.

## Commands Used

### 1. chmod 400

```bash
chmod 400 your-key.pem
````

**Use:** Restricts the private key file permissions so that only the owner can read the key. This helps protect the SSH private key and is commonly required before using an AWS `.pem` key.

### 2. SSH

```bash
ssh -i your-key.pem ubuntu@<your-instance-ip>
```

**Use:** Connect securely to the Ubuntu EC2 instance using SSH and the private key.

### 3. whoami

```bash
whoami
```

**Use:** Check the username of the currently logged-in user.

### 4. hostname

```bash
hostname
```

**Use:** Display the hostname of the EC2 instance.

### 5. pwd

# Part 2: Install Docker & Nginx

## Step 1: Update System

### Update Package List

```bash
sudo apt update
````

**Use:** Refreshes the local package index and retrieves information about available package updates.

### Upgrade Installed Packages

```bash
sudo apt upgrade -y
```

**Use:** Installs the latest available versions of the currently installed packages.

---

## Step 2: Install Docker

```bash
sudo apt install docker.io -y
```

**Use:** Installs Docker Engine and the required Docker packages on the Ubuntu EC2 instance.

### Start Docker

```bash
sudo systemctl start docker
```

**Use:** Starts the Docker service.

### Enable Docker

```bash
sudo systemctl enable docker
```

**Use:** Configures Docker to start automatically when the server boots.

---

## Step 3: Install Nginx

```bash
sudo apt install nginx -y
```

**Use:** Installs the Nginx web server on the EC2 instance.

---

## Verify Nginx Is Running

```bash
sudo systemctl status nginx
```

**Use:** Checks the current status of the Nginx service and verifies whether it is running.

### Check Nginx Version

```bash
nginx -v
```

**Use:** Displays the installed Nginx version.

```
```bash
pwd
```

**Use:** Display the current working directory.

# Part 3: Security Group Configuration

## Security Group Configuration

Configured the AWS EC2 Security Group to allow HTTP traffic so that the Nginx web server could be accessed from the internet.

### Inbound Rule

| Type | Protocol | Port | Purpose |
|---|---|---:|---|
| SSH | TCP | 22 | Remote SSH access |
| HTTP | TCP | 80 | Nginx web access |

---

## Test Web Access

Opened the following URL in a browser:

```text
http://<your-instance-ip>
```
## What I Learned
1.Learned how AWS Security Groups control inbound traffic to an EC2 instance.

2.Understood why port 80 must be allowed for HTTP web access.

3.Learned how to verify a web server through its public IP address.

4.Verified that Nginx was accessible from the internet through the EC2 instance.

# Part 4: Extract Nginx Logs

## Step 1: View Nginx Logs

### View Access Logs

```bash
sudo cat /var/log/nginx/access.log
````

**Use:** Displays the Nginx access logs and shows requests received by the web server.

### View Error Logs

```bash
sudo cat /var/log/nginx/error.log
```

**Use:** Displays Nginx error logs to help identify server and configuration issues.

### View Recent Logs

```bash
sudo tail -n 50 /var/log/nginx/access.log
```

**Use:** Displays the last 50 entries from the Nginx access log.

---

## Step 2: Save Logs to File

```bash
sudo cp /var/log/nginx/access.log ~/nginx-logs.txt
```

**Use:** Copies the Nginx access log to the user's home directory and saves it as `nginx-logs.txt`.

### Verify the File

```bash
ls -lh ~/nginx-logs.txt
```

**Use:** Verifies that the log file was created and displays its file size.

---

## Step 3: Download Log File to Local Machine

Run the following command from a **new terminal on your local machine**:

### AWS

```bash
scp -i your-key.pem ubuntu@<your-instance-ip>:~/nginx-logs.txt .
```

**Use:** Securely copies `nginx-logs.txt` from the AWS EC2 instance to the current directory on the local machine.

### SCP Flow

```text
EC2 Instance
     |
     | SCP over SSH
     ↓
Local Machine
```

## What I Learned

* Learned where Nginx stores access and error logs.
* Learned how to view and inspect Nginx logs.
* Learned how to save server logs into a separate file.
* Learned how to securely transfer logs from an EC2 instance to a local machine using SCP.

```


