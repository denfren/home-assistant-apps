# HAProxy

HAProxy as TLS termination for home-assistant and other services.

## Usage

- place `fullchain.pem` and `privkey.pem` (names configureable) at `/ssl`. the container merges them so that they are available inside the container at `/data/combined.pem`
- create your HAProxy configuration at `/addon_configs/b3c03e35_haproxy/haproxy.cfg`
- configure home-assistant to trust the proxy (see below)

Example configuration:

```cfg
global
    # make logs visible in homeassistant
    log stdout format raw local0
    maxconn 1024

defaults
    log global
    mode http
    option httplog
    option dontlog-normal
    timeout connect 5s
    timeout client 30s
    timeout server 10s

frontend main
    bind *:80
    bind *:443 ssl crt /data/combined.pem

    # redirect to ssl
    http-request redirect scheme https unless { ssl_fc }

    default_backend homeassistant

backend homeassistant
    server ha homeassistant:8123
```

### Home Assistant configuration

Add the following to your `configuration.yaml`:

```yaml
http:
  use_x_forwarded_for: true
  trusted_proxies:
    - 172.30.33.0/24
```

Remove any existing `ssl_certificate`, `ssl_key`, or `server_port` settings from the http section.

## Certificate handling

The add-on automatically:
- Combines your certificate and key into a single PEM file for HAProxy whenever any of the source files change
- safely reloads haproxy service when config was edited. See log for errors

## Troubleshooting

### 400 Bad Request
Ensure `trusted_proxies` is configured in your Home Assistant `configuration.yaml`.

### Connection refused
Check that your `haproxy.cfg` exists at `/addon_configs/b3c03e35_haproxy/haproxy.cfg`.

### Certificate errors
Verify your certificate files exist in `/ssl` and match the configured filenames.

