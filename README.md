# PROGETTO_SERVER_BANCA
Progetto svolto individualmente che riprende una simulazione realistica in sistema operativo GNU/LINUX

# Contesto del progetto

Il progetto simula un server **GNU/Linux** che ospita un’applicazione bancaria fittizia, utilizzata da clienti simulati per effettuare operazioni tipiche come:

- accessi
- prelievi
- depositi
- bonifici

Il sistema è progettato per generare **traffico applicativo e di rete artificiale ma realistico**, con l’obiettivo di analizzare il comportamento del server, delle connessioni di rete, delle porte e dei socket attivi.

L’intero scenario riproduce una situazione **plausibile e credibile** che potrebbe verificarsi su un server reale in ambiente GNU/Linux.

---

## Elenco dei problemi

1. [Rilevamento di flussi anomali di bonifici in ingresso (AML)](#rilevamento-di-flussi-anomali-di-bonifici-in-ingresso-anti-money-laundering)
2. [Individuazione di accessi simultanei sospetti dallo stesso account](#individuazione-di-accessi-simultanei-sospetti-dallo-stesso-account)
3. [Analisi degli accessi notturni fuori dal profilo abituale](#analisi-degli-accessi-notturni-fuori-dal-profilo-abituale)
4. [Rilevamento ATM che comunicano su porte non autorizzate](#rilevamento-atm-che-comunicano-su-porte-non-autorizzate)
5. [Rilevamento tentativi di brute-force sulle API del server](#rilevamento-tentativi-di-brute-force-sulle-api-del-server)
6. [Correlazione tra anomalie di rete e degrado del servizio bancario](correlazione-tra-anomalie-di-rete-e-degrado-del-servizio-bancario)
7. [Rilevamento di pattern anomali nell’utilizzo delle API bancarie](rilevamento-di-pattern-anomali-nell-utilizzo-delle-api-bancarie)


---

## Dataset utilizzato

Il progetto utilizza un file CSV denominato **`clienti_banca.csv`**, che rappresenta l’anagrafica statica dei clienti della banca.

### Intestazione del file

```csv
customer_id,first_name,last_name,tax_code,email,phone_number,address,city,postal_code,country,password_hash,two_factor_enabled,account_status,account_id,iban,account_type,account_balance,currency,card_id,card_number,card_expiry,card_status,last_login,opened_at
```

Il dataset contiene esclusivamente dati simulati ed è utilizzato come base informativa, non come database reale.

I dati anagrafici sono separati dai log applicativi per mantenere una struttura coerente e realistica.

---

### Generazione del traffico e degli eventi

Il traffico applicativo viene generato tramite **script Bash** pianificati con cron, che simulano il comportamento di più clienti che interagiscono contemporaneamente con il server.

Gli script simulano:

- accessi e disconnessioni
- prelievi e depositi
- bonifici verso IBAN differenti
- accessi da indirizzi IP diversi
- sessioni concorrenti

---

## Log degli eventi applicativi

Ogni interazione con il server genera un evento registrato in un file di log strutturato, che rappresenta la **principale fonte** di raccoglimento dati del progetto.

Gli eventi applicativi vengono registrati in tempo reale in un **database SQLite**, scelto per la sua leggerezza, affidabilità e idoneità alla gestione di log cronologici in ambienti GNU/Linux.

### Esempio di file .log

```sql
timestamp,customer_id,ip_address,azione,importo,iban_destinatario,session_duration,source_type
2026-02-02 10:15:03,1023,192.168.1.45,LOGIN,,,,USER
2026-02-02 10:18:21,1023,192.168.1.45,BONIFICO,500,IT60X0542811101000000123456,180,USER

```

### Azioni registrate
- **`LOGIN`**
- **`LOGOUT`**
- **`PRELIEVO`**
- **`DEPOSITO`**
- **`BONIFICO`**

---

## Analisi di rete e di sistema
Il focus principale del progetto **non è l’elaborazione dei dati anagrafici**, ma l’analisi del comportamento del server dal punto di vista di **rete e di sistema**.

### Oggetti dell'analisi
- porte aperte
- socket attivi
- servizi in ascolto
- connessioni simultanee
- utilizzo anomalo delle risorse di rete

---

## Interazione con il dataset clienti
L'interazione con il file **`clienti_banca.csv`** è limitata al recupero delle informazioni di supporto, ad esempio per verificare lo stato di un account o la mail di chi accede.

### Esempio

```terminal
grep ",active," clienti_banca.csv
```
la maggior parte delle analisi viene effettuata **senza interrogare direttamente l'anagrafica**, concentrandosi sui log applicativi e sullo stato del sistema.

---

## Simulazione di dispositivi fisici (ATM)

Il progetto include la simulazione di **dispositivi fisici bancari**, in particolare **ATM (Automated Teller Machine)**, che interagiscono con il server tramite rete, analogamente a quanto avverrebbe in un contesto reale.

Gli ATM simulati utilizzano una subnet dedicata **`(192.168.100.0/24)`**, separata dal traffico degli utenti, al fine di **facilitare l’analisi** delle connessioni di rete e l’individuazione di comportamenti anomali.”

Gli ATM sono trattati come **entità distinte dagli utenti finali**, caratterizzate da:
- indirizzo IP dedicato
- comportamento automatico
- operazioni ripetitive (prelievi, interrogazioni)
- assenza di interazione diretta con l’interfaccia utente

---

La simulazione degli ATM consente di introdurre una componente “fisica” nell’ecosistema del progetto, mantenendo un approccio coerente con un ambiente GNU/Linux e con l’analisi delle risorse di rete.


Per distinguere le operazioni effettuate dagli utenti da quelle generate da dispositivi fisici, il database degli eventi include un campo aggiuntivo che identifica la **tipologia di sorgente** dell’evento.

### Schema logico degli eventi:

```sql
timestamp,customer_id,ip_address,azione,importo,iban_destinatario,session_duration,source_type
```
Dove **`source_type`** può assumere valori come:
- **`USER`** -> operazione effettuata da un cliente
- **`ATM`** -> operazione effettuata da un dispositivo fisico

--- 

# Problemi affrontati:

### Rilevamento di flussi anomali di bonifici in ingresso (Anti-Money Laundering)
Identificare automaticamente gli utenti che ricevono un numero elevato di bonifici in un breve intervallo di tempo da IBAN differenti.
Il sistema analizza il database degli eventi, individua i conti potenzialmente sospetti e invia una comunicazione di verifica all’utente per prevenire frodi o attività di riciclaggio.

### [Elenco dei problemi](#elenco-dei-problemi)
--- 

### Individuazione di accessi simultanei sospetti dallo stesso account
Rilevare utenti che risultano attivi sul server con più **sessioni contemporanee provenienti da IP diversi**, possibile compromissione delle credenziali.

**Focus tecnico**
- socket attivi
- **`ss`**, **`lsof`**
- correlazione indirizzo IP <-> customer_id

### [Elenco dei problemi](#elenco-dei-problemi)
---

### Analisi degli accessi notturni fuori dal profilo abituale
Identificare utenti che accedono in fasce orarie anomale rispetto al loro storico, potenziale furto di account.

**Focus tecnico**
- timestamp
- finestre temporali
- nessuna interrogazione diretta al DB utenti

### [Elenco dei problemi](#elenco-dei-problemi)
---

## Rilevamento ATM che comunicano su porte non autorizzate
Verificare che gli IP riservati agli ATM comunichino **solo sulle porte previste**. Qualsiasi deviazione è potenziale compromissione fisica o di rete.

**Focus tecnico**
- **`ss -tulnp`**
- **`nmap`**
- porte & socket
- subnet ATM dedicata

### [Elenco dei problemi](#elenco-dei-problemi)
---

## Rilevamento tentativi di brute-force sulle API del server
Analizzare connessioni ripetute e ravvicinate verso le porte del servizio bancario per individuare tentativi di accesso automatizzati.

**Focus tecnico**
- **`netstat`**
- **`ss`**
- frequenza delle connessioni
- porte specifiche

### [Elenco dei problemi](#elenco-dei-problemi)
---

## Correlazione tra anomalie di rete e degrado del servizio bancario
Analizzare se rallentamenti, timeout o interruzioni del servizio bancario coincidono con:
- picchi di connessioni
- scansioni di porte
- aumento anomalo dei socket attivi
Le banche subiscono spesso attacchi a bassa intensità che non buttano giù il servizio, ma lo degradano.

**Focus tecnico**
- numero di socket
- stato porte
- traffico locale
- simulazione stress controllato

### [Elenco dei problemi](#elenco-dei-problemi)
---

## Rilevamento di pattern anomali nell’utilizzo delle API bancarie
Analizzare il traffico diretto alle API del server bancario per individuare **utilizzi anomali o potenzialmente malevoli**, come:
- chiamate API troppo frequenti
- sequenze di endpoint non coerenti con il normale flusso applicativo
- utilizzo delle API da IP non autorizzati o fuori contesto (es. IP ATM che invocano API web)

L’obiettivo è individuare **abusi delle API** che potrebbero indicare automazioni fraudolente, reverse engineering dell’app o tentativi di bypass dei controlli applicativi.

**Focus tecnico**
- **`curl`** -> generazione richieste
- **`ss -tuln`**, **`ss -tan`** -> socket API
- **`lsof -i`** -> processi in ascolto
- **`nmap`** -> esposizione endpoint
- analisi **porte** + **rete**, non DB

### [Elenco dei problemi](#elenco-dei-problemi)
---

