# Formazione CM – Ansible, Jenkins e Container

Questo repository contiene il materiale sviluppato durante il modulo formativo **Configuration Management**, con focus su automazione tramite 
**Ansible**, utilizzo di **Jenkins** e gestione di **container Docker/Podman**.

Il progetto dimostra come:
- costruire immagini container tramite Ansible
- pubblicarle su un registry
- avviare container applicativi
- integrare il tutto in una pipeline Jenkins

---

## Obiettivi del progetto

Gli obiettivi principali del laboratorio sono:

- utilizzare Ansible per orchestrare build, push e run di container
- creare ruoli Ansible modulari e riutilizzabili
- automatizzare la gestione di un registry container
- configurare Jenkins Agent tramite Ansible
- integrare Ansible all’interno di una pipeline Jenkins

---

## Tecnologie utilizzate

- Ansible
- Jenkins
- Docker / Podman
- Git
- YAML
- Linux (AlmaLinux, Ubuntu)

---

## Jenkins Pipeline

Il file `Jenkinsfile` definisce una pipeline dichiarativa che esegue le seguenti fasi:

- checkout del codice sorgente
- build dell’immagine container utilizzando Docker o Podman
- tagging dell’immagine per il registry locale
- push dell’immagine verso il registry locale
- deploy dell’applicazione tramite playbook Ansible

La pipeline utilizza un Jenkins Agent dedicato ed è compatibile sia con Docker che con Podman.

---

## Dockerfile

Sono presenti diversi Dockerfile, utilizzati per scopi differenti:

### build_ubuntu/
Immagine base Ubuntu utilizzata per la build applicativa.

### build_almalinux/
Immagine base AlmaLinux utilizzata per test e ambienti alternativi.

### server_deploy/
Immagine utilizzata come nodo di deploy, contenente:
- SSH server
- Ansible
- Docker CLI
- Java (necessaria per Jenkins Agent)

---

## Playbook Ansible

### container-playbook.yml
Configura un registry Docker locale eseguendo le seguenti operazioni:
- installazione di Docker
- avvio del servizio Docker
- avvio di un registry locale sulla porta 5000

### build_and_run.yml
Esegue le seguenti operazioni:
- build delle immagini Ubuntu e AlmaLinux
- avvio dei container con servizio SSH esposto
- iniezione della chiave SSH pubblica nei container
- verifica dello stato dei container avviati

### deploy_app.yml
Gestisce il deploy applicativo:
- pull dell’immagine dal registry locale
- stop e rimozione del container esistente
- avvio del nuovo container applicativo

---

## Inventory Ansible

Il file `inventory/hosts.ini` contiene la definizione degli host gestiti da Ansible, inclusi:
- host di deploy
- eventuali target utilizzati per test o runtime

---

## Ruoli Ansible

### build_images
Responsabile della build delle immagini container a partire dai Dockerfile.

### push_images
Gestisce il push delle immagini verso il registry container.

### registry
Configura il registry container e gestisce le credenziali di accesso.

### run_containers
Avvia e gestisce i container runtime.

### jenkins_agent
Configura un Jenkins Agent basato su container.

