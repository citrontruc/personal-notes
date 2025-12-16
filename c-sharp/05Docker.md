# Docker

You first have to build your app and then run that build in another image that is lighter.

```docker
# Build step
FROM mcr.microsoft.com/dotnet/sdk:9.0 AS build
WORKDIR /src
COPY TodoApi.sln .
COPY TodoApi.csproj .

# Put restore before publish to cache the step
RUN dotnet restore TodoApi.sln
COPY . .
RUN dotnet publish TodoApi.csproj -c Release -o /app/publish --no-restore

##########################################################################
# Run step
FROM mcr.microsoft.com/dotnet/aspnet:9.0 AS final
WORKDIR /app
# Do apt installs here.
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        libgdiplus \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Copy published artifacts from build (done as root so ownership is set correctly)
COPY --from=build /app/publish/ /app/

# Add a non-root user and give him rights on our project.
RUN addgroup --gid 1000 nonroot && \
    adduser nonroot --system -gid 1000 --home /home/nonroot && \
    chown -R nonroot:nonroot /app
USER nonroot

# Can't use https without a certificate, we default to http for now
EXPOSE 5158

# Set the entrypoint to run the built application.
ENTRYPOINT ["dotnet", "TodoApi.dll"]
```
