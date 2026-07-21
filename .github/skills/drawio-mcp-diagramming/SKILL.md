---
name: drawio-mcp-diagramming
description: Create and edit architecture diagrams using the Draw.io MCP server. Supports both the hosted App Server (`drawio/create_diagram`) and the stdio Tool Server (`drawio/open_drawio_xml`, `drawio/open_drawio_mermaid`, `drawio/search_shapes`, etc.) with Azure2 and AWS4 icon rendering guidance. Supports XML, Mermaid, and CSV input. Uses `drawio/search_shapes` for icon discovery.
metadata:
  author: Thomas Thornton
  version: "1.1.0"
  last-updated: "2026-07-21"
---

# Draw.io MCP Diagramming Skill

Use this skill to create or update diagrams through the Draw.io MCP server and to avoid common Azure and AWS icon rendering problems.

See [references/REFERENCE.md](references/REFERENCE.md) for additional reference artifacts.

For diagrams that use only basic shapes (flowcharts, UML, ERD, org charts, mind maps, timelines, wireframes), skip icon discovery and proceed directly to `drawio/create_diagram` or `drawio/open_drawio_mermaid`.

## When to Use

- The user asks to create or refine architecture diagrams (Azure, AWS, multi-cloud, or generic).
- The user wants draw.io/diagrams.net output from an MCP workflow.
- The user asks for **Mermaid → draw.io** conversion.
- The user asks for **CSV → draw.io** conversion (org charts, flowcharts from tabular data).
- The user needs Azure service icons in diagrams.
- The user needs AWS service icons in diagrams.
- The user reports that Azure or AWS icons/shapes are not appearing.
- The user asks for an **auth or identity flow** (OAuth 2.0, OIDC, JWT validation, SSO, login, token exchange, Entra, Cognito).
- The user asks for an **API or microservice interaction diagram** (request/response chain, service-to-service calls, API gateway flow).
- The user asks for a **CI/CD pipeline or deployment workflow** (build, test, deploy stages, GitHub Actions, Azure DevOps, approval gates).
- The user wants to **edit an existing multi-page `.drawio` file** (Tool Server only).

## Required Tooling

Draw.io provides two MCP server variants. The skill works with either; call the tools that match the configured server.

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

- MCP tools: `drawio/open_drawio_xml`, `drawio/open_drawio_mermaid`, `drawio/open_drawio_csv`, `drawio/search_shapes`, `drawio/list_pages`, `drawio/get_page`, `drawio/set_page`
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

Supported inputs: XML, Mermaid, CSV.
Optional layout pass: `routing: "libavoid"` on `open_drawio_xml`.

> Use **Option A** if your host supports MCP Apps inline rendering (Claude.ai, Cursor ≥ 2.6) or if the "Open in draw.io" button workflow is acceptable. Use **Option B** for VS Code / GitHub Copilot or any standard MCP client.

### Icon discovery tool (both servers)

- MCP tool: `drawio/search_shapes` — search 10,000+ draw.io shapes and return ready-to-use style strings.

### Tool name detection

MCP hosts may register tools with a server prefix (e.g. `mcp_drawio-mcp-ap_create_diagram` and `mcp_drawio-mcp-ap_search_shapes`). If `tool_search` does not surface the drawio tools, inspect the available or deferred tools list and call the exact names shown there. Do not assume a tool is unavailable if it appears in the deferred list; use the exact registered name.

### VS Code / GitHub Copilot: run shape searches sequentially

In VS Code and GitHub Copilot, parallel tool calls are cancelled if the user sends a new message while they are in flight. Always run `drawio/search_shapes` calls **one at a time** — never in parallel batches.

### XML hard constraints and edge routing rules

The draw.io MCP server enforces strict XML rules and generated diagrams frequently suffer from overlapping arrows. Before generating any XML, read [references/xml-authoring-rules.md](references/xml-authoring-rules.md) which covers:

- **Hard constraints** — forbidden constructs that cause the server to return a render error (XML comments, duplicate IDs, self-closing geometry with children, unescaped characters)
- **Edge density rules** — max edges per node, fan-out consolidation, exit/entry connection points, cross-zone waypoints
- **Pre-generation edge checklist** — run before writing edge XML for any infrastructure diagram

## Recommended Workflow

