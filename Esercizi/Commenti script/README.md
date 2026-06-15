
## Example 2-1
```bash

# Cleanup
# Run as root, of course.

cd /var/log
cat /dev/null > messages
cat /dev/null > wtmp
echo "Log files cleaned up." 
```

Questo script serve a svuotare il contenuto dei file di log di messages e wtmp.


## Example 9-5
```bash

#!/bin/bash
# am-i-root.sh:   Am I root or not?

ROOT_UID=0   # Root has $UID 0.

if [ "$UID" -eq "$ROOT_UID" ]  # Will the real "root" please stand up?
then
  echo "You are root."
else
  echo "You are just an ordinary user (but mom loves you just the same)."
fi

exit 0

# ============================================================= #
# Code below will not execute, because the script already exited.

# An alternate method of getting to the root of matters:

ROOTUSER_NAME=root

username=`id -nu`              # Or...   username=`whoami`
if [ "$username" = "$ROOTUSER_NAME" ]
then
  echo "Rooty, toot, toot. You are root."
else
  echo "You are just a regular fella."
fi
```

Questo script serve a verificare se l'utente che lo esegue è root oppure no. Nel primo vediamo che confronta l'UID dell'utente corrente con 0, che identifica l'account root. Se l'utente è root, viene visualizzato un messaggio di conferma altrimenti viene indicato che si tratta di un utente normale.

Nel secondo invece verifica se l'utente corrente è root confrontando il nome dell'utente in esecuzione. Il comando id -nu recupera il nome dell'utente attualmente connesso e se corrisponde a root viene visualizzato il messagio di conferma.


## Example 4-5
```bash

#!/bin/bash

# Call this script with at least 10 parameters, for example
# ./scriptname 1 2 3 4 5 6 7 8 9 10
MINPARAMS=10

echo

echo "The name of this script is \"$0\"."
# Adds ./ for current directory
echo "The name of this script is \"`basename $0`\"."
# Strips out path name info (see 'basename')

echo

if [ -n "$1" ]              # Tested variable is quoted.
then
 echo "Parameter #1 is $1"  # Need quotes to escape #
fi 

if [ -n "$2" ]
then
 echo "Parameter #2 is $2"
fi 

if [ -n "$3" ]
then
 echo "Parameter #3 is $3"
fi 

# ...


if [ -n "${10}" ]  # Parameters > $9 must be enclosed in {brackets}.
then
 echo "Parameter #10 is ${10}"
fi 

echo "-----------------------------------"
echo "All the command-line parameters are: "$*""

if [ $# -lt "$MINPARAMS" ]
then
  echo
  echo "This script needs at least $MINPARAMS command-line arguments!"
fi  

echo

exit 0
```

Questo script mostra come gestire i parametri passati da riga di comando, controlla la presenza dei parametri ricevuti e ne stampa il valore, mostra anche come accedere a parametri con indice superiore a 9 utilizzando le parentesi graffe ad esempio ${10}. Infine se vengono passati meno di 10 argomenti, viene mostrato un messaggio che avvisa l'utente del numero minimo richiesto.


## Example 4-2
```bash

#!/bin/bash
# Naked variables

echo

# When is a variable "naked", i.e., lacking the '$' in front?
# When it is being assigned, rather than referenced.

# Assignment
a=879
echo "The value of \"a\" is $a."

# Assignment using 'let'
let a=16+5
echo "The value of \"a\" is now $a."

echo

# In a 'for' loop (really, a type of disguised assignment):
echo -n "Values of \"a\" in the loop are: "
for a in 7 8 9 11
do
  echo -n "$a "
done

echo
echo

# In a 'read' statement (also a type of assignment):
echo -n "Enter \"a\" "
read a
echo "The value of \"a\" is now $a."

echo

exit 0
```

