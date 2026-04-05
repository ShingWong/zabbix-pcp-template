# PCP Monitoring Template Guide
## Training Guide - Understanding Our Custom Template

---

## 📚 **Overview**

This document explains our custom **PCP Monitoring via Cockpit** template, including its purpose, structure, and how it works. This guide is designed for administrators who need to understand the template without diving into operational details.

---

## 🎯 **What is This Template?**

Our **PCP_Monitoring_via_Cockpit** template is a Zabbix template that defines how we collect system metrics from Linux hosts using the Performance Co-Pilot (PCP) framework.

**Template Name**: `PCP_Monitoring_via_Cockpit`  
**Template ID**: `10782`  
**Host Type**: Linux servers with PCP/pmcd installed  
**Data Source**: HTTP endpoint on pmproxy (port 44323)

---

## 🏗️ **Template Architecture**

### **How the Template Works**

```
┌─────────────────────────────────────────────────────────────────┐
│                    Template Structure                            │
└─────────────────────────────────────────────────────────────────┘

  Template: PCP_Monitoring_via_Cockpit
  ├── 8 Items (metrics being collected)
  ├── Preprocessing rules (how to parse data)
  └── Application groups (logical grouping)

  Each Item:
  ├── Name: Human-readable label
  ├── Key: Unique identifier (pcp_*)
  ├── Type: HTTP agent
  ├── URL: Where to fetch data
  ├── Preprocessing: How to extract value
  └── Update interval: How often to collect
```

### **Data Flow**

```
Zabbix Server (Port 8088)
  │
  ├──► HTTP GET Request
  │    URL: http://{HOST.CONN}:44323/metrics?names=mem.util.used
  │
  ├──► pmproxy (Port 44323)
  │    └──► Returns Prometheus format metrics
  │
  ├──► Zabbix Preprocessing
  │    └──► Regex extraction + unit conversion
  │
  └──► Store in database
       └──► Available for triggers, graphs, dashboards
```

---

## 📊 **Metrics Collected (8 Core Metrics)**

### **1. Memory Used**
- **Key**: `pcp_mem_used`
- **What it measures**: Amount of physical memory currently in use
- **PCP Metric**: `mem.util.used`
- **Units**: Kilobytes (KB)
- **Update interval**: 30 seconds
- **Why it matters**: Memory pressure is a leading indicator of system performance issues

### **2. System Load (1 minute)**
- **Key**: `pcp_load_1min`
- **What it measures**: Average system load over the last minute
- **PCP Metric**: `kernel.all.load`
- **Units**: Load average (float)
- **Update interval**: 30 seconds
- **Why it matters**: Indicates how busy the system is (CPU, I/O, and disk combined)

### **3. Network In Bytes**
- **Key**: `pcp_network_in_bytes`
- **What it measures**: Network bytes received per second
- **PCP Metric**: `network.interface.in.bytes`
- **Units**: Bytes per second
- **Update interval**: 30 seconds
- **Why it matters**: Network traffic patterns help identify bottlenecks and anomalies

### **4. CPU Utilization**
- **Key**: `pcp_cpu_util`
- **What it measures**: CPU user time (counter metric)
- **PCP Metric**: `kernel.all.cpu.user`
- **Units**: Milliseconds (counter)
- **Update interval**: 30 seconds
- **Why it matters**: CPU usage is fundamental to system performance monitoring

### **5. Disk Read Bytes**
- **Key**: `pcp_disk_read_bytes`
- **What it measures**: Total bytes read from all disk devices
- **PCP Metric**: `disk.all.read_bytes`
- **Units**: Bytes (counter)
- **Update interval**: 30 seconds
- **Why it matters**: Disk I/O is often the bottleneck in database and file servers

### **6. Disk Write Bytes**
- **Key**: `pcp_disk_write_bytes`
- **What it measures**: Total bytes written to all disk devices
- **PCP Metric**: `disk.all.write_bytes`
- **Units**: Bytes (counter)
- **Update interval**: 30 seconds
- **Why it matters**: Write patterns indicate application behavior and potential issues

### **7-8. Test Variants**
- **Keys**: `pcp_mem_used_test`, `pcp_cpu_util_test`
- **Purpose**: Duplicate metrics used for testing and validation
- **Note**: These are identical to the main metrics but with separate keys

---

## 🔧 **Preprocessing Explained**

### **What is Preprocessing?**

Preprocessing is how Zabbix transforms raw data from the source into the value we want to store. Since pmproxy returns metrics in **Prometheus text format**, we need to extract the actual number.

### **Example: Memory Used**

**Raw Data from pmproxy:**
```
# HELP mem_util_used used memory metric from /proc/meminfo
# TYPE mem_util_used gauge
mem_util_used{agent="linux",hostname="<hostname>",...} 195625
```

**Preprocessing Steps:**
1. **Regex extraction**: `mem_util_used\{[^}]+\}\s+(\d+)\n\1`
   - Matches the metric line
   - Captures the number (195625)
   - Outputs just the captured value
2. **Multiply by 1024**: Converts KB to bytes

**Result**: `200,317,952` bytes stored in Zabbix

### **Preprocessing Types Used**

