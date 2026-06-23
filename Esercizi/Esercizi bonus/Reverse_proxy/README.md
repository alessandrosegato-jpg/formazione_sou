# Reverse Proxy
### Introduzione 
L'architettura di questo esercizio è basata su tre macchine: una macchina che fa da reverse proxy server, che dirige il traffico verso altre due macchine che fanno da server backend.
Io ho deciso di creare una configurazione con HaProxy in modo tale che il primo server backend sia costantemente attivo e tutto il traffico sia indirizzato esattamente li, allo stesso tempo il secondo server backend si trova "dietro di lui", è configurato in modo che quando il primo server andrà DOWN per qualsiasi motivo, ci sarà in automatico il secondo server che entrerà al posto suo e infine quando il primo server (principale) sarà di nuovo UP tornerà lui al posto del suo sostituto.
### Configurazione macchine
Ho realizzato tutto tramite  Vagrant, le tre macchine sono collegate alla stessa rete privata: 192.168.3.0/24 ed è tutto completamente portabile grazie al provisioning creato che adesso vedremo meglio nel dettaglio.
### Proxy-1

```bash
mkdir -p /etc/ssl/mycerts
openssl req -new -x509 -days 365 -nodes -subj "/C=IT/ST=RM/L=Roma/O=Lab/CN=Proxy-1" -out /etc/ssl/mycerts/server.crt -keyout /etc/ssl/mycerts/server.key
cat /etc/ssl/mycerts/server.crt /etc/ssl/mycerts/server.key > /etc/ssl/mycerts/proxy.pem
```
Dato che le richieste del client al server proxy saranno in https, partiamo con il generare un certificato autofirmato, creiamo una cartella per i nostri certificati, dentro utilizziamo questo comando openssl per generare sia la chiave che il certificato autofirmato.
- `-new:` serve per generare un nuovo certificato
- `-x509:` serve per generare direttamente un certificato x509 autofirmato invece di una CSR da inviare a una CA.
- `-days 365:` il certificato sarà valido per 365 giorni
- `-nodes:` la chiave sarà utilizzabile senza inserire password all'avvio del servizio.
- `-subj:` ci permette di specifica i dati del propetario del certificato
- `-out:` ci permette di inserire il percorso dove viene salvato il certificato
- `-keyout:` ci permette di inserire il percorso dove viene salvata la chiave privata

Infine utilizziamo il comando cat con i due percorsi della chiave e del certificato per unirli in un unico file, dato che haproxy lo richiede.
#### Configurazione del Haproxy

```bash
global
 daemon
 log /dev/log local0

defaults
 mode http
 log global
 timeout connect 5s
 timeout client 30s
 timeout server 30s

frontend https_front
 bind *:443 ssl crt /etc/ssl/mycerts/proxy.pem

 default_backend siti_backend

backend siti_backend
 balance first

 server web1 192.168.3.3:80 check fall 3 rise 2 
 server web2 192.168.3.4:80 check backup fall 3 rise 2
```
##### Global
Questa sezione contiene impostazioni generali di HaProxy:
- `daemon:` fa partire HaProxy in background
- `log /dev/log local0:`  dice come e dove salvare i log
##### Defaults
Questa sezione contiene i parametri che vengono applicati automaticamente a frontend e backend:
- `mode http:` mette HaProxy in http
- `log global:` usa la configurazione di log definita prima in global
- `timeout connect 5s:` tempo massimo per aprire una connessione verso il server backend 
- `timeout client 30s:` tempo massimo di inattività del client
- `timeout server 30s:` tempo massimo di inattività del server backend
##### Frontend
Il frontend è il punto di ingresso delle connessioni:
- `bind *:443 ssl crt /etc/ssl/mycerts/proxy.pem:` indica che il frontend è in ascolto su tutte le interfacce di rete sulla porta 443
- `ssl:` abilta https
- `crt:` specifichiamo il certificato mettendo il path completo dopo
- `default_backend siti_backend:` tutte le richieste ricevute vengono inoltrate al backend
##### Backend
Contiene i server reali che gestiscono il traffico:
- `balance first:` è un algoritmo di bilanciamento: quindi con `first`viene usato sempre il primo server, gli altri quando il primo non è disponibile
- `server web1 192.168.3.3:80 check fall 3 rise 2:` qui definiamo i server con nome, ip e porta. `Check` serve per abilitare degli healt check e controllare periodicamente se il server è UP. `Fall 3` il server viene marcato come DOWN dopo 3 tentativi falliti. `Rise 2` dopo essere andato DOWN, se fa 2 controlli consecutivi riusciti torna UP.
- `server web2 192.168.3.4:80 check backup fall 3 rise 2:` questo è il secondo server, `backup` indica che è un server di riserva ed entra quando il primo non è disponibile

Completata la configurazione restartiamo il servizio.
#### Firewall

```bash
firewall-cmd --permanent --add-port=443/tcp
firewall-cmd --reload
```
Utilizziamo firewalld per aggiungere la porta 443 permanentemente e restartiamo il servizio.
### Web-1 e Web-2
Queste macchine sono state configurate come server web.

```bash
echo "Benvenuto su Web-1" > /var/www/html/index.html
systemctl restart httpd
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --reload
```
- `echo "Benvenuto su Web-1" > /var/www/html/index.html:` utilizziamo `echo` e successivamente la frase che vogliamo inserire nel nostro sito, per poi indirizzarlo nella cartella di default di apache creando un `index.html` e restartare il servizio
- `firewall-cmd --permanent --add-port=80/tcp:` anche qui utilizziamo firewalld per aggiungere la porta 80 e lo restartiamo