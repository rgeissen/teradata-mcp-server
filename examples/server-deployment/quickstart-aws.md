# Teradata MCP Server on AWS EC2 — **Basic Deployment** 

> Goal: spin up the **Teradata MCP Server** on a small EC2 instance so you can connect it using **Claude Desktop** or your prefered desktop application over HTTP.  
> DO NOT USE AS IS FOR PRODUCTION DEPLOYMENT

---

## 1) EC2 sizing & OS

You can run the Teradata MCP server on a relatively small instance using a Linux OS.

| Size | vCPU | RAM | Why |
|---|---:|---:|---|
| **t3a.medium** *(recommended)* | 2 | 4 GiB | Great cost/perf for small teams (≈5–15 users). |
| t3a.large | 2 | 8 GiB | If you want to co-locate heavier front-end applications (eg. Flowwise, n8n, your chat bots...). |
| t4g.medium *(ARM)* | 2 | 4 GiB | Cheaper/faster if all deps are arm64; otherwise stick to x86 if unsure / planning to experiment. |

- **AMI**: Amazon Linux 2023 (or Ubuntu 22.04).  
- **Storage**: 30 GiB gp3 EBS.  (default)
- **Network**: (default)

---

## 2) Open the port & bind correctly

You will need to allow inbound traffic using ssh to configure your system and TCP for the end-users

- In the AWS Console, select your instance > Security > **Security Group > Edit inbound rules**:  
  - Inbound: TCP, type SSH, port **22** (default), 
  - Inbound: TCP, type Custom TCP, port **8001** (source = your end user IP list/range).  

---

## 3) Software Install

Install `uv` to install the MCP server

```bash
# On Amazon Linux
sudo dnf update -y
curl -LsSf https://astral.sh/uv/install.sh | sh
```
Install the MCP server

```bash
uv tool install teradata-mcp-server
```

## 4) Configure the server

You can create a configuration file to store your server settings:

```bash
cat > .env <<'EOF'
# --- Required: DB connection ---
DATABASE_URI=teradata://<USER>:<PASS>@<HOST>:1025/<DB_OR_USER>

# --- Server (Streamable HTTP) ---
MCP_TRANSPORT=streamable-http
MCP_HOST=0.0.0.0
MCP_PORT=8001

EOF
```

Edit the database connection string as with the Teradata system and user credentials for the MCP server. 

## 5) Run the server

You may now run the server as a background process:

```bash
nohup teradata-mcp-server > mcp.log &
```
After a few seconds, validate that the server is started: `tail -50 mcp.log`


## 6) Test the server connectivity

You can test the connectivity to the server from your clients servers/workstations with curl: `curl -I http://<EC2_PUBLIC_IP>:8001/mcp`.

If your MCP service is reachable, you should see this type of output:

```
HTTP/1.1 307 Temporary Redirect
date: Tue, 14 Oct 2025 15:39:58 GMT
server: uvicorn
...
```

 If this returns a `Failed to connect to ...: Connection refused`, it is likely an issue in your network configuration.

## 7) Configure the clients

For Visual Studio Code, follow the usual [http setup instructions](../../docs/client_guide/Visual_Studio_Code.md#Add-the-http-server-in-VS-Code).

For Claude desktop, you can use mcp-remote connect to your server.

```json
{
  "mcpServers": {
    "teradata_mcp_remote_aws": {
      "command": "npx",
      "args": [
        "mcp-remote",
        "http://<EC2_PUBLIC_IP>:8001/mcp/",
        "--allow-http"
      ]
    }
  }
}
```

If you have enabled basic authentication and setup a database proxy user, you may pass the database user credentials in the header:

```json
{
  "mcpServers": {
    "teradata_mcp_remote_aws": {
      "command": "npx",
      "args": [
        "mcp-remote",
        "http://<EC2_PUBLIC_IP>:8001/mcp/",
        "--header",
        "Authorization: Basic ${AUTH_TOKEN}",
        "--allow-http"
      ],
      "env": {
        "AUTH_TOKEN": "<YOUR_DB_USER:PASSWORD_u64>" 
      }
    }
  }
}
```