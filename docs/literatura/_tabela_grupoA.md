# Grupo A — o que já foi feito no IDMT-ISA-ELECTRIC-ENGINE e no A2212

Levantamento de 14/10/2026. Detalhes e justificativas nas fichas individuais.

| id | método | base | split | número reportado | risco de vazamento | leitura |
|---|---|---|---|---|---|---|
| grollmisch2019 | \|STFT\| (janela 4096, 50 %) → DNN 256-64-16-3; voto por arquivo | IDMT | oficial: treino `pure`, teste nas condições com ruído; validação = 10 % aleatório de quadros (5×) | A_frame 82,3 % (stress) a 100 % (atmo_low); A_file 86,7–100 % | médio (sem vazamento de segmento; classe = unidade física) | completa |
| strantzalis2022 | log-mel 30×32 → CNN 2D (5,5 KB RAM, 7,7 KB flash) / MFCC → CNN 1D no Edge Impulse; STM32 Cortex-M4 | IDMT (todas as condições juntas) | 25 % teste aleatório de espectrogramas sobrepostos, depois 25 % validação | 99,87 % (A) / 97,8 % (B); tempo real: resposta às trocas de 4,55 s / 3,83 s | alto | completa (PMC) |
| etnext2026 | DWT sym4 7 níveis + histogramas signum/ternários (5 376) → NCA/Chi2 → Cubic SVM / Fine kNN | IDMT (repack Kaggle, 2 021 recortes) e A2212 (segmentos de 3 s) | 10-fold sobre segmentos; seleção de atributos e escolha do vetor dentro do mesmo CV | IDMT: SVM 100 %, kNN 99,80 %; A2212: SVM 99,95 %, kNN 99,16 % | alto | completa |
| melcnn2025 | mel 128 (FFT 2048, hop 512) → 32×32 → CNN 2,4 M parâmetros | IDMT (2 378 recortes, todas as condições) | 70/15/15 por arquivo (= por recorte) + 5-fold × 10 sementes | teste 99,7 %; 5-fold 99,5 ± 0,3 %; kNN+MFCC 91,8 % | alto | completa (PMC) |
| estacio2023 | artigo de dados (só correlação de Pearson de 20 atributos) | A2212 | — | sem classificação | n/a (estrutural: rolamento = M4; velocidade difere por classe) | completa |
| kaggle_afroz | não encontrado | provavelmente IDMT (repack do mesmo usuário) | não encontrado | não encontrado | não avaliado | **acesso bloqueado** (página só com JavaScript; exige navegador/CLI do Kaggle) |
| ertarin2025 | imagens SDP → CNN própria / MobileNetV2 / InceptionV3 | IDMT (2 378 recortes, todas as condições) | 80/10/10 aleatório estratificado por estado e ruído | 3 classes: 96,34 % (CNN própria); 15 classes: 76,02 %; 12 classes: 81,39 % | alto | completa |
| tekin2025 | DWT db7 nível 7 + 8–11 estatísticas → bagged trees | A2212 (4 partes por arquivo: 2 canais × 2 metades) | não descrito (avaliação cobre as 176 partes) | 89,205 %, kappa 0,821 | alto | completa |
| yordanov2024 | SDP + generalização de domínio + classificador de imagem | "base de sons de motores" (IDMT provável, não confirmado) | não encontrado | não encontrado | indeterminado | só resumo |

Candidato não fichado: Vazquez Matute, Pelaez, Davalos Carrera, Loayza Paredes, "Fault Diagnosing for Electric
Motors Based in Noise Data Classification Using Deep Learning" (conference object, REDI/CEDIA; ensemble de CNNs
sobre espectrogramas, > 97 % de acurácia, AUC > 99 %). O resumo não nomeia a base; não foi confirmado que use o
IDMT ou o A2212.

## Leitura transversal

Nenhum trabalho encontrado separa recortes por gravação fora do split oficial do IDMT, e nenhum separa classe de
identidade do motor (no IDMT isso é impossível por construção; no A2212 só daria com o par healthy/propeller dos
motores 1 e 2). O único número limpo de vazamento de segmento é o de `grollmisch2019`, e é justamente o
único que mostra queda forte por condição (stress test 82–87 %). Os trabalhos com 99–100 % misturam treino e
teste oficiais e embaralham recortes. Lacunas que ficam abertas para o projeto: resultados por condição no split
oficial com pipelines baratas; métricas de sequência (atraso e trocas espúrias), que só `strantzalis2022` mede;
custo medido em MCU, também só em `strantzalis2022`.
