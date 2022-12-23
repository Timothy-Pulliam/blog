## Authenticate Python apps

In order to access Azure resrouces, we must authenticate our application.

![Auth Strategies](/Timothy-Pulliam/azure-python-sdk/python-sdk-auth-strategy.png "Auth Strategies")

For local development, we can authenticate our application by simply using `az login`. The Python SDK will find our credentials in the `~/.azure/` directory.

There are many ways to pass credentials to our python code. The easiest way is to use [DefaultAzureCredential](https://learn.microsoft.com/en-us/python/api/azure-identity/azure.identity.defaultazurecredential?view=azure-python). This allows us to use credentials found in the `~/.azure/` directory after a successful `az login`. DefaultAzureCredential looks for credentials in the following order and provides and provides the user with an access token.

1) A service principal configured by environment variables. See EnvironmentCredential for more details.

2) An Azure managed identity. See ManagedIdentityCredential for more details.

3) On Windows only: a user who has signed in with a Microsoft application, such as Visual Studio. If multiple identities are in the cache, then the value of the environment variable AZURE_USERNAME is used to select which identity to use. See SharedTokenCacheCredential for more details.

4) The user currently signed in to Visual Studio Code.

5) **The identity currently logged in to the Azure CLI.**

6) The identity currently logged in to Azure PowerShell.

We will be authenticating using option number 5, as it is the easiest to set up.

```bash
$ pip install azure-identity
```

```python
from azure.identity import DefaultAzureCredential
from azure.storage.blob import BlobServiceClient

# Acquire a credential object
credential = DefaultAzureCredential()

blob_service_client = BlobServiceClient(
        account_url="https://<my_account_name>.blob.core.windows.net",
        credential=credential)
```