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






````markdown
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

```bash
pwd
```

**Use:** Display the current working directory.

```
```
4.Learned how SSH key pairs are used for secure authentication.

5.Learned how to verify a successful SSH connection using basic Linux commands.
