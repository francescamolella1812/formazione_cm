# build_almalinux/Dockerfile
# syntax=docker/dockerfile:1.4
FROM almalinux:9

# installa openssh-server e sudo
RUN dnf -y install openssh-server sudo && dnf clean all

# genera host keys SSH
RUN ssh-keygen -A

# crea utente sshuser e directory .ssh
RUN useradd -m -s /bin/bash sshuser
RUN mkdir -p /home/sshuser/.ssh && chmod 700 /home/sshuser/.ssh
RUN mkdir -p /run/sshd

# usa secret per la chiave pubblica (non rimane nei layer)
RUN --mount=type=secret,id=ssh_pub_key \
    cat /run/secrets/ssh_pub_key > /home/sshuser/.ssh/authorized_keys && \
    chown sshuser:sshuser /home/sshuser/.ssh/authorized_keys && \
    chmod 600 /home/sshuser/.ssh/authorized_keys

# configura sudo senza password
RUN echo "sshuser ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/sshuser && chmod 0440 /etc/sudoers.d/sshuser

# disabilita login root con password
RUN sed -i 's/^#PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config || true
RUN sed -i 's/^#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config || true

EXPOSE 22
CMD ["/usr/sbin/sshd","-D"]

