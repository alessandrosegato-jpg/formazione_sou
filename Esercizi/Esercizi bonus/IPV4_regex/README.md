# Analizzatore di Indirizzi IPv4 in Bash

## Descrizione

Questo script Bash permette di:

1. Richiedere all'utente un indirizzo IPv4.
2. Verificare che l'indirizzo inserito sia valido.
3. Identificare alcune categorie speciali di indirizzi IPv4 (privati, loopback, multicast, ecc.).
4. Mostrare la funzione e la classe dell'indirizzo IP.

---

## Funzionamento

### 1. Definizione della regex per un ottetto IPv4

```bash
ott='(25[0-5]|2[0-4][0-9]|1[0-9]{2}|[1-9]?[0-9])'
```

La variabile `ott` contiene una **espressione regolare** che verifica che ogni ottetto dell'indirizzo IP sia compreso tra **0 e 255**.


| Regex      | Range  |
| ------------- | --------- |
| `25[0-5]`     | 250 - 255 |
| `2[0-4][0-9]` | 200 - 249 |
| `1[0-9]{2}`   | 100 - 199 |
| `[1-9]?[0-9]` | 0 - 99    |

---

### 2. Validazione dell'indirizzo IPv4

```bash
while true; do
    read -p "Inserisci un IPV4 valido: " ip

    if [[ $ip =~ ^$ott\.$ott\.$ott\.$ott$ ]]; then
        break
    fi

    echo
    echo "Indirizzo IP non valido, riprova.. "
done
```

Il ciclo continua a richiedere un indirizzo IP finché l'utente non inserisce un IPv4 valido.

L'espressione:

```bash
^$ott\.$ott\.$ott\.$ott$
```

verifica che siano presenti:

* 4 ottetti
* separati da punti
* ciascuno compreso tra 0 e 255

Se l'indirizzo non è valido, viene mostrato il messaggio:

```text
Indirizzo IP non valido, riprova..
```

---

### 3. Classificazione dell'indirizzo IP

Dopo la validazione, una serie di condizioni `if / elif` controlla a quale categoria appartiene l'indirizzo.

---

## Categorie

### Indirizzi Zero (0.0.0.0/8)

```bash
^0\.($ott\.){2}$ott$
```

---

### Indirizzi Privati Classe A

```bash
^10\.($ott\.){2}$ott$
```

Range:

```text
10.0.0.0 - 10.255.255.255
```

---

### Indirizzi Loopback

```bash
^127\.($ott\.){2}$ott$
```


Range:

```text
127.0.0.0 - 127.255.255.255
```

---

### Indirizzi Link Local

```bash
^169\.254\.$ott\.$ott$
```
Range:

```text
169.254.0.0 - 169.254.255.255
```

---

### Indirizzi Privati Classe B

```bash
^172\.(1[6-9]|2[0-9]|3[0-1])\.$ott\.$ott$
```

Range:

```text
172.16.0.0 - 172.31.255.255
```

---

### Indirizzi Riservati IANA

```bash
^192\.0\.2\.$ott$
```

Range:

```text
192.0.2.0 - 192.0.2.255
```

---

### Indirizzi Anycast

```bash
^192\.88\.99\.$ott$
```
Range:

```text
192.88.99.0 - 192.88.99.255
```


---

### Indirizzi Privati Classe C

```bash
^192\.168\.$ott\.$ott$
```

Range:

```text
192.168.0.0 - 192.168.255.255
```

---

### Indirizzi Benchmark

```bash
^198\.(1[8-9])\.$ott\.$ott$
```
Range:

```text
198.18.0.0 - 198.19.255.255
```


---

### Indirizzi Multicast

```bash
^(22[4-9]|23[0-9])\.($ott\.){2}$ott$
```

Range:

```text
224.0.0.0 - 239.255.255.255
```

---

### Indirizzi Classe E (Riservati)

```bash
^(24[0-9]|25[0-5])\.($ott\.){2}$ott$
```

Range:

```text
240.0.0.0 - 255.255.255.255
```

---

### Indirizzi Pubblici

Se l'indirizzo non rientra in nessuna delle categorie precedenti:

```bash
else
        echo
        echo "Indirizzo IP: $ip"
        echo "Funzione: IP pubblico"

fi
```

---

## Esempi di esecuzione

### Esempio 1

Input:

```text
192.168.1.10
```

Output:

```text
Indirizzo IP: 192.168.1.10
Funzione: IP privati
Classe: C
```

---

### Esempio 2

Input:

```text
127.0.0.1
```

Output:

```text
Indirizzo IP: 127.0.0.1
Funzione: Indirizzi loopback
Classe: A
```

---

### Esempio 3

Input:

```text
8.8.8.8
```

Output:

```text
Indirizzo IP: 8.8.8.8
Funzione: IP pubblico
```

