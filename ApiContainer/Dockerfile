# See https://aka.ms/customizecontainer to learn how to customize your debug container and how Visual Studio uses this Dockerfile to build your images for faster debugging.

# This stage is used when running from VS in fast mode (Default for Debug configuration)
# FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
# USER $APP_UID
# WORKDIR /app
# EXPOSE 8080
# EXPOSE 8081


# # This stage is used to build the service project
# FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
# ARG BUILD_CONFIGURATION=Release
# WORKDIR /src
# COPY ["ApiContainer/ApiContainer.csproj", "ApiContainer/"]
# RUN dotnet restore "./ApiContainer/ApiContainer.csproj"
# COPY . .
# WORKDIR "/src/ApiContainer"
# RUN dotnet build "./ApiContainer.csproj" -c $BUILD_CONFIGURATION -o /app/build

# # This stage is used to publish the service project to be copied to the final stage
# FROM build AS publish
# ARG BUILD_CONFIGURATION=Release
# RUN dotnet publish "./ApiContainer.csproj" -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=false

# # This stage is used in production or when running from VS in regular mode (Default when not using the Debug configuration)
# FROM base AS final
# WORKDIR /app
# COPY --from=publish /app/publish .
# ENTRYPOINT ["dotnet", "ApiContainer.dll"]


# Base image with ASP.NET 8 runtime
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
EXPOSE 80

# Build image with .NET 8 SDK
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY ["ApiContainer/ApiContainer.csproj", "ApiContainer/"]
WORKDIR /src/ApiContainer
RUN dotnet restore
COPY . .
RUN dotnet publish -c Release -o /app/publish

# Final runtime image
FROM base AS final
WORKDIR /app
COPY --from=build /app/publish .
ENTRYPOINT ["dotnet", "ApiContainer.dll"]

# Optional: health check
# HEALTHCHECK CMD curl --fail http://localhost/health || exit 1