Questo script mostra diversi modi per utilizzare una variabile, inizialmente assegna alla variabile a il valore 879, dopo utilizza il comando let per eseguire un'operazione aritmetica e aggiornare il valore della variabile, ma dimostra anche come una variabile può assumere un valore automatico dentro ad un ciclo for, come nell'esempio che gli viene asseganto 7 8 9 11 e salva solo l'ultimo assegnato. Infine utilizza il comando read per far aprire un promt e far assegnare all'utente il valore e salvarlo nella variabile a.


## Example 6-1
```bash

#!/bin/bash

echo hello
echo $?    # Exit status 0 returned because command executed successfully.

lskdf      # Unrecognized command.
echo $?    # Non-zero exit status returned -- command failed to execute.

echo

exit 113   # Will return 113 to shell.
           # To verify this, type "echo $?" after script terminates.

#  By convention, an 'exit 0' indicates success,
#+ while a non-zero exit value means an error or anomalous condition.
#  See the "Exit Codes With Special Meanings" appendix.
```

Questo script Bash mostra come funzionano i codici di uscita, dopo aver eseguito il comando echo hello, stampa il valore della variabile speciale $?, che contiene il codice di uscita dell'ultimo comando eseguito, in questo caso 0 che indica che è stato eseguito corretamente. Successivamente prova ad eseguire lskdf, un comando inesistente, che restituisce un valore diverso da 0 e genera un errore. Infine termina con exit 113, restituendo il codice 113 alla shell.


## Example 37-5
```bash

#!/bin/bash4
# fetch_address.sh

declare -A address
#       -A option declares associative array.

address[Charles]="414 W. 10th Ave., Baltimore, MD 21236"
address[John]="202 E. 3rd St., New York, NY 10009"
address[Wilma]="1854 Vermont Ave, Los Angeles, CA 90023"


echo "Charles's address is ${address[Charles]}."
# Charles's address is 414 W. 10th Ave., Baltimore, MD 21236.
echo "Wilma's address is ${address[Wilma]}."
# Wilma's address is 1854 Vermont Ave, Los Angeles, CA 90023.
echo "John's address is ${address[John]}."
# John's address is 202 E. 3rd St., New York, NY 10009.

echo

echo "${!address[*]}"   # The array indices ...
# Charles John Wilma
```

Questo script dimostra l’uso degli array associativi, una struttura che permette di associare più valori ad una variabile. Viene dichiarato un array chiamato address con l’opzione -A e vengono assegnati alcuni indirizzi utilizzando come chiavi i nomi delle persone e stampa l'indirizzo di ciascun nome utlizzando ${address[Nome]}, infine ${!address[*]} restituisce tutte le chiavi dell’array.


## Example 32-7
```bash

#! /bin/bash
# progress-bar2.sh
# Author: Graham Ewart (with reformatting by ABS Guide author).
# Used in ABS Guide with permission (thanks!).

# Invoke this script with bash. It doesn't work with sh.

interval=1
long_interval=10

{
     trap "exit" SIGUSR1
     sleep $interval; sleep $interval
     while true
     do
       echo -n '.'     # Use dots.
       sleep $interval
     done; } &         # Start a progress bar as a background process.

pid=$!
trap "echo !; kill -USR1 $pid; wait $pid"  EXIT        # To handle ^C.

echo -n 'Long-running process '
sleep $long_interval
echo ' Finished!'

kill -USR1 $pid
wait $pid              # Stop the progress bar.
trap EXIT

exit $?
```

Questo script mostra una barra di progresso mentre c'è un processo in esecuzione. All’inizio avvia un processo in background che stampa continuamente punti (.), questo processo si interrompe quando riceve un segnale (SIGUSR1) o ovviamente quando termina. Lo script salva il PID del processo in background e usa trap per gestire l’uscita corretta e nel frattempo il programma principale simula un’operazione lunga con sleep e stampa un messaggio di completamento.
Alla fine invia il segnale per fermare la barra di progresso e attende che termina il processo in background.