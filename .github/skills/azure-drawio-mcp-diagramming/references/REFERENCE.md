# Draw.io Azure2 References

This folder contains reference artifacts for the `azure-drawio-mcp-diagramming` skill.

## Icon Discovery

The primary way to find Azure2 icons is the MCP tool:

- `drawio/search_shapes` — search 10,000+ draw.io shapes and return exact style strings.
- Example queries: `"azure virtual machine"`, `"azure key vault"`, `"azure load balancer"`.

Use the returned `style` value directly in the XML `mxCell`. Do not use unconfirmed icon paths.

## Reference Files

- `layout-antipatterns.md`
  - Worked examples of layout problems (stacked edges, repeated labels, observability inside VNet, etc.)
  - Derived from real diagram review sessions.
  - Use this as the first reference when a diagram looks cluttered or has overlapping lines/labels.

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
- Include a traffic legend only when it improves readability
- Add a network isolation explanation box only when the topology needs the extra explanation
- Use a larger canvas (1900x1500) to accommodate the multi-VNet topology
- Color-code different zones only when they represent real boundaries (perimeter/ingress in yellow,
  internal in green, management in blue, VNet Peering in grey, External Services in orange)
- Show VNet peering connections and external services in separate zones when they help explain the topology
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