# Kong Workspaces for KongAir

This document explains how the KongAir project has been refactored to use Kong workspaces.

## Workspace Structure

The project now uses separate Kong workspaces for each team:

1. **flight-data** - Contains the Flights API and Routes API services
2. **sales** - Contains the Bookings API and Customer API services
3. **experience** - Contains the Experience API service

The platform team acts as a supervisor and doesn't have its own workspace. Instead, platform configurations (consumers, plugins, vaults) are merged into each team's workspace.

## How It Works

The implementation uses two GitHub workflows:

1. **stage-changes-for-kong-with-workspaces.yaml**
   - Converts OpenAPI specs to Kong configurations
   - Generates platform base configuration (shared across all workspaces)
   - Merges platform configuration with each team's configuration
   - Creates a PR with the changes

2. **deploy-kong-workspaces.yaml**
   - First creates the workspaces in Kong
   - Then deploys each workspace configuration separately
   - Uses workspace-specific admin tokens for security

## Required Secrets

To deploy the workspaces, you need to set up the following GitHub secrets:

- `KONG_ADMIN_URL`: The Kong Admin API URL
- `KONG_ADMIN_TOKEN`: Admin token with permissions to create workspaces
- `FLIGHT_DATA_ADMIN_TOKEN`: Admin token for the flight-data workspace
- `SALES_ADMIN_TOKEN`: Admin token for the sales workspace
- `EXPERIENCE_ADMIN_TOKEN`: Admin token for the experience workspace

## Local Testing

To test the workspace configuration locally:

```bash
# Create workspaces
deck sync --kong-addr http://localhost:8001 platform/kong/.generated/workspaces.yaml

# Deploy flight-data workspace
deck sync --kong-addr http://localhost:8001 --workspace flight-data platform/kong/.generated/workspaces/flight-data.yaml

# Deploy sales workspace
deck sync --kong-addr http://localhost:8001 --workspace sales platform/kong/.generated/workspaces/sales.yaml

# Deploy experience workspace
deck sync --kong-addr http://localhost:8001 --workspace experience platform/kong/.generated/workspaces/experience.yaml
```

## Benefits of This Approach

1. **Consistent Platform Configuration**: Platform team's configurations are applied consistently across all workspaces
2. **Security Isolation**: Each team uses their own admin token
3. **Independent Deployments**: Teams can deploy changes to their workspace without affecting others
4. **Simplified Management**: Teams only need to manage their own services and routes
5. **Clear Ownership**: Each API belongs to a specific team/workspace