| Type | Name | Purpose |
|------|------|---------|
| **5** | Regular expression | Extract value from Prometheus text |
| **2** | Multiply | Convert units (KB to bytes) |

---

## 📦 **Template Components**

### **Items**
- **8 total items** defined in the template
- Each item specifies:
  - Name, key, type, URL, update interval
  - Value type (numeric float or unsigned)
  - Preprocessing rules

### **Applications**
- Items are grouped into logical applications:
  - **PCP**: All PCP metrics
  - **Memory**: Memory-related metrics
  - **CPU**: CPU-related metrics
  - **Disk**: Disk I/O metrics
  - **Network**: Network metrics

### **Triggers**
- The template itself does not include triggers
- Triggers are created per-host or per-environment
- Use the `MANUAL_TRIGGER_SETUP.md` guide for trigger creation

---

## 🔄 **How to Apply the Template**

### **Via Zabbix Web UI**

1. **Navigate**: Configuration → Hosts
2. **Select host**: Click on the host name
3. **Go to Templates tab**
4. **Link new template**: Search for "PCP_Monitoring_via_Cockpit"
5. **Click Update**

### **Via API**

```bash
curl -s "http://localhost:8088/api_jsonrpc.php" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <TOKEN>" \
  -d '{
    "jsonrpc": "2.0",
    "method": "template.massadd",
    "params": {
      "templates": [{"templateid": "10782"}],
      "hosts": [{"hostid": "10779"}]
    },
    "id": 1
  }'
```

---

## 📋 **Template vs Host Items**

### **Template Items**
- Defined in the template
- Inherited by all hosts using the template
- Cannot be modified on individual hosts
- Updated by modifying the template

### **Host-Specific Items**
- Created directly on the host
- Not part of the template
- Can be customized per host
- Useful for host-specific metrics

**Best Practice**: Use template items for standard metrics, host-specific items for custom needs.

---

## 🎓 **Key Concepts**

### **1. HTTP Agent Type**
- Zabbix makes HTTP GET requests to fetch data
- No agent software needed on target systems
- URL format: `http://{HOST.CONN}:44323/metrics?names=<metric_name>`
- `{HOST.CONN}` is replaced with the host's IP address

### **2. Prometheus Format**
- PCP exports metrics in Prometheus text format
- Format: `metric_name{labels} value`
- Zabbix uses Regex to extract the value
- Labels provide metadata (hostname, agent, etc.)

### **3. Counter vs Gauge**
- **Gauge**: Current value (e.g., memory used)
  - Can go up or down
  - Stored directly
- **Counter**: Cumulative value (e.g., CPU time, disk bytes)
  - Only increases (resets on reboot)
  - Use `delta()` or `rate()` functions in triggers

### **4. Update Intervals**
- All metrics update every **30 seconds**
- Configurable per-item
- Balance between data freshness and system load

---

## 🚀 **Extending the Template**

### **Adding New Metrics**

To add new metrics to the template:

1. **Identify the PCP metric name**:
   ```bash
   curl http://<ZABBIX_SERVER>:44323/metrics | grep <keyword>
   ```

2. **Create new item in template**:
   - Name: Human-readable label
   - Key: `pcp_<metric_name>`
   - URL: `http://{HOST.CONN}:44323/metrics?names=<pcp_metric>`
   - Preprocessing: Regex pattern for that metric

3. **Test the item**:
   - Verify URL returns data
   - Check preprocessing extracts correct value
   - Confirm data appears in "Latest data"

### **Available Metric Categories**

Our comprehensive PCP template (`template_linux_with_pcp.xml`) includes **51 metrics** across:
- **Memory**: Used, free, cached, buffers, swap
- **CPU**: User, system, idle, wait, interrupt
- **Disk**: Read/write bytes, operations, latency
- **Network**: In/out bytes, packets, errors
- **Processes**: Count, threads, context switches
- **Filesystem**: Space, inodes, open files

---

## ⚠️ **Important Notes**

### **Template Inheritance**
- Items defined in templates are **read-only** on hosts
- To modify an item, update the template
- Changes propagate to all linked hosts

### **Preprocessing is Critical**
- Incorrect regex = no data collected
- Test regex against actual pmproxy output
- Use the "Test" button in Zabbix UI to validate

### **Metric Names Matter**
- PCP metric names are case-sensitive
- Use exact names from PCP documentation
- Verify with `curl` before creating items

### **Performance Considerations**
- 30-second intervals balance freshness and load
- Each HTTP request has minimal overhead
- pmproxy handles multiple concurrent requests efficiently

---

## 📚 **Related Documentation**

- **[Agent Types Overview](01_AGENT_TYPES_OVERVIEW.md)** - Understanding Windows, Linux, and PCP agents
- **[Linux Agent Deployment](03_LINUX_AGENT_DEPLOYMENT.md)** - Adding new Linux hosts
- **[Manual Trigger Setup](MANUAL_TRIGGER_SETUP.md)** - Creating alert triggers
- **[PCP Monitoring Update](PCP_MONITORING_UPDATE_2026-04-03.md)** - Detailed status report

---

**Questions?** Contact the monitoring team or refer to the admin documentation.