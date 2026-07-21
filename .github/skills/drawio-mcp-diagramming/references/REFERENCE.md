# Draw.io Azure2 & AWS4 References

This folder contains reference artifacts for the `drawio-mcp-diagramming` skill.

## Icon Discovery

The primary way to find Azure2 and AWS4 icons is the MCP tool:

- `drawio/search_shapes` — search 10,000+ draw.io shapes and return exact style strings.
- Example Azure queries: `"azure virtual machine"`, `"azure key vault"`, `"azure load balancer"`.
- Example AWS queries: `"aws lambda"`, `"aws s3"`, `"aws ec2"`, `"aws rds"`.

Use the returned `style` value directly in the XML `mxCell`. Do not use unconfirmed icon paths.

## Reference Files

- `layout-antipatterns.md`
  - Worked examples of layout problems (stacked edges, repeated labels, observability inside VNet/VPC, etc.)
  - Derived from real diagram review sessions.
  - Use this as the first reference when a diagram looks cluttered or has overlapping lines/labels.

- `topology-patterns.md`
  - Complete `mxGraphModel` XML examples for Azure (VNet → Subnet → Resource) and AWS (VPC → Public/Private Subnets → ALB/ECS) topology diagrams.
  - Use when building or debugging a network topology diagram.

- `standalone-file-requirements.md`
  - Required XML attributes when writing a `.drawio` file directly (MCP tool unavailable): `as="geometry"` on every `<mxGeometry>`, and standard `mxGraphModel` layout attributes.
  - Includes a full minimal wrapper template.

## Notes

- Always confirm icon style strings via `drawio/search_shapes` before use.
- If render review shows bad/missing icons, use `drawio/search_shapes` for alternative paths and substitute.

## Example Prompt Templates

### Azure Network Topology Diagram (Infrastructure Focus)

```text
Create a professional Azure network topology diagram from my Terraform infrastructure
in the components/ folder, emphasizing network isolation and traffic flows.

Requirements:
- Show VNet architecture with clear network boundaries (use thick borders strokeWidth=4
  for VNets, dashed borders strokeWidth=2 dashPattern=8 8 for subnets)
- Position all resources (VMs, databases, load balancers, etc.) inside their
  respective subnets to show network isolation
- Label all traffic flows with protocols and ports (e.g., HTTPS:443,
  PostgreSQL:5432, HTTP:8080)
- Add a network isolation explanation box showing the visual conventions
- Use a larger canvas (1900x1500) to accommodate the multi-VNet topology
- Color-code different zones (DMZ VNet in yellow, Internal VNet in green,
  Management zone in blue, VNet Peering in grey, External Services in orange)
- Show VNet peering connections and external services in separate zones
- Use Azure2 icons from draw.io MCP

Focus on the networking aspects - how components are isolated, how traffic flows
between them, and what the network boundaries are.
```

### Basic Azure Architecture Diagram

```text
Use drawio/create_diagram to generate a hub-spoke Azure architecture diagram.
Use Azure2 image styles (image=img/lib/azure2/...) for all Azure resources.
Include [list services] and show ingress/egress/telemetry flows.
```

### AWS Network Topology Diagram (Infrastructure Focus)

```text
Create a professional AWS network topology diagram emphasising VPC design,
subnet tiers, and traffic flows.

Requirements:
- Show VPC architecture with clear network boundaries (use thick borders strokeWidth=4
  for VPCs, dashed borders strokeWidth=2 dashPattern=8 8 for subnets)
- Group subnets by Availability Zone using light grey AZ containers
- Colour-code subnet tiers: public (light green), private (light blue),
  isolated/database (light orange)
- Position all resources (EC2, RDS, Lambda, ALB, etc.) inside their respective subnets
- Show Internet Gateway and NAT Gateway for public/private subnet egress
- Label all traffic flows with protocols and ports (e.g., HTTPS:443,
  PostgreSQL:5432, SSH:22)
- Add a network isolation explanation box (VPC thick borders, subnet dashed borders,
  SG/NACL annotations, VPC Endpoints for private AWS service access)
- Use a larger canvas (1900x1500) for multi-VPC/multi-account topologies
- Separate internet/edge services zone (CloudFront, Route53, WAF, Shield)
  and VPC Peering / Transit Gateway zone
- Use AWS4 icons from draw.io MCP

Focus on the networking aspects - VPC isolation boundaries, AZ redundancy,
traffic routing, and security controls.
```

