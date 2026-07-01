# Port Scanner in Bash

## Descrizione

Questo progetto consiste in un **Port Scanner** realizzato in **Bash** che utilizza il comando `nc` (Netcat) per verificare quali porte TCP risultano aperte su un host.

Lo script esegue una scansione di un intervallo di porte specificato dall'utente e mostra solamente le porte aperte, indicando al termine il numero totale di porte trovate.

---

## Funzionamento

Lo script richiede tre argomenti da riga di comando:

1. Indirizzo IP del target.
2. Porta iniziale.
3. Porta finale.

Prima di avviare la scansione vengono effettuati alcuni controlli:

* verifica che siano stati passati esattamente tre argomenti;
* controlla che l'indirizzo IP sia valido;
* verifica che le porte siano numeriche;
* controlla che il range di porte sia compreso tra **1** e **65535** e che la porta iniziale non sia maggiore della finale.

Se uno dei controlli fallisce, lo script termina mostrando un messaggio di errore.

---



## Output di esempio

```bash
./port_scanner 192.168.1.1 1 1000
```



```text
========================================
             PORT SCANNER
========================================
 Target : 192.168.1.1
 Range  : 1 - 1000
========================================

 Porta 22: APERTA
 Porta 80: APERTA
 Porta 443: APERTA

========================================
 Scansione completata.
 Porte aperte trovate: 3
========================================
```

---

## Come funziona

```bash
for ((i=porta1; i<=porta2; i++))
do
        echo | nc -w 1 $ip $i &>/dev/null

        if [ $? -eq 0 ]; then
                echo " Porta $i: APERTA"                
        ((aperte++))
        fi
done
```
Per ogni porta appartenente all'intervallo indicato viene eseguito il comando:
```bash
echo | nc -w 1 $ip $i &>/dev/null

if [ $? -eq 0 ]; then
                echo " Porta $i: APERTA"                
        ((aperte++))
        fi
```
dove:

* `nc -w 1` imposta un timeout di 1 secondo;
* il codice di uscita del comando (`$?`) viene utilizzato per determinare se la porta è aperta, se il valore restituito è `0`, la porta viene considerata aperta e viene visualizzata a schermo.

---

## Differenza tra TCP e UDP

Questo progetto utilizza TCP perché consente di verificare facilmente lo stato di una porta tramite il tentativo di connessione effettuato con **Netcat** (`nc`). La scansione delle porte UDP è più complessa, poiché l'assenza di risposta da parte dell'host non indica necessariamente che la porta sia chiusa: il pacchetto potrebbe essere stato semplicemente ignorato oppure perso durante la trasmissione.

### TCP

TCP **(Transmission Control Protocol)** è un protocollo orientato alla connessione, cioè prima di trasmettere dati stabilisce una connessione tra client e server tramite il  **three-way handshake**. Questo garantisce che entrambe le parti siano pronte a comunicare.

Caratteristiche:

* consegna affidabile dei dati;
* controllo degli errori;
* ritrasmissione dei pacchetti persi;
* mantenimento dell'ordine dei pacchetti.

Grazie a queste caratteristiche è possibile determinare con maggiore precisione se una porta è aperta.

### UDP

UDP **(User Datagram Protocol)** è invece un protocollo **senza connessione (connectionless)**.

A differenza di TCP:

* non stabilisce una connessione prima dell'invio dei dati;
* non garantisce la consegna dei pacchetti;
* non controlla eventuali errori di trasmissione;
* non ritrasmette i pacchetti persi;
* non garantisce che i pacchetti arrivino nell'ordine corretto.

Per questo motivo UDP viene spesso definito **non affidabile**. Ciò non significa che sia "difettoso", ma semplicemente che privilegia la velocità rispetto all'affidabilità.

