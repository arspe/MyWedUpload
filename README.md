# WedPhotoUpload — Guida completa (da zero)

Questa guida presume di ripartire puliti: nessun servizio Render attivo,
nessuna credenziale Google già creata. Segui i passi **in ordine**, uno alla
volta. Ogni passo ha un "✅ Come verificare che ha funzionato" — non passare
al passo successivo finché quella verifica non è andata a buon fine.

Il progetto è diviso in due parti, che vivono in **due posti diversi**:

| Parte | Cosa contiene | Dove va pubblicata |
|---|---|---|
| Backend | `server.js`, `package.json`, ecc. (tutto tranne `wedding-page/`) | **Render** |
| Pagina del matrimonio | `wedding-page/index.html` | **GitHub Pages** |

---

## PARTE 1 — Credenziali Google (10 minuti)

### 1.1 Crea il progetto Google Cloud

1. Vai su **console.cloud.google.com**, accedi con l'account Gmail che
   userete per ricevere le foto (es. quello a cui è collegato il vostro Drive).
2. In alto clicca sul selettore progetto → **Nuovo progetto**.
3. Nome: `Matrimonio` (o quello che preferisci) → **Crea**.
4. Aspetta la notifica che il progetto è pronto, poi selezionalo dal menu in alto.

✅ **Come verificare**: in alto, accanto al logo "Google Cloud", vedi scritto
il nome del progetto che hai appena creato (non "My First Project" o altro).

### 1.2 Abilita la Google Drive API

1. Menu ☰ (in alto a sinistra) → **API e servizi** → **Libreria**.
2. Cerca `Google Drive API`.
3. Cliccaci sopra → **Abilita**.

✅ **Come verificare**: dopo il click su "Abilita", la pagina cambia e mostra
un pulsante "Gestisci" al posto di "Abilita" — significa che è attiva.

### 1.3 Configura la schermata di consenso OAuth

1. Menu ☰ → **API e servizi** → **Schermata consenso OAuth**.
2. Tipo utente: **Esterno** → Crea.
3. Compila: nome app (es. "WedPhotoUpload"), la tua email in "Email di
   supporto utente", la tua email anche in "Dati di contatto sviluppatore".
4. Salva e continua (puoi lasciare vuote le sezioni Ambiti/Scopes qui).
5. Nella sezione **Utenti di test**, clicca "Aggiungi utenti" e inserisci
   l'indirizzo Gmail che userai per il login (lo stesso account del Drive).
6. Salva e continua fino alla fine.

✅ **Come verificare**: nella pagina "Schermata consenso OAuth", alla voce
"Stato di pubblicazione" vedi "Test", e il tuo indirizzo Gmail compare
nell'elenco "Utenti di test".

⚠️ Finché l'app resta in stato "Test" (va benissimo per il matrimonio),
**solo** gli indirizzi email che aggiungi come "utenti di test" possono
completare il login. Se provi ad accedere con un account diverso, Google
blocca l'accesso — non è un errore, è previsto.

### 1.4 Crea le credenziali OAuth

1. Menu ☰ → **API e servizi** → **Credenziali**.
2. **Crea credenziali** → **ID client OAuth**.
3. Tipo di applicazione: **Applicazione web**.
4. Nome: es. "WedPhotoUpload Web".
5. **Lascia vuoto per ora** il campo "URI di reindirizzamento autorizzati"
   — lo compileremo nella Parte 3, quando avremo l'indirizzo Render reale.
   (Se lo lasci vuoto, Google potrebbe darti un avviso: puoi ignorarlo e creare comunque.)
6. Clicca **Crea**.
7. Si apre una finestra con **Client ID** e **Client Secret**: copiali
   entrambi da qualche parte al sicuro (es. le note del telefono), ti
   serviranno tra poco.

✅ **Come verificare**: nella pagina "Credenziali", sotto "ID client OAuth
2.0", vedi una riga con il nome che hai scelto. Cliccandoci sopra, in cima
alla pagina vedi di nuovo il Client ID (una stringa che finisce con
`.apps.googleusercontent.com`) e più sotto il Client Secret (inizia con `GOCSPX-`).

