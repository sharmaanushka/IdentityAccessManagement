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

4. Joiner process

Now let's implement the Joiner part of JML.

Suppose Rahul joins the Finance department.

HR creates Rahul in Workday.
Rahul joins
    │
    ▼
Workday
    │
    │ Employee created
    ▼
SailPoint
    │
    │ Read department = Finance
    │ Job = Financial Analyst
    │ Location = India
    ▼
Role / Access Policies
    │
    ├──── Finance Role
    │
    ├──── Microsoft 365
    │
    ├──── Finance Application
    │
    └──── Required Groups
              │
              ▼
       Entra ID / AD
              │
              ▼
        Applications
So Rahul doesn't need to submit 10 separate access requests.

His birthright access can be automatically assigned based on his attributes.

5. RBAC — Role-Based Access Control

Now we introduce RBAC.

Instead of saying:

Rahul gets Application A + Group B + Permission C + Database D...

We create roles.

Employee
   │
   └── Finance Department
           │
           └── Financial Analyst
                    │
                    ├── Finance App
                    ├── Microsoft 365
                    └── Finance SharePoint

                USER
               │
               ▼
        ┌─────────────┐
        │ Department  │
        │   Finance   │
        └──────┬──────┘
               │
               ▼
        ┌─────────────┐
        │    ROLE     │
        │ Financial   │
        │   Analyst   │
        └──────┬──────┘
               │
       ┌───────┼────────┐
       ▼       ▼        ▼
   Finance   SharePoint  M365
     App

     The role maps the employee to the required access.

6. Least Privilege

RBAC alone isn't enough.

We also apply Least Privilege.

The principle is:

Give the user only the access required to perform their job — nothing more.

For example:

A Finance Analyst may need:

Finance Application → Read/Write
Reports → Read
Payroll → No Access
Admin Console → No Access
Production Database → No Access

                 Finance Analyst
                       │
              ┌────────┼────────┐
              │        │        │
              ▼        ▼        ▼
           Finance   Reports   M365
           R/W       Read      Standard
              
              X Payroll Admin
              X Production Admin
              X Security Admin
  "RBAC gives the appropriate role, while least privilege ensures that the permissions within that role are limited to what is actually required."

  7. Authentication — Entra ID

Now we have to answer:

How does the user actually log in?

That's where Microsoft Entra ID comes in.

                     USER
                      │
                      ▼
              ┌──────────────┐
              │ Microsoft    │
              │ Entra ID     │
              └──────┬───────┘
                     │
              Authentication
                     │
             ┌───────┴────────┐
             │                │
             ▼                ▼
            MFA       Conditional Access
             │                │
             └───────┬────────┘
                     │
                     ▼
                  SSO
                     │
          ┌──────────┼───────────┐
          ▼          ▼           ▼
       SaaS App   Internal App   M365

       Entra ID handles things like:

Authentication
SSO
MFA
Conditional Access
Device-based policies
Risk-based policies

8. MFA

We don't want:
Username + Password
to be enough.

So we introduce MFA.

Example:
User
 │
 ▼
Username + Password
 │
 ▼
Entra ID
 │
 ▼
MFA
 │
 ├── Microsoft Authenticator
 ├── FIDO2
 └── Other approved factor
 │
 ▼
Access Granted

9. Zero Trust

Now I would bring in Zero Trust.

The principle is:

Never trust automatically. Always verify.

So even if Rahul is an employee, we don't automatically trust the request.

We evaluate:

Who are you?
     +
What are you accessing?
     +
From where?
     +
What device are you using?
     +
Is the sign-in risky?
     +
Do you actually need this access?

                     USER
                       │
                       ▼
                ┌─────────────┐
                │ Entra ID    │
                └──────┬──────┘
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
     Identity        Device         Location
        │              │              │
        ▼              ▼              ▼
     MFA / Risk    Compliant?      Trusted?
        │              │              │
        └──────────────┼──────────────┘
                       ▼
                Access Decision
                       │
                 ┌─────┴─────┐
                 ▼           ▼
               Allow        Deny
10. SSO

Once authenticated through Entra ID, the user shouldn't have to repeatedly enter credentials.

So we use Single Sign-On.

                    USER
                     │
                     ▼
                Entra ID
                     │
                 Authenticate
                     │
                     ▼
                    SSO
                     │
       ┌─────────────┼─────────────┐
       ▼             ▼             ▼
    Salesforce      SAP        Internal App
Depending on the application, protocols such as SAML or OpenID Connect/OAuth can be used.

11. On-premises applications

Now imagine the company still has legacy applications that depend on Active Directory.

Then our architecture can contain both Entra ID and AD.

                    SailPoint
                       │
             ┌─────────┴──────────┐
             │                    │
             ▼                    ▼
        Entra ID              AD
             │                    │
             │                    │
       Cloud Apps          On-Prem Apps
  AD can continue supporting legacy/on-prem applications, while Entra ID handles modern cloud authentication.

Synchronization can connect the on-premises identity environment with Entra ID where required.

12. Access request

Now consider something outside Rahul's normal role.

Rahul needs temporary access to a reporting application.

He requests:

