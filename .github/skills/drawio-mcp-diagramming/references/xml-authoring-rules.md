# XML Authoring Rules for draw.io MCP

Hard constraints and edge-routing best practices for writing `mxGraphModel` XML that renders correctly in the draw.io MCP server (both App Server and Tool Server variants).

---

## Hard Constraints

Violating any of these causes render errors or silent failures from the MCP server.

| Constraint | Detail |
|---|---|
| **No XML comments** | `<!-- ... -->` anywhere in the XML causes the server to reject the payload with an explicit error. Remove all comments before submitting. |
| **Unique `id` values** | Every `mxCell` must have a globally unique `id` within the document. Duplicate IDs cause unpredictable rendering. |
| **Valid edge references** | Edge cells must use `edge="1"` and reference `source` and `target` IDs that exist in the same document. |
| **No self-closing geometry with children** | If a `<mxGeometry>` needs child elements (e.g. an `<Array>` waypoint), write it as an open/close pair — not self-closing. |
| **No unescaped XML characters in styles** | Style strings must not contain raw `<`, `>`, or `&`. Use `&lt;`, `&gt;`, `&amp;` if needed. |
| **`html=1` required for HTML labels** | Cell `value` attributes that use `<b>`, `<br>`, `<i>`, or any HTML tag must have `html=1` in the cell style. Without it the tags render as literal text. Newlines using `&#xa;` work without `html=1`. |

---

## Z-Order: Define Zone Backgrounds Before Icons

The draw.io MCP server renders cells in document order — cells defined later appear on top. Always define background zone rectangles (VNet containers, swimlane columns, legend boxes) **before** the service icons and edges they contain. If an icon is defined before its parent zone rectangle, the rectangle will render on top and cover the icon.

---

## VS Code / GitHub Copilot: Sequential Shape Searches

In VS Code and GitHub Copilot, **parallel tool calls are cancelled if the user sends a new message** while they are in flight.

Always run `drawio/search_shapes` calls **one at a time** (sequentially) — never in parallel batches — to avoid losing results mid-search.

---

## Icon Size Normalisation

`drawio/search_shapes` returns varying default dimensions (e.g. 65×60, 68×68, 64×64). Use these default sizes as returned — they reflect the icon's intended aspect ratio. When normalising a row of icons for visual consistency, 64×64 is a common safe size. Never scale an icon that has `aspect=fixed` in its style to a size that changes the aspect ratio.

---

## `routing: "libavoid"` Does Not Fix Stacked Same-Source Edges

`routing: "libavoid"` reroutes edges around obstacles and node shapes. It does **not** separate two edges that both originate from the same point on the same node. If multiple edges leave a node without distinct `exitX`/`exitY` values, they will stack on top of each other regardless of `routing: "libavoid"`.

**Rule**: Always assign `exitX`/`exitY` (and `entryX`/`entryY`) before relying on a layout pass to clean up routing. The layout pass is a finishing step, not a substitute for correct connection-point assignment.

`postLayout: "elk"` fully re-lays out the entire graph (moves nodes and routes edges). Use it when you want the engine to arrange everything. Use `routing: "libavoid"` only when you want to keep your manual node positions and just improve edge paths.

---

## Edge Density and Routing Rules

Overlapping arrows are the most common visual quality problem in generated diagrams. Apply these rules before writing any edge XML.

### Edge count limits

| Situation | Rule |
|---|---|
| A node is the source of 3+ edges going to **different zones** | Assign a distinct `exitX`/`exitY` to each edge. Never let all edges leave from the same connection point. |
| Fan-out to multiple **same-tier targets** (e.g. API gateway → 3 backends, same protocol) | Use **one aggregated edge** labelled e.g. `"HTTP:80 → backends"` instead of 3 identical arrows. |
| Observability (diagnostic logs from multiple services to Log Analytics / CloudWatch) | Draw **one representative edge** from the most significant source, labelled `"Diagnostic settings (all services)"`. Max 2 observability edges in any diagram. |
| Security / identity pattern (Managed Identity → Key Vault, or IAM Role → Secrets Manager, from multiple services) | Draw **one amber dashed edge** from the primary service. Not one per service instance. |

