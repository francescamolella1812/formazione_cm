# build_ubuntu/Dockerfile
# syntax=docker/dockerfile:1.4

FROM ubuntu:22.04

RUN apt-get update && apt-get install -y openssh-server sudo && rm -rf /var/lib/apt/lists/*

RUN ssh-keygen -A
RUN useradd -m -s /bin/bash sshuser
RUN mkdir -p /home/sshuser/.ssh && chmod 700 /home/sshuser/.ssh
RUN mkdir -p /run/sshd
RUN echo "sshuser ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/sshuser && chmod 0440 /etc/sudoers.d/sshuser
RUN sed -i 's/^#PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config || true
RUN sed -i 's/^#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config || true

# Usa la chiave pubblica come secret
RUN --mount=type=secret,id=ssh_pub_key \
    cat /run/secrets/ssh_pub_key > /home/sshuser/.ssh/authorized_keys && \
    chown sshuser:sshuser /home/sshuser/.ssh/authorized_keys && \
    chmod 600 /home/sshuser/.ssh/authorized_keys

EXPOSE 22
CMD ["/usr/sbin/sshd","-D"]