🔒 **Non condividere questi due valori con nessuno.** Se per errore li
incolli in una chat o in un posto pubblico, torna su questa pagina e clicca
"Rigenera secret" per crearne uno nuovo.

### 1.5 (Facoltativo) Crea la cartella su Google Drive

1. Vai su drive.google.com, crea una cartella (es. "Foto Matrimonio").
2. Aprila: nell'indirizzo del browser vedrai qualcosa come
   `.../folders/1a2B3c4D5e...`
3. Copia la parte dopo `/folders/` — è l'ID della cartella.

Se salti questo passo, le foto finiranno nella radice del tuo Drive: va
bene comunque, ma è più ordinato averle in una cartella dedicata.

---

## PARTE 2 — Pubblica il backend su Render (10 minuti)

### 2.1 Carica il backend su GitHub

1. Vai su github.com → **New repository**.
2. Nome: es. `wedphotoupload` → Create repository (può essere privato).
3. Carica **tutto il contenuto di questa cartella tranne `wedding-page/`**
   (quindi: `server.js`, `package.json`, `.gitignore`, `.env.example`,
   questo `README.md`). Il file `.env` (se lo hai creato per test locali)
   **non va mai caricato** — è escluso automaticamente da `.gitignore`.

✅ **Come verificare**: aprendo il repository su GitHub, vedi il file
`server.js` nella lista principale (non dentro nessuna sottocartella).

### 2.2 Crea il Web Service su Render

1. Vai su **render.com** → registrati o accedi (puoi usare "Sign in with GitHub").
2. Dashboard → **New** → **Web Service**.
3. Collega il repository `wedphotoupload` che hai appena creato.
4. Impostazioni:
   - **Name**: scegli un nome semplice e ragionevolmente unico, es.
     `wedphotoupload-armando-vanessa` (nomi troppo generici come "upload"
     rischiano di essere già occupati, e Render aggiunge un `-1` scomodo).
   - **Region**: Frankfurt (o la più vicina a te).
   - **Branch**: main (o quello che hai usato).
   - **Runtime**: **Node** (non Docker).
   - **Build Command**: `npm install`
   - **Start Command**: `npm start`
   - **Instance Type**: Free.
5. **Non cliccare ancora "Create Web Service"** — prima scorri fino a
   "Environment Variables" e aggiungi queste (i valori li completiamo dopo,
   per ora crea le chiavi anche vuote):

   | Chiave | Valore per ora |
   |---|---|
   | `GOOGLE_CLIENT_ID` | *(il Client ID copiato al passo 1.4)* |
   | `GOOGLE_CLIENT_SECRET` | *(il Client Secret copiato al passo 1.4)* |
   | `GOOGLE_REDIRECT_URI` | lascia vuoto, lo mettiamo al passo 2.4 |
   | `GOOGLE_DRIVE_FOLDER_ID` | *(l'ID cartella del passo 1.5, se l'hai fatto)* |
   | `WEDDING_TOKEN` | inventa una parola/frase senza spazi, es. `Sposi2026Ricordi` |
   | `ADMIN_SECRET` | inventa una password lunga e casuale, es. `Kx9mPz3qLw7nRt2vBs8e` |
   | `SESSION_SECRET` | un'altra stringa casuale qualsiasi, es. `Hj4wQe9rTy2uIo6pAs1d` |
   | `ALLOWED_ORIGIN` | lascia vuoto per ora, lo mettiamo al passo 3.3 |

6. Ora clicca **Create Web Service**. Render inizia il primo deploy (ci
   vuole 1-2 minuti).

✅ **Come verificare**: aspetta che in alto compaia un pallino verde con
scritto **"Live"**. Se invece vedi "Deploy failed", vai su "Logs" e leggi
l'errore in rosso.

### 2.3 Trova e annota il tuo indirizzo Render

1. In cima alla pagina del servizio, sotto il nome, trovi l'indirizzo
   pubblico assegnato, tipo `https://wedphotoupload-armando-vanessa.onrender.com`.
