# Project 01 — Two-Tier Private Network

## Objective
Build a private, segmented network in Azure that models a real
two-tier application design: a public-facing web layer and a
private application layer that only the web layer can reach.

## Network Design
- **VNet:** `vnet-portfolio-uks-01` — address space `10.10.0.0/16`
- **Subnets:**
  - `snet-web` — `10.10.1.0/24`
  - `snet-app` — `10.10.2.0/24`

The address space was split so the web and app tiers are on separate
subnets, which lets each tier have its own NSG and its own security
boundary — you can't enforce "only web talks to app" if they're on
the same subnet.

## NSG Rules (`nsg-app-uks-01`)

| Priority | Name | Source | Destination | Port | Action |
|---|---|---|---|---|---|
| 100 | Allow-Web-To-App-8080 | 10.10.1.0/24 | 10.10.2.0/24 | TCP 8080 | Allow |
| 200 | Deny-Other-VNet-Inbound | VirtualNetwork | Any | Any | Deny |

**Why both rules?**
- Rule 100 permits only web-subnet traffic to reach the app subnet,
  and only on port 8080 — nothing else gets through.
- Rule 200 overrides Azure's default `AllowVnetInBound` rule, which
  would otherwise let *any* resource inside the VNet reach the app
  subnet. Without rule 200, rule 100 alone doesn't actually make the
  app subnet private.
- NSG rules are evaluated by priority, lowest number first, and stop
  at the first match — so rule 100 must sit at a lower number than
  rule 200 for this to work correctly.

## Bug Found and Fixed
The rules were originally created using placeholder address ranges
(`10.0.0.0/24`) from an earlier design draft, before the VNet was
actually built with its real address space (`10.10.0.0/16`). This
meant `Allow-Web-To-App-8080` didn't match any real traffic — it was
a dead rule that looked correct in the portal but would have silently
blocked all legitimate web-to-app traffic once VMs were deployed.

Caught it by running:
```bash
az network nsg rule list --resource-group RG-cloud-portfolio-uks-01 --nsg-name nsg-app-uks-01 --output table
```
and comparing the output against the actual subnet ranges. Fixed with:
```bash
az network nsg rule update \
  --resource-group RG-cloud-portfolio-uks-01 \
  --nsg-name nsg-app-uks-01 \
  --name Allow-Web-To-App-8080 \
  --source-address-prefixes 10.10.1.0/24 \
  --destination-address-prefixes 10.10.2.0/24
```
Verified the fix by re-running the list command and confirming the
addresses matched the real VNet design.

**Lesson:** always verify NSG rules against the actual deployed
network, not against the design on paper — the two can silently
drift apart.

## Cost Discipline
Set up an Azure Budget (`portfolio-budget`, $100/month threshold,
evaluated on **Actual** spend rather than Forecasted, with alerts at
50% and 90%) before deploying any compute resources, since VMs are
the first resources in this project capable of running up real cost.

## Status / Next Steps
- ✅ VNet and subnets created
- ✅ NSG rules created, tested for correctness, bug fixed
- ✅ Private DNS zone created and linked to the VNet
- ❌ No VM deployed yet — the rules have never been tested against
  real traffic
- ❌ DNS records not yet created or resolved
- **Next:** deploy a jump-box VM in `snet-web` to validate the NSG
  rules end-to-end and test DNS resolution against the private zone
