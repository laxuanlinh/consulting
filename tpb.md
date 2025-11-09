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

# Partner customer data management
- Master customer data is stored in old core with CIF and ID card number as identifier
- Each partner can enrich the customer data and store in the Customer service of their own cluster
- The partner-specific data could be stored as document data in MongoDB/DynamoDB...
- A Mapping service is needed to map a customer to multiple partners when they're onboarded to each app
- Credit limit and credit scores are shared between all partners and stored in old core
  
# Partner account data management
- All account master data is stored in new core, including account ID, balances, interest rate...
- An Account service in the shared layer is needed to filter and limit access of each partner to their own account data, via partner ID or access control of the Core (Tuum)
- Product specific behavior like interest accrual, billing, repayment, ... are done by the new core and the shared integration layer
- Partners can only modify the parameters of offered products.
- These parameters are stored in Partner-product service
- Each partner has access to the kafka topic for their account updates (interst accrual, billing, repayment...)
- Partner-specific services can be used to store partner-specific account data in their own clusters (customer, account, parameters...)
- Partner-specific services can provide isolation and integration via API and webhook
- All updates to an account have to go through shared services such as AML/Fraud, credit limit, credit score...

# Partner Onboarding Steps
## 1. Registration
- Partner registers in onboarding portal.
- Bank reviews and approves.
- Bank creates identity in IDP and provides:
  - Credentials
  - Portal URL
  - Sandbox API URL
  - Production API URL (inactive until go-live)
## 2. Product Setup
- Partner logs into the Partner Portal (Sandbox).
- Partner selects product templates:
  - CASA
  - Deposit
  - Loan
  - Card
- Partner configures product parameters (fees, limits, etc.).
- Configurations are stored in Partner-Product Service.
## 3. Sandbox Testing
- Partner integrates with Sandbox APIs.
- Partner tests behaviors and lifecycle (Tuum doesn't support this)
- Bank enforced controls (not customizable) eKYC, AML, Credit scoring / limits
## 4. Optional Dedicated Cluster
- Partner requests dedicated cluster if needed.
- Bank reviews and approves.
- Cluster is provisioned via CloudFormation.
## 5. White-Label App
- Partner receives white-label app configuration template.
- Partner customizes logo, colors, text, feature flags.
- Partner submits configuration.
- CI/CD pipeline builds sandbox app.
- Partner reviews and approves.
- CI/CD pipeline builds production app and submits to app stores.
## 6. Production Launch
- Partner switches to production.
- Partner Portal provides:
  - User analytics
  - Product and feature flag management
  - Transaction monitoring (read-only)
- Bank continues to handle regulatory and core ledger operations.

# Customer Onboarding Journey
- Customer selects **Register** in partner app.
- ID photo is captured and sent to **Shared Layer → Old Core**.
- Old Core performs OCR, checks existing customer (CIF), blacklist, AML, fraud, and credit score.
- If approved, Old Core returns CIF to partner services.
- Customer completes **liveness selfie** check (Shared Layer → Old Core → App).
- Customer enters phone number; Old Core verifies uniqueness and sends OTP.
- If verified → **Registration complete** (CIF active).

- If product feature flags are enabled and customer opts in:
  - Partner services evaluate eligibility based on risk/credit info.
  - Partner services returns product offer with credit limits/interest rates...
  - If customer consents → create account in **New Core** (and in **CMS** if a card is included).

# Transfer between 2 cores

## Old Core to New Core
- Old Core receives the transfer request
- Old Core posts the payment in FCC
- Old Core sends a transfer event to New Core via Kafka
- New Core processes the event and returns a success response via Kafka

## New Core to Old Core
- New Core receives the transfer request
- New Core posts internally and also sends a payment event to Old Core via Kafka
- Old Core posts the payment in FCC and returns a Kafka success response
- New Core also returns a Kafka success response

## Inbound Transfer
- SWIFT/NAPAS sends inbound transfer to FC Payment Hub
- FC Payment Hub checks recipient account
- If account belongs to Old Core → send to Old Core (FCC)
- If account belongs to New Core → send to New Core via Kafka
- New Core returns Kafka success to Payment Hub
- Payment Hub returns success to SWIFT/NAPAS

## Outbound Transfer
- New Core receives outbound transfer request
- New Core posts the payment and returns success
- New Core sends outbound payment to FC Payment Hub
- FC Payment Hub sends to SWIFT/NAPAS
- SWIFT/NAPAS returns success
- Payment Hub returns success to New Core

