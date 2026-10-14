# Estado da arte nas bases do MotorSound

Consolidado em 14/10/2026 a partir das fichas desta pasta (tabelas parciais em
`_tabela_grupoA.md`, `_tabela_grupoB.md`, `_tabela_grupoC.md`). Os números são os
publicados; nenhum foi reproduzido ainda. "Leitura" diz se a ficha se baseou no texto
completo ou só no resumo.

## 1. Trabalhos que usam o IDMT-ISA-ELECTRIC-ENGINE ou o A2212

| id | método | base | split | número reportado | risco de vazamento | leitura |
|---|---|---|---|---|---|---|
| grollmisch2019 | \|STFT\| (4096, 50 %) → DNN 256-64-16-3, voto por arquivo | IDMT | **oficial**: treino `pure`, teste nas condições ruidosas; validação = 10 % de quadros aleatórios | quadro: 82,3 % (stress) a 100 % (atmo_low); arquivo: 86,7–100 % | médio: sem vazamento de segmento no teste, mas classe = unidade física; validação interna otimista | completa |
| strantzalis2022 | log-mel 30×32 → CNN 2 conv (STM32: 5,5 kB RAM, 7,7 kB flash, 40,8 ms); MFCC → Conv1D (Edge Impulse) | IDMT, todas as condições juntas | 25 % de espectrogramas sobrepostos sorteados para teste | 99,87 % / 97,8 %; atraso de resposta a trocas 4,55 s / 3,83 s | **alto** | completa |
| etnext2026 | DWT sym4 7 níveis + histogramas ternários → NCA/Chi2 → SVM/kNN | IDMT (repack Kaggle, 2 021 recortes) e A2212 | 10-fold sobre segmentos; seleção de atributos dentro do mesmo CV | IDMT 100 % (SVM), 99,80 % (kNN); A2212 99,95 % (SVM) | **alto** | completa |
| melcnn2025 | mel 128 → 32×32 → CNN de 2,4 M parâmetros | IDMT (2 378 recortes) | 70/15/15 "por arquivo" = por recorte de 3 s | 99,7 %; kNN+MFCC 91,8 % | **alto** | completa |
| ertarin2025 | imagens SDP → CNN | IDMT (2 378 recortes) | 80/10/10 aleatório | 96,34 % (3 classes) | **alto** | completa |
| tekin2025 | DWT db7 + estatísticas → bagged trees | A2212 (4 pedaços por arquivo) | não descrito | 89,2 %, κ 0,821 | **alto** | completa |
| vanaraj2026 | ZCR, RMS, centróide, MFCC → Voting | BLDC 3 classes, provavelmente A2212 (não declarado) | não encontrado | ~98 % | alto (presumido) | só resumo |
| estacio2023 | artigo de dados, sem classificação | A2212 | — | — | estrutural: rolamento = M4; faixas de velocidade diferem por classe | completa |
| yordanov2024 | SDP + generalização de domínio | provavelmente IDMT | não encontrado | não encontrado | indeterminado | só resumo |
| kaggle_afroz | não encontrado | IDMT/A2212 | não encontrado | não encontrado | não avaliado | **acesso bloqueado** (exige navegador logado) |

**Leitura transversal.** Todo resultado de 96–100 % nestas bases embaralha recortes da mesma
gravação entre treino e teste e mistura o treino e o teste oficiais do IDMT. O único número sem
vazamento de segmento é o de grollmisch2019, que é também o único a mostrar queda forte por
condição (stress test 82 %). Nenhum trabalho separa classe de identidade do motor; no IDMT isso é
impossível por construção (docs/DATASETS.md §1.5) e no A2212 só o par Healthy × Propeller dos
motores 1 e 2 permite. Só strantzalis2022 mede custo em MCU e resposta a trocas de estado.

## 2. Rastreamento de rotação sem tacômetro

| id | método | dados | número | causal | leitura |
|---|---|---|---|---|---|
| peeters2019 | revisão: demodulação de fase, TKEO, MOPA, ViBES, cepstro, máximo + Vold-Kalman | vibração (turbina, motor aeronáutico) com encoder/tacômetro | só em figuras (não encontrado no texto) | ViBES e máximo sim | completa |
| fsst_viterbi2015 | synchrosqueezing com deslocamento + Viterbi sobre o TFR | simulado + experimento | não informado no resumo | provavelmente não | só resumo |
| yoo2023 | MOPA com peso por harmônico (energia / nº de componentes por entropia de Rényi) | engrenagem simulada; Surveillance 8 | MAE 0,0391 vs 0,0792 (MOPA), unidade não informada | não | completa |
| wheat2025 | Harmonic Feature Space com f_o **conhecido** + regressão em (velocidade, carga) treinada em sãos | vibração CMHT, 6 rolamentos | 94 % (FFT 58 %) | sim, dado f_o | completa (preprint) |

Viterbi sobre representação tempo-frequência para seguir a rotação **não é novo** (Xi 2015 e a
literatura de HMM citada em peeters2019). O que o MotorSound pode reivindicar é o conjunto vindo da
transcrição: estado de silêncio, custo de oitava, emissão mista, em áudio de microfone e com
orçamento de MCU, e o uso da rotação rastreada como feature de estado de operação.

## 3. Features clássicas, embarcado e anomalia

| id | método | base | split | número | custo MCU | risco | leitura |
|---|---|---|---|---|---|---|---|
| glowacz2018 | FFT 1 s, 28 bins selecionados → média mais próxima | própria, não pública | 30 s treino / 500 s teste, sem separação por gravação descrita | 94,6 % | não medido | médio-alto | completa |
| tinyml_acoustic2025 | 13 MFCC → MLP int8 | UrbanSound8K binário (não é máquina) | 80/20 aleatório | AUC 0,970 (int8) | ~60 kB, **sem hardware** | alto | completa |
| koizumi2020 | AE denso em log-mel, escore = MSE (DCASE2020 T2) | ToyADMOS + MIMII | só normais no treino | AUC 0,65–0,85 | est. ~270 kB int8 | baixo | completa + código |
| purohit2019 | AE 320-64-64-8 (MIMII) | MIMII 6/0/−6 dB | só normais no treino | AUC 0,67–0,94 a 6 dB | est. ~51 kB int8 | baixo-médio | completa |
| harada2023 | AE + Mahalanobis seletiva | DCASE2022 T2 | só normais, hiperparâmetros fixos | bearing AUC 0,85 / 0,77 | Σ⁻¹ 640² não cabe | baixo | completa |

## 4. Lacunas que os dados do projeto podem ocupar

Resultados por condição de ruído com split por gravação e pipelines baratas; auditoria explícita
de quanto da acurácia publicada vem do embaralhamento de recortes; métricas de sequência (atraso,
trocas espúrias) com HMM online; custo estimado/medido em MCU ao lado da acurácia; e a confusão
classe × unidade física declarada em vez de escondida.
