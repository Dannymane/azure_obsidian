## 1.1 Shared responsibility models SaaS PaaS IaaS

![[Pasted image 20250607183044.png]]

**Infrastructure as a service** - you manage **VMs, OS, runtime, data, apps**. Examples: Azure Virtual Machines, Azure VNet.
**Platform as a service** - you manage **apps and data**. Examples: Azure App Service, Azure SQL Database.
**Software as a service** - You just **use the software**. Examples: Microsoft 365, Dynamics 365, Outlook 365, Notion

From costumer perspective the **SaaS means**: "I just log in and use the app. I don’t install, update, or manage anything — not the infrastructure, not the app, not the data.

While e.g. Zoom is SaaS from costumer perspective, from developer perspective **They _do_ manage the app**, the backend services, the infrastructure, security, monitoring, etc. (probably a mix of IaaS and PaaS)