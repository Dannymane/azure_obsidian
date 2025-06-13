## Azure CLI

Login to Azure account in CLI:
just 
```bash
az login
```
hasn't worked because of Multi-Factor Authentication (MFA) if required on my account

To login in use this - it will redirect to browser for login:
```bash
az login --use-device-code
```
