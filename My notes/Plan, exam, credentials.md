daniel:User1234

# 1       Plan

4 modules every week

[https://skillcertpro.com/product/developing-solutions-for-microsoft-azure-az-204-practice-exam-test/](https://skillcertpro.com/product/developing-solutions-for-microsoft-azure-az-204-practice-exam-test/) (not recommended)

Learn downloaded git repo [https://github.com/arvigeus/AZ-204](https://github.com/arvigeus/AZ-204)

Take test (the answers in git repo as well) [https://az-204.vercel.app/?id=585a8c1fb45e87c4d5f9c9134633e8239f53275f34a6df54c3c7db350c4d1100](https://az-204.vercel.app/?id=585a8c1fb45e87c4d5f9c9134633e8239f53275f34a6df54c3c7db350c4d1100)

Good practice test (if not enough) [https://www.whizlabs.com/microsoft-azure-certification-az-204/](https://www.whizlabs.com/microsoft-azure-certification-az-204/)

Test questions are downloaded in pdf (2025), more tests if not enough: [https://www.dumps-files.com/files/AZ-204/](https://www.dumps-files.com/files/AZ-204/)

# 2       Exam

Free practical assessment (test exam) [https://learn.microsoft.com/en-ca/credentials/certifications/exams/az-204/practice/assessment?assessment-type=practice&assessmentId=35](https://learn.microsoft.com/en-ca/credentials/certifications/exams/az-204/practice/assessment?assessment-type=practice&assessmentId=35)

Channel with practice questions [https://www.youtube.com/watch?v=si37TjkmWBA](https://www.youtube.com/watch?v=si37TjkmWBA)and good video [https://www.youtube.com/watch?v=k4Gm7HnIpcg](https://www.youtube.com/watch?v=k4Gm7HnIpcg)

# 3       Azure SQL Server

Server name: dy-azure-sql-server

Db name: sr-db

Local db can be saved as .bacpac file (MS SQL Server Management Studio -> right click on db -> Tasks -> Export Data Tier application), but to restore from .bacpac file you must:
-       Load .bacpac to Azure Storage – Blob Container,
-       Create new Azure SQL database ,
-       During creation configuration select import from and select file in Azure Storage.

 To create db from code-first app:
-       Create empty db,
-       Paste connection string to your app, don't forget to set new db name, otherwise the command below will create new db on Azure server with default pricing tier. You can find the connection string in Azure SQL -> your db -> Settings
-       Execute EF CLI command like  

dotnet ef database update --project SchoolRegister.DAL --startup-project SchoolRegister.Web

It will build program and create all tables and relations in db.

# 4       Azure Storage      

Account name: dystorage

# 5 App Service
App name: schoolRegister
## Details

Subscription
b36acc92-a970-465a-9d17-106df4854e76

Resource Group
main-rg

Name
dy-school-register

Secure unique default hostname
Enabled

Publish
Code

Runtime stack
.NET 8 (LTS)

## App Service Plan (New)

Name
ASP-mainrg-af33

Operating System
Windows

Region
Poland Central

SKU
Free

ACU
Shared infrastructure

Memory
1 GB memory

## Monitor + secure

Application Insights
Not enabled

## Deployment

Basic authentication
Disabled

Continuous deployment
Not enabled / Set up after app creation