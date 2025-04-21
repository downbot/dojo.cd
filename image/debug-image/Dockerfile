FROM alpine:latest

# Set timezone
ENV TZ="America/Buenos_Aires"
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# Install debugging tools
RUN apk add --no-cache \
    bash \
    curl wget \
    vim nano \
    net-tools tcpdump bind-tools iputils \
    strace lsof \
    socat \
    jq yq \
    htop procps \
    zip unzip \
    ca-certificates && \
    update-ca-certificates

# Add a non-root user for security purposes
RUN addgroup -S debug && adduser -S debug -G debug
USER debug

# Set the default shell to bash
CMD ["/bin/bash"]
