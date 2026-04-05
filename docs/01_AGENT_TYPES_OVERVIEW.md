# Zabbix Agent Types & Monitoring Architecture
## Training Guide - Understanding Agent Technologies

---

## 📚 **Overview**

This document explains the three main agent types used in our Zabbix monitoring infrastructure:
1. **Windows Agent** - For monitoring Windows systems
2. **Linux Agent** - For monitoring Linux systems  
3. **PCP Agent (Performance Co-Pilot)** - Our custom lightweight agent for Linux systems

Understanding these agents is essential for effectively deploying and managing our monitoring system.

---

## 🏗️ **Architecture Basics**

### **How Zabbix Agents Work**

```
┌─────────────────┐         ┌─────────────────┐         ┌─────────────────┐
│   Zabbix Server │◄────────│   Zabbix Agent  │────────►│   Monitored     │
│   (Port 8088)  │  JSON   │   (Active/      │  Metrics│   Host          │
│                 │  API   │    Passive)     │         │                 │
└─────────────────┘         └─────────────────┘         └─────────────────┘
```

**Key Concepts:**
- **Passive checks**: Server connects to agent, requests data
- **Active checks**: Agent connects to server, pushes data
- **Items**: Individual metrics being collected
- **Triggers**: Threshold-based alert conditions
- **Templates**: Reusable sets of items, triggers, and graphs

---

## 🪟 **Windows Agent**

### **What is the Windows Agent?**

The **Zabbix Windows Agent** is a native Windows service that collects performance metrics from Windows systems. It uses Windows-specific APIs and performance counters to gather data.

### **Key Features**

✅ **Native Windows Integration**
- Uses Windows Performance Counters
- Integrates with Windows Event Log
- Supports WMI queries
- Accesses Windows Registry

✅ **Installation Options**
- MSI installer package
- Portable executable
- Registered as Windows Service

✅ **Common Metrics Collected**
- CPU utilization
- Memory usage
- Disk space and I/O
- Network interfaces
- Windows Services status
- Event Log entries
- Performance counters (custom)

### **How It Works**

```
Windows System                      Zabbix Agent                      Zabbix Server
┌──────────────────┐              ┌──────────────────┐              ┌──────────────────┐
│ Performance      │   Query     │                  │   Push/       │                  │
│ Counters         │────────────►│  Zabbix Agent   │─────────────►│  Zabbix          │
│ (PDH API)        │◄────────────│  Service        │    Pull      │  Server          │
└──────────────────┘              └──────────────────┘              └──────────────────┘
```

### **Configuration File**

**Location**: `C:\Program Files\Zabbix Agent\zabbix_agentd.conf`

```ini
# Basic Settings
Server=zabbix-server.company.com
ServerActive=zabbix-server.company.com
Hostname=WIN-SERVER-01

# Agent Settings
ListenPort=10050
RefreshActiveChecks=120

# Remote Commands
EnableRemoteCommands=1

# User Parameters (Custom Metrics)
UnsafeUserParameters=1
```

### **Advantages**

- ✅ Deep Windows integration
- ✅ Native performance counter access
- ✅ Event Log monitoring
- ✅ Reliable and well-tested
- ✅ Low resource footprint

### **Limitations**

- ⚠️ Windows only
- ⚠️ Requires firewall exceptions (port 10050)
- ⚠️ Agent updates require package deployment
- ⚠️ Some metrics require administrative access

---

## 🐧 **Linux Agent (Zabbix Agent 2)**

### **What is the Linux Agent?**

The **Zabbix Linux Agent** is a daemon that runs on Linux systems to collect local metrics. It can be the original Zabbix Agent or the newer **Zabbix Agent 2** (written in Go).

### **Key Features**

✅ **Cross-Distribution Compatibility**
- Works on RHEL, CentOS, Ubuntu, Debian, SUSE, etc.
- Native systemd support
- Compatible with most Linux kernels

