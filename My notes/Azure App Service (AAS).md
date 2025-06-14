[Azure App Service documentation - Azure App Service \| Microsoft Learn](https://learn.microsoft.com/en-us/azure/app-service/)
## 1.1      Explore Azure App Service

Azure App Service is an HTTP-based service for hosting web applications, REST APIs, and mobile back ends.

scale up/down – scale number of cores/RAM of the machine. (vertical scaling)

scale out/in – scale the number of machines

-       Windows apps are run without container
-       Linux based apps are run within single container – Azure creates it under the hood (in **built-in image**)
-       You can run your app in **custom container**. To configure custom container(s) you can use:
1. Orchestration platforms: **Windows containers** or **Docker Compose** (.yml file). *For Kubernetes use **Azure Kubernetes Service** **(AKS)** instead of AAS.* 
2. Registry: **Docker Hub** or **Azure Container Registry** or any other public/private registry. Containers are pulled from registry.

For Linux containers **custom container** reads app files faster than **built-in image** because in built-in image the app files are in separate from container place – in Azure Storage.  
  

**App Service Environment** **(ASE)**

-       A premium, isolated version of App Service.
-       Runs inside your own virtual network (VNet).
-       You get your own VM
-       Guarantees static inbound and outbound IP addresses
-       only Isolated and IsolatedV2 plans, these plans always have AS Environment

Ideal for high-security, high-scale, or compliance-restricted scenarios.

> [!quote] **Scale unit**
> **Scale unit** - a collection of physical or virtual servers that host and run customer applications. Although managed by Azure, these scale units are shared across many App Service Plans and customers. They include **front-end load balancers**. Scale unit consists of **workers** (VM managed by Azure), when you define the number of VM for plan - you define the number of workers). One scale unit has multiple Inbound IPs and Outbound IPs.

****
**App Service plan**. Each App Service plan defines:

-       Operating System (Windows, Linux)
-       Region (West US, East US, etc.)
-       Number of VM instances
-       Size of VM instances (Small, Medium, Large)
-       Pricing tier (Free, Shared, Basic, Standard, Premium, PremiumV2, PremiumV3, Isolated, IsolatedV2)

Free and Shared tier – can’t scale out, your apps are on the same VM (and other customers apps)

Basic, Standard, Premium, PremiumV2, and PremiumV3 ­– you can scale out, your own dedicated VM but on physical machine with other customer VMs – compute isolation.

Isolated and IsolatedV2 tiers run dedicated Azure VMs on dedicated Azure Virtual Networks. Dedicated physical machine. Network isolation on top of compute isolation. Maximum scale-out capabilities.

Scaling out the plan:
-       An app runs on all the VM instances configured in the App Service plan.
-       If multiple apps are in the same App Service plan, they all share the same VM instances.
-       If you have multiple deployment slots for an app, all deployment slots also run on the same VM instances.

If the plan is configured to run five VM instances, then all apps in the plan run on all five instances – every app will be run on every of 5 VM, but requests from users to app will be divided between VM equally (1 request handles 1 VM). But if you have a singleton service – the singleton state will be encapsulated within VM and other VM won’t track state changes. Multiple apps can be deployed within one plan.

**App creation + Deployment in AAS**
Automated deployment: Azure DevOps, GitHub, BitBucket  
Manual deployment (these methods will create new App Service App and you can configure the created app plan e.g. in Azure platform):
-       GIT – generate Git URL in Azure and paste it to local git repo as **remote** and git push azure master – Azure automatically builds and deploys the app
-       CLI - navigate to app folder in PowerShell 
``` sh
az webapp up 
--name myapp #dy-school-register if you deploy to app in picture above
--resource-group myrg
```
   – the fastest way to deploy in Azure.
-       ZIP – pack to ZIP and sent via POST to Azure, example:  
``` sh
curl -X POST -u <username>:<password> \
--data-binary @app.zip \
https://<your-app-name>.scm.azurewebsites.net/api/zipdeploy
```

