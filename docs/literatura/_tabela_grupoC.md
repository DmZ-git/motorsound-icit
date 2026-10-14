# Grupo C: features clássicas, embarcado e anomalia (tabela-resumo)

Os detalhes e as fontes de cada linha estão em `docs/literatura/<id>.md`. "Est." indica uma
estimativa nossa, que não foi reportada pelo artigo.

| id | método | base | split | número | custo MCU | risco de vazamento | leitura |
|---|---|---|---|---|---|---|---|
| vanaraj2026 | ZCR, RMS, centróide e MFCC com Voting Classifier | BLDC (normal/rolamento/hélice); base não declarada no resumo, provavelmente a A2212 | não encontrado | ~98 % de acurácia | não reportado | **alto** (presumido: segmentos de ~10 s fatiados; rolamento só no motor 4) | só resumo (paywall) |
| glowacz2018 | FFT de 1 s, 28 bins escolhidos (MSAF-15-MULTIEXPANDED-8-GROUPS), com NN, NM (Manhattan), SOM e MLP 28-100-5 | própria (motor de comutador, 5 estados, 44,1 kHz), não pública | 30 amostras de 1 s no treino e 500 no teste; separação por gravação não descrita | TERCM 94,6 % (NM), 94,2 % (NN), 89 % (BNN), 88,4 % (SOM) | não há MCU; est. FFT de 32 k pontos ≈ 128 kB, NM ≈ 0,5 kB | **médio-alto** (segmentos da mesma trilha; um estado por sessão) | texto completo |
| tinyml_acoustic2025 | 13 MFCC (40 ms, 50 %), MLP 128-64, int8 em TFLite | UrbanSound8K reagrupado em binário (não é som de máquina; supervisionado) | 80/20 aleatório estratificado, sem os folds oficiais | acc 0,95 / AUC 0,991 (float); acc 0,91 / AUC 0,970 (int8) | ~60 kB de modelo; **sem implantação em hardware** | **alto** (fatias da mesma gravação nos dois lados; suporte da Tab. III inconsistente) | texto completo |
| koizumi2020 | AE denso sobre log-mel (5 quadros), escore = MSE de reconstrução | DCASE2020 T2 (ToyADMOS + MIMII, 16 kHz) | só normais no treino; teste com arquivos distintos por Machine ID | AUC médio no dev: 0,65 (fan) a 0,85 (slider); pAUC de 0,51 a 0,67 | não reportado; est. ≈ 270 k parâmetros (≈ 270 kB em int8) | **baixo** | texto completo + código |
| purohit2019 | AE 320-64-64-8, log-mel 64 × 5 | MIMII (16 kHz, 10 s, ruído de fábrica a 6/0/−6 dB) | todos os anômalos e igual número de normais sorteados vão para o teste; o resto dos normais é treino | AUC médio a 6 dB: valve 0,67, pump 0,81, fan 0,94, slide 0,90 (cai até 0,53–0,70 a −6 dB) | não reportado; est. ≈ 51 k parâmetros (≈ 51 kB em int8) | **baixo-médio** (normal e anômalo em sessões diferentes) | texto completo |
| harada2023 | AE com Mahalanobis seletiva (fonte/alvo) sobre o resíduo; requisito "First-shot" | DCASE2022 T2 (inclui *bearing*) | só normais; hiperparâmetros fixos; 3 seeds | bearing, AUC fonte/alvo: AE 0,71/0,60; Mahalanobis 0,85/0,77 | não reportado; est. Σ⁻¹ 640 × 640 ≈ 1,6 MB (não cabe no ESP32); a variante em baixa dimensão é trivial | **baixo** (o artigo denuncia o vazamento por ajuste no teste) | texto completo |

Não foi feita ficha do IDNN (Suefusa et al., ICASSP 2020, "Anomalous Sound Detection Based on
Interpolation Deep Neural Network"). Ele aparece citado em koizumi2020 como a base do vencedor da
DCASE2020, mas o texto não foi lido.
