# Copyright 2020 Google LLC
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

FROM python:3.9-slim as base

FROM base as builder

RUN apt-get -qq update \
    && apt-get install -y --no-install-recommends \
        wget g++ \
    && rm -rf /var/lib/apt/lists/*

# Download the grpc health probe
RUN GRPC_HEALTH_PROBE_VERSION=v0.4.11 && \
    wget -qO/bin/grpc_health_probe https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/${GRPC_HEALTH_PROBE_VERSION}/grpc_health_probe-linux-amd64 && \
    chmod +x /bin/grpc_health_probe

# get packages
COPY requirements.txt .
RUN pip install -r requirements.txt
RUN pip install --no-cache-dir newrelic
RUN pip install --no-cache-dir opencensus-ext-newrelic

FROM base as final
# Enable unbuffered logging
ENV PYTHONUNBUFFERED=1
# Enable Profiler
ENV ENABLE_PROFILER=1

WORKDIR /email_server

# Grab packages from builder
COPY --from=builder /usr/local/lib/python3.9/ /usr/local/lib/python3.9/
COPY --from=builder /bin/grpc_health_probe /bin/grpc_health_probe

ENV NEW_RELIC_CONFIG_FILE='/email_server/newrelic.ini'
ENV NEW_RELIC_INSERT_KEY='1e13529aa9c622f52ad2ab475d2bb7b66ab9NRAL'
ENV GCP_PROJECT_ID='imperial-data-362815'

# Add the application
COPY . .

EXPOSE 8080
# ENTRYPOINT ["newrelic-admin", "run-program"]
CMD ["python", "email_server.py"]