1. **Identify the input format and diagram type**
   - For flowcharts, sequence diagrams, ERD, mind maps, Gantt, timelines, kanban: prefer **Mermaid** if the Tool Server is available, or use the App Server's `mermaid` parameter.
   - For org charts or flowcharts from tabular data: use **CSV** with the Tool Server (`drawio/open_drawio_csv`).
   - For diagrams with named services, vendor shapes, or pictorial icons: use **XML** (`drawio/create_diagram` or `drawio/open_drawio_xml`).

2. **Use `drawio/search_shapes` for any non-geometric shape** — it searches all 10,000+ shapes across every draw.io library and returns ready-to-use style strings.
   - Use it for cloud services (Azure, AWS, GCP), network equipment (Cisco, Juniper), container/orchestration tools (Kubernetes, Docker), brand logos (Slack, GitHub), IT infrastructure shapes, and any other named component.
   - Example queries: `"azure virtual machine"`, `"aws lambda"`, `"cisco router"`, `"kubernetes pod"`, `"slack"`, `"docker"`.
   - Use the returned style string directly in the XML cell — do not guess or fabricate style strings.
   - Skip `search_shapes` only for diagrams that use purely geometric shapes: rectangles, diamonds, circles, and arrows.

4. **When to use `search_shapes` vs skip it** — if a shape has a recognised name, brand, or product identity, always look it up via `search_shapes` first. Only skip it for standard geometric diagrams (flowcharts, UML, ERD, org charts, mind maps, timelines, wireframes) that need no pictorial icons. For sequence and flow diagrams, apply Sequence and Flow Diagram Patterns (see section below).

5. **For Azure infrastructure/network diagrams**: apply Professional Network Topology Patterns (see Azure section below):
   - Use larger canvas (1900x1500)
   - VNets with thick borders (strokeWidth=4)
   - Subnets with dashed borders (strokeWidth=2, dashPattern=8 8)
   - Position resources inside their subnets
   - Label all traffic flows with protocols/ports
   - Include network isolation explanation box

6. **For AWS infrastructure/network diagrams**: apply AWS Network Topology Patterns (see AWS section below):
   - Use larger canvas (1900x1500) for multi-VPC/account topologies
   - VPCs with thick borders (strokeWidth=4)
   - Subnets (public/private) with dashed borders (strokeWidth=2, dashPattern=8 8)
   - Position resources inside their respective subnets
   - Label all traffic flows with protocols/ports
   - Include security group / NACL notation

7. **Build the payload**
   - XML: valid `mxGraphModel` using verified icons/style strings.
   - Mermaid: valid Mermaid.js definition (App Server: pass as `mermaid`; Tool Server: use `drawio/open_drawio_mermaid`).
   - CSV: valid CSV content (Tool Server: use `drawio/open_drawio_csv`).

8. **Call the appropriate tool**
   - App Server: `drawio/create_diagram` with `xml` or `mermaid`.
   - Tool Server: `drawio/open_drawio_xml`, `drawio/open_drawio_mermaid`, or `drawio/open_drawio_csv`.

9. **Apply layout passes when useful**
   - App Server: `postLayout: "elk"` for a full re-layout (moves nodes + routes edges), or `routing: "libavoid"` to improve edge paths **while keeping your manual node positions**.
   - Tool Server: `routing: "libavoid"` on `open_drawio_xml`.
   - Do **not** combine `postLayout` and `routing` — ELK already routes its own edges.
   - `routing: "libavoid"` does **not** separate same-source stacked edges. If multiple edges leave the same node at the same point, assign `exitX`/`exitY` values first — the layout pass is a finishing step, not a substitute.

10. If user wants a file artifact, save as `.drawio` wrapped in `<mxfile><diagram>...</diagram></mxfile>`.

11. Keep labels concise and explicit (service name + role).

12. Prefer one icon per major component or service; use edges for flow semantics (ingress/egress/dependency/telemetry).

## Input Format Quick Reference

Choose the input that matches the diagram type and configured server.

| Input | Best for | App Server | Tool Server |
|---|---|---|---|
| **XML** | Architecture/topology diagrams with vendor or pictorial icons, custom layouts | `drawio/create_diagram` with `xml` | `drawio/open_drawio_xml` |
| **Mermaid** | Flowcharts, sequence, class, ER, state, mindmap, Gantt, timeline, kanban | `drawio/create_diagram` with `mermaid` | `drawio/open_drawio_mermaid` |
| **CSV** | Org charts, flowcharts, simple diagrams from tabular data | Not supported | `drawio/open_drawio_csv` |

