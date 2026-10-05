# Tunnel-Cloudflare

Prerequisites

Linux server

Cloudflare account

Domain managed by Cloudflare

cloudflared installed on the origin server

A web service running locally, for example http://localhost:80

1. Verify cloudflared Installation

First, verify that cloudflared is installed on the origin server:

cloudflared --version

<img width="1427" height="633" alt="image" src="https://github.com/user-attachments/assets/e4c6e962-650f-4cac-adfc-e68deb181c61" />

2. Authenticate cloudflared

Authenticate the server with your Cloudflare account:

cloudflared tunnel login

<img width="1906" height="437" alt="image" src="https://github.com/user-attachments/assets/eaeefcbe-ac07-47a9-b3ec-1f5a99141be1" />

The command will provide a URL. Open the URL in a browser to continue the authentication process.

Select the Cloudflare account that will be used for the tunnel and authorize the request.

<img width="1826" height="617" alt="image" src="https://github.com/user-attachments/assets/e2b21d9e-a285-4299-a127-5084f2050b1b" />

3. Create the Tunnel

Create a new tunnel using the following command:

cloudflared tunnel create test-tunnel

Replace test-tunnel with the name you want to use for your tunnel.

<img width="1901" height="225" alt="image" src="https://github.com/user-attachments/assets/91d3144d-4f24-409d-9299-611397070cfc" />

Verify that the tunnel was created successfully:

cloudflared tunnel list

<img width="1542" height="192" alt="image" src="https://github.com/user-attachments/assets/862c32b0-9c58-47ca-ad32-711c440617ca" />

<img width="1885" height="580" alt="image" src="https://github.com/user-attachments/assets/7ce36e6e-5277-401d-bb00-15b94fb83377" />


Important: Save the Tunnel ID. It will be required later when creating the tunnel configuration.

4. Create the DNS Route

Create the DNS route for the tunnel:

cloudflared tunnel route dns test-tunnel "website.IDZONE.sxplab.com"

Replace:

test-tunnel with your tunnel name.

IDZONE with the appropriate DNS zone identifier.

<img width="1891" height="122" alt="image" src="https://github.com/user-attachments/assets/6a41162b-9c4f-4fd0-8576-75e3d71989c8" />

This command creates the required DNS record and associates the hostname with the Cloudflare Tunnel.

5. Configure the Tunnel

Create the Cloudflare Tunnel configuration file:

cat << EOF > ~/.cloudflared/config.yaml
tunnel: $TUNNELID
credentials-file: /home/cloudflare/.cloudflared/$TUNNELID.json

warp-routing:
  enabled: true

ingress:
  - hostname: website.$DNSZONE
    service: http://localhost:80
  - service: http_status:404
EOF

Replace the following variables:

Variable

Description

$TUNNELID

Tunnel ID generated when the tunnel was created

$DNSZONE

DNS zone / hostname used for the tunnel



Configuration Example

The resulting configuration should look similar to:

tunnel: 12345678-1234-1234-1234-123456789abc
credentials-file: /home/cloudflare/.cloudflared/12345678-1234-1234-1234-123456789abc.json

warp-routing:
  enabled: true

ingress:
  - hostname: website.example.com
    service: http://localhost:80
  - service: http_status:404

Security note: Do not publish your actual Tunnel ID credentials file or sensitive Cloudflare credentials in a public repository.

<img width="1206" height="347" alt="image" src="https://github.com/user-attachments/assets/dea7d62c-74d2-48a8-95ea-0bca54994454" />


6. Verify the Configuration

Verify that the configuration file was created correctly:

cat ~/.cloudflared/config.yaml

<img width="1193" height="341" alt="image" src="https://github.com/user-attachments/assets/6b6386de-fa25-4678-9c9b-3adff6872b01" />



You can also validate the configuration before starting the tunnel:

cloudflared tunnel ingress validate

7. Start the Tunnel

Start the tunnel using:

cloudflared tunnel run test-tunnel

Replace test-tunnel with the name of your tunnel.



Once the tunnel is running, cloudflared establishes outbound connections to Cloudflare.

<img width="1890" height="667" alt="image" src="https://github.com/user-attachments/assets/d4c259b1-93f6-4a97-becb-ee94b5a69909" />

<img width="1517" height="527" alt="image" src="https://github.com/user-attachments/assets/779f7dae-9c82-4a54-9c7c-72124496391d" />



8. Verify the Application

Once the tunnel is running, access the configured hostname:

https://website.example.com

Traffic is forwarded through Cloudflare to the local service:

Internet
    │
    ▼
Cloudflare
    │
    │ Cloudflare Tunnel
    ▼
cloudflared
    │
    ▼
localhost:80
    │
    ▼
Web Application

Summary

The main workflow is:

1. Verify cloudflared
        ↓
2. Authenticate with Cloudflare
        ↓
3. Create the Tunnel
        ↓
4. Create the DNS route
        ↓
5. Configure config.yaml
        ↓
6. Validate configuration
        ↓
7. Start the Tunnel
        ↓
8. Test the application

This configuration allows the origin server to publish an internal web service through Cloudflare without requiring direct inbound exposure of the origin server.
