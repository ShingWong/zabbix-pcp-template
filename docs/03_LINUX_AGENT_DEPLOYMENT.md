# Linux Host Deployment Guide
## Training Guide - Adding Linux Hosts with Our Script

---

## 📚 **Overview**

This document provides step-by-step instructions for adding new Linux hosts to our Zabbix monitoring system using our custom deployment script. This guide is designed for system administrators who need to onboard new servers quickly and reliably.

---

## 🎯 **What Does This Script Do?**

Our deployment script automates the entire process of adding a Linux host to Zabbix monitoring:

1. **Installs PCP** (Performance Co-Pilot) on the target host
2. **Configures pmcd** (Performance Metrics Collector Daemon)
3. **Registers the host** with Zabbix server
4. **Applies the PCP template** to the new host
5. **Verifies connectivity** and data collection
6. **Reports status** with success/failure details

---

## 📋 **Prerequisites**

### **Before Running the Script**

✅ **Target Host Requirements:**
- Linux operating system (RHEL, CentOS, Ubuntu, Debian, SUSE)
- Root or sudo access
- Network connectivity to Zabbix server (port 8088)
- Network connectivity to pmproxy (port 44323)
- SSH access from deployment machine

✅ **Zabbix Server Requirements:**
- Zabbix server running and accessible
- Admin credentials available
- PCP template already created (`PCP_Monitoring_via_Cockpit`)
- pmproxy service running on port 44323

✅ **Network Requirements:**
```
Target Host ──────────────► Zabbix Server (Port 8088)
               HTTP (API)

Target Host ──────────────► pmproxy (Port 44323)
               PCP Protocol
```

### **Required Information**

| Parameter | Description | Example |
|-----------|-------------|---------|
| `--server` | Zabbix server IP/hostname | `<ZABBIX_SERVER>` |
| `--hostname` | Host name in Zabbix | `linux-server-01` |
| `--host-ip` | Target host IP address | `<TARGET_HOST>` |
| `--template` | Template name (optional) | `PCP_Monitoring_via_Cockpit` |
| `--group` | Host group (optional) | `Linux servers` |

---

## 🚀 **Quick Start**

### **Basic Usage**

```bash
# Download the script
curl -O http://<ZABBIX_SERVER>:8088/scripts/deploy_linux_agent.sh

# Make it executable
chmod +x deploy_linux_agent.sh

# Run with minimal parameters
sudo ./deploy_linux_agent.sh \
  --server <ZABBIX_SERVER> \
  --hostname $(hostname) \
  --host-ip $(hostname -I | awk '{print $1}')
```

### **Full Example**

```bash
sudo ./deploy_linux_agent.sh \
  --server <ZABBIX_SERVER> \
  --hostname prod-web-01 \
  --host-ip <TARGET_HOST> \
  --template PCP_Monitoring_via_Cockpit \
  --group "Production Servers" \
  --verbose
```

---

## 📖 **Step-by-Step Process**

### **Step 1: Prepare the Target Host**

```bash
# SSH to the target host
ssh user@<TARGET_HOST>

# Verify connectivity to Zabbix server
curl -s http://<ZABBIX_SERVER>:8088 > /dev/null && echo "Zabbix accessible" || echo "Zabbix NOT accessible"

# Verify connectivity to pmproxy
curl -s http://<ZABBIX_SERVER>:44323/metrics > /dev/null && echo "pmproxy accessible" || echo "pmproxy NOT accessible"
```

### **Step 2: Run the Deployment Script**

```bash
# Download and run the script
curl -s http://<ZABBIX_SERVER>:8088/scripts/deploy_linux_agent.sh | sudo bash -s -- \
  --server <ZABBIX_SERVER> \
  --hostname $(hostname) \
  --host-ip $(hostname -I | awk '{print $1}')
```

### **Step 3: Script Execution Flow**

The script performs these steps automatically:

```
┌─────────────────────────────────────────────────────────────────┐
│                    Script Execution Flow                         │
└─────────────────────────────────────────────────────────────────┘

  Step 1: Validation
  ├── Check root/sudo access
  ├── Verify network connectivity
  └── Validate parameters

  Step 2: Install PCP
  ├── Detect OS (RHEL/Ubuntu/Debian)
  ├── Install pcp package
  ├── Install pcp-pmda-root
  └── Start pmcd service

  Step 3: Configure PCP
  ├── Enable pmcd service
  ├── Configure access control
  └── Verify pmcd is running

  Step 4: Register with Zabbix
  ├── Authenticate with Zabbix API
  ├── Create host in Zabbix
  ├── Apply PCP template
  └── Set host properties

  Step 5: Verification
  ├── Test data collection
  ├── Verify metrics in Zabbix
  └── Report status
```

### **Step 4: Verify Deployment**

After the script completes, verify the host is monitoring correctly:

