# Topology Pattern XML Examples

Complete example `mxGraphModel` XML for Azure and AWS infrastructure topology diagrams.

Arrow routing default: use `edgeStyle=orthogonalEdgeStyle` and apply `routing: "libavoid"` for hand-placed layouts (or `postLayout: "elk"` when you want full auto-layout). Add manual `exitX`/`entryX` or `<Array>` waypoints only when critical overlaps remain after routing.

---

## Azure Infrastructure Topology

```xml
<mxGraphModel pageWidth="1900" pageHeight="1500">
  <mxCell id="vnet" value="Internal VNet - 10.x.0.0/16"
    style="rounded=0;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;
    verticalAlign=top;fontSize=16;fontStyle=1;strokeWidth=4;"
    vertex="1" parent="1"><mxGeometry x="220" y="200" width="1340" height="820"/></mxCell>
  <mxCell id="subnet" value="Application Subnet - 10.x.2.0/24"
    style="rounded=1;whiteSpace=wrap;html=1;fillColor=#e6f4ea;strokeColor=#82b366;
    verticalAlign=top;fontSize=13;fontStyle=1;strokeWidth=2;dashed=1;dashPattern=8 8;"
    vertex="1" parent="1"><mxGeometry x="260" y="280" width="480" height="300"/></mxCell>
  <mxCell id="vm" value="App VM"
    style="image;aspect=fixed;html=1;points=[];align=center;
    image=img/lib/azure2/compute/Virtual_Machine.svg;"
    vertex="1" parent="1"><mxGeometry x="300" y="350" width="64" height="59"/></mxCell>
  <mxCell id="e1" value="HTTPS:443"
    style="edgeStyle=orthogonalEdgeStyle;strokeWidth=3;strokeColor=#0078D4;"
    edge="1" source="inet" target="vm" parent="1"><mxGeometry relative="1"/></mxCell>
  <mxCell id="iso" value="&lt;b&gt;Network Isolation&lt;/b&gt;&lt;br&gt;VNets: Thick borders&lt;br&gt;Subnets: Dashed borders&lt;br&gt;NSGs control traffic"
    style="text;html=1;strokeColor=#d6b656;fillColor=#fff9cc;align=left;
    verticalAlign=top;spacingLeft=10;rounded=1;"
    vertex="1" parent="1"><mxGeometry x="20" y="20" width="220" height="100"/></mxCell>
  <mxCell id="ext" value="External Services"
    style="rounded=0;whiteSpace=wrap;html=1;fillColor=#ffe6cc;strokeColor=#d79b00;
    verticalAlign=top;fontStyle=1;strokeWidth=2;"
    vertex="1" parent="1"><mxGeometry x="20" y="200" width="180" height="200"/></mxCell>
</mxGraphModel>
```

---

## AWS Infrastructure Topology

```xml
<mxGraphModel pageWidth="1900" pageHeight="1500">
  <mxCell id="vpc" value="Production VPC - 10.x.0.0/16"
    style="rounded=0;whiteSpace=wrap;html=1;fillColor=#d5e8d4;strokeColor=#82b366;
    verticalAlign=top;fontSize=16;fontStyle=1;strokeWidth=4;"
    vertex="1" parent="1"><mxGeometry x="220" y="200" width="1200" height="900"/></mxCell>
  <mxCell id="pub" value="Public Subnet - us-east-1a - 10.x.1.0/24"
    style="rounded=1;whiteSpace=wrap;html=1;fillColor=#e6f4ea;strokeColor=#82b366;
    verticalAlign=top;fontSize=12;fontStyle=1;strokeWidth=2;dashed=1;dashPattern=8 8;"
    vertex="1" parent="1"><mxGeometry x="260" y="280" width="500" height="200"/></mxCell>
  <mxCell id="prv" value="Private Subnet - us-east-1a - 10.x.2.0/24"
    style="rounded=1;whiteSpace=wrap;html=1;fillColor=#EFF7FF;strokeColor=#6c8ebf;
    verticalAlign=top;fontSize=12;fontStyle=1;strokeWidth=2;dashed=1;dashPattern=8 8;"
    vertex="1" parent="1"><mxGeometry x="260" y="520" width="500" height="200"/></mxCell>
  <mxCell id="alb" value="ALB"
    style="shape=mxgraph.aws4.application_load_balancer;fillColor=#8C4FFF;
    fontColor=#ffffff;strokeColor=none;"
    vertex="1" parent="1"><mxGeometry x="320" y="340" width="64" height="64"/></mxCell>
  <mxCell id="ecs" value="ECS Task"
    style="shape=mxgraph.aws4.ecs;fillColor=#ED7100;fontColor=#ffffff;strokeColor=none;"
    vertex="1" parent="1"><mxGeometry x="320" y="580" width="64" height="64"/></mxCell>
  <mxCell id="e1" value="HTTPS:443"
    style="edgeStyle=orthogonalEdgeStyle;strokeWidth=3;strokeColor=#0078D4;"
    edge="1" source="igw" target="alb" parent="1"><mxGeometry relative="1"/></mxCell>
  <mxCell id="e2" value="HTTP:8080"
    style="edgeStyle=orthogonalEdgeStyle;strokeWidth=2;strokeColor=#00897B;"
    edge="1" source="alb" target="ecs" parent="1"><mxGeometry relative="1"/></mxCell>
  <mxCell id="iso" value="&lt;b&gt;Network Isolation&lt;/b&gt;&lt;br&gt;VPCs: Thick borders&lt;br&gt;Public/Private/Isolated subnets&lt;br&gt;Security Groups + NACLs"
    style="text;html=1;strokeColor=#d6b656;fillColor=#fff9cc;align=left;
    verticalAlign=top;spacingLeft=10;rounded=1;"
    vertex="1" parent="1"><mxGeometry x="20" y="20" width="240" height="110"/></mxCell>
</mxGraphModel>
```

