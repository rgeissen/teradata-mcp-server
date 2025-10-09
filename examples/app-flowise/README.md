# Flowise Example with Teradata MCP

Use this example to locally test Flowise and the Teradata MCP server with recommended defaults. 
Refer to the [Flowise client guide](./docs/client_guide/Flowise_with_teradata_mcp_Guide.md) for walkthrough and screenshots.

## Prerequisites
- Docker Engine with the compose plugin available locally
- Teradata database connection details.
- Optional: Enterprise Vector Store connection details: PEM certificate, access token and URI.

## Quick start
1. From the repo root run:
   ```bash
   cd examples/app-flowise
   cp env .env   
   ```

2. Optional: Edit and update the `.env` file with your preferred configuration details. 
   If you don't the Teradata connection details will be inherited from your current environment variables, at least the DATABASE_URI variable is required.

3. Launch the stack with docker compose (this will build MCP server image from `../Dockerfile`):
:
   ```bash
   export DATABASE_URI=teradata://username:password@host:1025  # Optional - ignore if you have already defined it in your .env file or current profile
   docker compose --env-file .env up  -d --remove-orphans
   ```
4. Optional: monitor the logs
   ```bash
   docker logs teradata-mcp-server -f
   docker logs flowise -f
   ```
   When the services are ready, both containers report `healthy`, 


5. Open Flowise at http://localhost:3000/.

6. Shut down the stack:
   ```bash
   docker compose down
   ```

## Customisation hints
- Edit `docker-compose.yaml` to change exposed ports, attach additional volumes, or swap container images.
- The `env` file controls MCP transport options, Teradata connection pooling, and Flowise ports; adjust to match your infrastructure.
