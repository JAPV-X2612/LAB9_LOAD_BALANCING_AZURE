# ☁️ Scaling in Azure with Virtual Machines, Scale Sets and Service Plans

## 🎓 **Colombian School of Engineering - Julio Garavito**
### **Software Architectures - ARSW**

---

## 👥 **Team Members**

- [Jesús Alfonso Pinzón Vega](https://github.com/JAPV-X2612)
- [David Felipe Velásquez Contreras](https://github.com/DavidVCAI)

---

## 📚 **Project Overview**

This laboratory explores **cloud scalability strategies** using **Microsoft Azure** infrastructure. The project implements a **Node.js Fibonacci calculator** application to demonstrate the differences between **vertical scaling** (increasing VM resources) and **horizontal scaling** (distributing load across multiple VMs with a load balancer).

### 🎯 **Learning Objectives**

- Understand **vertical vs. horizontal scalability** patterns in cloud environments
- Configure and deploy **Azure Virtual Machines** with proper networking
- Implement **Azure Load Balancer** for traffic distribution
- Analyze **CPU consumption** and **response times** under concurrent load
- Compare **cost-effectiveness** of different scaling strategies
- Use **Newman/Postman** for load testing and performance benchmarking
- Apply **Infrastructure as Code** concepts for reproducible deployments

### 🔬 **Quality Scenario**

**Scalability Requirement:**
> *When a set of users concurrently query the nth Fibonacci number (greater than 1,000,000) and the system operates under normal conditions, all requests must be answered and the system's CPU consumption cannot exceed 70%.*

---

## 🏗️ **Architecture Overview**

This laboratory implements two distinct architectural approaches:

### 📊 **Part 1: Vertical Scaling Architecture**

```
┌─────────────────────────────────────────┐
│         Internet / Users                │
│      (Concurrent Requests)              │
└──────────────────┬──────────────────────┘
                   │ HTTP GET
                   │ /fibonacci/:nth
┌──────────────────▼──────────────────────┐
│      Azure Virtual Machine              │
│    ┌─────────────────────────────┐      │
│    │   Node.js Express Server    │      │
│    │   - FibonacciApp.js         │      │
│    │   - Port 3000               │      │
│    └──────────────┬──────────────┘      │
│                   │                     │
│    ┌──────────────▼──────────────┐      │
│    │   FibonacciService.js       │      │
│    │   - Iterative Algorithm     │      │
│    │   - BigInteger Operations   │      │
│    └─────────────────────────────┘      │
│                                         │
│  Size: B1ls → B2ms (Vertical Scaling)   │
│  CPU: 1 vCPU → 2 vCPUs                  │
│  RAM: 0.5 GB → 8 GB                     │
└─────────────────────────────────────────┘
```

### 🌐 **Part 2: Horizontal Scaling Architecture**

```
┌───────────────────────────────────────────────────┐
│              Internet / Users                     │
│           (Concurrent Requests)                   │
└───────────────────────┬───────────────────────────┘
                        │ HTTP
                        │
┌───────────────────────▼───────────────────────────┐
│          Azure Load Balancer                      │
│  ┌─────────────────────────────────────────┐      │
│  │  Frontend IP Configuration              │      │
│  │  - Public IP: 52.155.223.248            │      │
│  │  - Port: 80 → Backend 3000              │      │
│  └─────────────────────────────────────────┘      │
│  ┌─────────────────────────────────────────┐      │
│  │  Load Balancing Rule                    │      │
│  │  - Algorithm: Round Robin               │      │
│  │  - Health Probe: TCP 3000               │      │
│  └─────────────────────────────────────────┘      │
└───────────────────────┬───────────────────────────┘
                        │
        ┌───────────────┼───────────────┐
        │               │               │
┌───────▼──────┐ ┌──────▼──────┐ ┌──────▼──────┐
│   VM1        │ │   VM2       │ │   VM3       │
│ Zone 1       │ │ Zone 2      │ │ Zone 3      │
│              │ │             │ │             │
│ Node.js:3000 │ │ Node.js:3000│ │ Node.js:3000│
│ B1ls         │ │ B1ls        │ │ B1ls        │
└──────────────┘ └─────────────┘ └─────────────┘
        │               │               │
        └───────────────┼───────────────┘
                        │
            ┌───────────▼──────────┐
            │  Backend Pool        │
            │  - VM1, VM2, VM3     │
            │  - Health Monitoring │
            └──────────────────────┘
```

---

## 🛠️ **Technology Stack**

| Category | Technology | Version | Purpose |
|----------|-----------|---------|---------|
| **Cloud Provider** | *Microsoft Azure* | - | Infrastructure and VM hosting |
| **Operating System** | *Ubuntu Server* | 18.04 LTS | Linux-based VM image |
| **Runtime** | *Node.js* | 10.x+ | JavaScript runtime environment |
| **Web Framework** | *Express.js* | 4.17.1 | HTTP server and routing |
| **Math Library** | *big-integer* | 1.6.47 | Large number calculations |
| **Process Manager** | *Forever* | - | Keep Node.js app running |
| **Load Testing** | *Newman* | 7.x+ | Postman CLI for automation |
| **Version Control** | *Git* | - | Source code management |

---

## 📋 **Prerequisites**

Before starting this laboratory, ensure you have:

### ☁️ **Azure Account**
- ✅ Free Azure account with **$100 USD credit** for students
- ✅ Access to [Azure Portal](https://portal.azure.com/)
- ✅ Basic understanding of Azure Resource Groups

### 💻 **Local Environment**
- ✅ **Git** installed and configured
- ✅ **SSH key pair** generated (`ssh-keygen`)
- ✅ **Node.js** and **npm** installed locally
- ✅ **Newman** installed globally: `npm install -g newman`
- ✅ **Postman** (optional, for GUI testing)

### 📚 **Knowledge**
- ✅ Basic **Linux command line** usage
- ✅ Understanding of **HTTP/REST** concepts
- ✅ Familiarity with **SSH** connections
- ✅ Basic **networking** concepts (ports, IPs, firewalls)

---

## 🚀 **Part 1: Vertical Scaling**

### 📖 **Concept: Vertical Scaling (Scale Up)**

**Vertical scaling** increases the capacity of a single server by adding more resources (CPU, RAM, storage). It's the simplest approach but has physical limitations.

**Advantages:**
- ✅ Simple implementation
- ✅ No application changes required
- ✅ No data synchronization issues
- ✅ Straightforward monitoring

**Disadvantages:**
- ❌ Physical hardware limits
- ❌ Single point of failure
- ❌ Requires downtime for upgrades
- ❌ Cost increases exponentially

---

### 🔧 **Step 1: Create Virtual Machine**

#### 1.1 Basic Configuration

Navigate to [Azure Portal](https://portal.azure.com/) and create a new VM with these settings:

| Parameter | Value | Description |
|-----------|-------|-------------|
| **Resource Group** | `SCALABILITY_LAB` | Logical container for resources |
| **VM Name** | `VERTICAL-SCALABILITY` | Identifier for the virtual machine |
| **Region** | `East US` | Azure datacenter location |
| **Image** | `Ubuntu Server 18.04 LTS` | Operating system |
| **Size** | `Standard B1ls` | 1 vCPU, 0.5 GB RAM |
| **Authentication** | `SSH public key` | Secure connection method |
| **Username** | `scalability_lab` | Admin user account |
| **SSH Key** | Generate new or use existing | For remote access |

<img src="assets/images/part1/part1-vm-basic-config.png" alt="VM Basic Configuration" width="70%" height="auto">

**SSH Key Options:**
- **Option A:** Generate new key pair (Azure creates it)
    - Key pair name: `scalability-lab-key`
    - Azure downloads `.pem` file automatically

- **Option B:** Use existing public key
  ```bash
  # Generate locally
  ssh-keygen -t rsa -b 4096 -C "scalability_lab@azure"
  
  # Copy public key content
  cat ~/.ssh/id_rsa.pub
  ```

#### 1.2 Networking Configuration

Configure network settings:

| Parameter | Value | Notes |
|-----------|-------|-------|
| **Virtual Network** | (default) | Auto-created |
| **Subnet** | (default) | Auto-created |
| **Public IP** | Create new | Required for external access |
| **NIC Security Group** | Basic | Simplified firewall rules |
| **Public Inbound Ports** | Allow SSH (22) | Enable remote connection |
| **Load Balancing** | None | Not needed for Part 1 |

**Important:** Do NOT open port 3000 yet. This will be configured later via Inbound Security Rule.

#### 1.3 Other Sections

Leave default values for:
- **Disks:** Standard SSD
- **Management:** Monitoring enabled
- **Advanced:** No extensions
- **Tags:** (optional)

Click **"Review + create"** → **"Create"**

⏱️ **Deployment time:** ~3-5 minutes

---

### 🔌 **Step 2: Connect to VM**

#### 2.1 Obtain Public IP

After deployment completes:
1. Navigate to the VM resource
2. Click **"Connect"** → **"SSH"**
3. Copy the public IP address

#### 2.2 SSH Connection

```bash
# Using generated key (adjust path to your .pem file)
chmod 400 ~/Downloads/scalability-lab-key.pem
ssh -i ~/Downloads/scalability-lab-key.pem scalability_lab@<VM_PUBLIC_IP>

# OR using existing key
ssh scalability_lab@<VM_PUBLIC_IP>
```

**Expected Output:**
```
Welcome to Ubuntu 18.04.x LTS (GNU/Linux ...)
```

---

### ⚙️ **Step 3: Install Node.js**

#### 3.1 Install NVM (Node Version Manager)

```bash
# Download and install NVM
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.34.0/install.sh | bash

# Load NVM into current session
source ~/.bashrc

# Verify installation
nvm --version
```

#### 3.2 Install Node.js

```bash
# Install latest LTS version
nvm install node

# Verify installation
node --version  # Should show v10.x.x or higher
npm --version   # Should show 6.x.x or higher
```

---

### 📦 **Step 4: Deploy FibonacciApp**

#### 4.1 Clone Repository

```bash
# Clone your repository (replace with your actual repo URL)
git clone https://github.com/<your-username>/ARSW_LOAD-BALANCING_AZURE.git

# Navigate to application directory
cd ARSW_LOAD-BALANCING_AZURE/FibonacciApp
```

#### 4.2 Install Dependencies

```bash
# Install npm packages
npm install

# Verify dependencies
ls node_modules/  # Should see express, big-integer
```

**Expected `package.json` dependencies:**
```json
{
  "dependencies": {
    "big-integer": "^1.6.47",
    "express": "^4.17.1"
  }
}
```

---

### 🏃 **Step 5: Run Application**

#### 5.1 Install Forever (Process Manager)

```bash
# Install forever globally
npm install -g forever

# Verify installation
forever --version
```

#### 5.2 Start Application

```bash
# Start app with forever (keeps running after SSH disconnect)
forever start FibonacciApp.js

# Verify it's running
forever list

# Check logs
forever logs
```

**Expected Output:**
```
info:    Forever processing file: FibonacciApp.js
data:    uid  command         script           forever pid
data:    [0] node FibonacciApp.js 12345   12346
```

**Alternative (for testing):**
```bash
# Run directly (stops when SSH disconnects)
node FibonacciApp.js

# Output:
# Fibonacci App listening on port 3000!
```

---

### 🔥 **Step 6: Configure Firewall (Inbound Rule)**

#### 6.1 Create Inbound Port Rule

1. In Azure Portal, navigate to your VM
2. Go to **"Networking"** section
3. Click **"Add inbound port rule"**
4. Configure as follows:

| Parameter | Value | Description |
|-----------|-------|-------------|
| **Source** | Any | Allow from all IPs |
| **Source port ranges** | * | All source ports |
| **Destination** | Any | VM's IP |
| **Service** | Custom | Non-standard port |
| **Destination port ranges** | `3000` | Node.js app port |
| **Protocol** | TCP | HTTP protocol |
| **Action** | Allow | Permit traffic |
| **Priority** | 300 | Rule precedence |
| **Name** | `Port_3000` | Descriptive name |

<img src="assets/images/part1/part1-vm-3000InboudRule.png" alt="Inbound Rule for Port 3000" width="80%" height="auto">

#### 6.2 Verify Application Access

Open browser and test:

```
http://<VM_PUBLIC_IP>:3000/
```
**Expected:** `Hello World`

```
http://<VM_PUBLIC_IP>:3000/fibonacci/6
```
**Expected:** `The answer is 8`

✅ **Application is now publicly accessible!**

---

### 📊 **Step 7: Performance Testing - Response Times**

#### 7.1 Browser Console Testing

Use browser Developer Tools (F12) → Network tab to measure response times for different Fibonacci values.

Test with these values:
- 1000000
- 1010000
- 1020000
- 1030000
- 1040000
- 1050000
- 1060000
- 1070000
- 1080000
- 1090000

**Testing procedure:**
1. Open Developer Tools (F12)
2. Go to **Network** tab
3. Clear network log
4. Navigate to: `http://<VM_IP>:3000/fibonacci/<nth>`
5. Check **"Time"** column for total duration

<img src="assets/images/part1/part1-response-times-b1ls.png" alt="Response Times for B1ls VM Size" width="80%" height="auto">

#### 7.2 Response Time Table (B1ls Size)

| Fibonacci nth | Response Time (ms) | Notes |
|---------------|-------------------|-------|
| 1,000,000 | _____ | *Record your measurement* |
| 1,010,000 | _____ | *Record your measurement* |
| 1,020,000 | _____ | *Record your measurement* |
| 1,030,000 | _____ | *Record your measurement* |
| 1,040,000 | _____ | *Record your measurement* |
| 1,050,000 | _____ | *Record your measurement* |
| 1,060,000 | _____ | *Record your measurement* |
| 1,070,000 | _____ | *Record your measurement* |
| 1,080,000 | _____ | *Record your measurement* |
| 1,090,000 | _____ | *Record your measurement* |

---

### 📈 **Step 8: Monitor CPU Usage**

#### 8.1 Access VM Metrics

1. In Azure Portal, go to your VM
2. Click **"Metrics"** in left menu
3. Select **"Percentage CPU"** metric
4. Adjust time range to **"Last 30 minutes"**

⏱️ **Note:** Metrics may take 5-10 minutes to appear after requests.

<img src="assets/images/part1/part1-vm-cpu-b1ls.png" alt="CPU Usage for B1ls" width="80%" height="auto">

#### 8.2 CPU Analysis

**Expected behavior with B1ls:**
- CPU spikes to **90-100%** during Fibonacci calculations
- Sustained high CPU usage for large numbers
- Slow recovery between requests

**Why does it consume so much CPU?**

The `FibonacciService.js` uses an **iterative algorithm** which is computationally expensive:

```javascript
// Simplified version of the algorithm
for (var i = 0; i < nth - 1; i++) {
    answer = nth_2.add(nth_1)  // BigInteger addition
    nth_2 = nth_1
    nth_1 = answer
}
```

For `nth = 1,000,000`:
- **1 million iterations**
- **BigInteger operations** (slower than primitive types)
- **Memory allocations** for large numbers
- **No memoization** or optimization

---

### 🔄 **Step 9: Load Testing with Newman**

#### 9.1 Install Newman Locally

On your **local machine** (not the VM):

```bash
# Install Newman globally
npm install -g newman

# Verify installation
newman --version
```

#### 9.2 Configure Postman Environment

Navigate to `FibonacciApp/postman/part1/` directory.

Edit `[ARSW_LOAD-BALANCING_AZURE].postman_environment.json`:

```json
{
  "id": "373cbe10-3dd7-4a9a-8519-bb96bfcdf4b5",
  "name": "[ARSW_LOAD-BALANCING_AZURE]",
  "values": [
    {
      "key": "VM1",
      "value": "<YOUR_VM_PUBLIC_IP>",  // ⬅️ CHANGE THIS
      "enabled": true
    },
    {
      "key": "nth",
      "value": "1000000",
      "enabled": true
    }
  ]
}
```

#### 9.3 Run Load Test

Execute concurrent requests:

```bash
# Navigate to postman directory
cd FibonacciApp/postman/part1

# Run 2 parallel Newman instances with 10 iterations each
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json \
  -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json \
  -n 10 &

newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json \
  -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json \
  -n 10
```

**Total requests:** 20 (2 processes × 10 iterations)

<img src="assets/images/part1/part1-newman-results-b1ls.png" alt="Newman Load Test Results - B1ls" width="80%" height="auto">

---

