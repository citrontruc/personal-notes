# Azure

## table of content

- [Azure](#azure)
  - [table of content](#table-of-content)
  - [Keyvault](#keyvault)
    - [Accessing keyvault](#accessing-keyvault)
    - [Using keyvault](#using-keyvault)

## Keyvault

### Accessing keyvault

In order to access your keyvault, you need to define a shared identity so that the caller can access your keyvault. Go in "Access policies", create an access policy (with Get and List as access rights). Assign this identity to your azure app service resource.

### Using keyvault

In order to configure a key vault in your dotnet app, use the following code:

```cs
if (environment != Environments.Development)
{
    var secretClient = new SecretClient(
            new Uri("https://pluralsightdemokeyvault.vault.azure.net/"),
            new DefaultAzureCredential());
            
    builder.Configuration.AddAzureKeyVault(secretClient,
        new KeyVaultSecretManager());
}
```

When choosing a name for your secret in Azure Key vault, use -- to indicate children in a json. For example:

```json
{
    "Authentication" : {
        "SecretForKey" : "value"
    }
}
```

becomes **Authentication--SecretForKey**.
