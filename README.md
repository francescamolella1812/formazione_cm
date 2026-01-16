# Formazione CM – Ansible, Docker/Podman, Jenkins, CI/CD

Questo repository contiene gli esercizi del modulo formativo "Configuration Management" con focus su:
- gestione di infrastrutture containerizzate con Docker o Podman tramite Ansible
- creazione di playbook e ruoli Ansible riutilizzabili
- configurazione di un registry container locale
- utilizzo di Ansible Vault per cifrare credenziali e file sensibili
- integrazione CI/CD tramite Jenkins per build, push e deploy di immagini

## Prerequisiti

- Docker o Podman installato
- Ansible installato
- accesso a un ambiente in cui eseguire Jenkins (anche in container)

## Struttura del repository

## Modulo formativo – Step e contenuti

### Step 1 – Primo playbook (Docker Registry)

Il playbook `container-playbook.yml` configura un registry Docker locale eseguendo le seguenti operazioni:
- installazione di Docker
- avvio del servizio Docker
- avvio di un container registry sulla porta 5000 (senza autenticazione)

File coinvolti:
- `container-playbook.yml`
- `inventory/hosts.ini`

---

### Step 2 – Build di container (due sistemi operativi)

Il playbook `build_and_run.yml` costruisce almeno due immagini (Ubuntu e AlmaLinux) e avvia container con le seguenti caratteristiche:
- servizio SSH attivo e in ascolto sulla porta 22 del container
- utente dedicato con accesso tramite chiave SSH
- possibilità di eseguire comandi con privilegi sudo

Inoltre il playbook:
- mappa le porte SSH del container su porte host differenti (ad esempio 2222 e 2223)
- inietta automaticamente la chiave pubblica locale nel file `authorized_keys`
- verifica che il servizio SSH sia effettivamente raggiungibile

File coinvolti:
- `build_and_run.yml`
- `build_ubuntu/Dockerfile`
- `build_almalinux/Dockerfile`

---

### Step 3 – Creazione di ruoli Ansible

La directory `roles/` contiene ruoli Ansible che separano le responsabilità in componenti riutilizzabili:

- `registry`  
  Creazione e configurazione del registry container

- `build_images`  
  Build delle immagini container a partire dai Dockerfile

- `push_images`  
  Push delle immagini verso il registry

- `run_containers`  
  Avvio dei container evitando conflitti di porte

- `jenkins_agent`  
  Preparazione e configurazione di un Jenkins Agent basato su container

Nota: un obiettivo del modulo e' rendere i ruoli parametrizzabili e, quando possibile, compatibili sia con Docker che con Podman.

---

### Step 4 – Ansible Vault

Il modulo richiede l'utilizzo di Ansible Vault per cifrare dati sensibili, ad esempio:
- credenziali di accesso al registry
- password degli utenti (se presenti)
- file contenenti variabili riservate

File tipicamente da cifrare:
- `roles/registry/vars/registry_credentials.yml`

---

### Step 5 – Jenkins e Ansible (CI/CD)

Il file `Jenkinsfile` implementa una pipeline dichiarativa che esegue le seguenti operazioni:
- checkout del repository Git
- build di un'immagine container utilizzando Docker o Podman
- tagging dell'immagine con un valore progressivo basato su `BUILD_NUMBER`
- push dell'immagine su un registry locale (`localhost:5000`)
- deploy dell'applicazione tramite il playbook `deploy_app.yml`, passando il build number come variabile

File coinvolti:
- `Jenkinsfile`
- `deploy_app.yml`

---

## Descrizione dei file principali

### Jenkinsfile

Pipeline CI/CD che esegue build, tagging, push e deploy.  
La pipeline seleziona automaticamente Docker o Podman in base all'engine disponibile sull'agent Jenkins.

Parametri principali utilizzati dalla pipeline:
- registry: `localhost:5000`
- image name: `myapp`
- tag: `BUILD_NUMBER`

---

### deploy_app.yml

Playbook che esegue un deploy applicativo semplificato sul nodo target:
- pull dell'immagine dal registry locale
- stop e rimozione del container precedente, se esistente
- avvio di un nuovo container esponendo l'applicazione sulla porta 8080

---

### server_deploy/Dockerfile

Dockerfile utilizzato per costruire un container impiegabile come nodo o agent di deploy, contenente:
- server SSH
- Ansible
- Java (necessaria per Jenkins remoting e agent)
- Docker CLI (senza Docker daemon)


