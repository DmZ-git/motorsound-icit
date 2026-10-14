# estacio2023 — Artigo de dados da base BLDC A2212

## 1. Referência

R. S. Prieto Estacio, D. A. Bravo Montenegro, C. F. Rengifo Rodas. "Dataset of audio signals from brushless
DC motors for predictive maintenance". *Data in Brief* 50:109569, 2023. DOI 10.1016/j.dib.2023.109569.
PMC10539630: https://pmc.ncbi.nlm.nih.gov/articles/PMC10539630/
Dados: Mendeley Data, DOI 10.17632/j4yr5fmhv4.1 (https://data.mendeley.com/datasets/j4yr5fmhv4/1);
espelho no Kaggle (amirberenji/brushless-dc-motor-sound-dataset-for-pdm), usado pelo ETNeXt.

## 2. Leitura

**Texto completo** (artigo de dados curto).

## 3. Aquisição e parâmetros

- 43 wav de ~10 s, **16 kHz**, microfone condensador MCJR-005, gravação no Audacity.
- Motores A2212 com ESC de 30 A, LiPo 11,1 V / 2 100 mAh, velocidade controlada por ESP32 (potenciômetro ou
  slider Blynk); a velocidade é incrementada de um passo fixo por categoria. O nome do arquivo é a velocidade.
- Estrutura: Bearing/M4 (1700–2000, 7 arquivos); Healthy/M1 (1400–2000, 8), M2 (1400–2000, 8),
  M3 (1450–1950, 6); Propeller/M1 (1400–1850, 6), M2 (1400–2000, 8).
- O texto diz que cada arquivo carregado vira "um array 160 000 × 1" (mono). Na verificação local os arquivos
  são **estéreo** e um deles está a 44,1 kHz — divergência entre artigo e arquivos.
- Única análise: correlação de Pearson entre 20 atributos estatísticos/espectrais e as 3 classes (Fig. 2);
  a lista dos 20 atributos só aparece na figura (não encontrada no texto).
- Limitação declarada: ambiente controlado com pouco ruído externo.

## 4. Protocolo de split e risco de vazamento

Não há classificação nem split no artigo. Riscos estruturais para quem usar a base:
(a) **rolamento só no motor 4** → classe confundida com identidade do motor;
(b) **faixas de velocidade diferentes por classe** (rolamento só 1700–2000; hélice no M1 só até 1850) →
a velocidade é um atalho possível;
(c) só 43 gravações, então qualquer recorte em segmentos exige validação agrupada por arquivo
(e, idealmente, por motor).

**Risco de vazamento: não se aplica ao artigo (sem resultados)**; risco estrutural alto para trabalhos derivados.

## 5. Números reportados

Nenhum resultado de classificação.

## 6. Veredito

Fonte primária da base, não da pipeline. Usar para `docs/DATASETS.md` e para declarar as confusões
(rolamento × M4, velocidade × classe) em toda tabela.
