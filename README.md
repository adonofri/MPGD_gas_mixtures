# MPGD_gas_mixtures
Studies about gain performance for different gas mixtures in MM detectors
INPUTS: https://cernbox.cern.ch/files/spaces/eos/user/a/adonofri/MPGD?sort-by=name&sort-dir=asc&items-per-page=100&files-spaces-generic-view-mode=resource-table&tiles-size=2


### `fits_maker.ipynb`

Questo notebook esegue il **fit degli spettri Fe55** a partire dai file TXT con e senza background (BKG), per più tensioni e per più miscele di gas.

In particolare:

- indicizza automaticamente i file per miscela/camera/tensione;
- costruisce coppie `signal + BKG` e produce un report di pairing;
- sottrae il background e costruisce gli istogrammi ROOT;
- individua il picco con logica robusta (TSpectrum + fallback);
- stima range di fit e parametri iniziali;
- confronta modello a **1 gaussiana** vs **2 gaussiane (con escape)**;
- applica regole specifiche per tensione (es. 545 V, 600 V nella miscela 88/10/2);
- salva:
  - file ROOT con istogrammi, funzioni fit e residui,
  - PNG di fit, background e spettro totale,
  - tabella riepilogativa unica (`output/fit_summary_all_mixtures.csv`) con `mu`, `sigma`, `chi2/ndf`, modello usato.

---

### `draw_gain_from_peaks.ipynb`

Questo notebook costruisce le **curve di guadagno** a partire dai risultati dei fit (posizione del picco `mu1`) presenti nei ROOT prodotti da `fits_maker`.

Funzionalità principali:

- lettura del TTree `fitResults` per ogni miscela;
- ordinamento per tensione;
- applicazione di scaling configurabile del guadagno (es. ×10 sopra una soglia);
- stima errori pesata con qualità del fit (`chi2/ndf`);
- fit esponenziale del guadagno (in log) su intervallo configurabile;
- grafico con:
  - punti con error bar,
  - fit esponenziale,
  - pull `(data-fit)/σ`;
- export per miscela di:
  - tabella CSV del guadagno,
  - PNG della curva,
  - `gain_data.py` e `gain_data.json` riutilizzabili da altri notebook.

---

### `voltage_current_plots.ipynb`

Questo notebook elabora e visualizza le **curve di tensione/corrente** con e senza sorgente, e le confronta con il guadagno.

Cosa fa:

- legge file di corrente/tensione per ciascuna miscela (`V2`, `I2`);
- interpola `I_no_sorg` sulla griglia di tensione della misura con sorgente;
- calcola `I_diff = I_sorg - I_no_sorg_interp`;
- **esclude i punti non validi** se manca `I_sorg` o `I_no_sorg_interp`;
- produce output separati per miscela:
  - canvas ROOT/PNG delle serie temporali,
  - CSV delle tabelle interpolate e differenze;
- genera plot `I_diff` vs `V` (con binning in tensione e fit esponenziale);
- carica `gain_data` e costruisce il plot combinato **corrente + guadagno** su doppio asse;
- permette regole dedicate per miscela (es. uso dei punti di guadagno solo sopra una soglia di tensione).

---

## Workflow consigliato

1. Eseguire `fits_maker.ipynb` per ottenere fit spettrali e file ROOT.
2. Eseguire `draw_gain_from_peaks.ipynb` per calcolare ed esportare le curve di guadagno.
3. Eseguire `voltage_current_plots.ipynb` per analisi corrente/tensione e confronto con guadagno.