### Basic AWS Architecture Diagram

```text
Use drawio/create_diagram to generate a 3-tier AWS architecture diagram.
Use AWS4 image styles (image=img/lib/aws4/...) for all AWS resources.
Include [list services] and show ingress/egress/data flows.
```

### Multi-Cloud (Azure + AWS) Architecture Diagram

```text
Use drawio/create_diagram to generate a multi-cloud architecture diagram
showing both Azure and AWS components connected via [VPN/ExpressRoute/Direct Connect].

Use Azure2 image styles (image=img/lib/azure2/...) for Azure resources.
Use AWS4 image styles (image=img/lib/aws4/...) for AWS resources.
Show connectivity, data replication, and identity federation between the clouds.
```

## Diagram-Type Prompt Presets

Use these compact prompts to route the agent to the right input format and conventions.

### Flowchart

```text
Create a flowchart showing the decision flow for [process]. Use Mermaid if possible. Keep it left-to-right, label decision diamonds clearly, and use orthogonal edges.
```

### Sequence diagram

```text
Draw a sequence diagram for [interaction]. Show participants [A, B, C], label each message, and use activation bars. Use Mermaid.
```

### Entity Relationship diagram

```text
Generate an ER diagram from this SQL/schema: [schema]. Use Mermaid erDiagram syntax. Mark primary keys and foreign-key relationships.
```

### C4 model

```text
Create a C4 Container diagram for [system]. Show users, containers, databases, and external systems. Use official C4 shapes and keep the diagram at container scope only.
```

### Azure network topology

```text
Create a professional Azure network topology diagram for [description]. Use Azure2 icons, VNets with thick borders, subnets with dashed borders, position resources inside subnets, label traffic with protocols/ports, and include a traffic legend and network isolation explanation box.
```

### AWS network topology

```text
Create a professional AWS network topology diagram for [description]. Use AWS4 icons with correct fill colours, VPCs with thick borders, public/private/isolated subnets with dashed borders, and label all traffic flows.
```

### Cross-functional swimlane

```text
Create a cross-functional flowchart for [process] with swimlanes for [actors]. Number the steps, show decision points, and use the standard edge colour conventions.
```

## Input Format Details

### Mermaid

Mermaid is the fastest path for many standard diagram types. The draw.io server parses Mermaid natively and converts it to editable draw.io XML.

- Use Mermaid when the user asks for flowcharts, sequence diagrams, ER diagrams, class diagrams, state diagrams, mind maps, Gantt charts, timelines, or kanban boards.
- Use XML when the user needs official cloud icons, precise positioning, complex containers, or custom styling.

Example App Server call:

```json
{
  "mermaid": "flowchart LR\n  A[Start] --> B{Decision?}\n  B -->|Yes| C[Do thing]\n  B -->|No| D[Skip]"
}
```

Example Tool Server call:

```json
{
  "content": "flowchart LR\n  A[Start] --> B{Decision?}\n  B -->|Yes| C[Do thing]\n  B -->|No| D[Skip]"
}
```

> For complex flowcharts (≥ ~20 nodes, ≥ 3 decision diamonds, feedback edges, or ≥ 3 endpoints), the App Server's native Mermaid layout can become cramped. Add `postLayout: "elk"` to re-layout the result. The flow direction is taken from the Mermaid code (`TD/TB` vs `LR/RL`).

### CSV

CSV input is useful for org charts and simple labeled diagrams. The first row is the header; each subsequent row becomes a node. Edge relationships are typically expressed with `id` and `parent` columns. See the Tool Server documentation for the exact CSV schema.

### Multi-page `.drawio` files (Tool Server only)