Use Mermaid for standard diagram types; use XML when the user needs pictorial or vendor-specific icons, precise positioning, complex containers, or custom styling. See [references/REFERENCE.md](references/REFERENCE.md) for Mermaid/CSV examples and multi-page editing details.

## Visual Quality Guardrails

Apply these defaults unless the user explicitly asks for a dense/technical view:

- Use 3-4 major lanes/zones max (for example Source → Process → Destination).
- Keep primary flow left-to-right with a single main path.
- Use stage numbering (`1`, `2`, `3`, `4`) instead of many edge labels.
- Keep one icon per major component; avoid icon-per-step layouts.
- Limit cross-lane dashed lines to one security/auth line and one optional telemetry line.
- **Edge density**: nodes with 3+ outgoing edges must use `exitX`/`exitY` connection points to fan them out. Fan-out to same-tier targets (e.g. one gateway → 3 backends, same protocol) should use one aggregated edge not separate identical arrows. See [references/xml-authoring-rules.md](references/xml-authoring-rules.md) for exit-point values and consolidation patterns.
- Keep text concise (single purpose per box) and avoid multiline overload.
- **Animated flow on connectors**: adding `flowAnimation=1;` to any edge style renders a moving dot that travels along the arrow, making directional flow immediately visible without extra labels — ideal for data-flow and pipeline diagrams. The animation is preserved in SVG export and the draw.io desktop app. By default, ask the user whether they want any flow arrows animated before generating the diagram — *"Would you like any of the flow arrows animated to show traffic direction? If so, which ones?"* Apply `flowAnimation=1;` only to the edges the user identifies. If the user has already indicated they want a static/clean diagram, skip the question.
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
- **Label all traffic arrows** with protocols and ports, using this colour palette:
  - HTTPS:443 — **Azure blue** (`#0078D4`, thick solid) for internet ingress; prominent but professional
  - HTTP:80/8080/8090/8095 — **Teal** (`#00897B`, solid) for backend pool traffic; signals allowed/healthy east-west flow
  - PostgreSQL:5432 — **Indigo** (`#5C6BC0`, dashed) for database connections; purple/indigo conventionally marks the data tier
  - NFS/Gluster — **Green** (`#43A047`, solid) for shared storage flows
  - RBAC/Identity/SMTP — **Amber** (`#F57C00`, dashed) for management/control-plane traffic
  - Denied/Blocked (WAF, NSG deny rules) — **Red** (`#C62828`) — reserve red exclusively for blocked or denied traffic
- Use `edgeStyle=orthogonalEdgeStyle` for clean routing
- Include `<Array>` waypoints for complex routing
- **Direction animation on key edges**: `flowAnimation=1;` adds a moving dot along a connector arrow, making ingress paths, egress routes, and replication flows readable at a glance — the effect renders in SVG export and draw.io desktop and works on any edge style. Before generating the diagram, ask the user: *"Would you like any of the traffic arrows animated to show flow direction? If so, which ones?"* Apply `flowAnimation=1;` only to the edges they identify. Example style for an animated internet ingress arrow: `style="edgeStyle=orthogonalEdgeStyle;flowAnimation=1;strokeWidth=3;strokeColor=#0078D4;"`

### Essential Components

Include two annotation boxes in every Azure topology diagram:
1. **Network Isolation Explanation** (top-left, `fillColor=#fff9cc`) — visual conventions: VNet thick borders, subnet dashed borders, NSG/DNS notes
2. **Zone Separation** — VNet Peering zone (grey `fillColor=#f5f5f5`) and External Services zone (orange `fillColor=#ffe6cc`)

For a complete example, see [references/topology-patterns.md](references/topology-patterns.md).

### Professional Topology Checklist (Azure)
- [ ] VNets have thick borders (strokeWidth=4)
- [ ] Subnets have dashed borders (strokeWidth=2, dashPattern=8 8)
- [ ] All resources positioned inside their subnets
- [ ] Traffic arrows labelled with protocols and ports using the standard colour palette
- [ ] Network isolation explanation box included
- [ ] Color-coded zones for different purposes
- [ ] Canvas sized appropriately (1900x1500 for complex infra)
- [ ] VNet peering connections shown in separate zone
- [ ] External services grouped in separate zone
- [ ] Animation preference confirmed with user before generating (*"Would you like any flow arrows animated? If so, which ones?"*)

## Professional Network Topology Patterns (AWS Infrastructure)

