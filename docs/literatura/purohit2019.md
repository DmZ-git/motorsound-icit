# purohit2019 — MIMII Dataset e benchmark de autoencoder

## 1. Referência

Harsh Purohit, Ryo Tanabe, Kenji Ichige, Takashi Endo, Yuki Nikaido, Kaori Suefusa, Yohei Kawaguchi.
"MIMII Dataset: Sound Dataset for Malfunctioning Industrial Machine Investigation and Inspection".
*Proc. of the Detection and Classification of Acoustic Scenes and Events 2019 Workshop (DCASE2019)*,
2019. arXiv:1909.09347. DOI: não encontrado.
Links: https://arxiv.org/abs/1909.09347 ; dados: https://zenodo.org/record/3384388

## 2. Leitura

**Texto completo** (PDF do arXiv, v1).

## 3. Pré-processamento, parâmetros e arquitetura

- Base: válvulas, bombas, ventiladores, trilhos deslizantes; 7 modelos de produto por tipo; arranjo
  de 8 microfones (TAMAGO-03), 50 cm da máquina (10 cm para válvulas); segmentos de **10 s, 16 bit,
  16 kHz**; cada máquina gravada em sessão separada. 26 092 segmentos normais e 6 065 anômalos.
  Anomalias reais: contaminação, vazamento, desbalanceamento, mudança de tensão, entupimento, dano
  no trilho, correia frouxa, falta de graxa.
- Ruído de fábrica real misturado a SNR de **6, 0 e −6 dB** (potência média do ruído ajustada à
  potência média da máquina).
- Features: só canal 1; log-mel com **frame 1024, hop 512, 64 mels**; **5 quadros** concatenados →
  vetor de **320 dim**.
- AE: encoder FC(320,64,ReLU) → FC(64,64,ReLU) → FC(64,8,ReLU); decoder FC(8,64,ReLU) →
  FC(64,64,ReLU) → FC(64,320, linear). Perda ‖x − D(E(x))‖². **Adam, 50 épocas**; lr e lote não
  informados.
- Escore por segmento: erro de reconstrução (agregação sobre quadros: não detalhada; supõe-se média).

## 4. Protocolo de split e risco de vazamento

- Por tipo e modelo de máquina: **todos os anômalos vão para o teste**, o mesmo número de normais é
  sorteado para o teste, o resto dos normais é treino. AE por tipo/modelo. AUC médio de **3 execuções**.
- Unidade = segmento de 10 s (arquivo). O texto não diz se segmentos consecutivos de uma mesma
  sessão longa foram separados; como o cenário é monitorar a mesma máquina, ter a mesma máquina em
  treino e teste é o objetivo, não vazamento.
- **Risco de vazamento: baixo-médio.** Baixo quanto ao classificador (não supervisionado, anômalos
  nunca entram no treino). Médio porque normais e anômalos foram gravados em sessões diferentes; o
  AE pode estar detectando "sessão diferente" (posição, acústica da sala, ruído misturado) além da
  falha. O mesmo vale para o nosso IDMT.

## 5. Números reportados (MIMII, AUC médio por tipo; SNR 6 / 0 / −6 dB)

| Tipo | 6 dB | 0 dB | −6 dB |
|---|---|---|---|
| Valve | 0,67 | 0,61 | 0,53 |
| Pump | 0,81 | 0,74 | 0,68 |
| Fan | 0,94 | 0,84 | 0,70 |
| Slide rail | 0,90 | 0,80 | 0,70 |

Há valores por ID (ex.: bomba ID 04: 0,99 / 0,95 / 0,93; bomba ID 02: 0,45 / 0,46 / 0,52). Conclusão
dos autores: sons estacionários (ventilador) são mais fáceis; válvulas (impulsivas) são difíceis; a
degradação com SNR é forte.

## 6. Custo embarcado

Não reportado. Estimativa nossa: ≈ **50,8 k parâmetros** (≈ 203 kB float32, ≈ 51 kB int8); ≈ 0,1
MFLOP por quadro, ~3 MFLOP/s a 31 quadros/s. Cabe folgado no ESP32 em int8.

## 7. Para reproduzir / veredito

Supor: lr, lote, agregação do escore, seed do sorteio dos normais de teste. Dados públicos.
**Veredito: reproduzível** (com suposições menores) no MIMII; no MotorSound, **inspirada em**.

## 8. Relevância para o MotorSound → `p30_anomalia` (variante `p30b_ae_pequeno`)

É a versão **pequena** do AE (64-64-8) e por isso a mais adequada ao ESP32; também é o único dos
três que mede degradação por SNR, o que conversa diretamente com as condições de ruído do IDMT
(pure, talking, white noise, atmo) e com a nossa bancada `ruidos/`.

- **extrair**: log-mel 64 bandas, n_fft 1024, hop 512 a 16 kHz, 5 quadros → 320 dim.
- **treinar**: só estado normal, agrupado por gravação.
- **prever/limiar**: como em `koizumi2020.md` (percentil 95 da validação normal; AUC e pAUC p = 0,1).
- **experimento derivado**: curva AUC × SNR usando o ruído da bancada, comparável à Tabela 3 deste
  artigo; e repetir para o rastreador Decifrae para ver qual degrada menos.
