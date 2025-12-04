# Network Debugging Runbook

A comprehensive guide for troubleshooting network-related issues when calling URLs, including Cloudflare CLI usage and debugging scenarios.

## Table of Contents
- [Quick Diagnostic Commands](#quick-diagnostic-commands)
- [URL Connection Issues](#url-connection-issues)
- [DNS Debugging](#dns-debugging)
- [SSL/TLS Debugging](#ssltls-debugging)
- [Cloudflare CLI Debugging](#cloudflare-cli-debugging)
- [Firewall and Proxy Issues](#firewall-and-proxy-issues)
- [Performance Debugging](#performance-debugging)
- [Common Error Codes](#common-error-codes)
- [Advanced Debugging Techniques](#advanced-debugging-techniques)

## Quick Diagnostic Commands

### Basic Connectivity Tests
```bash
# Test basic connectivity to a host
ping google.com

# Test port connectivity
telnet example.com 80
nc -zv example.com 443

# Check if DNS is resolving
nslookup example.com
dig example.com

# Trace route to destination
traceroute example.com
mtr example.com  # Better alternative to traceroute
```

### HTTP/HTTPS Testing
```bash
# Basic HTTP request with curl
curl -I https://example.com

# Verbose curl output for debugging
curl -vvv https://example.com

# Follow redirects and show timing
curl -L -w "@curl-format.txt" https://example.com

# Test with specific HTTP version
curl --http2 https://example.com
curl --http1.1 https://example.com
```

## URL Connection Issues

### Common Connection Problems

#### Connection Refused
```bash
# Check if service is running on target port
sudo netstat -tlnp | grep :80
sudo ss -tlnp | grep :80

# Check local firewall rules
sudo iptables -L
sudo ufw status

# Test from different source
curl --interface eth1 https://example.com
curl --local-port 12345 https://example.com
```

#### Timeout Issues
```bash
# Set custom timeout values
curl --connect-timeout 10 --max-time 30 https://example.com

# Test with wget
wget --timeout=30 --tries=1 https://example.com

# Check for packet loss
ping -c 10 example.com
```

#### HTTP Error Codes
```bash
# Get detailed response headers
curl -D headers.txt https://example.com

# Follow redirects manually
curl -I https://example.com
# Follow Location header if 3xx response

# Test with different User-Agent
curl -A "Mozilla/5.0 (compatible; Bot/1.0)" https://example.com
```

### Testing with Different Tools

#### Using wget
```bash
# Download with verbose output
wget -v --server-response https://example.com

# Spider mode (don't download)
wget --spider https://example.com

# Test with proxy
wget --proxy=on --proxy-server=proxy.example.com:8080 https://example.com
```

## DNS Debugging

### DNS Resolution Issues
```bash
# Check system DNS configuration
cat /etc/resolv.conf
systemd-resolve --status

# Test different DNS servers
dig @8.8.8.8 example.com
dig @1.1.1.1 example.com
dig @208.67.222.222 example.com  # OpenDNS

# Check DNS propagation
dig example.com +trace
```

### DNS Record Types
```bash
# A records (IPv4)
dig A example.com

# AAAA records (IPv6)
dig AAAA example.com

# CNAME records
dig CNAME www.example.com

# MX records (mail)
dig MX example.com

# TXT records
dig TXT example.com

# NS records (nameservers)
dig NS example.com

# SOA records
dig SOA example.com
```

### DNS Cache Issues
```bash
# Flush DNS cache (systemd-resolved)
sudo systemd-resolve --flush-caches

# Flush DNS cache (dnsmasq)
sudo service dnsmasq restart

# Flush DNS cache (nscd)
sudo service nscd restart

# Check DNS cache statistics
sudo systemd-resolve --statistics
```

## SSL/TLS Debugging

### SSL Certificate Issues
```bash
# Check SSL certificate details
openssl s_client -connect example.com:443 -servername example.com

# Check certificate expiry
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -dates

# Test specific SSL/TLS version
openssl s_client -connect example.com:443 -tls1_2
openssl s_client -connect example.com:443 -tls1_3

# Check certificate chain
openssl s_client -connect example.com:443 -showcerts
```

### SSL Verification Issues
```bash
# Skip SSL verification (for testing only)
curl -k https://example.com

# Use specific CA bundle
curl --cacert /path/to/ca-bundle.crt https://example.com

# Check SSL with verbose output
curl -vvv https://example.com 2>&1 | grep -i ssl
```

### SSL Tools
```bash
# Using gnutls-cli
gnutls-cli --port 443 example.com

# Using nmap for SSL enumeration
nmap --script ssl-enum-ciphers -p 443 example.com

# Check SSL using sslscan
sslscan example.com

# Online SSL checker alternative
echo "Use: https://www.ssllabs.com/ssltest/"
```

## Cloudflare CLI Debugging

### Installation and Setup
```bash
# Install Cloudflare CLI
curl -L https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64 -o cloudflared
sudo mv cloudflared /usr/local/bin/
sudo chmod +x /usr/local/bin/cloudflared

# Alternative: using package manager
# brew install cloudflare/cloudflare/cloudflared  # macOS
# snap install cloudflared  # Ubuntu

# Login to Cloudflare
cloudflared login
```

### Zone and DNS Management
```bash
# List zones
cloudflared dns list

# Get zone information
cloudflared dns zone-info example.com

# List DNS records
cloudflared dns records example.com

# Add DNS record
cloudflared dns create example.com A subdomain 192.168.1.100

# Update DNS record
cloudflared dns update example.com A subdomain 192.168.1.101

# Delete DNS record
cloudflared dns delete example.com A subdomain
```

### Tunnel Debugging
```bash
# Create tunnel
cloudflared tunnel create my-tunnel

# List tunnels
cloudflared tunnel list

# Route traffic through tunnel
cloudflared tunnel route dns my-tunnel example.com

# Run tunnel with config
cloudflared tunnel --config /path/to/config.yml run

# Debug tunnel connectivity
cloudflared tunnel --loglevel debug run my-tunnel
```

### Cloudflare Network Diagnostics
```bash
# Test connectivity to Cloudflare edge
curl -H "CF-Connecting-IP: 1.1.1.1" https://example.com

# Check Cloudflare headers
curl -I https://example.com | grep -i cloudflare

# Test bypass cache
curl -H "Cache-Control: no-cache" https://example.com

# Check if site is behind Cloudflare
dig example.com | grep -E "(cloudflare|CF)"
```

### Cloudflare Troubleshooting Commands
```bash
# Check Cloudflare status
curl -s https://www.cloudflarestatus.com/api/v2/status.json | jq '.status.indicator'

# Test from different Cloudflare edge locations
curl -H "CF-IPCountry: US" https://example.com
curl -H "CF-IPCountry: GB" https://example.com

# Debug Worker scripts
cloudflared wrangler tail --format pretty

# Check SSL mode
curl -H "CF-Visitor: {\"scheme\":\"https\"}" http://example.com
```

## Firewall and Proxy Issues

### Local Firewall Debugging
```bash
# Check iptables rules
sudo iptables -L -n -v
sudo iptables -L OUTPUT -n -v

# Check UFW status
sudo ufw status verbose

# Temporarily disable firewall (for testing)
sudo ufw disable
# Remember to re-enable: sudo ufw enable
```

### Proxy Detection and Testing
```bash
# Check environment proxy settings
echo $http_proxy
echo $https_proxy
echo $no_proxy

# Test with proxy
curl --proxy http://proxy.example.com:8080 https://example.com

# Test without proxy
curl --noproxy "*" https://example.com

# Detect proxy configuration
curl -I http://www.whatismyipaddress.com/

# Test SOCKS proxy
curl --socks5 proxy.example.com:1080 https://example.com
```

### Corporate Network Issues
```bash
# Test internal vs external connectivity
curl -I https://internal.company.com
curl -I https://google.com

# Check corporate proxy settings
cat /etc/environment | grep -i proxy
grep -i proxy ~/.bashrc ~/.zshrc

# Test with different ports
for port in 80 443 8080 8443; do
  nc -zv proxy.company.com $port
done
```

## Performance Debugging

### Network Performance Testing
```bash
# Bandwidth testing
wget -O /dev/null http://speedtest.wdc01.softlayer.com/downloads/test100.zip

# Using iperf3 (if available on both ends)
iperf3 -c speedtest.example.com

# MTU discovery
ping -M do -s 1472 example.com
tracepath example.com
```

### Latency and Timing Analysis
```bash
# Detailed timing with curl
curl -w "@curl-format.txt" -o /dev/null -s https://example.com

# Multiple requests for average timing
for i in {1..10}; do
  curl -w "%{time_total}\n" -o /dev/null -s https://example.com
done

# Using ab (Apache Bench) for load testing
ab -n 100 -c 10 https://example.com/

# Using wrk for modern load testing
wrk -t12 -c400 -d30s https://example.com/
```

## Common Error Codes

### HTTP Status Codes
- **4xx Client Errors:**
  - 400: Bad Request - Check request syntax
  - 401: Unauthorized - Check authentication
  - 403: Forbidden - Check permissions/firewall
  - 404: Not Found - Check URL/path
  - 429: Too Many Requests - Rate limiting

- **5xx Server Errors:**
  - 500: Internal Server Error - Server-side issue
  - 502: Bad Gateway - Proxy/load balancer issue
  - 503: Service Unavailable - Server overloaded/maintenance
  - 504: Gateway Timeout - Upstream server timeout

### curl Error Codes
```bash
# Common curl errors and solutions
# (6) Could not resolve host
# Solution: Check DNS configuration

# (7) Failed to connect to host
# Solution: Check firewall, proxy settings

# (28) Operation timed out
# Solution: Increase timeout, check network

# (35) SSL connect error
# Solution: Check SSL certificate, try -k flag for testing

# (52) Empty reply from server
# Solution: Check if service is running on correct port

# (56) Recv failure: Connection reset by peer
# Solution: Check firewall, rate limiting, or server issues
```

## Advanced Debugging Techniques

### Network Packet Analysis
```bash
# Capture packets with tcpdump
sudo tcpdump -i any host example.com

# Capture HTTP traffic
sudo tcpdump -A -i any port 80 or port 443

# Save capture for analysis
sudo tcpdump -w capture.pcap host example.com

# Analyze with Wireshark (if GUI available)
wireshark capture.pcap
```

### System-level Debugging
```bash
# Check system network configuration
ip addr show
ip route show

# Check listening ports
sudo netstat -tuln
sudo ss -tuln

# Monitor network connections
watch -n 1 'netstat -an | grep :80'

# Check system limits
ulimit -n  # File descriptor limit
cat /proc/sys/net/core/somaxconn  # Socket connection limit
```

### Debugging with strace
```bash
# Trace system calls for curl
strace -e trace=network curl https://example.com

# Trace file operations
strace -e trace=file curl https://example.com

# Full trace (verbose)
strace -f curl https://example.com 2>&1 | grep -E "(connect|sendto|recvfrom)"
```

### Load Balancer and CDN Debugging
```bash
# Check which server is responding
curl -I https://example.com | grep -i server

# Test direct server access (bypass CDN)
curl -I https://origin-server.example.com
curl -H "Host: example.com" https://origin-ip-address/

# Check CDN cache status
curl -I https://example.com | grep -i cache

# Force cache miss
curl -H "Cache-Control: no-cache" -H "Pragma: no-cache" https://example.com
```
