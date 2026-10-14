# koizumi2020 — DCASE 2020 Task 2: baseline de autoencoder para detecção não supervisionada de som anômalo

## 1. Referência

Yuma Koizumi, Yohei Kawaguchi, Keisuke Imoto, Toshiki Nakamura, Yuki Nikaido, Ryo Tanabe,
Harsh Purohit, Kaori Suefusa, Takashi Endo, Masahiro Yasuda, Noboru Harada.
"Description and Discussion on DCASE2020 Challenge Task2: Unsupervised Anomalous Sound Detection for
Machine Condition Monitoring". *Proc. of the Detection and Classification of Acoustic Scenes and Events
2020 Workshop (DCASE2020)*, Tóquio, 2–3/11/2020. arXiv:2006.05822. DOI: não encontrado.
Links: https://arxiv.org/abs/2006.05822 ; código oficial: https://github.com/y-kawagu/dcase2020_task2_baseline

## 2. Leitura

**Texto completo** (PDF do arXiv, v2) **e o código oficial** (`baseline.yaml`, `keras_model.py`, `README.md`).
A Tabela 1 do PDF (AUC/pAUC por Machine ID) saiu embaralhada na extração de texto; os números por ID
não foram transcritos. As médias por tipo de máquina abaixo vêm do README do código.

## 3. Pré-processamento, parâmetros e arquitetura

- Dados: partes do ToyADMOS (ToyCar, ToyConveyor) e do MIMII (fan, pump, slider, valve), só o canal 1,
  clipes de ~10 s, **16 kHz**, já misturados com ruído de fábrica real.
- Features: log-mel; quadros concatenados ±P em torno de t.
  - **Divergência artigo × código:** o artigo diz F = 64 mels e "uma camada de entrada, três ocultas de
    128, gargalo 8"; o `baseline.yaml` usa **n_mels = 128, frames = 5, n_fft = 1024, hop = 512, power = 2**
    (entrada de 640 dim) e o `keras_model.py` tem **4 camadas Dense(128) + BatchNorm + ReLU, gargalo
    Dense(8), 4 camadas Dense(128), saída Dense(640)**. Para reproduzir, seguir o código.
- Treino: só sons normais, MSE, **Adam lr 0,001, 100 épocas, lote 512**, `validation_split` 0,1,
  um AE por Machine Type/ID.
- Escore de anomalia por clipe: média sobre os T quadros de ‖ψ_t − D(E(ψ_t))‖² (Eq. 4).
- Decisão: anômalo se A(x) > φ; **o limiar não é definido** pela tarefa (as métricas são livres de limiar).
- Métricas: **AUC** e **pAUC** com FPR em [0, p], **p = 0,1**.

## 4. Protocolo de split e risco de vazamento

- Desenvolvimento: por Machine Type/ID, ~1 000 clipes normais de treino; teste com 100–200 clipes
  normais e 100–200 anômalos. Avaliação: IDs diferentes dos de desenvolvimento, ~400 clipes de teste por
  ID sem rótulo, e ~1 000 normais de treino adicionais desses IDs.
- Unidade = clipe de 10 s; os clipes de treino e teste são arquivos distintos. O AE é treinado e testado
  na mesma máquina (ID), o que é o cenário de monitoramento pretendido, não vazamento.
- **Risco de vazamento: baixo** para o protocolo da base. Ressalva (levantada depois em harada2023):
  equipes ajustavam hiperparâmetros por tipo de máquina usando o AUC calculado com as anomalias de teste,
  o que vaza informação do teste. O baseline em si usa hiperparâmetros fixos.

## 5. Números reportados

Médias por tipo de máquina, conjunto de desenvolvimento, README do código (AUC / pAUC):

| Tipo | AUC | pAUC |
|---|---|---|
| ToyCar | 0,801 | 0,672 |
| ToyConveyor | 0,727 | 0,607 |
| fan | 0,652 | 0,526 |
| pump | 0,726 | 0,600 |
| slider | 0,850 | 0,669 |
| valve | 0,665 | 0,506 |

O artigo reporta média ± desvio de 10 execuções por ID (Tabela 1; não transcrita). O baseline ficou
em 33º no ranking de equipes; 117 submissões de 40 equipes. As abordagens vencedoras foram
classificação de Machine ID (outlier exposure) e AE condicionado ao ID.

## 6. Custo embarcado

Não reportado. Estimativa nossa a partir do código: ≈ 266 k pesos densos + ~4 k de BatchNorm ≈
**270 k parâmetros** (≈ 1,1 MB em float32, ≈ 270 kB em int8). Por quadro de 32 ms, ≈ 0,53 MFLOP;
a 31 quadros/s ≈ 17 MFLOP/s, viável no ESP32-S3 em int8, mas grande para um ESP32 sem PSRAM.

## 7. Para reproduzir / veredito

Código, configuração e dados públicos; a única suposição é escolher entre a versão do artigo e a do
código (adotar a do código). **Veredito: reproduzível** (no DCASE2020). No MotorSound seria uma
**reimplementação inspirada em**, porque as bases são outras.

## 8. Relevância para o MotorSound → `p30_anomalia` (variante `p30a_ae`)

Baseline canônico de anomalia. Mapeamento:

- **extrair**: log-mel 128 bandas (IDMT a 44,1 kHz: reamostrar para 16 kHz para comparar com A2212;
  n_fft 1024, hop 512), contexto de 5 quadros → 640 dim. Versão embarcável: 64 mels, 3 quadros.
- **treinar**: só janelas de **estado normal** do treino (IDMT: `good` de `train/`; A2212: saudável).
- **prever**: escore = MSE médio de reconstrução na janela de avaliação; probabilidade = posto do escore
  na distribuição de escores de validação normal (calibração empírica).
- **limiar**: percentil 95 (ou 99) dos escores de **validação normal agrupada por gravação**, o que
  fixa FPR ≈ 5 % sem olhar anomalias; reportar também AUC e **pAUC (p = 0,1)** livres de limiar.
- **Armadilha específica do IDMT:** cada classe é uma unidade física diferente; um AE treinado no
  `good` vai separar `broken` e `heavyload` também por identidade do motor. Toda tabela de anomalia
  no IDMT precisa declarar isso. Na A2212, treinar no saudável do motor k e testar hélice do mesmo
  motor k (k = 1, 2) é o único teste limpo; rolamento (só motor 4, sem saudável do motor 4) é
  confundido com identidade.
