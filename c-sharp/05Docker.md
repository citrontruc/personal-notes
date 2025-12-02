# Docker

You first have to build your app and then run that build in another image that is lighter.

```docker
# ----------------------------------------------------------------------------------
# Stage 1: Build Stage
# Uses the .NET 9 SDK image to compile the application and publish artifacts.
# Use this stage for tools needed ONLY for compilation (e.g., Git, specific build tools).
# ----------------------------------------------------------------------------------
FROM mcr.microsoft.com/dotnet/sdk:9.0 AS build
WORKDIR /src

# Example of an installation needed ONLY for the build process:
# RUN apt-get update && apt-get install -y git && rm -rf /var/lib/apt/lists/*

# Copy the CSPROJ file first and restore dependencies.
COPY *.csproj .
RUN dotnet restore

# Copy the rest of the source code.
COPY . .

# Publish the application in Release configuration.
RUN dotnet publish -c Release -o /app/publish --no-restore

# ----------------------------------------------------------------------------------
# Stage 2: Final Stage (Includes Non-Root User for Security)
# Uses the much smaller ASP.NET Runtime image for the final container.
# Use this stage for packages required for the app to run (e.g., libraries, fonts).
# ----------------------------------------------------------------------------------
FROM mcr.microsoft.com/dotnet/aspnet:9.0 AS final
WORKDIR /app

# --- RUNTIME DEPENDENCIES (Place apt installs here) ---
# This is the most common place for apt installations, as runtime libraries 
# are required for the application to function.
# IMPORTANT: Run these commands as root (before USER app) and clean up the cache 
# in the same command to keep the image size small.
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        libgdiplus \
        # Add any other runtime packages here
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*
# -----------------------------------------------------

# --- SECURITY ENHANCEMENT ---
# Switch to the built-in 'app' non-root user provided by the base image.
USER app
# ----------------------------

# Expose the default port for ASP.NET Core applications
EXPOSE 8080

# Copy the published artifacts from the 'build' stage into the final container.
COPY --from=build --chown=app:app /app/publish .

# Set the entrypoint to run the built application.
ENTRYPOINT ["dotnet", "YourApp.dll"]
```
