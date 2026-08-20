# WedPhotoUpload

Backend che permette agli invitati di caricare foto e video del matrimonio
direttamente su una cartella Google Drive, senza bisogno di fare login.

Il progetto è diviso in due parti:
- **`wedding-page/index.html`** → va su **GitHub Pages** (la pagina pubblica del
  matrimonio, con il pulsante "Condividi con noi il tuo ricordo" già integrato).
- **tutto il resto** (`server.js`, ecc.) → va su **Render** (il backend: riceve
  i file dalla pagina e li carica su Google Drive).

## 1. Credenziali Google Drive (Google Cloud Console)

1. Vai su https://console.cloud.google.com/ e crea un nuovo progetto (es. "Matrimonio").
2. Menu ☰ → **API e servizi** → **Libreria** → cerca "Google Drive API" → **Abilita**.
3. **API e servizi** → **Schermata consenso OAuth**:
   - Tipo utente: **Esterno**.
   - Compila nome app, email di supporto, email sviluppatore.
   - In "Utenti di test" aggiungi il tuo indirizzo Gmail (quello del Drive che userete).
4. **API e servizi** → **Credenziali** → **Crea credenziali** → **ID client OAuth**:
   - Tipo applicazione: **Applicazione web**.
   - URI di reindirizzamento autorizzati: aggiungi sia
     `http://localhost:3000/oauth2callback` (per i test) sia
     `https://wedphotoupload.onrender.com/oauth2callback` (lo aggiornerai col
     tuo indirizzo reale dopo il deploy su Render).
   - Salva: otterrai **Client ID** e **Client Secret**.
5. (Facoltativo ma consigliato) Su Google Drive crea una cartella dedicata
   "Foto Matrimonio", apri l'URL della cartella e copia l'ID che trovi dopo
   `/folders/` — ti servirà per `GOOGLE_DRIVE_FOLDER_ID`.

## 2. Caricamento su GitHub

Servono **due repository separati** (o due branch, se preferisci):

1. **Repository backend** (es. `wedphotoupload`): carica tutto il contenuto di
   questa cartella **tranne** `wedding-page/` (esclusi anche `node_modules` e
   `.env`, già esclusi da `.gitignore`). Questo è quello che pubblicherai su Render.
2. **Repository/pagina GitHub Pages**: qui va **solo** il contenuto della
   cartella `wedding-page/` (cioè `index.html`), nel repository che già usi
   per la pagina del matrimonio.

## 3. Pubblicazione su Render

1. Vai su https://render.com/ → **New** → **Web Service**.
2. Collega il repository GitHub del **backend** (`wedphotoupload`, senza la
   cartella `wedding-page/`).
3. Impostazioni:
   - **Runtime**: Node
   - **Build command**: `npm install`
   - **Start command**: `npm start`
4. In **Environment**, aggiungi tutte le variabili elencate in `.env.example`
   (tranne `GOOGLE_REFRESH_TOKEN`, che aggiungerai al passo successivo).
   - Per `GOOGLE_REDIRECT_URI` usa l'indirizzo reale assegnato da Render, es.
     `https://wedphotoupload.onrender.com/oauth2callback`.
   - Per `ALLOWED_ORIGIN` usa l'indirizzo della tua pagina GitHub Pages, es.
     `https://tuonomeutente.github.io` (senza slash finale).
5. Fai il deploy. Render ti darà un indirizzo pubblico tipo
   `https://wedphotoupload.onrender.com`.
6. Torna su Google Cloud Console → Credenziali → il tuo client OAuth →
   verifica che l'URI di reindirizzamento coincida esattamente con quello reale.
7. Apri `wedding-page/index.html` e aggiorna la costante `BACKEND_URL` in
   cima allo `<script>` con il tuo indirizzo Render reale, poi ripubblica la
   pagina su GitHub Pages.

## 4. Primo collegamento a Google Drive (una tantum, solo voi due)

1. Apri nel browser:
   `https://wedphotoupload.onrender.com/auth/login?key=IL_TUO_ADMIN_SECRET`
2. Accedi con l'account Google Drive che volete usare e concedi il permesso.
3. Verrai reindirizzato a una pagina con un codice lungo (il "refresh token").
   Copialo.
4. Su Render → il tuo servizio → **Environment** → aggiungi
   `GOOGLE_REFRESH_TOKEN` con quel valore → salva (il servizio si riavvia da solo).

Da questo momento il backend può caricare file su Drive per conto vostro,
senza che gli invitati debbano mai autenticarsi.

## 5. Collegamento dalla pagina del matrimonio (GitHub Pages)

Il pulsante "Condividi con noi il tuo ricordo" è già integrato in
`wedding-page/index.html` e carica i file direttamente su `BACKEND_URL/api/upload`
(vedi punto 7 sopra). L'unica cosa da controllare è che il link che condividi
(o il QR code) includa il token nell'URL, così:

```
https://tuonomeutente.github.io/tuo-repo/?t=IL_TUO_WEDDING_TOKEN
```

(`IL_TUO_WEDDING_TOKEN` è lo stesso valore che hai messo nella variabile
`WEDDING_TOKEN` su Render.)

## 6. Test

1. Apri il link sopra da uno smartphone.
2. Scegli una foto, tocca "Carica".
3. Controlla che il file compaia nella cartella Google Drive configurata.

## 7. QR code definitivo

Genera un QR code che punti al link della pagina del matrimonio su GitHub
Pages (quella con il pulsante "Condividi le tue foto", che a sua volta porta
a `/upload.html?t=...`).

---

### Note di sicurezza

- Non condividere mai `GOOGLE_CLIENT_SECRET`, `GOOGLE_REFRESH_TOKEN`,
  `ADMIN_SECRET` o `WEDDING_TOKEN`: vivono solo nelle variabili d'ambiente
  di Render, mai nel codice su GitHub.
- Lo scope richiesto è `drive.file`: l'app può vedere e scrivere solo i file
  che essa stessa crea, non l'intero Drive.