### Exit and entry connection points

Use `exitX`, `exitY`, `entryX`, `entryY` on any node with more than two edges. Spread exit positions so edges leave from different faces:

| Face | Style fragment |
|---|---|
| Left centre | `exitX=0;exitY=0.5;exitDx=0;exitDy=0;` |
| Right centre | `exitX=1;exitY=0.5;exitDx=0;exitDy=0;` |
| Top centre | `exitX=0.5;exitY=0;exitDx=0;exitDy=0;` |
| Bottom centre | `exitX=0.5;exitY=1;exitDx=0;exitDy=0;` |
| Bottom left | `exitX=0.25;exitY=1;exitDx=0;exitDy=0;` |
| Bottom right | `exitX=0.75;exitY=1;exitDx=0;exitDy=0;` |

Example — API Management fanning out to three backends with staggered right-side exits:

```xml
<mxCell id="e-apim-svc1" value="HTTP:80"
  style="edgeStyle=orthogonalEdgeStyle;strokeColor=#00897B;strokeWidth=2;
         exitX=1;exitY=0.3;exitDx=0;exitDy=0;entryX=0;entryY=0.5;entryDx=0;entryDy=0;"
  edge="1" source="apim" target="appsvc" parent="1">
  <mxGeometry relative="1" as="geometry"/>
</mxCell>
<mxCell id="e-apim-svc2" value="HTTP:80"
  style="edgeStyle=orthogonalEdgeStyle;strokeColor=#00897B;strokeWidth=2;
         exitX=1;exitY=0.5;exitDx=0;exitDy=0;entryX=0;entryY=0.5;entryDx=0;entryDy=0;"
  edge="1" source="apim" target="functions" parent="1">
  <mxGeometry relative="1" as="geometry"/>
</mxCell>
<mxCell id="e-apim-svc3" value="HTTP:80"
  style="edgeStyle=orthogonalEdgeStyle;strokeColor=#00897B;strokeWidth=2;
         exitX=1;exitY=0.7;exitDx=0;exitDy=0;entryX=0;entryY=0.5;entryDx=0;entryDy=0;"
  edge="1" source="apim" target="aks" parent="1">
  <mxGeometry relative="1" as="geometry"/>
</mxCell>
```

### Waypoints for cross-zone edges

When an edge travels from inside a network zone (e.g. VNet subnet, VPC) down to a zone below (Security or Observability plane), add an explicit `<Array>` waypoint to route it cleanly without crossing unrelated nodes.

Set the waypoint `x` to the horizontal midpoint of the source node; set `y` to just below the bottom edge of its parent zone rectangle.

```xml
<mxCell id="e-apim-kv" value="HTTPS:443 (Managed Identity)"
  style="edgeStyle=orthogonalEdgeStyle;strokeColor=#F57C00;strokeWidth=2;dashed=1;
         exitX=0.5;exitY=1;exitDx=0;exitDy=0;"
  edge="1" source="apim" target="keyvault" parent="1">
  <mxGeometry relative="1" as="geometry">
    <Array as="points">
      <mxPoint x="622" y="990"/>
    </Array>
  </mxGeometry>
</mxCell>
```

### Pre-generation edge checklist

Run this check before writing edge XML for any infrastructure diagram:

- [ ] Count edges per source node — nodes with 3+ edges must use distinct `exitX`/`exitY` values
- [ ] Fan-out patterns (same edge type, multiple same-tier targets) → consolidated to one labelled edge
- [ ] Cross-zone edges (VNet/VPC → Security or Observability plane) → `<Array>` waypoints added
- [ ] No two edges share the same label leaving the same source node
- [ ] Observability telemetry: ≤ 2 edges total across the whole diagram
- [ ] Security/identity pattern: ≤ 1 amber dashed edge per service tier
