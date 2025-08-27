# This stage creates a user and exposes ports for the final image.
# Using 'AS base' allows the final stage to reference this configuration.
FROM mcr.microsoft.com/dotnet/aspnet:9.0 AS base
USER app
WORKDIR /app
EXPOSE 8080
EXPOSE 8081

# This is the build stage. It's separate from the runtime to keep the final image small.
FROM mcr.microsoft.com/dotnet/sdk:9.0 AS build
# A build argument that allows you to specify the build configuration (e.g., Debug or Release).
ARG BUILD_CONFIGURATION=Release
WORKDIR /src

# Copy the project file first to leverage Docker's layer caching.
COPY ["Wpm.Management.Api/Wpm.Management.Api.csproj", "Wpm.Management.Api/"]
# Restore dependencies. This only runs if the project file changes.
RUN dotnet restore "Wpm.Management.Api/Wpm.Management.Api.csproj"

# Copy the rest of the source code.
COPY . .
# Set the working directory to the project folder.
WORKDIR "/src/Wpm.Management.Api"
# Build and then publish the application to the /app/publish directory.
# The BUILD_CONFIGURATION argument is used here to select the correct configuration.
RUN dotnet build "Wpm.Management.Api.csproj" -c $BUILD_CONFIGURATION -o /app/build
RUN dotnet publish "Wpm.Management.Api.csproj" -c $BUILD_CONFIGURATION -o /app/publish

# This is the final stage that will be used for the production container.
# It's based on the 'base' stage created earlier, which has the user and ports configured.
FROM base AS final
WORKDIR /app
# Copy the published application from the 'build' stage to the final image.
COPY --from=build /app/publish .
# Set the command that will run when the container starts.
ENTRYPOINT ["dotnet", "Wpm.Management.Api.dll"]
