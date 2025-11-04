Key considerations:
# Shared data between 2 cores
- Who owns what: 
  - customer data: Old core (FCC and CIF service)
  - account data: Each core manages its own accounts
  - transaction data: Each core manages its own accounts
  - Accounting: For MVP1 FC GL, for MVP2 to be discussed about new GL
- Source of truth of each data type?
  - customer data: integration to query Old core (FCC and CIF service)
  - account data: integration to query new core
  - transaction data: integration to query new core
  - Accounting: For MVP1 FC GL
- Missing features of old core? branches: has not identified missing features of shared data
# Unique products (MVP1)
- Which products can new core provide:
  - CASA/Deposit/Secured and unsecured loan
- Missing features that can be handled by integration layer?
  - Fee 
  - Limit
  - CIAM
  - Card
  - LOS
  - AML/Fraud...
  - yKYC
- Bank ops and financial records impacted (backadated, adjusted txn):
  - Teller to support customers with digital accounts (new teller app?)
# Common products
- Who handles the common products?
  - Both, depends on the business
- Existing contracts?
  - Old core
  - During migration:
    - Migrate CASA/Deposit/Loan => to specify strategies
- Migration to new core ?
- Realign functions like fee calculation, interst ...
  - Fee and limit are handled by old core, called via API
  - Interset calculation can be done by products of new core
# Migrate MVP
- How to migrate balances?
  - Big bang: quick and cheap, risky
  - Phased: complex, takes longer, less risky
  - Shadow/mirror: daily sync, safer, longer
- How to migrate account?
- How to migrate card?
- Ensure 1 time migration with accurate balances and fees, components ...
  - 
- Downstream systems affected by migration/new systems?
  - Deploy some reused services to VPC, mirror sync data daily with transformer services
  - Replace some services with new solutions (credit, fee, trade finance, )
# Daily reconciliation
- Even after migratoin, coexist for a while, how to sync between the 2?
- Master data ownership, who own what afterwards
# Data warehouse
- MVP1: sync data to on prem data warehouse
- MVP2: move data to cloud with new solution for reporting and analyzing
# Operation overhead
- Modify or build new backoffice apps to manage new core
- Extra training for Accounting, teller, credit
- KNowledge transfer to internal IT team to self-sustain
# Partner management
- Multi tenant model with Keycloak/Okta/Kong to manage access to API with AuthN/AuthZ
- Shared database for customer and account data using CIF or stakeholder IDs to map to partner ID
- Kong to manage rate limiter, pricing, logging and audit trail
- Each partner own their own products with different parameters configurable, can be done via TM Smart Contracts, for Tuum and 10x, needs to manage product templates externally for each partner
- Dedicated partner-service to manage partners' customer, account data
- Shared integration layer and CBS
- Sandbox environment with smaller VPC that has enough functionalities to test configuring products, onboard customers and create accounts


# Phased migration
- MVP1 (12-18 months): build new digital bank with CASA/deposit and loan product, partner integration with sandbox and portal
- MVP1.5 (12 months): coexistence, move more products to new core, migrate account and balances data with re-routing traffic from app to new core for certain products, deploy reused components to cloud with mirror sync, replace some components with new solutions (GL, reporting, ML)
- MVP 2 (12 months): migrate all customer and product/accounts to new core. Deprecate the old core