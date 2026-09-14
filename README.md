# Pannello e-ink per Guest House — TRMNL DIY + Home Assistant

Pannello informativo su e-paper 7.5" per una struttura ricettiva: meteo,
vento, raccolta differenziata, Wi-Fi, e pagine dedicate di benvenuto/commiato
per gli ospiti — bilingue (italiano/inglese), a batteria, pilotabile da
Home Assistant.

![Pannello acceso](images/screenshots/pannello-home.jpg)

## Caratteristiche

- **Data, meteo attuale e previsioni a 5 giorni** (temperatura, umidità,
  temperatura percepita, vento con icone dinamiche in scala Beaufort)
- **Card raccolta differenziata** con icone per categoria, "fotografata" da
  una dashboard Home Assistant dedicata
- **Due QR code**: Wi-Fi (connessione automatica) e un secondo a scelta
  (es. captive portal con le info del soggiorno)
- **Pagine check-in / check-out** dedicate, con testo e icone di benvenuto
- **Bilingue IT/EN**, cambio lingua da un pulsante in Home Assistant
- **Gestione batteria**: deep sleep tra un aggiornamento e l'altro, un tasto
  fisico come risveglio immediato, due come navigazione tra le pagine
- **Pilotabile da remoto**: un helper Home Assistant decide quale pagina
  mostrare, letto automaticamente ad ogni risveglio del pannello

## Hardware

- [Seeed Studio TRMNL 7.5" (OG) DIY Kit](https://www.seeedstudio.com/) —
  ESP32-S3, e-paper 800×480, modello Waveshare `7.50inv2p`
- Batteria 2000mAh (montata in una cornice fotografica)
- Home Assistant con supervisor (per gli add-on)

## Stack software

- **ESPHome** (framework `esp-idf`, non Arduino — necessario per il
  supporto PSRAM con immagini di queste dimensioni)
- **Home Assistant**, con:
  - [HACS](https://hacs.xyz/) e i custom components: `button-card`,
    `card-mod`, `browser_mod`, `mushroom`, `hui-element`
  - Add-on [Puppet](https://github.com/balloob/home-assistant-addons)
    (screenshot headless di una dashboard, usato per la card raccolta)
  - Integrazione meteo con supporto a `weather.get_forecasts` (per le
    previsioni a 5 giorni)
  - [ha_garbage](https://github.com/Simonz82/ha_garbage) (o equivalente)
    per la raccolta differenziata

## Struttura del repository

```
esphome/
  pannello-eink-trmnl.yaml   # file principale del device
  secrets.yaml.example       # rinomina in secrets.yaml e compila
home-assistant/
  helpers.yaml                     # input_select per pagina e lingua
  automazione-timeout-pannello.yaml
  card-eink-raccolta-it.yaml       # card "fotografata" da Puppet (IT)
  card-eink-raccolta-en.yaml       # stessa card, tradotta al volo (EN)
  badge-guest-house.yaml           # badge con popup di controllo rapido
  template-meteo-vento.yaml        # sensori HA per le previsioni a 5 giorni
images/
  checkin.png / checkin_en.png
  checkout.png / checkout_en.png
  rifiuti_eink/                    # icone monocrome per categoria rifiuto
```

## Setup

### 1. Home Assistant

1. Installa via HACS: `button-card`, `card-mod`, `browser_mod`, `mushroom`
   (per il badge), e l'add-on **Puppet** dal repository
   `https://github.com/balloob/home-assistant-addons`
2. Crea un utente dedicato (es. "Puppet"), solo accesso locale, e genera
   per lui un token di accesso a lunga durata da usare nella configurazione
   dell'add-on
3. Crea una nuova vista/dashboard con percorso `eink-raccolta` (e una
   gemella `eink-raccolta-eng` per l'inglese), layout **Pannello (singola
   scheda)**, e incollaci il contenuto di `card-eink-raccolta-it.yaml` /
   `-en.yaml`
4. Aggiungi i template sensor per le previsioni meteo a 5 giorni — vedi
   `home-assistant/template-meteo-vento.yaml`
5. Crea i due helper (`home-assistant/helpers.yaml`): pagina del pannello
   (Home / Check-in / Check-out) e lingua (Italiano / English)
6. Aggiungi l'automazione di sicurezza che riporta il pannello su Home dopo
   16 ore (`automazione-timeout-pannello.yaml`)
7. Carica le immagini di `images/` in `config/www/`
8. (Opzionale) aggiungi il badge di controllo rapido
   (`badge-guest-house.yaml`) a una dashboard

### 2. ESPHome

1. Copia `esphome/secrets.yaml.example` in `secrets.yaml` e compilalo con
   le tue credenziali WiFi, la chiave API e la password OTA
2. Apri `pannello-eink-trmnl.yaml` e cambia `ha_ip` in cima al file con
   l'IP della tua istanza Home Assistant
3. Scarica i font necessari e mettili in `esphome/fonts/`:
   - [Material Design Icons](https://github.com/Templarian/MaterialDesign-Webfont)
     (`materialdesignicons-webfont.ttf`)
   - [Weather Icons](https://github.com/erikflowers/weather-icons)
     (`weathericons-regular-webfont.ttf`)
4. Compila e flasha da ESPHome Dashboard (framework **esp-idf**, non
   Arduino)

## Cose imparate facendolo (utile se adatti il progetto)

- Il modello di pannello `7.50inv2p` ha **polarità colore invertita**
  rispetto al normale: bisogna scambiare i colori "on"/"off" sia nel
  disegno diretto (`Color(0,0,0)`/`Color(255,255,255)` invertiti) sia nelle
  immagini scaricate (`it.image(..., color_off, color_on)` — ordine
  scambiato rispetto a quanto ci si aspetterebbe, perché ESPHome classifica
  come "on" i pixel *chiari*, non quelli scuri, per le immagini binarie)
- I font `gfonts://` di ESPHome non includono di default lettere accentate
  o cifre estese — vanno dichiarate esplicitamente in `glyphs:`
- `deep_sleep` e richieste HTTP vanno d'accordo solo se si aspetta
  esplicitamente la connessione WiFi/API (`wait_until`) prima di lanciare
  il fetch, altrimenti la richiesta parte troppo presto e fallisce
- Il pannello dorme la maggior parte del tempo: i comandi remoti (pulsanti
  Home Assistant) funzionano solo se il device è già sveglio in quel
  momento. Per un controllo affidabile anche a device addormentato, si usa
  un helper che il pannello **legge da solo** ad ogni risveglio, invece di
  aspettarsi un comando push

## Licenza

MIT — vedi [LICENSE](LICENSE).