✅ **Flexible Data Collection**
- System calls for metrics
- Proc filesystem reads (`/proc/meminfo`, `/proc/stat`)
- Command output parsing
- User parameters for custom scripts

✅ **Plugin System (Agent 2)**
- Built-in plugins for common services
- Extensible architecture
- Better performance than original agent

### **Common Metrics Collected**

```bash
# CPU Metrics
- system.cpu.util[,user]         # CPU user time
- system.cpu.load[,avg1]         # Load average (1 min)
- vfs.dev.read.count             # Disk read operations
- net.tcp.service[http]           # TCP service checks

# Memory Metrics  
- vm.memory.size[available]       # Available memory
- vm.memory.size[total]           # Total memory
- system.swap.size[,total]        # Swap size

# Filesystem Metrics
- vfs.fs.size[/,free]            # Filesystem free space
- vfs.fs.inode[/,free]           # Free inodes
```

### **How It Works**

```
Linux System                        Zabbix Agent                      Zabbix Server
┌──────────────────┐              ┌──────────────────┐              ┌──────────────────┐
│ /proc/meminfo     │   Read      │                  │   Push/       │                  │
│ /proc/stat        │────────────►│  Zabbix Agent    │─────────────►│  Zabbix          │
│ /proc/net/dev     │◄────────────│  (Agent 2)       │    Pull      │  Server          │
└──────────────────┘              └──────────────────┘              └──────────────────┘
```

### **Configuration File**

**Location**: `/etc/zabbix/zabbix_agent2.conf` or `/etc/zabbix/zabbix_agentd.conf`

```ini
# Server Configuration
Server=<ZABBIX_SERVER>
ServerActive=<ZABBIX_SERVER>
Hostname=linux-server-01

# Agent Settings
RefreshActiveChecks=60
BufferSend=5
BufferSize=100

# Plugin Settings (Agent 2)
Plugins.System.Run[\*]=system.run
```

### **Installation (Our Custom Script)**

```bash
# Using our deployment script
curl -O http://<ZABBIX_SERVER>:8088/scripts/deploy_linux_agent.sh
chmod +x deploy_linux_agent.sh
sudo ./deploy_linux_agent.sh --server <ZABBIX_SERVER> --hostname $(hostname)
```

### **Advantages**

- ✅ Cross-platform (all major Linux distros)
- ✅ Low resource usage
- ✅ Native Linux metrics access
- ✅ Plugin extensibility (Agent 2)
- ✅ Supports passive and active checks

### **Limitations**

- ⚠️ Requires root access for some metrics
- ⚠️ Firewall configuration needed (port 10050)
- ⚠️ Custom metrics require script maintenance
- ⚠️ Agent on every monitored host

---

## 📊 **PCP Agent (Performance Co-Pilot)**

### **What is PCP?**

**Performance Co-Pilot (PCP)** is an open-source framework for collecting, archiving, and analyzing system-level performance metrics. Our **custom PCP agent** is a lightweight, agentless approach that uses HTTP to expose metrics.

### **Why We Built a Custom PCP Agent?**

Our traditional Zabbix Linux Agent approach had limitations:
- **Agent fatigue**: Installing agents on every system
- **Resource overhead**: Agent running on every monitored host
- **Update complexity**: Rolling out agent updates
- **Firewall management**: Opening ports on many systems

**PCP Solution**: Instead of pushing an agent to every host, we deploy a **central pmproxy service** that:
1. Collects metrics from multiple hosts via PCP protocol
2. Exposes metrics via HTTP (Prometheus format)
3. Zabbix pulls metrics via HTTP (active checks)

### **Architecture**

