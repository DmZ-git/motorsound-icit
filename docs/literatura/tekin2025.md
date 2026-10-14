# tekin2025 — DWT + estatísticas + bagged trees no A2212

## 1. Referência

B. Tekin, T. Kaya. "Analysis of Brushless DC Motor Sounds with Machine Learning Methods Using Wavelet Transform
Based Features". *Firat University Journal of Experimental and Computational Engineering* 4(2):363–374, jun. 2025.
DOI 10.62520/fujece.1632384. https://dergipark.org.tr/en/pub/fujece/article/1632384

## 2. Leitura

**Texto completo** (PDF de 12 páginas, bilíngue turco/inglês).

## 3. Pré-processamento, parâmetros e arquitetura

- Base: A2212 (Estacio et al.), 43 wav de ~10 s a 16 kHz. A Tabela 1 lista healthy = 20 e bearing = 7
  (a base tem 22 healthy).
- Cada arquivo tem 2 colunas (canais estéreo); cada coluna é dividida em 2 metades → **4 partes por arquivo**.
- DWT **db7, nível 7**; limiarização dos coeficientes de detalhe; estatísticas: média, mediana, máx, mín,
  amplitude, desvio-padrão, MAD, norma (8 listadas, mas a Tabela 2 diz 11 atributos por parte).
  Normalização z-score.
- Amostras: bearing 28, healthy 88, propeller 60 → 176 (60 não bate com 14 arquivos × 4 = 56).
- Classificador: bagged trees (o texto sugere MATLAB; número de árvores não encontrado).

## 4. Protocolo de split e risco de vazamento

**Não descrito.** A matriz de confusão soma as 176 amostras, então a avaliação cobre todas — provavelmente
validação cruzada sobre as partes (não encontrado). As 4 partes de um mesmo arquivo (inclusive os dois canais
do **mesmo instante**) quase certamente caem em treino e teste. Rolamento = motor 4.

**Risco de vazamento: alto.**

## 5. Números reportados (A2212)

Acurácia **89,205 %**, kappa 0,821. Recall por classe: 80,8 / 92,3 / 88,1 %; precisão 75 / 95,5 / 86,7 %
(classes 1–3; o mapeamento para bearing/healthy/propeller segue a Tabela 2, rótulos 0/1/2 — ambíguo).

## 6. O que falta para reproduzir; veredito

Falta o protocolo de validação, o número de árvores e a lista exata dos 11 atributos. **Veredito: só inspiração.**
Interessa como referência de que, mesmo com vazamento provável, um extrator estatístico simples sobre DWT fica
abaixo de 90 % no A2212.
