# Zabbix DNS Monitoring Template

A Zabbix template for monitoring DNS server performance and availability across multiple DNS providers and domains.

## Features

- **Multi-DNS Server Monitoring**: Monitor multiple DNS servers simultaneously (CloudFlare, Google, Quad9, and custom servers)
- **Domain-Specific Testing**: Test DNS resolution for specific domains on each server
- **Performance Metrics**: Track DNS query response times with configurable thresholds
- **Availability Monitoring**: Check DNS server TCP port 53 availability

## Requirements

- Zabbix Server 7.4 or higher
- Zabbix Agent with active checks enabled on monitored hosts
- Network connectivity to DNS servers being monitored

## Installation

1. **Download the Template**
   ```bash
   wget https://raw.githubusercontent.com/McKinnonIT/template-dns-monitoring/refs/heads/main/template_dns_monitoring.yaml
   ```

2. **Import into Zabbix**
   - Log into your Zabbix web interface
   - Go to **Configuration** → **Templates**
   - Click **Import**
   - Choose the `template_dns_monitoring.yaml` file
   - Click **Import**

3. **Assign to Host**
   - Go to **Configuration** → **Hosts**
   - Select your target host (this can be your Zabbix server or a dedicated monitoring host)
   - Go to **Templates** tab
   - Add "DNS Monitoring" template
   - Click **Update**

## Configuration

### Master Item Script Configuration

The template uses a master item script (`dns.targets.config.json`) that defines which DNS servers and domains to monitor. This is the core configuration that drives the entire monitoring setup.

#### Accessing the Master Item Script

1. Go to **Configuration** → **Templates**
2. Find "DNS Monitoring" template and click on it
3. Go to **Items** tab
4. Find the item named "DNS Target Configuration"
5. Click on the item to edit it

#### Configuring DNS Servers and Domains

The master item contains a JavaScript configuration that you need to customize for your environment:

```javascript
// DNS Monitoring Configuration
// Customize this configuration to match your monitoring requirements
var config = [
    {
        "{#SERVER}": "1.1.1.1",
        "{#SERVERNAME}": "CloudFlare",
        "{#DOMAINS}": ["example.com", "google.com", "cloudflare.com"]
    },
    {
        "{#SERVER}": "8.8.8.8",
        "{#SERVERNAME}": "Google",
        "{#DOMAINS}": ["example.com", "google.com", "cloudflare.com"]
    },
    {
        "{#SERVER}": "9.9.9.9",
        "{#SERVERNAME}": "Quad9",
        "{#DOMAINS}": ["example.com", "google.com", "cloudflare.com"]
    }
];
```

#### Configuration Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| `{#SERVER}` | DNS server IP address | `"1.1.1.1"` |
| `{#SERVERNAME}` | Human-readable name for the DNS server | `"CloudFlare"` |
| `{#DOMAINS}` | Array of domains to test against this server | `["example.com", "google.com"]` |

#### Customization Examples

**Example 1: Corporate Environment**
```javascript
var config = [
    {
        "{#SERVER}": "192.168.1.10",
        "{#SERVERNAME}": "Internal DNS",
        "{#DOMAINS}": ["intranet.company.com", "mail.company.com", "wiki.company.com"]
    },
    {
        "{#SERVER}": "8.8.8.8",
        "{#SERVERNAME}": "Google Public",
        "{#DOMAINS}": ["google.com", "github.com", "stackoverflow.com"]
    }
];
```

**Example 2: ISP Monitoring**
```javascript
var config = [
    {
        "{#SERVER}": "208.67.222.222",
        "{#SERVERNAME}": "OpenDNS",
        "{#DOMAINS}": ["facebook.com", "twitter.com", "linkedin.com"]
    },
    {
        "{#SERVER}": "1.0.0.1",
        "{#SERVERNAME}": "CloudFlare Alt",
        "{#DOMAINS}": ["reddit.com", "youtube.com", "netflix.com"]
    }
];
```

### Important Notes for Master Item Configuration

- **Do NOT modify the JavaScript logic** below the configuration array - only modify the `config` array
- **Each DNS server can monitor different domains** - domains don't need to be identical across servers

### Trigger Configuration

The template includes pre-configured triggers that you can customize:

#### High DNS Latency Trigger
- **Expression**: `count(/DNS Monitoring/net.dns.perf[{#SERVER},{#DOMAIN}],5m,"gt",0.5)>=2`
- **Threshold**: 0.5 seconds (500ms)
- **Condition**: 2 or more queries exceed threshold within 5 minutes
- **Priority**: Warning

#### DNS Server Down Trigger
- **Expression**: `last(/DNS Monitoring/net.tcp.port[{#SERVER},53])=0`
- **Condition**: TCP port 53 not responding
- **Priority**: High

## Monitored Metrics

### Per Domain/Server Combination
- **DNS Resolution Result** (`net.dns.get[server,domain]`): The resolved IP address
- **DNS Query Performance** (`net.dns.perf[server,domain]`): Response time in seconds

### Per DNS Server
- **Average DNS Performance** (`dns.server.perf.avg[server]`): Average response time across all domains
- **DNS Server Availability** (`net.tcp.port[server,53]`): TCP port 53 connectivity

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly in a lab environment
5. Submit a pull request

## License

This template is released under the MIT License. See LICENSE file for details.

## Support

- **Issues**: Report bugs and feature requests via GitHub Issues
- **Documentation**: Check the Zabbix official documentation for net.dns.* items
- **Community**: Join the Zabbix community forums for additional support

## Version History

- **v1.0**: Initial release with basic DNS monitoring
- **v1.1**: Added Quad9 server, improved documentation
- **v1.2**: Fixed recovery expression for DNS server availability trigger

---

**Note**: Always test this template in a non-production environment before deploying to production systems.