```bash
# Check host status in Zabbix
curl -s "http://<ZABBIX_SERVER>:8088/api_jsonrpc.php" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <TOKEN>" \
  -d '{
    "jsonrpc": "2.0",
    "method": "host.get",
    "params": {
      "output": ["hostid", "host", "name", "status"],
      "filter": {"host": "linux-server-01"}
    },
    "id": 1
  }'

# Check if metrics are collecting
curl -s "http://<ZABBIX_SERVER>:44323/metrics?names=mem.util.used"
```

---

## 🔧 **Script Parameters**

### **Required Parameters**

| Parameter | Short | Description | Example |
|-----------|-------|-------------|---------|
| `--server` | `-s` | Zabbix server IP/hostname | `<ZABBIX_SERVER>` |
| `--hostname` | `-n` | Host name in Zabbix | `linux-server-01` |
| `--host-ip` | `-i` | Target host IP address | `<TARGET_HOST>` |

### **Optional Parameters**

| Parameter | Short | Description | Default |
|-----------|-------|-------------|---------|
| `--template` | `-t` | Template name | `PCP_Monitoring_via_Cockpit` |
| `--group` | `-g` | Host group | `Linux servers` |
| `--verbose` | `-v` | Enable verbose output | `false` |
| `--dry-run` | `-d` | Show what would be done | `false` |
| `--help` | `-h` | Show help message | - |

### **Examples**

```bash
# Minimal deployment
sudo ./deploy_linux_agent.sh -s <ZABBIX_SERVER> -n $(hostname) -i $(hostname -I | awk '{print $1}')

# With custom template and group
sudo ./deploy_linux_agent.sh \
  -s <ZABBIX_SERVER> \
  -n prod-db-01 \
  -i <DB_HOST> \
  -t "PCP_Monitoring_via_Cockpit" \
  -g "Database Servers"

# Dry run (no changes)
sudo ./deploy_linux_agent.sh \
  -s <ZABBIX_SERVER> \
  -n test-server-01 \
  -i <STAGING_HOST> \
  -d

# Verbose output
sudo ./deploy_linux_agent.sh \
  -s <ZABBIX_SERVER> \
  -n prod-app-01 \
  -i <APP_HOST> \
  -v
```

---

## 📊 **What Gets Installed**

### **On the Target Host**

```
┌─────────────────────────────────────────────────────────────────┐
│                    Target Host Components                        │
└─────────────────────────────────────────────────────────────────┘

  PCP Packages:
  ├── pcp                    # Core PCP framework
  ├── pcp-pmda-root          # Root PMDA (system metrics)
  └── pcp-pmda-linux         # Linux-specific PMDA (optional)

  Services:
  ├── pmcd                   # Performance Metrics Collector Daemon
  │   └── Port: 44321
  └── pmlogger               # PCP logger (optional, for archiving)

  Configuration:
  ├── /etc/pcp/              # PCP configuration directory
  ├── /etc/pcp/pmcd/         # PMCD configuration
  └── /etc/pcp/pmlogger/     # PMLogger configuration (if enabled)
```

### **On the Zabbix Server**

```
┌─────────────────────────────────────────────────────────────────┐
│                    Zabbix Server Changes                         │
└─────────────────────────────────────────────────────────────────┘

  Host Created:
  ├── Host name: <hostname>
  ├── IP address: <host-ip>
  ├── Template: PCP_Monitoring_via_Cockpit
  ├── Host group: Linux servers
  └── Status: Monitored

  Items Created (8):
  ├── pcp_mem_used
  ├── pcp_load_1min
  ├── pcp_network_in_bytes
  ├── pcp_cpu_util
  ├── pcp_disk_read_bytes
  ├── pcp_disk_write_bytes
  ├── pcp_mem_used_test
  └── pcp_cpu_util_test
```

---

## ✅ **Post-Deployment Verification**

### **1. Check Host Status**

```bash
# Via Zabbix Web UI
# Navigate to: Monitoring → Hosts
# Search for the new hostname
# Verify status is "Monitored" (green)

# Via API
curl -s "http://<ZABBIX_SERVER>:8088/api_jsonrpc.php" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <TOKEN>" \
  -d '{
    "jsonrpc": "2.0",
    "method": "host.get",
    "params": {
      "output": ["hostid", "host", "name", "status"],
      "filter": {"host": "linux-server-01"}
    },
    "id": 1
  }'
```

### **2. Check Data Collection**

```bash
# Via Zabbix Web UI
# Navigate to: Monitoring → Latest data
# Filter by host: <hostname>
# Verify all 8 PCP items show recent values

# Via API
curl -s "http://<ZABBIX_SERVER>:8088/api_jsonrpc.php" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <TOKEN>" \
  -d '{
    "jsonrpc": "2.0",
    "method": "item.get",
    "params": {
      "output": ["name", "lastvalue", "lastclock"],
      "hostids": "<HOST_ID>",
      "search": {"key_": "pcp_"}
    },
    "id": 1
  }'
```

### **3. Check pmcd Service**