2. **Copialo e tienilo a portata di mano** — lo useremo più volte da qui in poi.
3. Da questo momento in poi, in questa guida, ogni volta che leggi
   `TUO-INDIRIZZO-RENDER` intendo esattamente questo valore.

✅ **Come verificare**: apri `TUO-INDIRIZZO-RENDER/health` nel browser
(esempio: `https://wedphotoupload-armando-vanessa.onrender.com/health`).
Deve rispondere `{"ok":true,"driveCollegato":false}`. `driveCollegato:false`
è corretto per ora, lo sistemiamo alla Parte 4.

⚠️ Se invece resta su una schermata animata "Application Loading" per più
di 90 secondi senza cambiare, il servizio non è "Live" — torna al 2.2 e
controlla i Logs.

### 2.4 Completa GOOGLE_REDIRECT_URI

1. Render → il tuo servizio → **Environment**.
2. Modifica `GOOGLE_REDIRECT_URI` mettendo: `TUO-INDIRIZZO-RENDER/oauth2callback`
   (esempio: `https://wedphotoupload-armando-vanessa.onrender.com/oauth2callback`)
3. Salva. Render riavvia da solo il servizio (aspetta che torni "Live").

✅ **Come verificare**: rileggi la variabile e controlla che finisca
esattamente con `/oauth2callback`, senza spazi o doppio slash.

### 2.5 Registra lo stesso indirizzo su Google Cloud

1. Torna su Google Cloud Console → **Credenziali** → apri il tuo ID client OAuth.
2. In "URI di reindirizzamento autorizzati" → **Aggiungi URI**.
3. Incolla **esattamente** lo stesso valore del passo 2.4:
   `TUO-INDIRIZZO-RENDER/oauth2callback`
4. **Salva**.

✅ **Come verificare**: ricarica la pagina delle credenziali e controlla che
l'URI compaia nell'elenco, identico carattere per carattere a quello su Render.

⚠️ Questo è il passo dove nascono più errori. I due valori (su Render e su
Google) devono essere **identici al 100%**: stesso `https://`, stesso
dominio, nessuno spazio, nessun `/` in più o in meno alla fine.

---

## PARTE 3 — Pubblica la pagina del matrimonio (5 minuti)

### 3.1 Aggiorna l'indirizzo del backend nella pagina