-       FTP/S: Azure gives you FTP credentials and an FTP hostname. You use an FTP client (like FileZilla or VS Code extensions) to connect. You drag and drop your files into the web root folder.


> [!quote] First deployment/publishing new build
> Before deployment you can create configure App Service without code and than deploy your repository to configured App Service app using app name.
![[Pasted image 20250613105734.png]]
In Visual Studio publish your app "to folder" and than place files of this folder into .zip, than navigate in PowerShell to folder with this .zip and execute command:
``` bash
az webapp deploy 
--name dy-school-register
--resource-group main-rg 
--src-path releaseVersion.zip 
--type zip
```
> [!quote] 
> Visual studio have option to publish to Azure, so you can easily manually deploy your app, but it doesn't see the already created through Azure webApp. 






Slots allow firstly to deploy a new production build to a staging environment, warm up the necessary worker instance to match production scale and then swap staging and production slot.

**Sidecar container** – separate container related to main app container. Available up to nine sidecar containers for each sidecar-enabled **custom container** app. Add in **Deployment Center** in the app's management page.

**AAS Authentication and Authorization**

AAS provides own solution for authentication and authorization. User accounts managed through **Microsoft Entra ID**. It easily works with third party authentication (through MS/Google/Meta/X accounts etc.). ASP roles (attributes) can be connected to AAS authorization – the AAS can send configurated roles in token. But the passwords or passwords hashes are hidden and there is no access to them. Access another user account can be through code changes (turn off authorization etc.).

Authentication flows:
-       **Server-directed flow** – without provider SDK, typically for web projects. The authentication takes place on provider’s login page.
-       **Client-directed flow** – with provider SDK, typically for browser-less apps. The application signs users in to the provider manually: the provided credentials are sent throw SDK code to provider server and provider returns token to app and then the app submits the authentication token to App Service for validation. This applies to REST APIs, Azure Functions and native mobile apps.

Authentication can be configured to allow or not allow the unauthenticated requests. For allowed unauthenticated requests the provider will pass all requests to application, but only authenticated requests will have authentication info in the HTTP headers. For disallowed unauthenticated requests – disallowed unauthenticated requests will be redirected to another chosen auth provider `/.auth/login/<provider>` or simply rejected by HTTP 401 Unauthorized response. **This restriction applies to all calls to application, to any page.**

AAS writes all logs about authentication and authorization with details, it available to browse. Enabling application logging will write all traces in your log files. 

**App Service networking features**

Most plans deploy apps in shared servers, so different apps may have same network. That is why connecting and configuring the network are available only in App Service Environment (Iso IsoV2). But in other plans the network can be configured through **networking features**. The features dedicated to configure calls TO app can’t be used to configure calls FROM app, same for opposite.

