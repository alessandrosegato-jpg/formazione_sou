# Indovinello del Contadino, Lupo, Capra e Cavolo con Docker

## Descrizione

Questo progetto riproduce il classico indovinello del **contadino, del lupo, della capra e del cavolo** utilizzando **Docker**, **Bash**, **SSH** e **Ansible**.

L'idea è rappresentare le due sponde del fiume attraverso due macchine virtuali:

* **VM Sorgente (S)** → macchina locale
* **VM Destinazione (D)** → macchina remota

Ogni personaggio del gioco è rappresentato da un container Docker:

* `contadino`
* `lupo`
* `capra`
* `cavolo`

Spostare un personaggio da una sponda all'altra equivale ad arrestare il relativo container su una VM e avviarlo sull'altra.

---

# Obiettivo

Lo scopo del gioco è portare tutti i personaggi dalla sponda **S** alla sponda **D**, rispettando le regole del rompicapo:

* Il **lupo** non può rimanere da solo con la **capra**.
* La **capra** non può rimanere da sola con il **cavolo**.
* Solo il **contadino** può trasportare gli altri personaggi.

Se una delle regole viene violata, la partita termina con una sconfitta.

---

# Architettura

```
       VM S                   VM D
   ------------           ------------
    Docker                   Docker 

    contadino  <---SSH--->   contadino
    lupo                     lupo
    capra                    capra
    cavolo                   cavolo
```

Lo script utilizza SSH per avviare e fermare i container sulla macchina remota.

# Configurazione tramite Vagrant

Il `Vagrantfile` definisce entrambe le macchine virtuali.

## ubuntu1

Durante il provisioning vengono eseguite le seguenti operazioni:

* generazione di una chiave SSH ED25519 per l'utente `vagrant`;
* copia della chiave pubblica nella cartella condivisa;
* configurazione della rete privata con indirizzo `192.168.3.2`.

La chiave SSH verrà utilizzata per consentire il collegamento senza password verso la seconda macchina.

---

## ubuntu2

Durante il provisioning vengono eseguite le seguenti operazioni:

* aggiunta della chiave pubblica generata da ubuntu1 al file `authorized_keys`;
* configurazione della rete privata con indirizzo `192.168.3.3`;


---

# Provisioning con Ansible

Il playbook `playbook.yaml` automatizza completamente la configurazione delle macchine.

Le attività svolte sono:

1. Download dello script ufficiale di installazione di Docker.
2. Installazione di Docker.
3. Creazione dei quattro container:

* contadino
* lupo
* capra
* cavolo

I container vengono creati ma inizialmente lasciati in stato **stopped**.

Successivamente il playbook:

* aggiunge l'utente `vagrant` al gruppo `docker`;
* copia lo script del gioco nella home dell'utente:

```
/home/vagrant/indovinello
```

---

# Funzionamento dello script

Lo script Bash mantiene lo stato di ogni elemento mediante variabili:

* `contadino`
* `lupo`
* `capra`
* `cavolo`

Ogni variabile può assumere due valori:

* **S** → sponda sinistra
* **D** → sponda destra

Le funzioni principali sono:

* `container1()` → sposta il contadino;
* `container2()` → sposta il lupo;
* `container3()` → sposta la capra;
* `container4()` → sposta il cavolo;
* `vittoria()` → controlla la condizione di vittoria;
* `sconfitta()` → controlla le condizioni di sconfitta;
* `pulizia()` → arresta tutti i container all'uscita del programma.

Durante l'esecuzione il programma mostra i container presenti su ciascuna macchina utilizzando:

```
docker ps --format "{{.Names}}"
```

e

```
ssh vagrant@192.168.3.3 docker ps --format "{{.Names}}"
```
che attraverso questo parametro mostrerà solamente il nome del container, ovvero contadino, lupo, capra e cavolo.
