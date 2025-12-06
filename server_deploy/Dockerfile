# syntax=docker/dockerfile:1.4
FROM ubuntu:22.04

# ------------------------------
# Install OS utilities + SSH + sudo + Git
# ------------------------------
RUN apt-get update && apt-get install -y \
    openssh-server sudo curl gnupg lsb-release ca-certificates \
    git python3 python3-pip sshpass software-properties-common \
    && rm -rf /var/lib/apt/lists/*

# ------------------------------
# Install Ansible
# ------------------------------
RUN pip3 install ansible

# ------------------------------
# Install Java (required by Jenkins remoting)
# ------------------------------
RUN apt-get update && apt-get install -y openjdk-17-jre-headless \
    && rm -rf /var/lib/apt/lists/*

# ------------------------------
# Install ONLY Docker CLI (NO dockerd)
# ------------------------------
RUN curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
      | gpg --dearmor -o /usr/share/keyrings/docker-cli.gpg && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-cli.gpg] \
      https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) stable" \
      > /etc/apt/sources.list.d/docker.list && \
    apt-get update && apt-get install -y docker-ce-cli

# ------------------------------
# Create user for Jenkins agent
# ------------------------------
RUN useradd -m -s /bin/bash sshuser

# Allow sudo without password
RUN echo "sshuser ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/sshuser && \
    chmod 0440 /etc/sudoers.d/sshuser

# ------------------------------
# SSH configuration
# ------------------------------
RUN ssh-keygen -A

RUN mkdir -p /home/sshuser/.ssh && \
    chown sshuser:sshuser /home/sshuser/.ssh && \
    chmod 700 /home/sshuser/.ssh

RUN mkdir -p /run/sshd && chmod 755 /run/sshd

RUN --mount=type=secret,id=ssh_pub_key \
    cat /run/secrets/ssh_pub_key > /home/sshuser/.ssh/authorized_keys && \
    chown sshuser:sshuser /home/sshuser/.ssh/authorized_keys && \
    chmod 600 /home/sshuser/.ssh/authorized_keys

RUN sed -i 's/^#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config && \
    sed -i 's/^#PubkeyAuthentication yes/PubkeyAuthentication yes/' /etc/ssh/sshd_config

EXPOSE 22

CMD ["/usr/sbin/sshd", "-D"]

