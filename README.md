# Social Media Automation System for n8n - Documentazione Tecnica
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