Use these tools to inspect or edit one page of a multi-page `.drawio` file without rewriting the whole file:

- `drawio/list_pages` — list pages by index, id, name, and approximate size.
- `drawio/get_page` — retrieve the `mxGraphModel` XML for a single page.
- `drawio/set_page` — replace a single page's content.

Use these when the user says "update page 2 of my architecture diagram" or "add a new diagram page to this file".

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

## Known-Good AWS4 Icon Examples

AWS4 icons use stencil syntax: `shape=mxgraph.aws4.<name>`. Always confirm the exact style string via `drawio/search_shapes` before use.

```text
shape=mxgraph.aws4.ec2;fillColor=#ED7100;fontColor=#ffffff;strokeColor=none;
shape=mxgraph.aws4.lambda;fillColor=#ED7100;fontColor=#ffffff;strokeColor=none;
shape=mxgraph.aws4.elastic_container_service;fillColor=#ED7100;fontColor=#ffffff;strokeColor=none;
shape=mxgraph.aws4.elastic_kubernetes_service;fillColor=#ED7100;fontColor=#ffffff;strokeColor=none;
shape=mxgraph.aws4.application_load_balancer;fillColor=#8C4FFF;fontColor=#ffffff;strokeColor=none;
shape=mxgraph.aws4.cloudfront;fillColor=#8C4FFF;fontColor=#ffffff;strokeColor=none;
shape=mxgraph.aws4.route_53;fillColor=#8C4FFF;fontColor=#ffffff;strokeColor=none;
shape=mxgraph.aws4.vpc;fillColor=#8C4FFF;fontColor=#ffffff;strokeColor=none;
shape=mxgraph.aws4.transit_gateway;fillColor=#8C4FFF;fontColor=#ffffff;strokeColor=none;
shape=mxgraph.aws4.s3;fillColor=#3F8624;fontColor=#ffffff;strokeColor=none;
shape=mxgraph.aws4.efs;fillColor=#3F8624;fontColor=#ffffff;strokeColor=none;
shape=mxgraph.aws4.rds;fillColor=#C7131F;fontColor=#ffffff;strokeColor=none;
shape=mxgraph.aws4.dynamodb;fillColor=#C7131F;fontColor=#ffffff;strokeColor=none;
shape=mxgraph.aws4.elasticache;fillColor=#C7131F;fontColor=#ffffff;strokeColor=none;
shape=mxgraph.aws4.iam;fillColor=#DD344C;fontColor=#ffffff;strokeColor=none;
shape=mxgraph.aws4.key_management_service;fillColor=#DD344C;fontColor=#ffffff;strokeColor=none;
shape=mxgraph.aws4.waf;fillColor=#DD344C;fontColor=#ffffff;strokeColor=none;
shape=mxgraph.aws4.cognito;fillColor=#DD344C;fontColor=#ffffff;strokeColor=none;
shape=mxgraph.aws4.cloudwatch;fillColor=#E7157B;fontColor=#ffffff;strokeColor=none;
shape=mxgraph.aws4.cloudformation;fillColor=#E7157B;fontColor=#ffffff;strokeColor=none;
shape=mxgraph.aws4.cloudtrail;fillColor=#E7157B;fontColor=#ffffff;strokeColor=none;
shape=mxgraph.aws4.sqs;fillColor=#E7157B;fontColor=#ffffff;strokeColor=none;
shape=mxgraph.aws4.sns;fillColor=#E7157B;fontColor=#ffffff;strokeColor=none;
shape=mxgraph.aws4.eventbridge;fillColor=#E7157B;fontColor=#ffffff;strokeColor=none;
shape=mxgraph.aws4.api_gateway;fillColor=#8C4FFF;fontColor=#ffffff;strokeColor=none;
shape=mxgraph.aws4.codepipeline;fillColor=#C7131F;fontColor=#ffffff;strokeColor=none;
```

> **Note:** Shape names use underscores. If a shape does not render, use `drawio/search_shapes` for an alternative exact style string (e.g. query `"aws api gateway"`).