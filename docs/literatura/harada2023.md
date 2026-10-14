# harada2023 — First-shot ASD: AE com distância de Mahalanobis seletiva (baseline DCASE 2023 Task 2)

## 1. Referência

Noboru Harada, Daisuke Niizumi, Yasunori Ohishi, Daiki Takeuchi, Masahiro Yasuda.
"First-shot anomaly sound detection for machine condition monitoring: A domain generalization
baseline". Preprint arXiv:2303.00455v1 [eess.AS], 01/03/2023. Versão em anais revisados: não
encontrada nesta busca. DOI: não encontrado.
Link: https://arxiv.org/abs/2303.00455

## 2. Leitura

**Texto completo** (PDF do arXiv, v1). A Tabela I saiu com rótulos de linha desalinhados na extração;
abaixo só os números cuja posição é inequívoca (coluna `bearing`, que é estável em todas as linhas de seed).

## 3. Pré-processamento, parâmetros e arquitetura

- Dados: DCASE2022 Task 2 (domain generalization): ToyCar, ToyTrain, fan, gearbox, **bearing**,
  slider, valve; 3 seções por tipo; por seção, **990** clipes normais de 10 s do domínio fonte e só
  **10** do domínio alvo; teste com 50 normais + 50 anômalos por domínio e seção.
- Features: STFT com quadros de **64 ms, hop 50 %** (n_fft 1024, hop 512 a 16 kHz), **128 log-mel**,
  **5 quadros** → 640 dim.
- AE "First-shot compliant" (FS-AE): 3 camadas de 128 (Linear + BatchNorm + ReLU) no encoder, gargalo
  8, decoder simétrico; MSE; **100 épocas, lote 256, Adam lr 0,001**; um modelo **por seção**.
  Escore = MSE de reconstrução.
- **Selective Mahalanobis AE**: mesmo AE; após a última época calculam-se as covariâncias Σ_s e Σ_t
  do resíduo x − x̂ nos normais do domínio fonte e alvo; escore =
  min{Mahalanobis(x, x̂; Σ_s⁻¹), Mahalanobis(x, x̂; Σ_t⁻¹)}.
- Requisito "First-shot": **proibido** ajustar hiperparâmetros por tipo de máquina com métricas
  calculadas usando as anomalias do teste.
- Métricas: AUC por domínio, pAUC (p = 0,1), média harmônica (hmean) sobre tipos.

## 4. Protocolo de split e risco de vazamento

- Treino e teste são arquivos distintos da DCASE2022; só normais no treino; hiperparâmetros fixos para
  todas as máquinas. Três seeds.
- **Risco de vazamento: baixo.** O próprio artigo é sobre isso: denuncia que os cinco primeiros da
  DCASE2022 usaram ajuste por máquina com AUC de teste ("information from the anomaly samples had been
  leaked into the system"). É a referência a citar para justificar o nosso `congelar.py`.

## 5. Números reportados (DCASE2022 T2, coluna `bearing`, faixa sobre as linhas de seed/média)

| Sistema | AUC fonte | AUC alvo | pAUC |
|---|---|---|---|
| FS-AE (MSE) | 0,709 – 0,714 | 0,597 – 0,612 | 0,513 – 0,520 |
| Selective Mahalanobis AE | 0,849 – 0,853 | 0,768 – 0,780 | 0,601 – 0,610 |

O hmean geral dos dois baselines fica em torno de 0,51–0,56 (AUC alvo), bem abaixo do 1º colocado
da DCASE2022 (Liu, CQUPT). O ganho do Mahalanobis é grande justamente em `bearing` e `fan`.

## 6. Custo embarcado

Não reportado. Estimativa nossa: AE ≈ 0,26 M parâmetros; o Mahalanobis em 640 dim exige guardar
Σ⁻¹ de 640 × 640 = 409 600 floats (**≈ 1,6 MB** em float32) por domínio — **não cabe** na SRAM do
ESP32 e é caro (≈ 0,8 MFLOP por quadro). Variante viável: Mahalanobis no espaço do gargalo (8 dim,
64 floats) ou em PCA de 16–32 componentes do resíduo.

## 7. Para reproduzir / veredito

Código prometido no GitHub (repositório oficial DCASE2023 T2), dados públicos. Suposições menores
(normalização, regularização da covariância). **Veredito: reproduzível** na DCASE2022; no MotorSound,
**inspirada em**.

## 8. Relevância para o MotorSound → `p30_anomalia` (variante `p30c_mahalanobis`)

Mostra que um **escore gaussiano simples** (Mahalanobis) sobre uma representação fixa ganha do erro
de reconstrução em rolamento, que é o caso de falha da A2212. Isso sugere a variante mais leve possível,
sem rede:

- **extrair**: log-mel 64 bandas, média e desvio por janela de 1 s (128 dim), ou as features de
  `p04_classicas` (≈ 32 dim), ou as amplitudes das harmônicas da rotação vindas do rastreador Decifrae.
- **treinar**: média μ e covariância Σ (com encolhimento Ledoit-Wolf) **só dos normais**, agrupado
  por gravação. Alternativas na mesma interface: GMM com 1–4 componentes; kNN (distância média aos
  k vizinhos normais).
- **prever**: escore d²_M(x) = (x − μ)ᵀ Σ⁻¹ (x − μ); sob normalidade gaussiana segue χ² com D graus
  de liberdade, o que dá um limiar teórico além do empírico.
- **limiar**: percentil 95 da validação normal; comparar com o quantil χ²(0,95; D).
- **métricas**: AUC, pAUC (p = 0,1), e por sequência atraso de detecção e alarmes falsos por minuto.
- **custo_mcu**: D = 32 → Σ⁻¹ com 1 024 floats (4 kB), 2 k FLOP por janela: trivial no ESP32.
