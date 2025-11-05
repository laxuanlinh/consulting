@startuml
title Customer Onboarding (High Level)

participant CustomerApp
participant SharedLayer
participant PartnerServices
participant OldCore
participant NewCore
participant CMS

== Registration ==
CustomerApp -> SharedLayer: Send ID photo
SharedLayer -> OldCore: OCR + CIF lookup + AML/Fraud/Credit checks
OldCore --> PartnerServices: Return CIF + Risk result

PartnerServices --> CustomerApp: Proceed or Reject

== Liveness Check ==
CustomerApp -> SharedLayer: Send selfie
SharedLayer -> OldCore: Liveness + Face Match
OldCore --> CustomerApp: Result

== Phone Verification ==
CustomerApp -> OldCore: Enter phone + request OTP
OldCore --> CustomerApp: OTP verification result

== Registration Complete ==
OldCore --> CustomerApp: CIF active

== Optional Product Opening ==
CustomerApp -> PartnerServices: Request product (if enabled)
PartnerServices -> OldCore: Retrieve credit/risk/limits
PartnerServices --> CustomerApp: Offer terms

CustomerApp -> PartnerServices: Customer accepts

PartnerServices -> NewCore: Create account
NewCore --> PartnerServices: Account ID

opt If card is included
    PartnerServices -> CMS: Create card profile
    CMS --> PartnerServices: Card details
end

PartnerServices --> CustomerApp: Account/Card confirmed

@enduml