Rahul
  │
  ▼
Access Request
  │
  ▼
SailPoint
  │
  ▼
Manager Approval
  │
  ▼
Application Owner Approval
  │
  ▼
Access Granted

                USER
                 │
                 ▼
          Access Request
                 │
                 ▼
            SailPoint
                 │
                 ▼
          Policy Check
                 │
                 ▼
          Manager Approval
                 │
                 ▼
       Application Owner
            Approval
                 │
                 ▼
          Access Granted
  This gives us governance and accountability.

  13. Just-In-Time access

For privileged access, I wouldn't give permanent admin rights.

For example:

Normal user
     │
     │ Request
     ▼
Privileged Access
     │
     │ Approval
     ▼
Admin access
     │
     │ 2 hours
     ▼
Access automatically removed

                 USER
                   │
                   ▼
             Request Admin
                   │
                   ▼
             Approval / MFA
                   │
                   ▼
            JIT Access
                   │
             ┌─────┴─────┐
             │           │
          2 hours      Activity
             │          Logging
             ▼
       Access Removed
  This reduces the standing privilege problem.

  14. Mover process

Now suppose Rahul moves from:

Finance → IT

This is where many IAM implementations fail.

We shouldn't simply give him IT access.

We also need to remove the Finance access that he no longer requires.

             Rahul
               │
       Department Change
               │
               ▼
            Workday
               │
               ▼
           SailPoint
               │
       ┌───────┴────────┐
       ▼                ▼
Remove old role      Assign new role
 Finance Role           IT Role
       │                │
       ▼                ▼
Remove Finance       Add IT Access
 Access

 15. Leaver process

Now Rahul leaves the company.

HR marks him as terminated in Workday.

That event triggers deprovisioning.

              Employee leaves
                     │
                     ▼
                  Workday
                     │
              Termination Event
                     │
                     ▼
                 SailPoint
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
       Entra ID      AD       Applications
          │          │          │
          ▼          ▼          ▼
       Disable     Disable    Remove Access
                     │
                     ▼
              Account Disabled
    This should happen automatically and as quickly as possible according to the organization's termination policy.

   
16. Access Reviews / Certification

Even if our automated provisioning is working, we can't assume access remains appropriate forever.

Managers should periodically review their employees' access.
Manager
   │
   ▼
Quarterly Access Review
   │
   ├── Rahul → Finance App → REMOVE
   ├── Priya → HR App → KEEP
   └── Amit → Admin Access → REMOVE

                     SailPoint
                     │
              Access Certification
                     │
                     ▼
                  Manager
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
        User A     User B     User C
          │          │          │
        KEEP       REVOKE       KEEP
          │          │          │
          └──────────┼──────────┘
                     ▼
              Access Updated

17. Audit and logging

Every important IAM event should be auditable.

Workday ───────┐
               │
SailPoint ─────┤
               │
Entra ID ──────┼──────► Logs / SIEM
               │
AD ────────────┤
               │
Applications ──┘
                         │
                         ▼
                  Security Monitoring
                         │
                         ▼
                      Audit
          This becomes particularly important for compliance and investigations.

18. The complete architecture

                           ┌──────────────────┐
                         │     WORKDAY      │
                         │  HR Source of    │
                         │      Truth       │
                         └────────┬─────────┘
                                  │
                         Employee Lifecycle
                                  │
                                  ▼
                    ┌─────────────────────────┐
                    │       SAILPOINT         │
                    │                         │
                    │ Identity Governance     │
                    │ JML                     │
                    │ RBAC                    │
                    │ Access Requests          │
                    │ Approvals               │
                    │ Certifications          │
                    │ Least Privilege         │
                    └────────────┬────────────┘
                                 │
                   ┌─────────────┴─────────────┐
                   │                           │
                   ▼                           ▼
          ┌─────────────────┐          ┌─────────────────┐
          │   ENTRA ID      │          │       AD        │
          │                 │          │                 │
          │ Authentication  │          │ On-Prem Identity│
          │ MFA             │          │ Groups          │
          │ SSO             │          │ Legacy Apps     │
          │ Conditional     │          │                 │
          │ Access          │          │                 │
          └────────┬────────┘          └────────┬────────┘
                   │                            │
             ┌─────┴──────┐              ┌─────┴──────┐
             ▼            ▼              ▼            ▼
          SaaS Apps   Cloud Apps    On-Prem Apps   Legacy
             │            │              │
             └────────────┼──────────────┘
                          ▼
                  Business Applications
                          │
                          ▼
                  ┌───────────────┐
                  │ Logging / SIEM│
                  │ Audit / Alerts│
                  └───────────────┘
 19. Where Zero Trust fits

Don't think of Zero Trust as another box in the architecture.

It's the security philosophy applied across the architecture.

                    ZERO TRUST
                        │
        ┌───────────────┼────────────────┐
        ▼               ▼                ▼
   Verify Identity   Least Privilege   Continuous
        │               │              Evaluation
        ▼               ▼                ▼
       MFA             RBAC          Risk/Device
                                     Assessment
        │               │                │
        └───────────────┼────────────────┘
                        ▼
                 Access Decision


     
