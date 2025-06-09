## 1.1 Shared responsibility models (cloud service types) SaaS PaaS IaaS

![[Pasted image 20250607183044.png]]

**Applications**:
- **The code** (apps, services, microservices) that runs your business logic.
- **Configuring, updating, and securing the software** itself.
- **Application-layer controls**, like validating inputs, handling errors securely, and managing business logic.
**Network controls**:
- **Configuring firewalls**, **NSGs (Network Security Groups)**, **subnets**, **routing**, and **access rules**.
- Controls over **who can access what**, and **how traffic flows** in and out of your environment.
- Managing **virtual networks**, **VPNs**, **load balancers**, etc.

**Infrastructure as a service** - you manage **VMs, OS, runtime, data, apps**. Examples: Azure Virtual Machines, Azure VNet.
**Platform as a service** - you manage **apps and data**. Examples: Azure App Service, Azure SQL Database.
**Software as a service** - You just **use the software**. Examples: Microsoft 365, Dynamics 365, Outlook 365

From costumer perspective the **SaaS means**: "I just log in and use the app. I don’t install, update, or manage anything — not the infrastructure, not the app, not the data.

While e.g. Zoom is SaaS from costumer perspective, from developer perspective **They _do_ manage the app**, the backend services, the infrastructure, security, monitoring, etc. (probably a mix of IaaS and PaaS)