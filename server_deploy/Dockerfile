# BASE: Ubuntu stabile
FROM ubuntu:22.04

# Evita domande interattive
ENV DEBIAN_FRONTEND=noninteractive

# Installa pacchetti base + SSH + sudo + dipendenze Docker
RUN apt-get update && apt-get install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release \
    openssh-server \
    sudo \
    iptables \
    uidmap \
    && rm -rf /var/lib/apt/lists/*

# Installa Docker Engine (script ufficiale)
RUN curl -fsSL https://get.docker.com | sh

# Attiva cartella SSH
RUN mkdir -p /var/run/sshd

# Crea utente sshuser
RUN useradd -m -s /bin/bash sshuser && \
    usermod -aG docker sshuser && \
    echo "sshuser ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# Prepara directory .ssh
RUN mkdir -p /home/sshuser/.ssh && \
    chmod 700 /home/sshuser/.ssh && \
    chown sshuser:sshuser /home/sshuser/.ssh

# Copia chiave pubblica (come secret)
RUN --mount=type=secret,id=ssh_pub_key \
    cat /run/secrets/ssh_pub_key > /home/sshuser/.ssh/authorized_keys && \
    chown sshuser:sshuser /home/sshuser/.ssh/authorized_keys && \
    chmod 600 /home/sshuser/.ssh/authorized_keys

# Genera le host keys
RUN ssh-keygen -A

# Espone SSH
EXPOSE 22

# Avvia Docker daemon + SSH
CMD ["/bin/bash", "-c", "dockerd & exec /usr/sbin/sshd -D"]

