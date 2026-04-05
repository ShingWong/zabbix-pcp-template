# Zabbix PCP Monitoring Template

A clean, production-ready Zabbix template for monitoring Linux hosts via **Performance Co-Pilot (PCP)** and Cockpit integration.

## Overview

This template provides **agentless Linux monitoring** using PCP's HTTP API instead of traditional Zabbix agents. Instead of installing software on every monitored host, a central `pmproxy` service collects metrics and exposes them via HTTP in Prometheus format.

## Features

- ✅ **6 core metrics** out of the box (Memory, CPU, Load, Disk, Network)
- ✅ **Agentless** - No Zabbix agent required on target hosts
- ✅ **9000+ metrics** available via PCP (expandable)
- ✅ **HTTP agent** items with Regex preprocessing
- ✅ **Template-based** - Easy to apply to any Linux host
- ✅ **Counter support** - CPU and disk metrics with proper rate conversion
- ✅ **Unit conversion** - Automatic KB→bytes conversion

## Quick Start

### 1. Import the Template

```bash
# In Zabbix Web UI:
# Data collection → Templates → Import
# Select: templates/pcp_monitoring_template.xml
```

### 2. Add a Host

```bash
# In Zabbix Web UI:
# Data collection → Hosts → Create host
# - Host name: my-linux-server
# - Interface: HTTP agent, IP: <host-ip>, Port: 44323
# - Templates: Link "PCP Monitoring via Cockpit"
```

### 3. Ensure pmproxy is Running

On each monitored host:
```bash
sudo dnf install pcp pcp-pmda-root
sudo systemctl start pmcd
sudo systemctl enable pmcd
```

On the monitoring server (pmproxy):
```bash
pmproxy -l 0.0.0.0:44323 -h <target-host>:44321
```

## Metrics

| Metric | Key | PCP Metric | Update |
|--------|-----|------------|--------|
| Memory Used | `pcp_mem_used` | `mem.util.used` | 30s |
| System Load (1min) | `pcp_load_1min` | `kernel.all.load` | 30s |
| Network In Bytes | `pcp_network_in_bytes` | `network.interface.in.bytes` | 30s |
| CPU Utilization | `pcp_cpu_util` | `kernel.all.cpu.user` | 30s |
| Disk Read Bytes | `pcp_disk_read_bytes` | `disk.all.read_bytes` | 30s |
| Disk Write Bytes | `pcp_disk_write_bytes` | `disk.all.write_bytes` | 30s |

## Architecture

```
Linux Host (pmcd) ──PCP──► pmproxy (44323) ──HTTP──► Zabbix Server (8088)
```

## Project Structure

```
zabbix-pcp-template/
├── templates/
│   ├── pcp_monitoring_template.xml    # Zabbix template (import this)
│   ├── template_linux_with_pcp.xml    # Comprehensive template (51 metrics)
│   ├── pcp-discovery.sh               # Low-level discovery script
│   └── userparameter_pcp_discovery.conf # UserParameter config
├── scripts/
│   └── deploy_linux_agent.sh          # Host deployment script
├── docs/
│   ├── 01_AGENT_TYPES_OVERVIEW.md     # Windows vs Linux vs PCP agents
│   ├── 02_PCP_TEMPLATE_GUIDE.md       # Template documentation
│   └── 03_LINUX_AGENT_DEPLOYMENT.md   # Host deployment guide
└── examples/
    └── pcp_metrics_examples.md        # Example metric queries
```

## Requirements

- **Zabbix**: 7.0+ (tested with 7.4.8)
- **PCP**: 5.0+ on target hosts
- **pmproxy**: Running on monitoring server (port 44323)
- **Network**: HTTP connectivity from Zabbix to pmproxy

## License

MIT License - see [LICENSE](LICENSE)

## Author

Based on PCP monitoring templates from the Zabbix community, customized for Cockpit integration.
