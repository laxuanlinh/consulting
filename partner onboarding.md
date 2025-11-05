@startuml
title BaaS Partner Onboarding (High-Level)

participant Partner
participant OnboardingPortal
participant BankOps
participant IDP
participant PartnerProductService
participant SandboxAPIs
participant CICDPipeline
participant AppStores

== Registration ==
Partner -> OnboardingPortal: Register Partner
OnboardingPortal -> BankOps: Review Application
BankOps --> IDP: Create Partner Identity
BankOps -> Partner: Provide Credentials + URLs

== Product Setup ==
Partner -> OnboardingPortal: Login (Sandbox)
Partner -> OnboardingPortal: Select Product Templates
OnboardingPortal -> PartnerProductService: Store Partner Configurations

== Sandbox Testing ==
Partner -> SandboxAPIs: Test Product Lifecycle / Integrations
SandboxAPIs -> SandboxAPIs: Enforce Compliance (eKYC/AML/Fraud)

== Optional Dedicated Cluster ==
Partner -> BankOps: Request Dedicated Cluster
BankOps -> CICDPipeline: Provision Cluster (CloudFormation)

== White-Label App ==
Partner -> OnboardingPortal: Submit App Branding & Feature Flags
OnboardingPortal -> CICDPipeline: Trigger Build (Sandbox App)
CICDPipeline -> Partner: Provide App Build for Approval
Partner -> CICDPipeline: Approve Build
CICDPipeline -> AppStores: Deploy Production App

== Go Live ==
Partner -> OnboardingPortal: Monitor Users & Activity

@enduml