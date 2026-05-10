# Secrets

## Table of content

- [Secrets](#secrets)
  - [Table of content](#table-of-content)
  - [Add secrets to a project](#add-secrets-to-a-project)

## Add secrets to a project

In order to store secrets in dotnet, go to your csproj file and type the following command:

```sh
dotnet user-secrets init

// Add secrets
dotnet user-secrets set "OpenAI:ApiKey" "secret-key"
dotnet user-secrets set "Db:Password" "password"

// List secrets
dotnet user-secrets list

// edit secrets
dotnet user-secrets edit
```

Secrets should be stored at the following address: <~/.microsoft/usersecrets/%UserSecretsId%/secrets.json>
