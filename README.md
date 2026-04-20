# QuoteFlash ⚡
### Preventivo rapido per officine e gommisti · v2.2

PWA mobile-first di [PezzaliApp](https://pezzaliapp.com)

---

## Novità v2.2

Architettura più sicura per l'integrazione EPREL.

**Tre livelli di funzionamento dell'integrazione EPREL:**

1. **Solo link (default)** — scan QR → apre scheda EPREL in nuova tab + form con ID/link precompilati. Nessuna configurazione richiesta.
2. **Proxy configurato senza API key** — scan QR → l'app chiama il Worker → se il Worker estrae dati dal PDF pubblico EPREL li popola in automatico, altrimenti flusso manuale.
3. **Proxy con API key ufficiale** — scan QR → Worker con `EPREL_API_KEY` → dati popolati sempre, in automatico.

Passare dal livello 2 al livello 3 **non richiede modifiche alla PWA** — basta aggiungere la variabile d'ambiente sul Worker Cloudflare.

---

## 🔐 Architettura di sicurezza

```
[Gommista sul suo telefono]
          ↓  scansiona QR
[QuoteFlash PWA su GitHub Pages]   ← codice pubblico, ZERO segreti
          ↓  richiesta dati
[Cloudflare Worker privato]         ← EPREL_API_KEY criptata qui
          ↓  autenticato
[EPREL API ufficiale UE]
          ↓
[PWA riceve JSON e compila form]
```

La API key **non** è mai nel codice frontend. Sta solo nel Worker come variabile d'ambiente (Cloudflare Secret).

---

## Funzionalità

### Flusso preventivo a 4 step
1. **Cliente** — nome, targa, veicolo
2. **Pneumatici** — scan QR, ricerca per misura, aggiunta manuale; nuovi/usati; posizione ruota
3. **Servizi** — montaggio, equilibratura, convergenza, assetto, stoccaggio
4. **Output** — PDF, TXT, WhatsApp, webhook Make.com

### Scanner QR multi-formato
- **QR EPREL** → fetch automatico via Worker o flusso manuale
- **EAN-13** con validazione checksum
- **DOT** (WWYY) → calcolo età per usati
- **JSON custom QuoteFlash** → import diretto
- **URL** con misura estratta
- **Testo libero** con pattern misura ETRTO

### Posizione ruota
Set, Anteriore SX/DX, Posteriore SX/DX, Scorta — con posizione specifica la qty viene forzata a 1.

### Classi energetiche EPREL
Dropdown veloce nel form pneumatico: ⛽ Carburante (A–G), 💧 Bagnato (A–G), 🔊 Rumore (A–C).

### Output professionali
- **PDF** con sezioni Pneumatici/Servizi, ID EPREL, link ufficiale, classi energetiche, posizione, DOT
- **TXT copiabile** per WhatsApp/email
- **WhatsApp diretto** con link EPREL cliccabili
- **Make.com webhook** con payload arricchito
- **Offline-ready** via Service Worker
- **Installabile** come PWA

---

## Struttura file

```
quoteflash/
├── index.html       ← App completa (single file)
├── manifest.json    ← PWA manifest
├── sw.js            ← Service Worker
├── icons/
│   ├── icon-192.png
│   └── icon-512.png
└── README.md
```

---

## Impostazioni runtime (⚙)

Tutto salvato in `localStorage`, **mai** trasmesso al codice sorgente.

- Nome officina, telefono, indirizzo, P.IVA
- Webhook Make.com
- Numero WhatsApp default
- **EPREL Proxy URL** — URL del tuo Cloudflare Worker
- Fornitori B2B con placeholder `{w}/{h}/{r}`

---

## 🛠 Setup Cloudflare Worker per EPREL

1. Crea un Worker su [dash.cloudflare.com](https://dash.cloudflare.com) → Workers & Pages → Create Worker
2. Incolla il codice del proxy EPREL (file `eprel-proxy-worker.js` separato)
3. Deploy
4. Nelle impostazioni della PWA incolla l'URL del worker, es. `https://eprel-proxy.tuonome.workers.dev/?id={id}`

**Opzionale — API key ufficiale EPREL:**

Quando ottieni la API key da EPREL (via `ENER-ENERGY-LABELLING@ec.europa.eu`):

1. Cloudflare Dashboard → il tuo Worker → **Settings** → **Variables and Secrets**
2. Click **"Add variable"** → tipo **"Secret"** (non "Text"!)
3. Name: `EPREL_API_KEY` · Value: la tua key
4. Deploy

Il Worker userà automaticamente la key quando presente. Nessuna modifica alla PWA necessaria.

---

## Fornitori B2B — deep-link

In Impostazioni → Fornitori B2B aggiungi i portali con placeholder:

```
Nome: Il Mio Fornitore
URL:  https://b2b.tuofornitore.it/ricerca?w={w}&h={h}&r={r}
```

---

## Deploy su GitHub Pages

```bash
cd ~/Downloads
rm -rf quoteflash-v2
unzip -o quoteflash-v2.zip
cd quoteflash-v2
git init -b main
git add .
git commit -m "v2.2"
git remote add origin https://github.com/pezzaliapp/quoteflash.git
git push -u origin main --force
```

---

## 🔒 Cosa viene condiviso con il link dell'app

Le impostazioni utente (officina, P.IVA, fornitori personalizzati, URL proxy, preventivi) sono salvate in `localStorage` del browser e **non vengono mai incluse** nel codice sorgente pubblico. Altri utenti che aprono la PWA vedono l'app vuota con i soli default.

Nel codice sorgente pubblico non ci sono segreti: API key EPREL, webhook Make.com, URL worker personali devono stare solo nelle impostazioni runtime di ciascun utente.

---

## Changelog

### v2.2 (questa release)
- Rimosso campo "EPREL API Key" dalla PWA (la key va solo sul Worker come Secret)
- Architettura chiara: frontend pubblico + proxy privato
- 2 preset fornitori B2B italiani di esempio (Ciavarella, Intergomma)
- Flusso EPREL semplificato: proxy → dati; senza proxy → link + compilazione manuale

### v2.1
- Posizione ruota, note pneumatico

### v2
- Step Pneumatici separato
- Scanner QR/Barcode multi-formato  
- Ricerca per misura + deep-link fornitori B2B

### v1
- Preventivatore base con PDF/TXT/WhatsApp/Make.com

---

*by [PezzaliApp](https://pezzaliapp.com)*