When creating **AWS infrastructure network diagrams** with VPCs, subnets, and network isolation:

### Canvas Sizing
- Use larger canvas for complex infrastructure: `pageWidth="1900" pageHeight="1500"`
- Standard canvas may be too small for multi-VPC/multi-account topologies

### VPC and Subnet Visualization
- **VPCs**: Use thick borders (`strokeWidth=4`) and large containers
  - Production VPC: Green (`fillColor=#d5e8d4`, `strokeColor=#82b366`)
  - Development VPC: Blue (`fillColor=#dae8fc`, `strokeColor=#6c8ebf`)
  - Shared Services VPC: Yellow (`fillColor=#fff2cc`, `strokeColor=#d6b656`)
- **Subnets**: Use dashed borders (`strokeWidth=2`, `dashed=1`, `dashPattern=8 8`)
  - Public Subnets: Light green (`fillColor=#e6f4ea`, `strokeColor=#82b366`)
  - Private Subnets: Light blue (`fillColor=#EFF7FF`, `strokeColor=#6c8ebf`)
  - Isolated Subnets (databases): Light orange (`fillColor=#fff3e0`, `strokeColor=#e6821e`)
  - Position subnet containers **inside** VPC containers
  - Label with subnet name, AZ, and CIDR (e.g., "Public Subnet A - us-east-1a - 10.x.1.0/24")
- **Availability Zones**: Use light grey container inside VPC to group subnets per AZ

### Resource Positioning
- Position all resources **inside their respective subnet containers**
- EC2 instances, RDS, Lambda, etc. must be visually contained within their subnets
- Internet-facing resources (ALB, NAT Gateway, Bastion) go in **public subnets**
- Application servers / ECS tasks go in **private subnets**
- Databases (RDS, ElastiCache) go in **isolated subnets** with no outbound internet

### Traffic Flow Visualization
- **Label all traffic arrows** with protocols and ports, using the same colour palette as Azure:
  - HTTPS:443 *(internet ingress)* — **Azure blue** (`#0078D4`, thick solid) for external traffic entering via ALB/CloudFront
  - HTTP:80→HTTPS redirect — **Teal** (`#00897B`, solid) for healthy/redirected traffic
  - Port 5432/3306 — **Indigo** (`#5C6BC0`, dashed) for database connections
  - HTTPS:443 *(internal AWS service calls)* — **Green** (`#43A047`, solid) for traffic to VPC Endpoints and AWS-managed services (S3, SSM, Secrets Manager, etc.)
  - SSH:22 / SSM — **Amber** (`#F57C00`, dashed) for management / Bastion access
  - Denied/Blocked (WAF, Security Group deny rules) — **Red** (`#C62828`) — reserve red exclusively for blocked traffic
- Use `edgeStyle=orthogonalEdgeStyle` for clean routing
- Show NAT Gateway path for private subnet → internet egress
- **Direction animation on key edges**: `flowAnimation=1;` adds a moving dot along a connector arrow, making ingress paths, egress routes, and data-transfer flows readable at a glance — the effect renders in SVG export and draw.io desktop and can be applied to any edge style. Before generating the diagram, ask the user: *"Would you like any of the traffic arrows animated to show flow direction? If so, which ones?"* Apply `flowAnimation=1;` only to the edges they identify. Example style for an animated ingress path: `style="edgeStyle=orthogonalEdgeStyle;flowAnimation=1;strokeWidth=3;strokeColor=#0078D4;"`

### Essential Components

Include two annotation boxes in every AWS topology diagram:
1. **Network Isolation Explanation** (top-left) — visual conventions: VPC thick borders, subnet tiers (public/private/isolated), SG/NACL notes, VPC Endpoints
2. **Zone Separation** — Internet/Edge zone (orange), VPC Peering/Transit Gateway zone (grey), AWS Managed Services zone (purple)

For a complete example, see [references/topology-patterns.md](references/topology-patterns.md).

### Professional Topology Checklist (AWS)
- [ ] VPCs have thick borders (strokeWidth=4) and are colour-coded by environment
- [ ] Subnets have dashed borders (strokeWidth=2, dashPattern=8 8) and are colour-coded by tier (public/private/isolated)
- [ ] Availability Zone containers group subnets per AZ
- [ ] All resources positioned inside their respective subnets
- [ ] Internet Gateway and NAT Gateway shown for public/private egress
- [ ] Traffic arrows labelled with protocols and ports using the standard colour palette
- [ ] Security Group boundaries annotated where important
- [ ] Network isolation explanation box included
- [ ] Canvas sized appropriately (1900x1500 for complex infra)
- [ ] VPC Peering / Transit Gateway shown in separate zone
- [ ] Edge/internet services (CloudFront, Route53, WAF) in separate zone
- [ ] Animation preference confirmed with user before generating (*"Would you like any flow arrows animated? If so, which ones?"*)