---

## Azure Topology Guidance

### Canvas and zone sizing
- Use `pageWidth="1900" pageHeight="1500"` for complex multi-VNet diagrams.

### VNet and subnet styles

| Element | Style |
|---|---|
| VNet (DMZ) | `fillColor=#fff2cc;strokeColor=#d6b656;strokeWidth=4;` |
| VNet (Internal) | `fillColor=#d5e8d4;strokeColor=#82b366;strokeWidth=4;` |
| VNet (Management) | `fillColor=#dae8fc;strokeColor=#6c8ebf;strokeWidth=4;` |
| Subnet | `dashed=1;dashPattern=8 8;strokeWidth=2;` lighter shade of parent VNet colour |

Label subnets with name + CIDR. For delegated subnets, add delegation note to label.

### Traffic flow colour palette (Azure)

| Traffic type | Colour | Style |
|---|---|---|
| HTTPS:443 internet ingress | `#0078D4` Azure blue | solid, `strokeWidth=3` |
| HTTP:80/8080 backend | `#00897B` Teal | solid, `strokeWidth=2` |
| Database (PostgreSQL:5432) | `#5C6BC0` Indigo | dashed, `strokeWidth=2` |
| Storage (NFS/Gluster) | `#43A047` Green | solid, `strokeWidth=2` |
| Management/Identity | `#F57C00` Amber | dashed, `strokeWidth=2` |
| Denied/blocked | `#C62828` Red | solid — reserve exclusively for blocked traffic |

### Essential annotation boxes (every Azure topology)
1. **Network Isolation Explanation** — top-left, `fillColor=#fff9cc`: VNet/subnet conventions, NSG/DNS notes
2. **Zone Separation** — VNet Peering zone (`fillColor=#f5f5f5`) + External Services zone (`fillColor=#ffe6cc`)

### Topology checklist (Azure)
- [ ] VNets have thick borders (strokeWidth=4)
- [ ] Subnets have dashed borders (strokeWidth=2, dashPattern=8 8)
- [ ] All resources positioned inside their subnets
- [ ] Traffic arrows labelled with protocols and ports
- [ ] Network isolation explanation box included
- [ ] Color-coded zones for different purposes
- [ ] Canvas 1900×1500 for complex infra
- [ ] VNet peering connections shown in separate zone
- [ ] External services grouped in separate zone
- [ ] Animation preference confirmed before generating

---

## AWS Topology Guidance

### VPC and subnet styles

| Element | Style |
|---|---|
| VPC (Production) | `fillColor=#d5e8d4;strokeColor=#82b366;strokeWidth=4;` |
| VPC (Development) | `fillColor=#dae8fc;strokeColor=#6c8ebf;strokeWidth=4;` |
| VPC (Shared Services) | `fillColor=#fff2cc;strokeColor=#d6b656;strokeWidth=4;` |
| Public subnet | `fillColor=#e6f4ea;strokeColor=#82b366;dashed=1;dashPattern=8 8;strokeWidth=2;` |
| Private subnet | `fillColor=#EFF7FF;strokeColor=#6c8ebf;dashed=1;dashPattern=8 8;strokeWidth=2;` |
| Isolated subnet (databases) | `fillColor=#fff3e0;strokeColor=#e6821e;dashed=1;dashPattern=8 8;strokeWidth=2;` |
| AZ container | `fillColor=#f5f5f5;strokeColor=#999999;strokeWidth=1;` |

Label subnets with name, AZ, and CIDR.

### Resource placement rules (AWS)
- Internet-facing (ALB, NAT Gateway, Bastion) → **public subnets**
- Application servers / ECS tasks → **private subnets**
- Databases (RDS, ElastiCache) → **isolated subnets** (no outbound internet)

### Traffic flow colour palette (AWS)

Same palette as Azure, plus:
- HTTPS:443 to VPC Endpoints / AWS managed services: `#43A047` Green, solid
- SSH:22 / SSM management: `#F57C00` Amber, dashed
- Always show NAT Gateway path for private subnet → internet egress

### Essential annotation boxes (every AWS topology)
1. **Network Isolation Explanation** — top-left: VPC thick borders, subnet tiers, SG/NACL notes, VPC Endpoints
2. **Zone Separation** — Internet/Edge zone (orange), VPC Peering/TGW zone (grey), AWS Managed Services zone (purple)

### Topology checklist (AWS)
- [ ] VPCs have thick borders (strokeWidth=4) colour-coded by environment
- [ ] Subnets have dashed borders colour-coded by tier (public/private/isolated)
- [ ] AZ containers group subnets per Availability Zone
- [ ] All resources positioned inside their subnets
- [ ] Internet Gateway and NAT Gateway shown
- [ ] Traffic arrows labelled with protocols and ports
- [ ] Security Group boundaries annotated where important
- [ ] Network isolation explanation box included
- [ ] Canvas 1900×1500 for complex infra
- [ ] VPC Peering / Transit Gateway shown in separate zone
- [ ] Edge/internet services (CloudFront, Route53, WAF) in separate zone
- [ ] Animation preference confirmed before generating
