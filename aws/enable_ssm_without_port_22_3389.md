# AWS SSM Session Manager – Connection Flow

## 🔑 Prerequisites
1. **IAM Role on EC2 instance**  
   - Attach role with policy:  
     ```json
     arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
     ```

2. **SSM Agent**  
   - Preinstalled on most Amazon AMIs (Windows & Linux).  
   - Must be **running** on the instance.

---

## 🌐 Networking Flow
1. **SSM Agent on EC2 → talks to 3 AWS services** over HTTPS (TCP 443):
   - `ssm.<region>.amazonaws.com`
   - `ec2messages.<region>.amazonaws.com`
   - `ssmmessages.<region>.amazonaws.com`

2. **VPC Endpoints (Interface Endpoints)** are required if no Internet/NAT:  
   - Create 3 endpoints:  
     - `com.amazonaws.<region>.ssm`  
     - `com.amazonaws.<region>.ec2messages`  
     - `com.amazonaws.<region>.ssmmessages`  
   - Enable **Private DNS** so instances resolve endpoints to private IPs.  

3. **Security Groups**
   - Attach SG to the endpoints.  
   - Must allow **inbound TCP 443** from your instance SG.  
   - Instance SG must allow **outbound TCP 443**.

4. **NACLs**
   - Outbound → allow TCP 443.  
   - Inbound → allow **ephemeral ports (1024–65535)** for return traffic.  

5. **Windows Firewall / Linux iptables**
   - Must allow **outbound TCP 443**.

---

## 📡 Session Flow
1. User starts a session from **AWS Console/CLI**.  
2. Session Manager connects to the **SSM service**.  
3. SSM forwards the session request via **`ec2messages`**.  
4. Instance responds and establishes a secure channel using **`ssmmessages`**.  
5. Session runs entirely over **encrypted HTTPS (443)** inside the VPC.  

👉 No SSH (22) or RDP (3389) is required.

---

## ✅ Summary Formula
```text
IAM Role + SSM Agent + 3 VPC Endpoints
+ SG (443) + NACL (443 + ephemeral ports)
= Successful Session Manager Connection