```
┌──────────────────────────────────────────────────────────────────┐
│                     Our Monitoring Architecture                   │
└──────────────────────────────────────────────────────────────────┘

  ┌──────────────┐         ┌──────────────┐         ┌──────────────┐
  │  Linux      │  PCP    │              │ HTTP    │   Zabbix    │
  │  Host 1     │────────►│   pmproxy    │────────►│   Server    │
  │  (pmcd)     │  Metrics│   (Port     │ Metrics │   (Port     │
  │              │         │   44323)    │         │   8088)     │
  └──────────────┘         └──────────────┘         └──────────────┘
                                 ▲
  ┌──────────────┐               │
  │  Linux      │  PCP           │
  │  Host 2     │────────────────┘
  │  (pmcd)     │
  └──────────────┘

  ┌──────────────┐         ┌──────────────┐
  │  Cockpit    │  HTTP   │   Zabbix     │
  │  Plugin     │────────►│   Items      │
  │  (on host)  │  Local  │   (Pull)     │
  └──────────────┘         └──────────────┘
```

### **Key Components**

#### **1. PMCD (Performance Metrics Collector Daemon)**
- Runs on each Linux host
- Collects metrics from various sources
- Responds to PCP protocol queries
- Default port: **44321**

#### **2. pmproxy**
- Our central aggregation service
- Acts as a proxy between Zabbix and pmcd agents
- Converts PCP metrics to Prometheus text format
- Exposes HTTP endpoint on **port 44323**
- Connects to multiple pmcd instances

#### **3. Zabbix HTTP Agent Items**
- Zabbix pulls metrics via HTTP GET requests
- No agent needed on target systems
- Prometheus format parsing (Regex preprocessing)
- Supports 9000+ pre-defined PCP metrics

### **Why PCP Over Traditional Agent?**

| Feature | Traditional Agent | PCP Agent |
|---------|------------------|-----------|
| **Installation** | Install on every host | Centralized proxy only |
| **Updates** | Update every agent | Update single pmproxy |
| **Resource** | Agent overhead on each host | Minimal (one service) |
| **Firewall** | Open port on each host | Single port (44323) |
| **Metrics** | Manual custom metrics | 9000+ pre-built metrics |
| **Protocol** | Zabbix agent protocol | Standard HTTP |
| **Scalability** | Agent per host | Single proxy scales well |

### **Available PCP Metrics (9000+)**

```bash
# System Metrics
kernel.all.load                     # System load average
mem.util.used                       # Memory used
disk.all.read_bytes                 # Disk read bytes
network.interface.in.bytes          # Network input bytes
proc.nprocs                         # Number of processes

# Application Metrics
postgresql.statements.count         # PostgreSQL query count
mysql.commands.count                # MySQL command count
apache.busy_workers                 # Apache busy workers
redis.clients.connected             # Redis connected clients

# Hardware Metrics
ipmitool.sdr.list                   # IPMI sensor data
lmsensors.fan.speed                 # Fan speed
ipmitool.sel.list                   # System Event Log
```

### **How Data Flows**

```
Step 1: pmproxy fetches metrics from pmcd agents
────────────────────────────────────────────────
pmproxy --pcp://<TARGET_HOST>:44321--> Host 1 (pmcd)
pmproxy --pcp://<TARGET_HOST_2>:44321--> Host 2 (pmcd)

Step 2: pmproxy converts to Prometheus format
────────────────────────────────────────────────
# PCP metric (internal)
kernel_all_load 1.25

# Prometheus format (HTTP output)
# HELP kernel_all_load 
# TYPE kernel_all_load gauge
kernel_all_load{hostname="server01",...} 1.25

Step 3: Zabbix pulls via HTTP
────────────────────────────────────────────────
Zabbix --> GET http://<ZABBIX_SERVER>:44323/metrics
       --> Regex preprocessing
       --> Store as item value
```

### **Configuration (pmproxy)**

```bash
# Start pmproxy with access control
pmproxy \
  -l 0.0.0.0:44323 \        # Listen on all interfaces
  -A +all \                  # Allow all clients  
  -h host1:44321 \           # Connect to Host 1 pmcd
  -h host2:44321            # Connect to Host 2 pmcd
```

### **Zabbix Item Configuration**

