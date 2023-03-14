FROM jpoon/oci-cli:latest


RUN apt-get update && \
    apt-get install -y curl gnupg2 wget ca-certificates python3 tar gzip bash && \
    rm -rf /var/lib/apt/lists/*

# Install kubectl
ENV KUBE_LATEST_VERSION="v1.21.5"

RUN curl -L https://storage.googleapis.com/kubernetes-release/release/$KUBE_LATEST_VERSION/bin/linux/amd64/kubectl -o /usr/local/bin/kubectl \
    && chmod +x /usr/local/bin/kubectl


# Add KubeConfig and OCI CLI config
RUN mkdir -p /root/.kube
COPY kubeconfig /root/.kube/config
RUN mkdir -p /root/.oci
COPY ociconfig /root/.oci/config
COPY private /root/.oci/private
RUN chmod 600 /root/.oci/private

ENTRYPOINT ["kubectl"]
