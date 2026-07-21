---
name: azure-drawio-mcp-diagramming
description: Create and edit Azure architecture diagrams using the Draw.io MCP server. Supports the hosted App Server (`drawio/create_diagram`) and the stdio Tool Server (`drawio/open_drawio_xml`, `drawio/open_drawio_mermaid`, `drawio/search_shapes`, etc.) with Azure2 icon rendering guidance. Uses `drawio/search_shapes` for icon discovery.
metadata:
  author: Thomas Thornton
  version: "1.1.0"
  last-updated: "2026-07-21"
---

# Azure Draw.io MCP Diagramming Skill

Use this skill to create or update Azure-focused diagrams through the Draw.io MCP server and to avoid common Azure icon rendering problems.

See [references/REFERENCE.md](references/REFERENCE.md) for Azure-specific reference artifacts.

For generic (non-Azure) diagrams, use the `drawio-mcp-diagramming` skill instead.

## When to Use

- The user asks to create or refine Azure architecture diagrams.
- The user wants draw.io/diagrams.net output from an MCP workflow.
- The user asks for **Mermaid → draw.io** conversion for an Azure diagram.
- The user needs Azure service icons in diagrams.
- The user reports that Azure icons/shapes are not appearing.
- The user wants to **edit an existing multi-page `.drawio` file** (Tool Server only).

## Required Tooling

This skill works with either Draw.io MCP server variant. Call the tools that match the configured server.

### Option A — Hosted App Server (inline / "Open in draw.io" button)

- MCP tool: `drawio/create_diagram`
- Workspace MCP config:

```json
{
  "servers": {
    "drawio": {
      "type": "http",
      "url": "https://mcp.draw.io/mcp"
    }
  }
}
```

Supported inputs: `xml` (draw.io XML), `mermaid` (Mermaid.js text).
Optional layout passes: `postLayout: "elk"`, `routing: "libavoid"`.

### Option B — stdio Tool Server (opens draw.io in browser)

- MCP tools: `drawio/open_drawio_xml`, `drawio/open_drawio_mermaid`, `drawio/search_shapes`, `drawio/list_pages`, `drawio/get_page`, `drawio/set_page`
- Workspace MCP config:

```json
{
  "servers": {
    "drawio": {
      "command": "npx",
      "args": ["-y", "@drawio/mcp"]
    }
  }
}
```

Supported inputs: XML, Mermaid.
Optional layout pass: `routing: "libavoid"` on `open_drawio_xml`.

> Use **Option A** if your host supports MCP Apps inline rendering (Claude.ai, Cursor ≥ 2.6) or if the "Open in draw.io" button workflow is acceptable. Use **Option B** for VS Code / GitHub Copilot or any standard MCP client.

### Azure icon discovery

- MCP tool: `drawio/search_shapes` — search 10,000+ draw.io shapes and return ready-to-use style strings.

## Recommended Workflow

1. **Identify the input format**
   - For simple Azure flowcharts, sequence diagrams, or ER diagrams: prefer **Mermaid** if the Tool Server is available.
   - For precise Azure architecture diagrams with official icons: use **XML**.

2. **Discover Azure icons with `drawio/search_shapes`**
   - Example queries: `"azure virtual machine"`, `"azure key vault"`, `"azure load balancer"`, `"azure cosmos db"`.
   - Use the returned style string directly in the XML cell.

3. **For Azure infrastructure/network diagrams**: apply Professional Network Topology Patterns (see section below):
   - Use larger canvas (1900x1500)
   - VNets with thick borders (strokeWidth=4)
   - Subnets with dashed borders (strokeWidth=2, dashPattern=8 8)
   - Position resources inside their subnets
   - Label all traffic flows with protocols/ports
   - Include traffic legend and network isolation explanation boxes

4. Build a valid `mxGraphModel` payload using verified icons/style strings.

5. **Call the appropriate tool**
   - App Server: `drawio/create_diagram` with `xml` or `mermaid`.
   - Tool Server: `drawio/open_drawio_xml` or `drawio/open_drawio_mermaid`.

6. **Apply layout passes when useful**
   - App Server: `postLayout: "elk"` for a full re-layout, or `routing: "libavoid"` to tidy connectors while keeping your positions.
   - Tool Server: `routing: "libavoid"` on `open_drawio_xml`.
   - Do **not** combine `postLayout` and `routing`.