## Sequence and Flow Diagram Patterns

Use this section for diagrams that show **temporal flows** — what happens in order — rather than infrastructure topology. No shape lookup via `drawio/search_shapes` is required.

### When to Apply

| Diagram type | Keywords | Layout |
|---|---|---|
| Auth / authorisation flow | OAuth, OIDC, JWT, SSO, login, token exchange, Entra, Cognito | Swimlane interaction flow |
| API / microservice call chain | REST, GraphQL, request/response, service-to-service, API gateway | Swimlane or vertical flowchart |
| CI/CD pipeline | pipeline, build, deploy, release, GitHub Actions, Azure DevOps, approval gate | Horizontal pipeline flowchart |

### Layout Approach

**Swimlane interaction flow** (auth / API flows with 2–5 actors):
- Represent each actor as a labelled header rectangle at the top, with a matching full-height light-coloured background column below it
- Steps flow top-to-bottom within each column; number them (`1.`, `2.`, `3.`) in the label so execution order is unambiguous
- All step boxes and edges live at `parent="1"` (root) — no nested swimlane cell geometry required
- Edges cross between columns with `edgeStyle=orthogonalEdgeStyle;`
- Canvas: `pageWidth="1400" pageHeight="900"` for 3 actors; add ~420 px width per additional actor

**Horizontal pipeline flowchart** (CI/CD):
- Stages flow left-to-right: Source → Build → Test → Staging → Approval → Production
- Use `rounded=1` rectangles for stages, `rhombus` shape for gate / decision points
- Colour-code each stage box using the Stage Colours table below
- Failure branch goes downward from the gate with a red edge to a Rollback/Notify step
- Canvas: `pageWidth="1700" pageHeight="600"`

### Colour Conventions

**Edge colours** (consistent with topology palette):

| Meaning | `strokeColor` | Style |
|---|---|---|
| Primary request / call | `#0078D4` Azure blue | solid, `strokeWidth=2` |
| Success response / return | `#00897B` Teal | solid, `strokeWidth=2` |
| Token / credential / redirect | `#F57C00` Amber | `dashed=1`, `strokeWidth=2` |
| Async / event-driven call | `#5C6BC0` Indigo | `dashed=1`, `strokeWidth=2` |
| Error / rejection / rollback | `#C62828` Red | solid, `strokeWidth=2` |
| Optional / conditional | `#666666` Grey | `dashed=1`, `strokeWidth=1` |

**Participant lane colours** (swimlane header + column background at `opacity=30`):

| Actor type | `fillColor` | `strokeColor` |
|---|---|---|
| User / browser / client | `#dae8fc` | `#6c8ebf` |
| Identity provider (Entra, Cognito, Okta) | `#e6f4ea` | `#82b366` |
| API / backend service | `#fff3e0` | `#e6821e` |
| Database / data store | `#f5f5f5` | `#666666` |
| Managed service / external system | `#f3e5f5` | `#7B1FA2` |

**Stage fill colours** (CI/CD pipeline):

| Stage | `fillColor` | `fontColor` |
|---|---|---|
| Source / Trigger | `#0078D4` | `#ffffff` |
| Build | `#00897B` | `#ffffff` |
| Test / Quality Gate | `#F57C00` | `#ffffff` |
| Deploy to Staging | `#5C6BC0` | `#ffffff` |
| Approval Gate | `#795548` | `#ffffff` |
| Deploy to Production | `#43A047` | `#ffffff` |
| Rollback / Failure | `#C62828` | `#ffffff` |

### Flow Animation

`flowAnimation=1;` works on sequence/flow edges exactly as in topology diagrams. Apply to primary call paths or pipeline stage transitions. Always ask the user before applying.

### Checklist (Sequence/Flow Diagrams)

- [ ] Diagram type identified (auth flow / API flow / CI/CD pipeline)
- [ ] Actors / participants labelled clearly
- [ ] Steps numbered in execution order
- [ ] Edge colours consistent with conventions above
- [ ] Error / failure paths shown in red
- [ ] Animation preference confirmed with user before generating
- [ ] Canvas sized appropriately for participant count and step depth

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

