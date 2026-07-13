# Linux Forensics, Log Analysis & Incident Response Lab

Questo repository raccoglie strumenti, comandi e playbook operativi dedicati alla Digital Forensics, al Blue Teaming e alla Malware Analysis su sistemi Linux. L'obiettivo è documentare tramite scenari pratici come identificare gli Indicatori di Compromissione (IoC), analizzare vettori di attacco reali e isolare minacce in ambienti sandbox.

---

## 🔎 Modulo 1: Log Analysis & SSH Brute-Force Detection

Procedure di Incident Response per identificare, tracciare e mitigare attacchi di Brute Force diretti al servizio SSH analizzando i log di sistema centralizzati.

### 1. Identificazione degli IP Attaccanti (Live Parsing)
Il seguente comando analizza il flusso dei log di autenticazione in tempo reale, estrae gli indirizzi IPv4 associati a tentativi falliti e restituisce una classifica aggiornata degli host malevoli:

**BASH**
-

`sudo journalctl -u ssh -f | grep --line-buffered "Failed password" | grep -oE '([0-9]{1,3}\.){3}[0-9]{1,3}' | sort | uniq -c | sort -rn`

-
Per comprendere se l'attacco mira ad account di sistema reali o a dizionari standard (admin, root), questo filtro isola gli username presi di mira:

`sudo journalctl -u ssh --no-pager | grep "Failed password" | grep -oE 'for (invalid user )?[a-zA-A0-9_-]+' | sed 's/for //g' | sort | uniq -c | sort -rn`

## Analisi della Regular Expressioni
-

L'estrazione degli Indicatori di Compromissione (IoC) legati alla rete si basa sulla seguente espressione regolare: 
                                       
                                                      *$$([0-9]{1,3}\.){3}[0-9]{1,3}$$*

`[0-9]{1,3}`: Identifica una sequenza numerica da 1 a 3 cifre (singolo ottetto, es: 192).

`\.`: Esegue l'escape del carattere punto fermo per interpretarlo letteralmente.

`([0-9]{1,3}\.){3}`: Ripete la struttura ottetto-punto esattamente per tre volte (es: 192.168.1.).

`[0-9]{1,3}`: Intercetta l'ultimo ottetto finale dell'indirizzo IP.

## Contenimento dell'Attacco (Mitigazione)
-

Isolato l'IP malevolo tramite l'analisi dei log, la minaccia viene bloccata immediatamente a livello di network firewall locale per interrompere il brute-force:

`sudo iptables -A INPUT -s [IP_ATTACCANTE] -j DROP`

## 🔬 Modulo 2: Triage Operativo & Malware Analysis
-

Metodologie investigative e comandi di analisi dinamica per individuare artefatti malevoli persistenti o volatili che operano direttamente all'interno della memoria del sistema.

Rilevamento di Processi Nascosti (Deleted Binaries)
Una tecnica di evasion comune consiste nell'avviare un processo dannoso e procedere all'immediata eliminazione del file binario dal disco per eludere i controlli dei software di sicurezza statici.

Il comando seguente esegue un triage sul file system virtuale /proc analizzando i link simbolici dei processi attivi, isolando quelli che puntano a file non più presenti sul file system:

`sudo find /proc/ -maxdepth 2 -name exe -type l -exec readlink {} \; 2>/dev/null | grep "deleted"`

**NOTE: Interpretazione dell'output: Se il comando restituisce un percorso associato alla stringa (deleted), indica la presenza di un processo attivo in esecuzione esclusivamente nella memoria RAM volatile.**

## Ispezione della Memoria di un Processo (Triage)
Per identificare stringhe sensibili (come indirizzi IP di server C2, chiavi cifrate o URL di download) all'interno di un processo marcato como sospetto, è possibile ispezionare lo spazio di memoria associato al suo PID senza interromperne l'esecuzione:

Bash

`sudo strings /proc/[PID]/mem 2>/dev/null | head -n 50`


## Configurazione del Laboratorio (Sandbox)
Tutti i test e le analisi documentate in questo repository sono stati eseguiti all'interno di un ambiente sicuro controllato:

Attaccante/Analizzatore: Kali Linux

Target/Vittima: Ubuntu Server 22.04 LTS

Rete: Configurazione in modalità **Host-Only** (senza accesso a Internet esterno) per garantire il contenimento sicuro delle minacce simulate.