7. If user wants a file artifact, save as `.drawio` wrapped in `<mxfile><diagram>...</diagram></mxfile>`.

8. Keep labels concise and explicit (service name + role).

9. For Azure diagrams, prefer one icon per major service and use edges for flow semantics (ingress/egress/peering/telemetry).

## Input Format Quick Reference

| Input | Best for | App Server | Tool Server |
|---|---|---|---|
| **XML** | Precise Azure architectures, network topology, custom layouts | `drawio/create_diagram` with `xml` | `drawio/open_drawio_xml` |
| **Mermaid** | Azure flowcharts, sequence, ER, state diagrams | `drawio/create_diagram` with `mermaid` | `drawio/open_drawio_mermaid` |

### Mermaid

Mermaid is the fastest path for standard Azure diagram types that do not require exact icon placement. The draw.io server parses Mermaid natively and converts it to editable draw.io XML.

- Use Mermaid for flowcharts, sequence diagrams, ER diagrams, and state diagrams.
- Use XML when the user needs official Azure icons, precise VNet/subnet positioning, or custom styling.

### Multi-page `.drawio` files (Tool Server only)

Use these tools to inspect or edit one page of a multi-page `.drawio` file without rewriting the whole file:

- `drawio/list_pages` — list pages by index, id, name, and approximate size.
- `drawio/get_page` — retrieve the `mxGraphModel` XML for a single page.
- `drawio/set_page` — replace a single page's content.

## Visual Quality Guardrails

Apply these defaults unless the user explicitly asks for a dense/technical view:

- Use 3-4 major lanes/zones max (for example Source, Pipeline, Azure target).
- Keep primary flow left-to-right with a single main path.
- Use stage numbering (`1`, `2`, `3`, `4`) instead of many edge labels.
- Keep one icon per major service; avoid icon-per-step layouts.
- Limit cross-lane dashed lines to one security/auth line and one optional telemetry line.
- Keep text concise (single purpose per box) and avoid multiline overload.
- Prefer a "clean" variant first; add detail only if requested.

For worked examples of common layout problems (stacked edges, repeated labels, observability inside VNet, etc.), see [references/layout-antipatterns.md](references/layout-antipatterns.md).

## Professional Network Topology Patterns (Azure Infrastructure)

When creating **Azure infrastructure network diagrams** with VNets, subnets, and network isolation:

### Canvas Sizing
- Use larger canvas for complex infrastructure: `pageWidth="1900" pageHeight="1500"`
- Standard canvas may be too small for multi-VNet topologies

### VNet and Subnet Visualization
- **VNets**: Use thick borders (`strokeWidth=4`) and large containers
  - DMZ VNet: Yellow (`fillColor=#fff2cc`, `strokeColor=#d6b656`)
  - Internal VNet: Green (`fillColor=#d5e8d4`, `strokeColor=#82b366`)
  - Management Zone: Blue (`fillColor=#dae8fc`, `strokeColor=#6c8ebf`)
- **Subnets**: Use dashed borders (`strokeWidth=2`, `dashed=1`, `dashPattern=8 8`)
  - Position subnet containers **inside** VNet containers
  - Use lighter shades of parent VNet color
  - Label with subnet name and CIDR (e.g., "Application Subnet - 10.x.2.0/24")
- **Delegated Subnets**: Add delegation info to label (e.g., "PostgreSQL Subnet - 10.x.4.0/24 (Delegated to Microsoft.DBforPostgreSQL/flexibleServers)")

### Resource Positioning
- Position all resources **inside their respective subnet containers**
- VMs, databases, load balancers must be visually contained within their subnets
- This clearly shows network isolation boundaries

### Traffic Flow Visualization
- **Label all traffic arrows** with protocols and ports:
  - HTTPS:443 (red thick arrows for internet ingress)
  - HTTP:8080/8090/8095 (gold arrows for backend pools)
  - PostgreSQL:5432 (blue dashed arrows for database connections)
  - NFS/Gluster (green arrows for shared storage)
  - RBAC/Identity/SMTP (orange dashed arrows for management/external)
- Use `edgeStyle=orthogonalEdgeStyle` for clean routing
- Include `<Array>` waypoints for complex routing

### Essential Components
1. **Traffic Legend Box** (bottom-left)
   - Show all 5 traffic types with color-coded arrows
   - Include protocol/port information
   - Use thick bordered white box (`strokeWidth=3`)

