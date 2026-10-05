Problem Statement:

Design an Identity and Access Management (IAM) architecture for an enterprise with 5,000 employees. The organization uses multiple business applications, cloud services, and on-premises systems. The architecture should provide secure user provisioning and deprovisioning, Single Sign-On (SSO), Multi-Factor Authentication (MFA), Role-Based Access Control (RBAC), least-privilege access, Just-In-Time (JIT) access, access reviews, and automated joiner-mover-leaver (JML) processes while following Zero Trust principles.

ARCHITECTURE 
Workday → SailPoint → Microsoft Entra ID / AD → Applications

Where:

Workday = authoritative source for employee identity
SailPoint = Identity Governance / lifecycle / access governance
Entra ID = authentication, SSO, MFA, Conditional Access
AD = on-prem identity where required
Applications = business systems where users actually consume access

1. Start with the requirement

We have an enterprise with 5,000 employees.

Assume the company has:

Cloud applications
On-premises applications
Microsoft 365
Internal applications
Different departments such as HR, Finance, IT, Sales
Different levels of access depending on job role


"I would first establish Workday as the authoritative source for employee identity and then design the IAM architecture around the complete Joiner-Mover-Leaver lifecycle."

                    ENTERPRISE – 5,000 USERS
                            │
             ┌──────────────┼──────────────┐
             │              │              │
           HR              IT            Business
             │              │            Applications
             └──────────────┼──────────────┘
                            │
                     IAM ARCHITECTURE
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
     Identity            Access             Governance
     Lifecycle           Control             & Audit

2. Workday becomes the authoritative identity source

I would not allow every application to independently create users.

Instead, Workday is the source of truth for employee information.

Employee joins company
        │
        ▼
      Workday
        │
        │ Employee ID
        │ Name
        │ Department
        │ Manager
        │ Job Title
        │ Location
        │ Employment Status
        ▼
    IAM Platform

    So when HR creates an employee in Workday, that becomes the trigger for the identity lifecycle.

                        HR
                     │
                     ▼
                 ┌─────────┐
                 │ Workday │
                 └────┬────┘
                      │
             Employee information
                      │
                      ▼
             ┌────────────────┐
             │ Identity        │
             │ Governance      │
             │ Platform        │
             └────────────────┘
Workday doesn't necessarily need to manage application permissions.

It tells us:

"This person exists, this is their job, this is their department, this is their manager, and this is whether they're active."

Access governance can then use this information to determine what access the employee should receive.

3. Why do we introduce SailPoint?
You can technically do:
Workday → Entra ID → Applications
But in a large enterprise, access governance becomes complicated.

For example:

Finance employee → SAP Finance
HR employee → Workday
Developer → GitHub
IT employee → privileged systems
Manager → additional approval rights

We don't want HR to manually manage all those permissions.

That's where SailPoint acts as the Identity Governance layer.

                       WORKDAY
                  Authoritative Source
                          │
                          ▼
                  ┌───────────────┐
                  │   SAILPOINT   │
                  │               │
                  │ IGA /         │
                  │ Governance    │
                  └───────┬───────┘
                          │
              ┌───────────┼────────────┐
              │           │            │
              ▼           ▼            ▼
           Entra ID      AD       Applications
Think of it simply as:

Workday tells us WHO the person is. SailPoint determines and governs WHAT access the person should have.