```bash
# On the target host
systemctl status pmcd

# Test pmcd connectivity
pminfo -f mem.util.used

# Check pmcd logs
journalctl -u pmcd -n 20
```

---

## ⚠️ **Troubleshooting**

### **Common Issues**

#### **1. PCP Installation Fails**

**Symptom**: Script fails during PCP installation

**Solution**:
```bash
# Check package manager
sudo apt-get update  # For Debian/Ubuntu
sudo dnf update      # For RHEL/CentOS

# Install PCP manually
sudo apt-get install pcp pcp-pmda-root  # Debian/Ubuntu
sudo dnf install pcp pcp-pmda-root      # RHEL/CentOS

# Start pmcd
sudo systemctl start pmcd
sudo systemctl enable pmcd
```

#### **2. Zabbix API Connection Fails**

**Symptom**: Script cannot connect to Zabbix API

**Solution**:
```bash
# Test connectivity
curl -s http://<ZABBIX_SERVER>:8088

# Check firewall
sudo firewall-cmd --list-ports | grep 8088

# Verify Zabbix is running
systemctl status zabbix-server
```

#### **3. No Data Collection**

**Symptom**: Host created but no metrics collecting

**Solution**:
```bash
# Check pmcd is running
systemctl status pmcd

# Test pmproxy connectivity
curl http://<ZABBIX_SERVER>:44323/metrics?names=mem.util.used

# Check Zabbix item configuration
# Navigate to: Configuration → Hosts → <hostname> → Items
# Verify URLs are correct
```

#### **4. Template Not Applied**

**Symptom**: Host created but no items

**Solution**:
```bash
# Verify template exists
curl -s "http://<ZABBIX_SERVER>:8088/api_jsonrpc.php" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <TOKEN>" \
  -d '{
    "jsonrpc": "2.0",
    "method": "template.get",
    "params": {
      "output": ["templateid", "host"],
      "filter": {"host": "PCP_Monitoring_via_Cockpit"}
    },
    "id": 1
  }'

# Manually apply template via Web UI
# Navigate to: Configuration → Hosts → <hostname> → Templates
# Link: PCP_Monitoring_via_Cockpit
```

---

## 📋 **Deployment Checklist**

### **Before Deployment**

- [ ] Target host is accessible via SSH
- [ ] Network connectivity to Zabbix server (port 8088)
- [ ] Network connectivity to pmproxy (port 44323)
- [ ] Root/sudo access on target host
- [ ] Hostname is unique in Zabbix
- [ ] IP address is correct

### **During Deployment**

- [ ] Script runs without errors
- [ ] PCP installed successfully
- [ ] pmcd service started
- [ ] Host created in Zabbix
- [ ] Template applied
- [ ] Items created

### **After Deployment**

- [ ] Host status is "Monitored"
- [ ] All 8 PCP items collecting data
- [ ] Latest data shows recent values
- [ ] No errors in Zabbix logs
- [ ] No errors in pmcd logs

---

## 🎓 **Best Practices**

### **1. Naming Conventions**

```bash
# Good hostnames
prod-web-01        # Environment-role-number
staging-db-01     # Environment-role-number
dev-app-01        # Environment-role-number

# Bad hostnames
server1           # Too generic
<TARGET_HOST>      # IP address as name
my-server         # No environment/role context
```

### **2. Host Groups**

Organize hosts logically:
- `Linux servers` - All Linux hosts
- `Production Servers` - Production environment
- `Staging Servers` - Staging environment
- `Database Servers` - Database hosts
- `Web Servers` - Web application hosts

### **3. Monitoring Strategy**

- Start with core 8 metrics (memory, CPU, disk, network, load)
- Add triggers after baseline is established
- Create dashboards for visualization
- Expand metrics as needed (51 available in comprehensive template)

### **4. Maintenance**

- Monitor the monitoring system itself
- Check Zabbix server health regularly
- Review metric collection status weekly
- Update PCP packages during maintenance windows
- Test failover procedures

---

## 📚 **Related Documentation**

- **[Agent Types Overview](01_AGENT_TYPES_OVERVIEW.md)** - Understanding Windows, Linux, and PCP agents
- **[PCP Template Guide](02_PCP_TEMPLATE_GUIDE.md)** - Understanding our custom PCP template
- **[Manual Trigger Setup](MANUAL_TRIGGER_SETUP.md)** - Creating alert triggers
- **[System Status Check](../scripts/check_system_status.sh)** - Verify system health

---

## 📞 **Support**

**For deployment issues:**
1. Check the troubleshooting section above
2. Run `./scripts/diagnose_pcp_issues.sh`
3. Review Zabbix logs: `journalctl -u zabbix-server`
4. Contact the monitoring team

**For script updates:**
- Script location: `scripts/deploy_linux_agent.sh`
- Template location: `templates/`
- Configuration: `configs/`

---

**Questions?** Contact the monitoring team or refer to the admin documentation.