2. **Network Isolation Explanation Box** (top-left)
   - Explain visual conventions:
     - "VNets: Thick borders"
     - "Subnets: Dashed borders"
     - "PostgreSQL subnet delegated"
     - "NSGs control traffic"
     - "Private DNS for internal resolution"
   - Use yellow background (`fillColor=#fff9cc`)

3. **Zone Separation**
   - VNet Peering Zone: Grey box (`fillColor=#f5f5f5`, `strokeColor=#666666`)
   - External Services Zone: Orange box (`fillColor=#ffe6cc`, `strokeColor=#d79b00`)

### Complete Example Structure
```xml
<mxGraphModel pageWidth="1900" pageHeight="1500">
  <!-- VNet Container with thick border -->
  <mxCell id="vnet" value="Internal VNet - 10.x.0.0/16" 
    style="rounded=0;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;
    verticalAlign=top;fontSize=16;fontStyle=1;align=center;strokeWidth=4;">
    <mxGeometry x="220" y="580" width="1340" height="820"/>
  </mxCell>
  
  <!-- Subnet Container with dashed border inside VNet -->
  <mxCell id="subnet-app" value="Application Subnet - 10.x.2.0/24"
    style="rounded=1;whiteSpace=wrap;html=1;fillColor=#e6f4ea;strokeColor=#82b366;
    verticalAlign=top;fontSize=13;fontStyle=1;align=center;strokeWidth=2;dashed=1;dashPattern=8 8;">
    <mxGeometry x="260" y="650" width="480" height="340"/>
  </mxCell>
  
  <!-- Resources inside subnet -->
  <mxCell id="vm" style="image;aspect=fixed;html=1;points=[];align=center;
    image=img/lib/azure2/compute/Virtual_Machine.svg;">
    <mxGeometry x="300" y="720" width="64" height="59"/>
  </mxCell>
  
  <!-- Labeled traffic edge -->
  <mxCell id="edge" value="PostgreSQL:5432" 
    style="edgeStyle=orthogonalEdgeStyle;strokeWidth=2;strokeColor=#6c8ebf;dashed=1;"
    edge="1" source="vm" target="postgres">
    <mxGeometry relative="1"/>
  </mxCell>
</mxGraphModel>
```

### Professional Topology Checklist
- [ ] VNets have thick borders (strokeWidth=4)
- [ ] Subnets have dashed borders (strokeWidth=2, dashPattern=8 8)
- [ ] All resources positioned inside their subnets
- [ ] Traffic arrows labeled with protocols and ports
- [ ] Traffic legend box included
- [ ] Network isolation explanation box included
- [ ] Color-coded zones for different purposes
- [ ] Canvas sized appropriately (1900x1500 for complex infra)
- [ ] VNet peering connections shown in separate zone
- [ ] External services grouped in separate zone

## Icon Reference Assets (Azure Diagrams)

This section applies only when the diagram includes Azure services/icons.

1. **Use `drawio/search_shapes`**
   - Search for the service by name and use the returned style string directly.
   - Example queries: `"azure virtual machine"`, `"azure key vault"`, `"azure load balancer"`, `"azure cosmos db"`.
   - This is the upstream-recommended path and covers all 10,000+ shapes.

2. **Hard gate**
   - If an icon style string cannot be confirmed via `drawio/search_shapes`, do **not** use it.
   - Find an alternative via `drawio/search_shapes` first.

3. **Render review fallback**
   - If diagram review shows wrong/missing icon rendering, use `drawio/search_shapes` for alternative style strings.
   - Substitute and regenerate the diagram.

## Azure Icon Caveats (Important)

Azure icon rendering in draw.io can fail for two common reasons:

1. **Wrong style type**
   - `shape=mxgraph.azure2.*` may not render in some hosts.
   - Prefer Azure2 image style entries:
   - `image;aspect=fixed;html=1;...;image=img/lib/azure2/<category>/<Icon_Name>.svg;`

2. **Library/environment mismatch**
   - Some embedded viewers/extensions do not resolve `img/lib/azure2/...` consistently.
   - If icons do not render in one host, test in `app.diagrams.net`.

## How to Discover Icons

Use `drawio/search_shapes` to find Azure icons and get exact style strings.

