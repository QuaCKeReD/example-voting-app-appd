# because of dotnet, we always build on amd64, and target platforms in cli
# dotnet doesn't support QEMU for building or running. 
# (errors common in arm/v7 32bit) https://github.com/dotnet/dotnet-docker/issues/1537
# https://hub.docker.com/_/microsoft-dotnet
# hadolint ignore=DL3029
# to build for a different platform than your host, use --platform=<platform>
# for example, if you were on Intel (amd64) and wanted to build for ARM, you would use:
# docker buildx build --platform "linux/arm64/v8" .
FROM --platform=${BUILDPLATFORM} mcr.microsoft.com/dotnet/sdk:8.0 as build
ARG TARGETPLATFORM
ARG TARGETARCH
ARG BUILDPLATFORM
RUN echo "I am running on $BUILDPLATFORM, building for $TARGETPLATFORM"

WORKDIR /source
COPY *.csproj .
RUN dotnet restore -a $TARGETARCH

COPY . .
RUN dotnet publish -c release -o /app -a $TARGETARCH --self-contained false --no-restore

# app image
FROM mcr.microsoft.com/dotnet/runtime:8.0
WORKDIR /app
COPY --from=build /app .

# Copy agent binaries to the image from current folder
COPY libappdprofiler.so .
COPY AppDynamics.Agent.netstandard.dll .
# COPY Worker.AppDynamicsConfig.json .

# Mandatory settings required to attach the agent to the .NET application
ENV CORECLR_PROFILER={57e1aa68-2229-41aa-9931-a6e93bbc64d8}
ENV CORECLR_ENABLE_PROFILING=1
ENV CORECLR_PROFILER_PATH=/app/libappdprofiler.so

# Configure connection to the controller
ENV APPDYNAMICS_CONTROLLER_HOST_NAME=34.105.231.232
ENV APPDYNAMICS_CONTROLLER_PORT=8090
ENV APPDYNAMICS_CONTROLLER_SSL_ENABLED=false
ENV APPDYNAMICS_AGENT_ACCOUNT_NAME=customer1
ENV APPDYNAMICS_AGENT_ACCOUNT_ACCESS_KEY=2d98af63-beb9-4748-bc50-68c3e61ab9a1

# Configure application identity in AppDynamics
ENV APPDYNAMICS_AGENT_APPLICATION_NAME="VotingApp"
ENV APPDYNAMICS_AGENT_TIER_NAME="worker"
ENV APPDYNAMICS_AGENT_REUSE_NODE_NAME=true
ENV APPDYNAMICS_AGENT_REUSE_NODE_NAME_PREFIX="worker"

ENTRYPOINT ["dotnet", "Worker.dll"]
