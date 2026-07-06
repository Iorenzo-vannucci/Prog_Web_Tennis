# WebTennis - Web App per la Gestione Sportiva del Circolo Tennis

Piattaforma web moderna e intuitiva progettata per la gestione organizzativa e sportiva di un circolo di tennis. Consente ai soci del club di registrarsi, controllare il proprio profilo, visualizzare le classifiche in tempo reale, analizzare le statistiche di gioco e registrare i risultati degli incontri. Gli amministratori dispongono di funzionalità avanzate per la supervisione e la rettifica dei punteggi con ricalcolo retroattivo delle classifiche.

L'applicazione è strutturata secondo un'architettura **Client-Server a tre livelli (3-tier)** senza l'ausilio di framework pesanti lato frontend, per garantire massima pulizia ed efficienza.

---

## 🛠️ Stack Tecnologico e Architettura

### 1. Frontend (Presentation Layer)
* **HTML5 & CSS3 (Vanilla)**: Layout interamente responsive realizzati tramite **CSS Grid** e **Flexbox** per garantire una visualizzazione ottimale su PC, tablet e smartphone (menu ad hamburger a scomparsa sotto i 768px).
* **JavaScript (ES6+)**: Logica client-side pura con caricamento dei dati asincrono (**Fetch API/AJAX**).
* **Google Charts API**: Integrazione per la generazione di grafici a torta interattivi sulle statistiche dei match (vinti/persi) nella scheda di ogni giocatore.
* **Material Symbols**: Set di icone utilizzato per arricchire l'interfaccia utente (incluso l'avatar dinamico basato sul genere).

### 2. Backend (Application Layer)
* **Node.js & Express.js**: Server applicativo leggero e modulare con gestione delle rotte API RESTful.
* **Middlewares Personalizzati**: Filtri di sicurezza dedicati sia per la visualizzazione delle pagine HTML private sia per la protezione degli endpoint delle API.

### 3. Database (Data Layer)
* **MySQL**: DB relazionale strutturato in tabelle normalizzate con vincoli di integrità referenziale e aggiornamenti a cascata.

---

## 🔒 Sicurezza e Gestione delle Sessioni

* **JSON Web Token (JWT)**: Autenticazione *stateless* e sicura. Al login, il server genera e firma un token contenente le info minime (`userId`, `isAdmin`).
* **Cookie HTTP-Only & SameSite**: Il token JWT viene salvato nel client tramite un cookie con flag `httpOnly: true` (inaccessibile via JavaScript per prevenire attacchi XSS) e `sameSite: 'strict'` (protezione contro CSRF).
* **Prevenzione SQL Injection**: Utilizzo sistematico di **Prepared Statements** e query parametrizzate (`?`) fornite dal driver `mysql2`.
* **Database Connection Pool**: Ottimizzazione delle performance tramite un pool limitato a un massimo di 10 connessioni riutilizzabili.

---

## 📂 Struttura del Database

Il database `circolo_tennis` è composto da tre tabelle principali:

1. **Utenti**: Contiene le credenziali e i dati anagrafici dei soci.
   * `genere`: ENUM('M', 'F') per la separazione delle classifiche e la visualizzazione degli avatar.
   * `is_admin`: TINYINT per discriminare i ruoli (Socio / Admin).
2. **Classifiche**: Legata in relazione **1-a-1** con la tabella Utenti. Contiene punteggi e statistiche aggregate (`partite_vinte`, `partite_perse`, `livello`). La chiave esterna ha clausola `ON DELETE CASCADE`.
3. **Partite**: Memorizza lo storico di tutti i match disputati.
   * Contiene chiavi esterne per il `giocatore1`, il `giocatore2` e l' `id_vincitore`.
   * Registra i punteggi per ogni set (fino a un massimo di 3 set, con il terzo opzionale).

---

## 🗂️ Organizzazione delle Pagine

### Area Pubblica (Accessibile a tutti)
* **Home (`index.html`)**: Vetrina del circolo con filosofia del club, attività (Scuola tennis, agonistica, ecc.) e presentazione dello staff.
* **Abbonati (`abbonati.html`)**: Piani di abbonamento mensili e annuali.
* **Contatti (`contatti.html`)**: Form di contatto e mappa interattiva via Google Maps iframe.
* **Autenticazione**: Pagine di `login.html` e `registrati.html` con validazione client-side nativa.

### Area Privata (Solo per soci registrati)
* **Classifica (`ranking.html`)**: Mostra le due graduatorie (Maschile e Femminile) ordinate per punteggio. Per i soli amministratori, sblocca in fondo alla pagina la visualizzazione dello **Storico Incontri Globale** di tutto il circolo.
* **Scheda Personale (`giocatore.html`)**: 
  * Mostra nome, livello, punti e statistiche di gioco.
  * Genera l'avatar del giocatore (`person` / `person_2`) in modo dinamico in base al genere.
  * Mostra il grafico delle performance (Google Charts).
  * Consente l'inserimento di un nuovo incontro (solo sul proprio profilo o tramite account amministrativo).
  * **Pannello Admin**: Se l'utente collegato è amministratore, sblocca il pulsante **Modifica Punteggio** sullo storico del giocatore per rettificare i risultati errati. La modifica ricalcola in automatico punti, vittorie e sconfitte di entrambi i giocatori in modo coerente e protetto da transazioni logiche.

---

## 🚀 Guida all'Avvio (Local Setup)

### 📌 Prerequisiti
* **Node.js** installato sul sistema.
* **MySQL Server** installato e in esecuzione in locale.

### 1. Preparazione del Database
Accedi alla console MySQL o usa un client grafico per eseguire i seguenti comandi:

```sql
-- 1. Crea il database
CREATE DATABASE circolo_tennis;
USE circolo_tennis;

-- 2. Crea l'utente per l'app e assegna i privilegi
CREATE USER 'admin_tennis'@'localhost' IDENTIFIED BY 'Pippo04!';
GRANT ALL PRIVILEGES ON circolo_tennis.* TO 'admin_tennis'@'localhost';
FLUSH PRIVILEGES;

-- 3. Importa la struttura delle tabelle dallo script setup.sql
-- (Esegui questo comando puntando al percorso assoluto del file setup.sql del progetto)
SOURCE /percorso/di/setup.sql;

-- 4. Inserisci l'utente amministratore di default
INSERT INTO Utenti (nome, cognome, email, password, genere, is_admin) 
VALUES ('Admin', 'Admin', 'admin', 'admin', 'M', 1);
```

> 💡 **Nota**: Se utilizzi credenziali o porte differenti per MySQL, assicurati di configurare opportunamente i parametri di connessione all'interno di `app.js` (linea 14-20).

### 2. Configurazione ed Esecuzione dell'Applicazione
Apri il terminale all'interno della cartella principale del progetto:

```bash
# Installa le dipendenze dichiarate in package.json
npm install

# Avvia il server in modalità di sviluppo (con nodemon o node a seconda degli script)
npm run dev
```

Se l'avvio va a buon fine, il terminale mostrerà i seguenti messaggi:
```text
Connesso al database MySQL con successo.
Server attivo su http://localhost:3000
```

Apri la pagina **`http://localhost:3000`** sul browser per iniziare a usare la piattaforma.
