FROM mcr.microsoft.com/dotnet/sdk:7.0 as build

# Args pour le build
ARG TARGETPLATFORM
ARG TARGETARCH
ARG BUILDPLATFORM

# Print des build args (on utilise run ?)
RUN echo "TARGETPLATFORM=$TARGETPLATFORM" && \
    echo "TARGETARCH=$TARGETARCH" && \
    echo "BUILDPLATFORM=$BUILDPLATFORM"

WORKDIR /app/source

COPY . /app/source/

# Build de l'application
RUN dotnet restore -a $TARGETARCH && \
    dotnet publish -c release -o /app -a $TARGETARCH --self-contained false --no-restore

FROM mcr.microsoft.com/dotnet/runtime:7.0

# Copier le build de l'application
WORKDIR /app
COPY --from=build /app .

ENTRYPOINT ["dotnet", "Worker.dll"]