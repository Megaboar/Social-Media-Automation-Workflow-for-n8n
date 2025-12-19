# Social Media Automation System for n8n - Facebook, Instagram, TikTok, YouTube, X e WhatsApp - Documentazione Tecnica
Questa documentazione descrive il funzionamento del sistema di automazione per la gestione, generazione e pubblicazione di contenuti social, suddiviso in due macro-workflow: Acquisizione/Generazione e Approvazione/Pubblicazione.

⚠️ NOTA IMPORTANTE SULLA GESTIONE MEDIA
Il sistema è progettato per la massima coerenza cross-platform. Per ogni sessione di invio, viene generata (o scaricata) una sola risorsa media (immagine o video) che verrà utilizzata indistintamente per tutti i social network selezionati.

Nessuna duplicazione: Non vengono generate immagini o video differenti per ogni social.

Efficienza: Questo approccio garantisce che il brand message sia identico su ogni piattaforma e ottimizza l'uso delle risorse API (OpenAI) e dello spazio di archiviazione.

⚠️ Nel codice JSON ho posizionato vari placeholder per tutte le occorrenze di dati sensibili.
Tutti i placeholder iniziano in questo modo:
<PUT HERE ...>
Cercate nel codice i placeholder e sostituiteli con i vosti dati e/o percorsi.

1. Architettura Generale
Il sistema gestisce il ciclo di vita completo di un post social in quattro fasi:

Ingresso: Ricezione dati da Google Form/Sheet.

Elaborazione Centralizzata: Normalizzazione e generazione/download del media "master".

Approvazione: Validazione umana tramite ricezione mail di approvazione gestita tramite Webhook.

Pubblicazione Multipiattaforma: Invio del media master e del testo specifico a Facebook, Instagram, TikTok, YouTube, X e WhatsApp.

2. Workflow 1: Acquisizione, Generazione e Archiviazione
Questo workflow gestisce la creazione dell'asset unico.

Fase 1: Normalizzazione Dati (Normalize Data1)
Riceve le risposte dal tab Risposte_del_modulo.

Identità Unica: Crea un session_id che lega tutti i post di una sottomissione allo stesso file media.

Mapping Media: Determina se la fonte è openai (AI-generated) o drive (upload utente).

Fase 2: Estrazione Task Unico (Extract Media Tasks1)
È il nodo che garantisce l'unicità del media. Invece di creare un task per ogni social, questo nodo estrae solo i task media unici per sessione.

Se l'utente seleziona 5 social, questo nodo istruisce il sistema a generare un solo file.

Fase 3: Loop di Produzione Asset
Generazione/Download: Il file viene creato da OpenAI o scaricato da Google Drive.

Asset Master: Il file viene caricato via FTP in una cartella specifica.

URL Pubblico: Viene generato un singolo public_url che servirà da sorgente per tutti i nodi di pubblicazione social successivi.

3. Workflow 2: Approvazione e Pubblicazione
Gestisce l'invio dell'asset master ai vari endpoint API.

Fase 1: Filtro di Approvazione
Il Webhook riceve il comando di sblocco. Il sistema recupera tutte le righe associate al session_id e, se approvate, le invia al routing.

Fase 2: Distribuzione Social (Switch Platform)
In questa fase, il flusso si divide per piattaforma, ma tutti i rami attingono allo stesso URL prodotto nel Workflow 1:

Facebook/Instagram: Caricano l'immagine/video unico tramite l'URL dell'asset master.

YouTube/TikTok: Utilizzano lo stesso file video per Shorts e post.

X/WhatsApp: Distribuiscono il medesimo media ai rispettivi destinatari.

4. Specifiche delle Colonne (Tab Variabili)
Per mantenere questo controllo centralizzato, il database Variabili_di_input-output utilizza:

common_image_url / common_video_url: I link agli asset generati validi per l'intera sessione.

media_source: Indica se il media comune proviene da AI o da un file personale caricato su Drive.

status: Monitora lo stato di ogni singolo post (es. "PUBLISHED" per Facebook, "ERROR" per Instagram), mantenendo il media invariato.

5. Vantaggi del Sistema
Risparmio Token: Generando un'unica immagine con DALL-E per 6 social, il costo API è ridotto all'osso.

Uniformità: Si evita il rischio che l'AI generi immagini leggermente diverse (es. un logo spostato) tra un social e l'altro.

Velocità: Il download da Google Drive avviene una sola volta, velocizzando l'intero processo di archiviazione FTP.

Documentazione Tecnica n8n Versione: 2.3 Focus: Asset Unico Cross-Social

