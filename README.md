# 🚀 Auto Scaling Group Setup with PM2, Bun, and Node.js on AWS

This guide helps developers set up an **Auto Scaling Group (ASG)** in AWS using EC2 instances configured to run a Bun-based project with **PM2** for process management. This supports both CI/CD pipelines and manual launch scenarios.

---

## 🛠 Prerequisites

- AWS account with permission to create EC2, Launch Templates, Load Balancers, and Auto Scaling Groups
- Bun project deployed to GitHub or available for `git pull`
- Node.js (v22.14.0) and Bun installed or installable via script
- AMI based on Amazon Linux or Ubuntu (depending on your path setup)

---

## 🧩 Folder Structure Example

```bash
ASG/
├── bin.ts
├── package.json / bun.lockb
├── .git/
```

---

## 🏗️ Step-by-Step Setup

### 1. ✅ Create and Prepare EC2 Instance

Install dependencies and verify your app is working. Then:

1. SSH into your EC2 instance
2. Clone your repo and setup Bun:
   ```bash
   git clone https://github.com/your-username/your-repo.git ASG
   cd ASG
   bun install
   bun run bin.ts
   ```

---

### 2. 📸 Create AMI Image

After setup:
- Go to **EC2 Dashboard → Instances**
- Select your configured EC2 instance
- Click **Actions → Image and templates → Create image**
- Name it (e.g., `asg-bun-app`) and create the image

---

### 3. 📄 Create Launch Template

While creating a **Launch Template**:
- Choose the **AMI** created in Step 2
- Go to **Advanced → User Data** and paste one of the scripts below

---

## 📜 User Data Scripts

### ➤ Option 1: CI/CD with Git Pull

```bash
#!/bin/bash
cd /home/ec2-user/ASG || exit
export PATH=$PATH:/home/ec2-user/.nvm/versions/node/v22.14.0/bin/
npm install -g pm2
git pull
bun install
pm2 start --interpreter /home/ec2-user/.nvm/versions/node/v22.14.0/bin/bun bin.ts
```

### ➤ Option 2: Without CI/CD (AMI has full app code)

```bash
#!/bin/bash
cd /home/ec2-user/ASG || exit
export PATH=$PATH:/home/ec2-user/.nvm/versions/node/v22.14.0/bin/
npm install -g pm2
pm2 start --interpreter /home/ec2-user/.nvm/versions/node/v22.14.0/bin/bun /home/ec2-user/ASG/bin.ts
```

---

## 📡 Attach Load Balancer (REQUIRED)

- Go to **EC2 → Load Balancers**
- Create an **Application Load Balancer**
- Use appropriate listener (e.g., HTTP on port 80)
- Link to a **Target Group**
- Register your EC2 instances with the target group

---

## 🔁 Create Auto Scaling Group (ASG)

1. Go to **EC2 → Auto Scaling Groups**
2. Click **Create Auto Scaling Group**
3. Link your **Launch Template**
4. Attach your **Load Balancer Target Group**
5. Choose **Network** and **Subnets**
6. Configure **Dynamic Scaling Policies**

---

### 🔄 Dynamic Auto Scaling Policies

1. After ASG is created, go to its settings
2. Select **Automatic Scaling**
3. Add policy to scale out when CPU > 70%
4. Add policy to scale in when CPU < 30%