1. Apri `wedding-page/index.html` con un editor di testo qualsiasi.
2. Cerca la riga (vicino all'inizio dello `<script>`):
   ```js
   const BACKEND_URL = '...';
   ```
3. Sostituiscila con il tuo indirizzo Render reale:
   ```js
   const BACKEND_URL = 'TUO-INDIRIZZO-RENDER';
   ```
   (senza `/` alla fine)

### 3.2 Carica la pagina su GitHub Pages

1. Crea un **secondo** repository GitHub (diverso da quello del backend),
   oppure usa quello che magari già usi per il sito del matrimonio.
2. Carica dentro **solo** `index.html` (il contenuto di `wedding-page/`).
3. Vai su quel repository → **Settings** → **Pages**.
4. In "Source" scegli il branch `main` e cartella `/ (root)` → **Save**.
5. Aspetta 1-2 minuti: GitHub ti mostrerà l'indirizzo pubblico, tipo
   `https://tuonomeutente.github.io/nomerepository/`.

✅ **Come verificare**: apri quell'indirizzo, deve mostrare la pagina del
matrimonio con lo stile grafico che hai preparato.

### 3.3 Completa ALLOWED_ORIGIN su Render

1. Prendi **solo la parte di dominio** dell'indirizzo GitHub Pages, cioè
   `https://tuonomeutente.github.io` (senza il nome del repository dopo,
   e senza slash finale).
2. Render → il tuo servizio → Environment → `ALLOWED_ORIGIN` → incolla
   quel valore → Salva.

✅ **Come verificare**: `ALLOWED_ORIGIN` deve contenere solo
`https://tuonomeutente.github.io`, niente altro dopo.

---

## PARTE 4 — Primo collegamento a Google Drive (una tantum)

Questo passo lo fate **solo voi due**, una volta sola. Gli invitati non lo
vedranno mai.

1. Apri (da un browser, loggato con l'account Gmail che hai aggiunto come
   "utente di test" al passo 1.3):
   ```
   TUO-INDIRIZZO-RENDER/auth/login?key=IL-TUO-ADMIN-SECRET
   ```
   (sostituisci `IL-TUO-ADMIN-SECRET` con il valore che hai messo nella
   variabile `ADMIN_SECRET` al passo 2.2)

2. **Non aprire mai `/oauth2callback` direttamente**: quella pagina si apre
   da sola, solo dopo che avrai dato il consenso a Google. Se la apri a mano
   vedrai "codice di autorizzazione mancante" — è normale, semplicemente
   non è il modo giusto di arrivarci.

3. Google ti mostra una schermata che chiede il permesso di accedere al
   Drive. Clicca **Continua** / **Consenti**.

4. Vieni riportato automaticamente sul tuo backend, con una pagina che dice
   "✅ Collegamento a Google Drive riuscito!" e mostra un codice lungo.

5. Copia quel codice → Render → Environment → aggiungi una nuova variabile
   `GOOGLE_REFRESH_TOKEN` → incolla il codice → Salva.

✅ **Come verificare**: apri di nuovo `TUO-INDIRIZZO-RENDER/health` — ora
deve rispondere `{"ok":true,"driveCollegato":true}` (`true`, non più `false`).

---

## PARTE 5 — Test finale

1. Apri sul telefono: `https://tuonomeutente.github.io/nomerepository/?t=IL-TUO-WEDDING-TOKEN`
   (sostituisci con il valore che hai messo in `WEDDING_TOKEN`)
2. Tocca il pulsante di condivisione foto, scegli una foto di prova, invia.
3. Controlla la cartella Google Drive: la foto deve comparire lì entro
   qualche secondo.

Se tutto funziona, genera il QR code puntando all'indirizzo del punto 1, e
sei pronto per il matrimonio.

---

## Errori frequenti e cosa significano

| Messaggio | Causa più probabile | Cosa fare |
|---|---|---|
| `Missing required parameter: client_id` | `GOOGLE_CLIENT_ID` è vuoto su Render | Vai al passo 2.2, ricontrolla il valore |
| `Errore 401: invalid_client` / `The OAuth client was not found` | `GOOGLE_CLIENT_ID` su Render non corrisponde a quello su Google Cloud | Ricopia il Client ID direttamente da Google Cloud Console (passo 1.4) dentro Render, senza passaggi intermedi |
| `redirect_uri_mismatch` | I due valori dei passi 2.4 e 2.5 non sono identici | Confrontali carattere per carattere |
| `This service has been suspended` | Il servizio Render è stato sospeso (limiti del piano, verifica account) | Controlla la dashboard Render e la tua email per il motivo |
| Pagina "Application Loading" bloccata a lungo | Il servizio si sta risvegliando dalla pausa (piano gratuito) oppure sta avendo un problema di avvio | Aspetta 90 secondi senza ricaricare; se non cambia, controlla i Logs |
| "Codice di autorizzazione mancante" su `/oauth2callback` | Hai aperto quella pagina direttamente invece di partire da `/auth/login` | Riparti dal passo 4.1 |
| L'indirizzo Render ha un `-1` alla fine diverso da quello che ti aspettavi | Il nome scelto per il servizio era già in uso; Render ne ha assegnato uno leggermente diverso | Usa l'indirizzo realmente assegnato ovunque (non quello che avevi in mente), aggiornando tutti i passi 2.3 in poi |

---

## Note di sicurezza

- `ADMIN_SECRET`, `GOOGLE_CLIENT_SECRET`, `GOOGLE_REFRESH_TOKEN` non vanno
  mai condivisi né inseriti nel codice caricato su GitHub — vivono solo
  nelle variabili d'ambiente di Render.
- Lo scope Google richiesto è `drive.file`: l'app può leggere e scrivere
  solo i file che crea lei stessa, non l'intero vostro Drive.
- `WEDDING_TOKEN` è l'unico valore pensato per essere condiviso (dentro il
  QR code) — è quello che impedisce a estranei di caricare file a caso.