## ⚖️ Licenza e Condizioni d'Uso
Questo progetto è rilasciato sotto licenza **PolyForm Noncommercial 1.0.0**. 
È consentito:
- Lo studio del codice e dei workflow.
- L'uso personale non a scopo di lucro.

È rigorosamente vietato:
- L'utilizzo dei workflow per attività commerciali, agenzie o servizi a pagamento senza autorizzazione.
- La vendita del codice o di derivati.

Per proposte di collaborazione o licenze commerciali, contattami su: orvio@yahoo.com


--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


# Social Media Automation System for n8n – Facebook, Instagram, TikTok, YouTube, X and WhatsApp – Technical Documentation

This document describes how the automation system works for managing, generating, and publishing social content. The system is split into two macro-workflows: **Acquisition/Generation** and **Approval/Publishing**.

---

## ⚠️ IMPORTANT NOTE ABOUT MEDIA HANDLING

The system is designed for maximum cross-platform consistency. For each submission session, **a single media asset** (image or video) is generated (or downloaded) and then used for **all** selected social networks.

* **No duplication:** No different images or videos are generated per social network.
* **Efficiency:** This approach ensures the brand message is identical on every platform and optimizes both API usage (OpenAI) and storage space.

⚠️ In the JSON code I placed several placeholders for every occurrence of sensitive data.
All placeholders start like this:

`<PUT HERE ...>`

Search the code for placeholders and replace them with your own data and/or paths.

---

## 1. General Architecture

The system manages the full lifecycle of a social post in four phases:

1. **Input:** Data received from Google Form/Sheet.
2. **Centralized Processing:** Normalization and generation/download of the “master” media.
3. **Approval:** Human validation via an approval email handled through a Webhook.
4. **Multi-Platform Publishing:** Sending the master media and platform-specific text to Facebook, Instagram, TikTok, YouTube, X, and WhatsApp.

---

## 2. Workflow 1: Acquisition, Generation, and Archiving

This workflow handles creation of the single unique asset.

### Phase 1: Data Normalization (Normalize Data1)

* Receives responses from the **Risposte_del_modulo** tab.
* **Unique identity:** Creates a `session_id` that links all posts in a submission to the same media file.
* **Media mapping:** Determines whether the source is `openai` (AI-generated) or `drive` (user upload).

### Phase 2: Single Task Extraction (Extract Media Tasks1)

This node guarantees media uniqueness. Instead of creating one task per social network, it extracts only the **unique media tasks per session**.

If the user selects 5 social networks, this node instructs the system to generate **only one file**.

### Phase 3: Asset Production Loop

* **Generation/Download:** The file is created by OpenAI or downloaded from Google Drive.
* **Master asset:** The file is uploaded via FTP into a specific folder.
* **Public URL:** A single `public_url` is generated and used as the source for all subsequent social publishing nodes.

---

## 3. Workflow 2: Approval and Publishing

This workflow sends the master asset to the various API endpoints.

### Phase 1: Approval Filter

The Webhook receives the unlock command. The system retrieves all rows associated with the `session_id` and, if approved, routes them to the publishing logic.

### Phase 2: Social Distribution (Switch Platform)

At this stage, the flow splits by platform, but all branches use the same URL produced in Workflow 1:

* **Facebook/Instagram:** Upload the single image/video via the master asset URL.
* **YouTube/TikTok:** Use the same video file for Shorts and posts.
* **X/WhatsApp:** Distribute the same media to their respective targets.

---

## 4. Column Specifications (Variables Tab)

To keep control centralized, the `Variabili_di_input-output` database uses:

* `common_image_url` / `common_video_url`: Links to the generated assets valid for the entire session.
* `media_source`: Indicates whether the shared media comes from AI or from a personal file uploaded to Drive.
* `status`: Tracks the status of each single post (e.g., `"PUBLISHED"` for Facebook, `"ERROR"` for Instagram), while keeping the media unchanged.

---

## 5. System Advantages

* **Token savings:** Generating a single DALL·E image for 6 social networks drastically reduces API cost.
* **Uniformity:** Avoids the risk that the AI generates slightly different images (e.g., a shifted logo) across platforms.
* **Speed:** Downloading from Google Drive happens only once, speeding up the entire FTP archiving process.

**n8n Technical Documentation – Version:** 2.3
**Focus:** Single cross-social asset

---

## ⚖️ License and Terms of Use

This project is released under the **PolyForm Noncommercial 1.0.0** license.

**Allowed:**

* Studying the code and workflows.
* Personal, non-profit use.

**Strictly prohibited:**

* Using the workflows for commercial activities, agencies, or paid services without authorization.
* Selling the code or derivatives.

For collaboration proposals or commercial licensing, contact me at: **[orvio@yahoo.com](mailto:orvio@yahoo.com)**
