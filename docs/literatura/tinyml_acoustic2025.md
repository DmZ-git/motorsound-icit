# tinyml_acoustic2025 — TinyML for Acoustic Anomaly Detection in IoT Sensor Networks

> Nota sobre o id: o id foi atribuído antes da leitura; o preprint é de **março de 2026**
> (arXiv 2603.26135). Mantido o id pedido para não quebrar referências.

## 1. Referência

Amar Almaini, Jakob Folz, Ghadeer Ashour. "TinyML for Acoustic Anomaly Detection in IoT Sensor
Networks". Preprint arXiv:2603.26135v1 [cs.LG], submetido em 27/03/2026. Veículo de publicação
revisado por pares: não encontrado. DOI: não encontrado (só o identificador arXiv).
Links: https://arxiv.org/abs/2603.26135 , https://arxiv.org/html/2603.26135

## 2. Leitura

**Texto completo** (versão HTML do arXiv).

## 3. Pré-processamento, parâmetros e classificador

- Base: **UrbanSound8K** (8 732 clipes, 10 classes de som urbano), **não é som de máquina**.
  Problema reformulado como binário: "normal" (ex.: engine idling, air conditioner, children playing)
  vs "anômalo" (ex.: siren, gunshot, jackhammer, dog bark). O mapeamento completo das 10 classes não
  é dado (só exemplos com "e.g."). Portanto **não é detecção de anomalia não supervisionada**: é
  classificação supervisionada binária com as duas classes vistas no treino.
- Áudio reamostrado a **16 kHz**, cortado/preenchido a duração fixa (duração não informada).
- **13 MFCCs**, quadros de **40 ms** com **50 %** de sobreposição; entrada **13 × 32**, achatada.
- Rede densa: 416 → **128 (ReLU) → Dropout 0,2 → 64 (ReLU) → Dropout 0,2 → 1 (sigmoide)**;
  ≈ 61 825 parâmetros. BCE, Adam lr 0,001, ReduceLROnPlateau, early stopping paciência 5
  (restaura melhores pesos), máx. 50 épocas (parou em 36; figuras dizem 35), lote 32 **e** 64
  (a Tabela I lista os dois), 20 % de validação.
- Quantização para 8 bits e conversão para TensorFlow Lite (Micro); tipo de quantização
  (pós-treino, inteira completa, faixa dinâmica, QAT) não informado.

## 4. Protocolo de split e risco de vazamento

- Split **aleatório estratificado 80/20** (6 985 treino / 1 747 teste). O artigo menciona que o
  UrbanSound8K tem folds padronizados, mas **não diz que os usou**.
- **Risco de vazamento: alto.** O UrbanSound8K é composto de fatias de ≤ 4 s cortadas de gravações
  de origem do Freesound, e os 10 folds oficiais existem justamente para que fatias da mesma
  gravação não caiam em treino e teste. Um 80/20 aleatório quebra isso. Há também inconsistências
  internas: o suporte da Tabela III soma 8 279, nem 1 747 (teste) nem 8 732 (total), o que sugere
  que a matriz de confusão foi calculada sobre quase toda a base, incluindo treino.

## 5. Números reportados (UrbanSound8K, binário)

| Modelo | Acurácia | F1 | ROC AUC | Avg. Precision |
|---|---|---|---|---|
| float32 | 0,95 | 0,95 | 0,991 | 0,992 |
| int8 | 0,91 | 0,91 | 0,970 | 0,970 |

Por classe (int8): normal P 0,88 / R 0,94 / F1 0,91; anômalo P 0,94 / R 0,89 / F1 0,91.

## 6. Custo embarcado

- Modelo int8 ≈ **60 kB**. Nada mais.
- **Não houve implantação em hardware.** ESP32 aparece só na revisão (trabalho de Hammad et al.).
  RAM, flash, latência e energia ficam para "trabalho futuro". O título promete TinyML, mas o artigo
  não mede nada no MCU.

## 7. Para reproduzir / veredito

Faltam: mapeamento completo das classes, duração fixa, lote, tipo de quantização, seed. O resultado
depende de um split com vazamento. **Veredito: só inspiração**, e fraca. Útil apenas como referência
de "como dimensionar um MLP sobre MFCC para TFLM" e como exemplo de armadilha de avaliação.

## 8. Relevância para o MotorSound

Pouca como método; útil como **contraexemplo** na seção de trabalhos relacionados (TinyML acústico
sem medição em MCU e com split que vaza) e como dimensionamento de referência:

- Uma `p05_mlp_mfcc` (opcional) com entrada 13 × N quadros, MLP 128-64, int8, cabe em ~60 kB de flash;
  serviria de ponto de comparação de custo contra `p04_classicas` e contra o rastreador Decifrae.
- Para `p30_anomalia`, não serve como modelo (é supervisionado). A lição é de protocolo: medir RAM,
  flash e latência reais no ESP32 é o que este artigo não fez e o que pode diferenciar o nosso.
- Métricas a herdar: além de acurácia, reportar ROC AUC e AP, e o delta float32 → int8 (aqui −4 p.p.
  de acurácia), que é um número que o revisor do ICIT vai pedir.