Inbound features:
- App-assigned address (configure unshared dedicated inbound address, etc.)
- Access restrictions (Restrict access to your app from a set of well-defined addresses)
- Service endpoints (Restrict access to your app from resources in a virtual network)
- Private endpoints (Expose your app on a private IP in your virtual network)
Outbound features:
- Virtual network integration (Access resources in an Azure virtual network in the same region, etc. This is useful when your resource like Azure SQL db is not available via internet - public ip, than you connect within inner virtual network)
- Gateway-required virtual network integration (Access resources in an Azure virtual network in a different region)
- Hybrid Connections (Access resources in a private network that's not connected to Azure)


**Worker** – a VM managed by Azure.
**DNS** **–** domain name system. It converts entered e. g. in browser domain like [https://yourapp.azurewebsites.net](https://yourapp.azurewebsites.net) to real ip address. The requests go to real ip address.  
![[Pasted image 20250427090628.png|400]]
**Inbound IP** – the IP that users can use to reach the app. AAS by default don't have a guaranteed dedicated Inbound IP — it's **shared**. You get a dedicated IP with App Service Environment. When client enter the azure app URL, the Azure DNS resolves it to IP address.  Multiple plans of different users can use same Inbound IP.

> [!info] ChatGPT 
> 1.     User accesses your app via a domain like:  [https://yourapp.azurewebsites.net](https://yourapp.azurewebsites.net)
> 2.     The DNS lookup resolves this to a shared inbound IP, like:  `20.42.72.18`
> 3. The request hits Azure’s **App Service Front End**, which is a **load balancer** that:  
> - Accepts traffic on that IP 
> - Inspects the **Host header** in the HTTP request:  
> `Host: yourapp.azurewebsites.net`
> 4. Azure knows which app matches `yourapp.azurewebsites.net` and routes the request to your specific app instance behind the scenes. 


 
**Outbound IP** – the IP from which app send requests to another resources out of App Service (API, servers, apps). If the app makes a request like:

`HttpClient.Get("https://api.something.com/data");`

The request will appear to come from one of the outbound IP addresses assigned to your App Service.

Outbound IPs are shared by all apps running on the same worker (or workers running within same unit scale – plan).    

``` sh
#the outbound IP addresses currently used by app (equal to all used outbound IP used by plan)
az webapp show \
    --resource-group <group_name> \
    --name <app_name> \
    --query outboundIpAddresses \
    --output tsv #tsv fomrat - tab separated values
```

A **resource group** is like a **folder** or **container** in Azure that holds all the related resources for an application or project (App Service app, App Service plan, accounts, databases, Key Vaults, Application Insights…).

For most plans the Azure might change the current used outbound IPs e. g. when switching the Plan tier or during scale in/out. But the new outbound IPs are taken from limited list of all possible outbound IPs for current scale unit.
``` sh
#Check all possible outbound Ips of plan (enter any app_name from plan)
az webapp show \
    --resource-group <group_name> \
    --name <app_name> \
    --query possibleOutboundIpAddresses \
    --output tsv
```

Azure App Service **does not guarantee fixed outbound IPs.** IPs might be changed without any interruption from developer.****

If you're using this IP to whitelist firewalls or APIs, you should always whitelist the full possibleOutboundIpAddresses set, not just the current active ones.

> [!quote] 
> App data like IPs are shown also on azure platform: ![[Pasted image 20250613185847.png]] 

**Deployment unit** or **webspace** - set of different customers plans that have same single **inbound IP** and same set of **outbound IPs**. (In Isolated case the single plan have dedicated own deployment unit)

> [!quote] **When inbound IP/ set of outbound IPs changes**
>- Delete an app and recreate it in a different resource group (deployment unit may change).
>- Delete the last app in a resource group _and_ region combination and recreate it (deployment unit may change). 
>- Changing the region of plan requires creating a new plan, so the list of possible outbound IPs will change as well.
>- only for **outbound**: The worker virtual machines are broken down in large part by the App Service plans. The Free, Shared, Basic, Standard, and Premium plans all use the same worker virtual machine type. The PremiumV2 plan uses another virtual machine type. PremiumV3 uses yet another virtual machine type. When you change the virtual machine family, you get a different set of outbound addresses. 
>- In some older scale units, both the **inbound** and **outbound** addresses change when you scale from Standard to PremiumV2.
>- only for **inbound**: Delete an existing IP-based TLS binding, such as during certificate renewal.
>-  only for **outbound**: Scale your app between the lower tiers (**Basic**, **Standard**, and **Premium**), the **PremiumV2** tier, the **PremiumV3** tier, and the **Pmv3** options within the **PremiumV3** tier (IP addresses may be added to or subtracted from the set).

There are several ways to guarantee static outbound IP, e. g. using App Service Environment.

The command `> az webapp up` without parameters performs the following actions:
·       Create a default resource group if one isn't specified.
·       Create a default app service plan.
·       Create an app with the specified name.
·       Zip deploy files from the current working directory to the web app.

## Creation of webapp in App Service UI
> [!info] ChatGPT 
> The "Create Web App" option in Azure App Service simply provisions the hosting **infrastructure (App Service + Plan)** After creating the Web App, you can **deploy your own repo**. 

![[Pasted image 20250611190347.png]]
![[Pasted image 20250611190359.png]]
![[Pasted image 20250611190423.png]]
![[Pasted image 20250611190432.png]]
![[Pasted image 20250611190440.png]]
Virtual network is not faster, but you may want do disable public access to your Azure SQL db for security reason, than you need use Virtual network integration feature to connect webapp to Azure SQL resource through inner virtual network. Recommended to connect via internet but with connection ip whitelist. 

![[Pasted image 20250611190702.png]]
![[Pasted image 20250611190715.png]]
## 1.2      Application settings configuration

**App settings** – variables passed as environment variables to the application code, they're injected into app environment at app startup. When you add, remove, or edit app settings, App Service triggers an app restart. They are always encrypted when stored. App settings example:

``` json
[
	{
		"name": "ConnectionStrings:MyDb",
		"value": "Server=myserver.database.windows.net;Database=mydb;UserId=myuser;Password=mypassword;",
		"slotSetting": false
	}

]
```

![[Pasted image 20250427101748.png|500]]


![[Pasted image 20250427101808.png|300]]

For ASP.NET and ASP.NET Core developers, setting app settings in App Service is like setting them in `<appSettings>` in _Web.config_ or _appsettings.json_, but the values in App Service override the ones in _Web.config_ or _appsettings.json_. You can keep development settings (for example, local MySQL password) in _Web.config_ or _appsettings.json_ and production secrets (for example, Azure MySQL database password) safely in App Service.

App settings are always encrypted when stored (encrypted-at-rest). App settings names can only contain letters, numbers (0-9), periods ("."), and underscores ("\_") Special characters in the value of an App Setting must be escaped as needed by the target OS.

The following setting:

``` json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  }
}
```

In Azure environmental variables must be written as:

``` json
{
  "name": "Logging:LogLevel:Default",
  "value": "Debug",
  "slotSetting": false
}
```

And in ASP.NET app you can access them by:

`var level = Configuration["Logging:LogLevel:Default"];`

But in Linux Systems environment variable names can’t contain colons “:”. The AAS in Linux-based plan (or in Linux custom containers) will inject name as

    "name": " Logging__LogLevel__Default ",

Exchanging : to double _

Connection strings can be written in special way – add “type” field and use keywords for certain db engine:

```json
{
  "name": "ICOM-CS",
  "value": "conn-string-1",
  "type": "SQLServer",
  "slotSetting": false
}
```

App service creates variable with prefix and you can access this CS:

```csharp
var conn = Environment.GetEnvironmentVariable("SQLCONNSTR_ICOM-CS");
```

·       SQLServer: `SQLCONNSTR_`
·       MySQL: `MYSQLCONNSTR_`
·       SQLAzure: `SQLAZURECONNSTR_`
·       Custom: `CUSTOMCONNSTR_`
·       PostgreSQL: `POSTGRESQLCONNSTR_`
·       Notification Hub: `NOTIFICATIONHUBCONNSTR_`
·       Service Bus: `SERVICEBUSCONNSTR_`
·       Event Hub: `EVENTHUBCONNSTR_`
·       Document DB: `DOCDBCONNSTR_`
·       Redis Cache: `REDISCACHECONNSTR_`

``` sh
#Add environmental variable in Bash
az webapp config appsettings set \
  --resource-group <group-name> \
  --name <app-name> \
  --settings key1=value1 key2=value2
```

**General settings**

Stack settings:
![[Pasted image 20250427110429.png|275]]
  

**Startup Command** - _(Linux/Custom Container)_

Platform settings:

-    **Platform bitness**: 32-bit or 64-bit. For Windows apps only.
-    **FTP state**: Allow only FTPS or disable FTP altogether.
-    **HTTP version**: Set to **2.0** to enable support for HTTPS/2 protocol.
-    **Web sockets**: For ASP.NET SignalR or socket.io, for example.
-    **Always On**: Keeps the app loaded even when there's no traffic. When **Always On** isn't turned on (default), the app is unloaded after 20 minutes without any incoming requests. The unloaded app can cause high latency for new requests because of its warm-up time. When **Always On** is turned on, the front-end load balancer sends a GET request to the application root every five minutes.
-   **ARR affinity**: In a multi-instance deployment, ensure that the client is routed to the same instance for the life of the session. You can set this option to **Off** for stateless applications.
-   **HTTPS Only**
-   **Minimum TLS version**

**Debugging**: Enable remote debugging for ASP.NET, ASP.NET Core, or Node.js apps. This option turns off automatically after 48 hours.

**Incoming client certificates: turns on** TLS mutual authentication. **By default, the server has a TLS certificate and client validates it. With** TLS mutual authentication server requires a TLS from client and communicates only if trust the client certificate. Trusted certificates are loaded in AAS.

## 1.3      Configuration > Path mappings

App repo are deployed to D:\home\site\wwwroot. To access files in root in code use

        var path = Path.Combine(_env.WebRootPath, "myfile.txt");

where _env  is IWebHostEnvironment that must be added through DI (not available in .NET Framework)

If your app root is in a different folder, or if your repository has more than one application, you can edit or add virtual applications and directories.

Public files like .js .html. .png can be accessible via URL:

[https://rekrutacja.invent.ag/Content/img/cz.png](https://rekrutacja.invent.ag/Content/img/cz.png) (returns picture)

In **Configuration** **->** **Path mappings**  you can define handler mappings for uncontainerized Windows apps that will define how to handle url access to files with selected extension. URL example:

https://yourapp.azurewebsites.net/myapp/index.php

this is a php script that won’t be handled properly without handler mappings because Azure App Service is **primarily for .NET, Node.js, Python, etc.**

But you can add php script index.php and define handler mapping for php with properties:

·       Extension: *.php

·       Script Processor: D:\home\site\wwwroot\php\php-cgi.exe

·       Arguments: (optional) like -f or --some-flag

Azure will run php-cgi.exe with your file as input.

For standard languages (like ASP.NET, Python), Azure already sets up the handler for you.

**Files storage**

Windows apps are deployed to D:\home\site\wwwroot and every redeploy, scaling out etc. will wipe out this path and load files again. But you have an access to D:\home and you can place files in other paths – the will persist after redeploying.

It doesn’t work in containerized apps. Containerized apps include all Linux apps and also the Windows and Linux custom containers running on App Service. In containerized apps you need to add **Azure Storage Mount ­– it’s the network file share (like local file system but in network).**

**Mount can have and choose type Azure Blobs or Azure Files**. Windows container apps only support Azure Files. Azure Blobs Mount is not the same as Azure Blobs.  Azure Blobs Mount in AAS is read-only and good for static files.    

Why would I need Storage Mount?

1.     **With a storage mount**. your data (e.g., logs, user uploads) lives ****outside the container****, so it ****survives restarts, redeployments, or scaling****.

2.     Sharing data between instances. If your app scales out (e.g., 3+ instances), each instance runs in a separate container. You might need to share files between them (e.g., a shared image folder)

3.     Separate code and data. Sometimes it’s helpful to **keep your application code and runtime separate from your data****,** like:  
- Configuration files  
- Large media files  
You don’t have to **rebuild your container** just to update data.

Example:

-       You have a photo-sharing app that lets users upload pictures

-       You want those photos to be available to all app instances and persist even after redeployment

-       So, you mount an Azure File Share to /mnt/photos in your container

You can use Azure Storage Mount also on windows app if the default file system not enough.

**Azure Storage Mount Cofigurations**

·       **Name**: The display name.

·       **Configuration options**: **Basic** or **Advanced**. Select **Basic** if the storage account isn't using service endpoints, private endpoints, or Azure Key Vault. Otherwise, select **Advanced**.

·       **Storage accounts**: The storage account with the container you want.

·       **Storage type**: **Azure Blobs** or **Azure Files**. Windows container apps only support Azure Files. Azure Blobs only supports read-only access.

·       **Storage container**: For basic configuration, the container you want.

·       **Share name**: For advanced configuration, the file share name.

·       **Access key**: For advanced configuration, the access key.

·       **Mount path**: The absolute path in your container to mount the custom storage.

·       **Deployment slot setting**: When checked, the storage mount settings also apply to deployment slots.
