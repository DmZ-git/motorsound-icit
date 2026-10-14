# ertarin2025 — Imagens SDP + CNN no IDMT

## 1. Referência

M. Ertarin, N. Yordanov, M. Zhilevski. "Health Status Classification of Electric Motors Using CNN-Based Models
and SDP Images Under Varying Noise Conditions". *Proc. 11th World Congress on Electrical Engineering and
Computer Systems and Sciences (EECSS'25)*, Paris, ago. 2025, Paper EEE 107. DOI 10.11159/eee25.107.
PDF: https://avestia.com/EECSS2025_Proceedings/files/paper/EEE/EEE_107.pdf

## 2. Leitura

**Texto completo** (PDF de 8 páginas).

## 3. Pré-processamento, parâmetros e arquitetura

- Base: IDMT (Zenodo 10.5281/zenodo.7551261), **2 378 amostras** (todas as condições; Tabela 1 mostra a
  contagem por condição, inclusive `pure`) → são os recortes de 3 s de `train_cut/` + `test_cut/` misturados.
- Representação: Symmetrized Dot Pattern (SDP) do sinal acústico como imagem. Parâmetros do SDP (atraso,
  ganho angular, número de espelhos, tamanho da imagem): **não encontrados**.
- Modelos: MobileNetV2, InceptionV3 e uma CNN própria (Conv 32/64/128, 3×3, ReLU, MaxPool 2×2, Dense 128,
  Dropout 0,5, softmax). Otimizador, épocas e lote: não encontrados (gráfico mostra ~50 épocas).
- Tarefas: C1 = 3 classes de estado; C2 = 15 classes (estado × ruído); C3 = 12 classes (sem stresstest).

## 4. Protocolo de split e risco de vazamento

Split **80/10/10** (1 896 / 236 / 246) estratificado por estado e por tipo de ruído, sobre os recortes.
Não há agrupamento por gravação: recortes contíguos da mesma gravação ficam em treino e teste, e as condições
de teste oficiais entram no treino.

**Risco de vazamento: alto.**

## 5. Números reportados (IDMT, teste de 246 recortes)

- C1 (3 classes): **CNN própria 96,34 %**, MobileNetV2 95,93 %, InceptionV3 94,31 %. Broken 100 %;
  os erros são good ↔ heavy load, quase todos na condição stresstest.
- C2 (15 classes): MobileNet 76,02 %, CNN própria 73,58 %, InceptionV3 73,17 %.
- C3 (12 classes, sem stresstest): MobileNet 81,39 %, CNN própria 77,49 %, InceptionV3 73,59 %.

## 6. O que falta para reproduzir; veredito

Faltam os parâmetros do SDP e do treino. **Veredito: só inspiração.** Dois pontos úteis: mesmo com vazamento,
a SDP fica em ~96 % (é uma representação fraca para esta tarefa), e a confusão good ↔ heavy load sob
stresstest coincide com o ponto fraco já visto em `grollmisch2019`.