- Example queries: `"azure virtual machine"`, `"azure key vault"`, `"azure load balancer"`, `"azure cosmos db"`.
- Use the returned `style` value directly on the `mxCell`.

A typical Azure2 image-style result looks like:

```text
image;aspect=fixed;html=1;points=[];align=center;image=img/lib/azure2/<category>/<Icon_Name>.svg;
```

For renderer resilience, absolute URLs also work:

```text
image;aspect=fixed;html=1;...;image=https://raw.githubusercontent.com/jgraph/drawio/dev/src/main/webapp/img/lib/azure2/networking/Application_Gateways.svg;
```

If local rendering still fails, open in `app.diagrams.net`.

## Known-Good Azure2 Icon Examples

```text
image=img/lib/azure2/networking/Front_Doors.svg
image=img/lib/azure2/networking/Private_Link_Hub.svg
image=img/lib/azure2/networking/Network_Watcher.svg
image=img/lib/azure2/app_services/API_Management_Services.svg
image=img/lib/azure2/app_services/App_Services.svg
image=img/lib/azure2/databases/Azure_Cosmos_DB.svg
image=img/lib/azure2/identity/Managed_Identities.svg
image=img/lib/azure2/management_governance/Policy.svg
image=img/lib/azure2/analytics/Log_Analytics_Workspaces.svg
image=img/lib/azure2/management_governance/Monitor.svg
image=img/lib/azure2/devops/Application_Insights.svg
image=img/lib/azure2/devops/API_Connections.svg
```

## Fallback Strategy if Icons Still Fail

If Azure icons still do not render:

- Do **not** generate the Azure diagram with an unresolved icon set.
- Use `drawio/search_shapes` to find alternative exact style strings.
- Return the missing icon list and propose verified replacements.
- After replacements validate to `OK`, then generate the diagram.

## Troubleshooting Checklist

- Confirm the configured MCP server appears in `MCP: List Servers`.
- Run `MCP: Reset Cached Tools` if tool list is stale.
- Ensure XML is well-formed (no malformed tags or invalid comments).
- Verify style uses `image=img/lib/azure2/...` for Azure2 icon mode.
- Reopen diagram in web draw.io if VS Code extension rendering differs.
- If an icon looks wrong, use `drawio/search_shapes` for an alternative exact style string.

## Pre-Flight Layout Checklist

Before finalising any Azure diagram, run through these checks:

- [ ] No overlapping nodes or labels
- [ ] No edges passing through unrelated shapes (use `routing: "libavoid"` if needed)
- [ ] Labels are readable and not clipped
- [ ] One icon per major Azure service; no icon-per-step clutter
- [ ] Edge colours consistently encode meaning (HTTPS, HTTP, database, management, blocked)
- [ ] VNet/subnet containers clearly group related resources
- [ ] Canvas size fits content without excessive whitespace
- [ ] Traffic legend box and network isolation explanation box present for topology diagrams
- [ ] All traffic flows labelled with protocols and ports
- [ ] Animation (`flowAnimation=1;`) only applied where the user requested it

## Prompt Template for Agents

See [references/REFERENCE.md](references/REFERENCE.md) for full example prompt templates.

## Definition of Done

- The correct input format and MCP tool were chosen (XML or Mermaid; App Server or Tool Server).
- For Azure diagrams: all icon/style strings confirmed via `drawio/search_shapes` before generating.
- If render issues found, alternative icon style strings sourced via `drawio/search_shapes` and substituted.
- Diagram generated with confirmed icon paths only.
- XML/Mermaid is valid and opens in draw.io.
- Azure resources are identifiable (icons and clear service labels).
- Layout passes applied appropriately (`postLayout: "elk"` and/or `routing: "libavoid"` for App Server; `routing: "libavoid"` for Tool Server `open_drawio_xml`).
- **For Azure infrastructure/network diagrams**:
  - VNets use thick borders (strokeWidth=4) and are color-coded
  - Subnets use dashed borders (strokeWidth=2, dashPattern=8 8)
  - All resources positioned inside their respective subnets
  - All traffic flows labeled with protocols and ports
  - Traffic legend box included (bottom-left)
  - Network isolation explanation box included (top-left)
  - Canvas appropriately sized (1900x1500 for complex topologies)
  - VNet peering and external services in separate zones
- Layout anti-patterns checked against [references/layout-antipatterns.md](references/layout-antipatterns.md) before finalising