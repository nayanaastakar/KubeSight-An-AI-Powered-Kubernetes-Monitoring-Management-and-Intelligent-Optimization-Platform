
# Stage 1: Get Trivy from the official image
FROM aquasec/trivy:0.58.0 AS trivy

# Stage 2: Backend application
FROM python:3.11-slim

WORKDIR /app

# Copy Trivy binary
COPY --from=trivy /usr/local/bin/trivy /usr/local/bin/trivy

# Install system dependencies, kubectl and Helm
RUN apt-get update && apt-get install -y \
    curl \
    wget \
    git \
    && curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" \
    && chmod +x kubectl \
    && mv kubectl /usr/local/bin/ \
    && curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

RUN chmod +x entrypoint.sh

RUN useradd -m -u 1000 kubesight \
    && ln -s /home/kubesight /home/nextsight \
    && chown -R kubesight:kubesight /app /home/kubesight

USER kubesight

ENV PYTHONUNBUFFERED=1
ENV K8S_IN_CLUSTER=true

EXPOSE 8000

ENTRYPOINT ["./entrypoint.sh"]
