# Copyright 2021 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# https://mcr.microsoft.com/product/dotnet/sdk
FROM mcr.microsoft.com/dotnet/sdk:8.0.301-noble as builder


# Set default value for PLATFORM
ARG PLATFORM

# Determine the architecture and set the PLATFORM variable accordingly
RUN ARCH=$(uname -m) && \
    if [ "$ARCH" = "x86_64" ]; then \
        echo "linux-amd64" > /etc/platform; \
    elif [ "$ARCH" = "aarch64" ]; then \
        echo "linux-arm64" > /etc/platform; \
    else \
        echo "unknown" > /etc/platform; \
    fi
RUN cat /etc/platform

WORKDIR /app
COPY cartservice.csproj .
RUN export PLATFORM=$(cat /etc/platform) && dotnet restore cartservice.csproj -r $PLATFORM
COPY . .

RUN export PLATFORM=$(cat /etc/platform) && \
    dotnet publish cartservice.csproj \
    -p:PublishSingleFile=true \
    -r $PLATFORM \
    --self-contained true \
    -p:PublishTrimmed=True \
    -p:TrimMode=Full \
    -c release \
    -o /cartservice \
    --no-restore

# https://mcr.microsoft.com/product/dotnet/runtime-deps
FROM mcr.microsoft.com/dotnet/runtime-deps:8.0.6-noble-chiseled

WORKDIR /app
COPY --from=builder /cartservice .
EXPOSE 7070
ENV DOTNET_EnableDiagnostics=0 \
    ASPNETCORE_HTTP_PORTS=7070
USER 1000
ENTRYPOINT ["/app/cartservice"]