Azure icons come from **two separate libraries**. `drawio/search_shapes` will return icons from either; use the style string exactly as returned.

### azure2 library (most Azure services)

Path pattern: `image=img/lib/azure2/<category>/<Icon_Name>.svg`

- `shape=mxgraph.azure2.*` may not render in some hosts. Prefer the `image;aspect=fixed;...` style form.
- Some embedded viewers do not resolve `img/lib/azure2/...` consistently — test in `app.diagrams.net` if icons are missing.

### mscae library (Sentinel, DNS Private Zones, and some older icons)

Path pattern: `image=img/lib/mscae/<Icon_Name>.svg`

- Icons from this library **must include `sketch=0`** in their style. Without it the shape renders with a hand-drawn sketch effect.
- Correct style prefix: `image;sketch=0;aspect=fixed;html=1;points=[];align=center;...`
- Common mscae icons: `Azure_Sentinel.svg`, `DNS_Private_Zones.svg`.
- If `drawio/search_shapes` returns a style containing `sketch=0`, the icon is from mscae — preserve that attribute.

## Icon Reference Assets (AWS Diagrams)

This section applies only when the diagram includes AWS services/icons.

> **Important difference from Azure**: AWS4 icons in draw.io are **stencil-based**, not individual SVG files. They are typically referenced using `shape=mxgraph.aws4.<name>`. `drawio/search_shapes` returns the correct style string, including fill colours.

1. **Use `drawio/search_shapes`**
   - Search for the service by name and use the returned style string directly.
   - Example queries: `"aws lambda"`, `"aws s3"`, `"aws ec2"`, `"aws rds"`.
   - This is the upstream-recommended path and covers all 10,000+ shapes.

2. **Hard gate**
   - If a shape style string cannot be confirmed via `drawio/search_shapes`, do **not** use it.
   - Find an alternative via `drawio/search_shapes` first.

3. **Render review fallback**
   - If diagram review shows wrong/missing shape rendering, use `drawio/search_shapes` for alternative style strings.
   - Substitute and regenerate the diagram.

## AWS Icon Caveats (Important)

AWS4 icon rendering in draw.io can fail for two common reasons:

1. **Wrong style approach**
   - Do **not** use `image=img/lib/aws4/...` — AWS4 icons are **stencils**, not SVG files.
   - The correct style is: `shape=mxgraph.aws4.<name>;fillColor=#E7157B;fontColor=#ffffff;strokeColor=none;`
   - Fill colour conventions:
     - Compute (orange): `fillColor=#ED7100`
     - Storage (green): `fillColor=#3F8624`
     - Database (blue): `fillColor=#C7131F` (for Aurora/RDS use red)
     - Networking (purple): `fillColor=#8C4FFF`
     - Security (red): `fillColor=#DD344C`
     - Management (orange-red): `fillColor=#E7157B`
     - General/generic: `fillColor=#232F3E` (AWS dark)

2. **Library/environment mismatch**
   - Some embedded viewers may not load the `mxgraph.aws4` stencil library.
   - If shapes do not render in VS Code, test in `app.diagrams.net`.

## How to Discover Shapes

`drawio/search_shapes` searches all 10,000+ shapes across every draw.io library and returns ready-to-use style strings. Use it for **any** shape that has a name, brand, or product identity — not just cloud providers.

Example queries by category:

| Category | Example queries |
|---|---|
| Azure | `"azure virtual machine"`, `"azure key vault"`, `"azure api management"` |
| AWS | `"aws lambda"`, `"aws s3"`, `"aws ec2"` |
| GCP | `"gcp compute engine"`, `"gcp cloud storage"` |
| Network equipment | `"cisco router"`, `"cisco firewall"`, `"juniper switch"` |
| Containers / orchestration | `"kubernetes pod"`, `"docker"`, `"helm"` |
| Brands / SaaS | `"slack"`, `"github"`, `"jira"`, `"salesforce"` |
| On-premises / IT | `"server"`, `"database"`, `"laptop"`, `"printer"` |

Always use the returned `style` value directly on the `mxCell` — never guess or fabricate a style string.

The style format varies by library:

