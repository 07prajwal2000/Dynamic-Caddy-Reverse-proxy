# Using Caddy Server as a Reverse proxy and DNS based dynamic Upstream configuration

### Create A records in DNS server 
Values will be the IP address of the backend.

I am using `technitium/dns-server` DNS server docker image.

The backend servers are just busybox httpd apps serving simple HTML containing hostname and IP address of the machine.

The below command is used to start the httpd server.

```bash
sh -c 'mkdir /webapp; cd /webapp; echo \"<h2>Hostname: $(hostname)</h2> <br> <h2>IP Address: $(hostname -i)</h2>\">index.html;httpd -f -p 0.0.0.0:8080'
```
NsLookup is a CLI tool to quickly look into DNS records with specified DNS name.
```bash
nslookup -type=a webapps.local 127.0.0.11
# type is A record | DNS name to look for | DNS server IP address for querying
```

Caddy server supports Load balancing with StickyCookie/ConsistentHashing techniques. I used Sticky Cookie for this example, and caddy searches for `server` cookie and, the value contains hashed (HMAC SHA256) hex-string IP address with port of the server (backend).
## Sample Caddy file
```caddyfile
:7999 {
    reverse_proxy * {
        lb_policy cookie server {
            fallback first
        }
        dynamic a {
            name      webapps.bbapps.local
            port      8080
            refresh   10s
            resolvers dnsserver
            dial_timeout        5s
            dial_fallback_delay 5s
            versions ipv4
        }
    }
}
```

Here is a sample C# and Js code to generate hash
```csharp
// 1st parameter is the secret, 2nd is IP address of the backend
Convert.ToHexString(HMACSHA256.HashData(""u8, "192.168.192.3:8080"u8)).ToLower(); 
```

```javascript
const crypto = require('crypto');
crypto.createHmac('SHA256', "").update("192.168.192.3:8080").digest('hex');
//                           ^ is password | ^ is IP address of backend
```

## More Info
- [Caddy Docs for Dynamic Upstreams](https://caddyserver.com/docs/caddyfile/directives/reverse_proxy#dynamic-upstreams)
- [Caddy Docs for Load balancer](https://caddyserver.com/docs/caddyfile/directives/reverse_proxy#load-balancing)
- [Open-Source DNS server](https://github.com/TechnitiumSoftware/DnsServer)