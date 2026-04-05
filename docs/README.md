# Zabbix Monitoring System - Training Documentation

## 📚 **Training Guide Index**

Welcome to the Zabbix Monitoring System training documentation. This directory contains comprehensive guides for understanding and operating our monitoring infrastructure.

---

## 🎯 **Training Modules**

### **Module 1: Agent Types Overview**
📄 **[01_AGENT_TYPES_OVERVIEW.md](01_AGENT_TYPES_OVERVIEW.md)**

**What you'll learn:**
- Three agent types: Windows, Linux, and PCP
- How each agent works and when to use them
- Architecture comparison and decision matrix
- Why we built a custom PCP agent

**Target audience:** All team members  
**Estimated time:** 20 minutes

---

### **Module 2: PCP Template Guide**
📄 **[02_PCP_TEMPLATE_GUIDE.md](02_PCP_TEMPLATE_GUIDE.md)**

**What you'll learn:**
- Structure and purpose of our custom PCP template
- 8 core metrics collected and why they matter
- How preprocessing works (Prometheus → Zabbix)
- How to extend the template with new metrics

**Target audience:** Monitoring administrators  
**Estimated time:** 15 minutes

---

### **Module 3: Linux Host Deployment**
📄 **[03_LINUX_AGENT_DEPLOYMENT.md](03_LINUX_AGENT_DEPLOYMENT.md)**

**What you'll learn:**
- Step-by-step guide to adding Linux hosts
- Using our custom deployment script
- Post-deployment verification
- Troubleshooting common issues

**Target audience:** System administrators  
**Estimated time:** 10 minutes (plus deployment time)

---

## 🚀 **Quick Start Path**

### **For New Team Members**
1. Start with **Module 1** (Agent Types Overview)
2. Read **Module 2** (PCP Template Guide)
3. Practice with **Module 3** (Linux Host Deployment)

### **For Experienced Admins**
1. Skip to **Module 3** for deployment procedures
2. Reference **Module 2** for template details
3. Use **Module 1** as a reference guide

---

## 📊 **System Quick Reference**

### **Access Information**
| Service | URL | Credentials |
|---------|-----|-------------|
| Zabbix Web UI | http://<ZABBIX_SERVER>:8088 | <admin_user> / <admin_password> |
| Zabbix Web UI | http://<ZABBIX_SERVER>:8088 | monitor@<domain> / <password> |
| Zabbix API | http://<ZABBIX_SERVER>:8088/api_jsonrpc.php | Bearer token |
| pmproxy | http://<ZABBIX_SERVER>:44323/metrics | No auth required |

### **Key Ports**
| Port | Service | Purpose |
|------|---------|---------|
| 8088 | Zabbix Web UI | Monitoring interface |
| 44323 | pmproxy | PCP metrics aggregation |
| 44321 | pmcd | PCP collector (per host) |

### **Template Information**
| Template | ID | Metrics | Purpose |
|----------|----|---------|---------|
| PCP_Monitoring_via_Cockpit | 10782 | 8 | Core Linux monitoring |
| template_linux_with_pcp.xml | N/A | 51 | Comprehensive Linux monitoring |

---

## 🛠️ **Common Tasks**

### **Add a New Linux Host**
```bash
sudo ./scripts/deploy_linux_agent.sh \
  --server <ZABBIX_SERVER> \
  --hostname $(hostname) \
  --host-ip $(hostname -I | awk '{print $1}')
```

### **Check System Status**
```bash
./scripts/check_system_status.sh
```

### **Diagnose PCP Issues**
```bash
./scripts/diagnose_pcp_issues.sh
```

### **Test PCP Metrics**
```bash
curl http://<ZABBIX_SERVER>:44323/metrics?names=mem.util.used
```

---

## 📚 **Additional Resources**

### **Project Documentation**
- **[README](../README.md)** - Main project overview
- **[Final Status Report](FINAL_STATUS_2026-04-03.md)** - Current system status
- **[Manual Trigger Setup](MANUAL_TRIGGER_SETUP.md)** - Creating alerts
- **[Next Steps Guide](NEXT_STEPS_MANUAL_GUIDE.md)** - Future enhancements

### **Scripts**
- `deploy_zabbix_root_8088.sh` - Main deployment script
- `deploy_linux_agent.sh` - Linux host deployment
- `check_system_status.sh` - System verification
- `diagnose_pcp_issues.sh` - Diagnostic tool
- `fix_pcp_preprocessing_api.py` - Preprocessing fixes

### **Templates**
- `template_linux_with_pcp.xml` - Comprehensive PCP template (51 metrics)
- `template_elasticsearch.xml` - Elasticsearch monitoring
- `template_mysql_with_pcp.xml` - MySQL with PCP

---

## 🎓 **Learning Path**

```
Beginner                          Intermediate                      Advanced
    │                                 │                                │
    ▼                                 ▼                                ▼
┌──────────────┐              ┌──────────────┐              ┌──────────────┐
│ Module 1     │              │ Module 2     │              │ Module 3     │
│ Agent Types  │─────────────►│ PCP Template │─────────────►│ Deployment   │
│ Overview     │              │ Guide        │              │ Guide        │
└──────────────┘              └──────────────┘              └──────────────┘
    │                                 │                                │
    ▼                                 ▼                                ▼
 Understand agents              Understand template              Deploy hosts
 Compare architectures          Extend metrics                   Troubleshoot
 Choose right agent             Preprocessing basics             Verify deployment
```

---

## 📞 **Support**

**For questions about:**
- **Agent types**: See Module 1
- **Template configuration**: See Module 2
- **Deployment issues**: See Module 3
- **General monitoring**: Contact the monitoring team

**Emergency contacts:**
- Monitoring team: [team-email@company.com](mailto:team-email@company.com)
- On-call engineer: [on-call@company.com](mailto:on-call@company.com)

---

## 📝 **Document History**

| Date | Version | Changes |
|------|---------|---------|
| 2026-04-03 | 1.0 | Initial training documentation created |

---

**Last Updated**: 2026-04-03  
**Maintained by**: Monitoring Team  
**Status**: ✅ Active