```text
# Image-based (Azure azure2, SVG files)
image;aspect=fixed;html=1;points=[];align=center;image=img/lib/azure2/<category>/<Name>.svg;

# Stencil-based (AWS4, shape library)
shape=mxgraph.aws4.<name>;fillColor=<color>;fontColor=#ffffff;strokeColor=none;

# Stencil-based (Cisco, Kubernetes, etc.)
shape=mxgraph.cisco.<category>.<name>;sketch=0;html=1;
```

## Fallback Strategy if Shapes Still Fail

If any shapes do not render correctly:

- Do **not** generate the diagram with an unresolved shape style.
- Use `drawio/search_shapes` to find alternative verified style strings.
- Return the list of unresolved shapes and propose verified replacements.
- After replacements validate to `OK`, then generate the diagram.

## Exporting Diagrams

| Format | How | Notes |
|---|---|---|
| **SVG** | `File → Export As → SVG` | Recommended — preserves `flowAnimation` moving-dot effects and all icon rendering. Use for sharing or embedding. |
| **PNG** | `File → Export As → PNG` | Static snapshot. `flowAnimation` effects are not captured; icons and colours are preserved. |
| **PDF** | `File → Export As → PDF` | Best for printed or document-embedded diagrams. Static only. |
| **.drawio file** | `File → Save As` | Preserves all XML, animation settings, and style attributes for future editing. |

> `flowAnimation=1` is only visible when the diagram is open in **draw.io desktop** or rendered as **SVG**. It does not appear in PNG or PDF exports — inform the user of this if they ask why the animation isn't showing.

## Troubleshooting Checklist

- Confirm the configured MCP server appears in `MCP: List Servers`.
- Run `MCP: Reset Cached Tools` if tool list is stale.
- **XML comments (`<!-- -->`) are forbidden** — the MCP server rejects them. Remove all comments before submitting.
- Ensure XML is otherwise well-formed (no malformed tags, no duplicate IDs, no unescaped `<`/`>`/`&` in style strings).
- **Z-order**: zone background rectangles must be defined **before** the icons they contain, or they will render on top of icons.
- **`html=1` in style** is required for any cell whose `value` contains HTML tags (`<b>`, `<br>`, `<i>`). Newlines via `&#xa;` work without it.
- **`sketch=0` in search results**: if `drawio/search_shapes` returns a style string containing `sketch=0`, preserve it exactly — omitting it enables the hand-drawn sketch rendering mode for that shape.
- **Icon sizes**: use dimensions as returned by `search_shapes`; they reflect the intended aspect ratio. When normalising a row of icons for visual consistency, 64×64 is a safe common size. Never change the aspect ratio of an icon that has `aspect=fixed` in its style.
- **Azure**: Verify style uses `image=img/lib/azure2/...` for Azure2 icon mode.
- **AWS**: Verify style uses `shape=mxgraph.aws4.<name>` for AWS4 icon mode (not `image=img/lib/aws4/...`).
- Reopen diagram in web draw.io if VS Code extension rendering differs.
- If an icon looks wrong, use `drawio/search_shapes` for an alternative exact style string.

## Prompt Templates and Checklists

See [references/REFERENCE.md](references/REFERENCE.md) for diagram-type prompt presets and [references/layout-antipatterns.md](references/layout-antipatterns.md) for the pre-flight layout checklist.

## Definition of Done

- The correct input format and MCP tool were chosen (XML, Mermaid, or CSV; App Server or Tool Server).
- All icon/style strings confirmed via `drawio/search_shapes` before generating; unconfirmed icons are not used.
- Diagram renders correctly; XML/Mermaid/CSV is valid and opens in draw.io.
- All named components identifiable via correct icons and clear labels.
- Layout passes applied appropriately (`postLayout: "elk"` and/or `routing: "libavoid"` for App Server; `routing: "libavoid"` for Tool Server `open_drawio_xml`).
- All applicable topology checklist items passed (borders, subnets, traffic labels, legend, isolation box, zones, canvas size).
- All applicable sequence/flow checklist items passed (numbered steps, colour-coded edges, error paths, canvas size).
- Animation preference confirmed; `flowAnimation=1;` applied only to user-identified edges.
- File artifact saved as `.drawio` (wrapped in `<mxfile>`) if requested.
- Edge density rules applied: nodes with 3+ edges use distinct exit/entry points; fan-out consolidated; cross-zone edges have waypoints. See [references/xml-authoring-rules.md](references/xml-authoring-rules.md).
- Layout anti-patterns checked against [references/layout-antipatterns.md](references/layout-antipatterns.md) before finalising.