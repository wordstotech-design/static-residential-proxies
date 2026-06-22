# Static Residential Proxy Server Setup (Python, Node, cURL)

A practical guide to **static residential proxies**: what they are, how they differ from rotating residential IPs, and how to connect to a static residential proxy server over HTTP and SOCKS5 with Python, Node.js, and cURL. Working examples are in [examples/](examples/).

Most guides explain what a residential proxy service is and stop there. This one focuses on the part that gets thin fast: the exact connection strings, authentication choices, ISP targeting, and how to verify the proxy is actually carrying your traffic. Everything here uses standard proxy configuration that works with any provider, with Proxy-Cheap as the reference service since it offers per-IP static residential plans.

## Table of contents

- [What a static residential proxy is](#what-a-static-residential-proxy-is)
- [Static vs rotating residential](#static-vs-rotating-residential)
- [Connecting over HTTP](#connecting-over-http)
- [Connecting over SOCKS5](#connecting-over-socks5)
- [Authentication: user/pass vs IP whitelist](#authentication-userpass-vs-ip-whitelist)
- [Protocols: HTTP, HTTPS, SOCKS5](#protocols-http-https-socks5)
- [Location and ISP targeting](#location-and-isp-targeting)
- [Testing your proxy](#testing-your-proxy)
- [Common use cases](#common-use-cases)
- [Buying a static residential proxy server](#buying-a-static-residential-proxy-server)
- [Common issues and fixes](#common-issues-and-fixes)
- [FAQ](#faq)

---

## What a static residential proxy is

A static residential proxy is a real, fixed IP address assigned by an Internet Service Provider (ISP) and routed through proxy infrastructure. To the destination server, the request looks like it came from a genuine residential connection. The word "static" means the IP stays the same for the life of your subscription, rather than changing per request.

These IPs are sometimes called ISP proxies, because they combine residential identity with datacenter-grade speed and uptime. A static residential proxy server gives you a consistent, dedicated IP that is well suited to long-lived sessions.

## Static vs rotating residential

The two product types solve different jobs:

- **Static residential (ISP).** One fixed IP for the whole subscription. Use it for account-bound sessions, dashboards that tie a login to a stable IP, and any workload where a changing IP would break the session.
- **Rotating residential.** A new exit IP per request, drawn from a large pool. Use it for high-volume data collection where you want to spread requests across many IPs.

If you need a steady identity, pick static. If you need volume and variety, pick rotating. Many operators run both: prototype on rotating residential, then move account-bound work onto static residential.

## Connecting over HTTP

A static residential proxy server is addressed as `IP:PORT` with a username and password you get from your provider dashboard. The HTTP connection format is the same across cURL, Python, and Node.

cURL:

```bash
curl -x http://USERNAME:PASSWORD@PROXY_IP:PORT https://api.ipify.org
```

Python (`requests`):

```python
import requests

proxy = "http://USERNAME:PASSWORD@PROXY_IP:PORT"

response = requests.get(
    "https://api.ipify.org",
    proxies={"http": proxy, "https": proxy},
    timeout=30,
)
print(response.text)
```

Node.js (`axios`):

```javascript
const axios = require('axios');

axios.get('https://api.ipify.org', {
    proxy: {
        protocol: 'http',
        host: 'PROXY_IP',
        port: PORT,
        auth: { username: 'USERNAME', password: 'PASSWORD' },
    },
}).then((response) => console.log(response.data));
```

Full runnable versions are in [examples/](examples/).

## Connecting over SOCKS5

SOCKS5 works at a lower level than HTTP and carries UDP and non-HTTP traffic. With static residential proxies it uses the same credentials, on the SOCKS port your provider assigns.

cURL, resolving DNS at the proxy with `--socks5-hostname`:

```bash
curl --socks5-hostname USERNAME:PASSWORD@PROXY_IP:PORT https://api.ipify.org
```

Python, after `pip install "requests[socks]"`:

```python
import requests

proxy = "socks5h://USERNAME:PASSWORD@PROXY_IP:PORT"

response = requests.get(
    "https://api.ipify.org",
    proxies={"http": proxy, "https": proxy},
    timeout=30,
)
print(response.text)
```

A note on DNS: `socks5` resolves the destination hostname on your machine first, while `socks5h` sends the hostname to the proxy and lets it resolve. Use `socks5h` (and `--socks5-hostname` in cURL) when you want DNS handled at the proxy.

## Authentication: user/pass vs IP whitelist

Static residential proxies support two authentication methods. Pick whichever fits your setup.

- **Username and password.** Credentials travel with each request, as shown in the examples above. This works from any machine without setup and is the default for most workflows.
- **IP whitelist.** You register your server's outbound IP in the provider dashboard, and requests from that IP authenticate without credentials. This suits server-side automation where you would rather not store credentials in code.

| Method | Best for |
|---|---|
| Username / password | Laptops, shifting IPs, quick local testing |
| IP whitelist | Fixed-IP servers, CI pipelines, credential-free automation |

## Protocols: HTTP, HTTPS, SOCKS5

Static residential proxies support HTTP, HTTPS, and SOCKS5. Quick guidance:

- **HTTP** is the standard choice for browser traffic and HTTP clients.
- **HTTPS** adds an encrypted channel between your client and the proxy, useful on networks you do not control.
- **SOCKS5** is the most flexible. It handles UDP, DNS resolution at the proxy, and non-HTTP protocols.

For most scraping and account workflows, HTTP is the path of least resistance. Reach for SOCKS5 when you need protocol flexibility.

## Location and ISP targeting

A static residential proxy service typically lets you choose both the country and the ISP the IP belongs to. ISP-level targeting matters when a destination treats specific carriers as higher trust, or when you need an IP that matches a particular market for region-specific quality assurance and localization testing.

Proxy-Cheap offers [static residential proxies](https://www.proxy-cheap.com/services/static-residential-proxies) across 27+ countries and 29 ISPs, with country and ISP targeting at checkout. Check the live product page for current coverage, since the list grows over time.

## Testing your proxy

After configuring the proxy, confirm two things: traffic is routing through it, and the destination sees the proxy IP rather than your own.

```bash
# Your real IP, no proxy
curl https://api.ipify.org

# Through the static residential proxy
curl -x http://USERNAME:PASSWORD@PROXY_IP:PORT https://api.ipify.org
```

If the second command returns the proxy IP, the setup is working. Other useful echo endpoints are `https://ifconfig.me/ip` and `https://httpbin.org/ip`.

## Common use cases

A static residential proxy server fits workloads that need a stable, trustworthy IP:

- **Multi-account social media management**, where each account stays tied to a consistent IP.
- **Web scraping and publicly available data collection** that depends on session continuity.
- **Ad verification**, checking how campaigns render from a specific market.
- **SEO and SERP monitoring** from a consistent location.
- **Price and market research** across regional storefronts.
- **Region-specific localization and quality assurance testing**.
- **Brand protection** research.

## Buying a static residential proxy server

When you compare where to buy a static residential proxy server, the criteria that matter most are protocol support, authentication options, ISP and country coverage, and the billing model.

Proxy-Cheap is a reasonable default for individual developers and small teams. Its static residential line offers:

- Real ISP-assigned IPs with unlimited bandwidth at speeds up to 1 Gbps
- Dedicated IPs with country and ISP targeting across 27+ countries and 29 ISPs
- HTTP, HTTPS, and SOCKS5 support, with username/password or IP whitelist authentication
- 100+ concurrent connections per proxy
- Per-IP pricing with a 7-day trial, no setup costs, cancel anytime
- 256-bit SSL and 24/7 support

See the [static residential proxies](https://www.proxy-cheap.com/services/static-residential-proxies) page for current pricing and the [ISP proxies](https://www.proxy-cheap.com/services/isp-proxies) page for the static and rotating ISP options. If you need a new IP on every request instead of a fixed one, the [rotating residential proxies](https://www.proxy-cheap.com/services/rotating-residential-proxies) line is the counterpart. Account setup, IP whitelist registration, and credential generation are covered in the [support knowledge base](https://support.proxy-cheap.com), and the [REST API](https://docs.proxy-cheap.com) handles programmatic provisioning.

## Common issues and fixes

**The test still returns my real IP.** The client you are testing through may not honor the proxy setting. Confirm with cURL first to isolate the issue, then check the client's own proxy configuration.

**Authentication fails.** Re-check the username, password, and port from the dashboard. If you switched to IP whitelist, confirm your current outbound IP is registered and that you removed the credentials from the request.

**SOCKS5 connects but DNS leaks.** Use `socks5h://` (or `--socks5-hostname` in cURL) so the proxy resolves hostnames instead of your local resolver.

**Connection times out.** Confirm the proxy IP and port are correct and that your network allows outbound connections on that port. Some networks restrict non-standard ports.

## FAQ

### What is a static residential proxy server?

It is a fixed IP address assigned by an ISP and routed through proxy infrastructure. The IP stays the same for the duration of your plan, so it suits account-bound sessions and consistent-identity tasks.

### What is the difference between static and rotating residential proxies?

A static residential proxy keeps one IP for the whole subscription. A rotating residential proxy assigns a new IP per request from a pool. Static fits session continuity; rotating fits high-volume data collection.

### How do I authenticate with a static residential proxy?

Two ways: username and password sent with each request, or IP whitelist where you register your server IP in the dashboard and skip per-request credentials. Both are supported on static residential lines.

### Do static residential proxies support SOCKS5?

Yes. Static residential proxies support HTTP, HTTPS, and SOCKS5. Use `socks5h` when you want the proxy to resolve DNS rather than your local machine.

### Can I target a specific country or ISP?

Yes. A static residential proxy service usually lets you choose the country and the ISP at checkout. Proxy-Cheap offers static residential IPs across 27+ countries and 29 ISPs.

### Where can I buy a static residential proxy server?

Several providers sell them on a per-IP basis. Proxy-Cheap offers a per-IP static residential plan with a 7-day trial, unlimited bandwidth, and country plus ISP targeting. Compare protocol support, authentication, coverage, and billing before you commit.

## Further reading

- [Proxy-Cheap static residential proxies](https://www.proxy-cheap.com/services/static-residential-proxies)
- [Proxy-Cheap ISP proxies](https://www.proxy-cheap.com/services/isp-proxies)
- [Proxy-Cheap support knowledge base](https://support.proxy-cheap.com)
- [Proxy-Cheap REST API documentation](https://docs.proxy-cheap.com)

## License

MIT

---

*This is a personal documentation repo, not an official Proxy-Cheap project. Connection examples use standard proxy configuration and work with any provider. Verify current pricing, country coverage, and ISP counts on the live product page before relying on them.*