```bash
# Item: Memory Used
Name: Memory Used
Type: HTTP agent
Key: pcp_mem_used
URL: http://<ZABBIX_SERVER>:44323/metrics?names=mem.util.used
Preprocessing:
  1. Regex: mem_util_used\{[^}]+\}\s+(\d+)\n\1
  2. Multiply by: 1024 (convert KB to bytes)
```

### **Advantages of Our PCP Approach**

✅ **Agentless**
- No agent installation on target systems
- Reduces maintenance overhead
- Faster deployment

✅ **Rich Metrics**
- 9000+ pre-built metrics available
- Covers Linux kernel, apps, hardware
- Continuously maintained by PCP community

✅ **Scalable Architecture**
- Single pmproxy can monitor many hosts
- Centralized metric aggregation
- Easy to add new hosts

✅ **Standard Protocol**
- Uses HTTP (port 44323)
- Prometheus text format
- Easy to integrate with other tools

✅ **Performance Data Quality**
- High-resolution metrics (sub-second)
- Historical data preservation
- Statistical aggregation functions

### **Limitations**

⚠️ **Linux Only**: PCP primarily supports Linux (Windows in development)
⚠️ **Network Dependency**: Requires connectivity to target systems
⚠️ **Setup Complexity**: Requires pmcd on each target host
⚠️ **Learning Curve**: New technology for team

---

## 🔄 **Comparison Matrix**

| Aspect | Windows Agent | Linux Agent | PCP Agent |
|--------|--------------|-------------|-----------|
| **Platform** | Windows only | Linux/Unix | Linux primarily |
| **Architecture** | Traditional agent | Traditional agent | Agentless/HTTP |
| **Installation** | Per-host install | Per-host install | Centralized proxy |
| **Metrics Count** | ~200 built-in | ~300 built-in | 9000+ built-in |
| **Resource Usage** | ~15-30 MB RAM | ~10-20 MB RAM | ~50 MB RAM (proxy) |
| **Updates** | Per-host | Per-host | Single point |
| **Latency** | Low | Low | Medium (network) |
| **Scalability** | Good | Good | Excellent |
| **Custom Metrics** | User parameters | User parameters | User parameters |

---

## 📋 **When to Use Which Agent?**

### **Use Windows Agent When:**
- Monitoring Windows servers or workstations
- Need deep Windows integration (Event Log, AD, etc.)
- Windows-specific performance counters required
- Have existing Zabbix Windows infrastructure

### **Use Linux Agent When:**
- Monitoring critical Linux servers
- Need real-time, low-latency metrics
- Firewall allows port 10050
- Prefer traditional agent architecture
- Need custom shell/python scripts

### **Use PCP Agent When:**
- Monitoring many Linux hosts at scale
- Want rich, pre-built metrics without custom scripts
- Prefer centralized metric collection
- Need hardware-level metrics (IPMI, sensors)
- Building long-term performance dashboards
- Moving away from per-host agents

---

## 🎓 **Key Takeaways**

1. **Three agent types serve different purposes**: Windows for Windows, Linux Agent for traditional Linux monitoring, PCP for scalable Linux observability

2. **PCP is agentless**: Instead of installing software on every system, we use HTTP to pull metrics through a central proxy

3. **Rich metrics library**: PCP provides 9000+ metrics covering kernel, applications, hardware, and middleware

4. **Our architecture**: Central pmproxy on port 44323 aggregates metrics from PCP-enabled hosts, Zabbix pulls via HTTP

5. **Choose wisely**: Select the right agent type based on platform, scale, and monitoring requirements

---

## 📚 **Next Steps**

- **[PCP Template Guide](PCP_TEMPLATE_GUIDE.md)** - Deep dive into our custom PCP monitoring template
- **[Linux Agent Deployment](LINUX_AGENT_DEPLOYMENT.md)** - Step-by-step guide to adding Linux hosts
- **[Zabbix Web UI Basics](ZABBIX_UI_TRAINING.md)** - Navigating and configuring Zabbix

---

**Questions?** Contact the monitoring team or refer to the admin documentation.