# Playbook Ansible – Esercitazione

Questo repository contiene un playbook Ansible composto da **due play indipendenti**, richiamabili singolarmente tramite tag. Gestione pacchetti, gestione utenti, uso di variabili, vault, loop, condizioni e manipolazione di file di configurazione.

## Struttura del file

Il playbook è diviso in due play, distinguibili tramite i tag `1` e `2`:

```
ansible-playbook playbook.yaml --tags "1"   # esegue solo Esercitazione1
ansible-playbook playbook.yaml --tags "2"   # esegue solo Esercitazione2
ansible-playbook playbook.yaml              # esegue entrambi
```

Entrambi i play:
- girano su `hosts: all`
- usano `become: true` per eseguire i task con privilegi elevati
- disabilitano il gathering automatico dei fact (`gather_facts: false`) per velocizzare l'esecuzione, dato che non sono necessari fact di sistema

---

## Play 1 — `Esercitazione1` (tag: `1`)

Questo play si occupa di installare/rimuovere pacchetti, creare utenti di sistema con vari parametri e leggere credenziali cifrate da un vault.

### Variabili

**`pacchetti`** – dizionario che mappa nome del pacchetto → stato desiderato:

| Pacchetto | Stato      | Effetto                     |
|-----------|-----------|------------------------------|
| nginx     | present   | installato se non presente   |
| ufw       | present   | installato se non presente   |
| curl      | absent    | rimosso se presente          |

**`utenti`** – lista di dizionari, uno per ogni utente da creare, con i campi:
- `name`: nome utente
- `shell`: shell di login
- `group`: gruppo primario
- `state`: stato desiderato
- `home`: percorso della home directory

Nell'esercitazione sono definiti due utenti: `mario` e `luigi`, entrambi con shell `/bin/bash` e gruppo `sudo`.

### Task

1. **Gestione pacchetti**
   Usa il modulo `ansible.builtin.package` in loop su `pacchetti | dict2items`, così da trasformare il dizionario in una lista di coppie `key`/`value` iterabili. Per ogni pacchetto imposta lo stato indicato nella variabile.

2. **Crea utenti**
   Usa il modulo `ansible.builtin.user` in loop sulla lista `utenti`, creando ciascun utente con i parametri definiti nella variabile.

3. **Carica il vault**
   Usa `ansible.builtin.include_vars` per caricare il contenuto del file `vault.yaml`, che deve contenere le variabili cifrate `password_mario` e `password_luigi`.

4. **Stampa le password di Mario / Luigi**
   Due task di `ansible.builtin.debug` che stampano a schermo il valore delle password caricate dal vault.

### File richiesto: `vault.yaml`

Il play si aspetta un file `vault.yaml` nella stessa directory del playbook (o comunque risolvibile dal path relativo), contenente almeno:

```yaml
password_mario: "una_password_sicura"
password_luigi: "un_altra_password_sicura"
```

Per proteggere il file con Ansible Vault:

```bash
ansible-vault encrypt vault.yaml
```

Eseguire il playbook passando la password del vault:

```bash
ansible-playbook playbook.yaml --tags "1" --ask-vault-pass

```
---

## Play 2 — `Esercitazione2` (tag: `2`)

Questo play imposta i limiti sul numero massimo di file aperti (`nofile`) per un insieme di utenti, tramite `/etc/security/limits.conf`, e configura l'accesso consentito in `/etc/security/access.conf`.

### Variabili

**`max_file`** – valore calcolato dinamicamente con un'espressione condizionale Jinja2:

```yaml
max_file: "{{ 10000 if ambiente == 'produzione' else 1000 }}"
```

- Se la variabile `ambiente` è uguale a `produzione` → `max_file = 10000`
- Altrimenti (es. `sviluppo`) → `max_file = 1000`

**`utenti`** – semplice lista di stringhe con i nomi degli utenti su cui applicare i limiti: `fabio`, `tozzi`.

### Task

1. **Imposta soft limit nofile**
   Usa `ansible.builtin.lineinfile` in loop su `utenti` per aggiungere/garantire la presenza della riga:
   ```
   <utente> soft nofile <max_file>
   ```

2. **Imposta hard limit nofile**
   Analogo al task precedente, ma aggiunge la riga:
   ```
   <utente> hard nofile <max_file>
   ```

3. **Aggiungi blocco in un file**
   Usa `ansible.builtin.blockinfile` per inserire, prima della riga `#-:ALL:ALL` nel file `/etc/security/access.conf`, un blocco generato con un ciclo Jinja2 `{% for %}` che produce una riga per ogni utente nel formato:
   ```
   +:<utente>:ALL
   ```

### Esecuzione con variabile `ambiente`

```bash
ansible-playbook playbook.yaml --tags "2" --extra-vars "ambiente=produzione"
```