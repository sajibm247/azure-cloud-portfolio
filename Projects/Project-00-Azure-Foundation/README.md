# Project 00 — Azure Foundation

## Objective
Learn the core building blocks of Azure architecture — the resource
group and storage account — by creating them for real rather than
just reading about them. This project also establishes the naming
and tagging conventions used across the rest of the portfolio.

## Resources Created
- **Resource Group:** `RG-cloud-portfolio-uks-01` (UK South)
- **Tags:** `Project=CloudPortfolio`, `Environment=Learning`, `Owner=Sajib`
- **Storage Account:** `stcloudportfolio2026`
  - Performance: Standard
  - Redundancy: LRS
- **Blob Container:** created inside the storage account
- **File Upload:** test file uploaded to confirm blob storage works end-to-end

## Design Decisions
- **Why UK South?** Matches my location, keeps latency low, and keeps
  every resource in this portfolio in one consistent region so cost
  and networking stay simple to reason about.
- **Why LRS (Locally Redundant Storage)?** This is a personal learning
  project with no production uptime requirement, so the cheapest
  redundancy tier is the right choice. Higher tiers (ZRS, GRS) matter
  when data needs to survive a datacenter or regional failure — not
  relevant here, but understanding *why* LRS is enough is the point.
- **Why these tags?** `Project` and `Environment` make it obvious at a
  glance what this resource is for and whether it's safe to delete.
  `Owner` matters in real teams so nobody deletes someone else's
  resource by mistake — good habit to build now.

## Security Settings
- **Secure transfer required:** Enabled — forces HTTPS for all
  connections to the storage account, rejecting plain HTTP.
- **Public blob access:** Disabled — containers and blobs are private
  by default; nothing is accessible without authentication
  
## What I Learned
- The Azure resource hierarchy: Tenant → Subscription → Resource Group → Resource
- Why resource groups exist as a management/billing/lifecycle boundary,
  not just a folder
- The practical difference between storage redundancy tiers
- How to run my first Azure CLI command (`az group show`)

## Open Question
- At what point would ZRS actually matter for a real small business app??
