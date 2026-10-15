# Tabelas de resultados (geradas)
Gerado por `experimentos/tabelas_resultados.py`; não editar à mão. Todas as acurácias são por janela, na validação (o teste do IDMT não foi lido).

## T1. Validação limpa e auditorias de vazamento e de nível
Origem: `out/experiments/20261014_185859_validacao.csv`, `out/experiments/20261014_203110_validacao_cnn.csv`

IC95 por bootstrap de gravações (A2212). 'motor fora' = deixa-um-motor-de-fora.

| pipeline                  |   IDMT blocos |   IDMT aleatório |   IDMT sem norm. |   A2212 arquivo |   A2212 aleatório |   A2212 motor fora |   Δ vazamento A2212 | IC95 A2212 arquivo   |
|:--------------------------|--------------:|-----------------:|-----------------:|----------------:|------------------:|-------------------:|--------------------:|:---------------------|
| b00_nivel                 |         0.991 |            0.992 |            0.994 |           0.728 |             0.73  |              0.693 |               0.002 | [0.61, 0.83]         |
| b01_forma_espectral       |         1     |            1     |            1     |           0.935 |             0.998 |              0.723 |               0.063 | [0.86, 1.00]         |
| p00_logmel_cnn            |         1     |            1     |            1     |           0.965 |             0.998 |              0.507 |               0.033 | [0.91, 1.00]         |
| p01_mfcc_cnn1d            |         1     |            1     |            1     |           0.874 |             0.891 |              0.542 |               0.016 | [0.81, 0.93]         |
| p02_grollmisch            |         1     |            1     |            1     |           0.947 |             1     |              0.73  |               0.053 | [0.88, 1.00]         |
| p03_etnext                |         1     |            1     |            1     |           0.905 |             0.981 |              0.381 |               0.077 | [0.83, 0.96]         |
| p03b_etnext_knn           |         1     |            1     |            1     |           0.77  |             0.953 |              0.379 |               0.184 | [0.68, 0.86]         |
| p04_classicas             |         1     |            1     |            1     |           0.979 |             0.998 |              0.777 |               0.019 | [0.95, 1.00]         |
| p04b_msaf                 |         0.875 |            0.894 |            0.875 |           0.344 |             0.374 |              0.326 |               0.03  | [0.21, 0.49]         |
| p04c_classicas_lr         |         0.997 |            1     |            1     |           0.974 |             0.995 |              0.749 |               0.021 | [0.94, 1.00]         |
| p05_melcnn                |         1     |            1     |            1     |           0.977 |             1     |              0.495 |               0.023 | [0.93, 1.00]         |
| p10_decifrae_pyin_viterbi |         1     |            1     |            1     |           0.744 |             0.793 |              0.528 |               0.049 | [0.64, 0.84]         |
| p11_decifrae_harmsum      |         1     |            1     |            1     |           0.751 |             0.788 |              0.528 |               0.037 | [0.64, 0.85]         |
| p11a_grade10              |         0.997 |            1     |            0.997 |           0.77  |             0.798 |              0.56  |               0.028 | [0.67, 0.86]         |
| p12_decifrae_sem_viterbi  |         0.997 |            0.997 |            0.997 |           0.728 |             0.793 |              0.533 |               0.065 | [0.62, 0.82]         |
| p20_hfs                   |         1     |            1     |            1     |           0.549 |             0.691 |              0.386 |               0.142 | [0.45, 0.64]         |
| p20b_so_frot              |         0.988 |            0.989 |            0.988 |           0.5   |             0.612 |              0.379 |               0.112 | [0.39, 0.60]         |
| p21_envelope              |         1     |            1     |            1     |           0.94  |             0.993 |              0.784 |               0.053 | [0.87, 0.99]         |
| p30_anomalia              |         0.632 |            0.633 |            0.632 |           0.702 |             0.716 |              0.628 |               0.014 | [0.58, 0.82]         |
| p30b_ae                   |         0.603 |            0.633 |            0.6   |           0.66  |             0.763 |              0.428 |               0.102 | [0.53, 0.79]         |

## T2. Robustez no IDMT (treino limpo, validação perturbada)
Origem: `out/experiments/20261014_192347_robustez_idmt.csv`, `out/experiments/20261014_205140_robustez_idmt.csv`

Um modelo por fold, treinado no limpo e aplicado a cada condição. Acaso = 0,33.

| pipeline                  |   limpo |   branco 20 |   branco 10 |   branco 5 |   branco 0 |   branco -5 |   rosa 20 |   rosa 10 |   rosa 5 |   rosa 0 |   marrom 0 |   percussivo 8 1.5 |   reverb 0.35 |   ganho 6 |
|:--------------------------|--------:|------------:|------------:|-----------:|-----------:|------------:|----------:|----------:|---------:|---------:|-----------:|-------------------:|--------------:|----------:|
| b00_nivel                 |    0.99 |        0.99 |        0.63 |       0.52 |       0.35 |        0.35 |      0.99 |      0.62 |     0.55 |     0.37 |       0.59 |               0.35 |          0.98 |      0.66 |
| b01_forma_espectral       |    1    |        1    |        0.64 |       0.35 |       0.35 |        0.35 |      1    |      0.64 |     0.36 |     0.35 |       1    |               1    |          1    |      1    |
| p00_logmel_cnn            |    1    |        1    |        0.64 |       0.64 |       0.45 |        0.35 |      1    |      0.72 |     0.64 |     0.64 |       1    |               1    |          1    |      1    |
| p01_mfcc_cnn1d            |    1    |        0.96 |        0.64 |       0.64 |       0.64 |        0.43 |      0.98 |      0.67 |     0.64 |     0.64 |       0.99 |               1    |          1    |      1    |
| p02_grollmisch            |    1    |        1    |        0.92 |       0.64 |       0.64 |        0.58 |      1    |      1    |     0.92 |     0.7  |       1    |               1    |          1    |      1    |
| p03_etnext                |    1    |        1    |        1    |       1    |       0.72 |        0.58 |      1    |      0.89 |     0.56 |     0.27 |       0.66 |               1    |          1    |      1    |
| p03b_etnext_knn           |    1    |        1    |        0.93 |       0.88 |       0.87 |        0.8  |      1    |      0.83 |     0.49 |     0.41 |       0.66 |               0.98 |          1    |      1    |
| p04_classicas             |    1    |        0.64 |        0.64 |       0.64 |       0.64 |        0.58 |      0.99 |      0.64 |     0.64 |     0.69 |       0.94 |               0.45 |          1    |      0.97 |
| p04b_msaf                 |    0.88 |        0.5  |        0.38 |       0.35 |       0.35 |        0.35 |      0.51 |      0.48 |     0.38 |     0.35 |       0.51 |               0.46 |          0.87 |      0.87 |
| p04c_classicas_lr         |    1    |        1    |        0.64 |       0.64 |       0.64 |        0.63 |      1    |      0.67 |     0.71 |     0.67 |       0.94 |               0.35 |          0.99 |      0.94 |
| p05_melcnn                |    1    |        1    |        0.64 |       0.47 |       0.41 |        0.41 |      1    |      0.69 |     0.64 |     0.54 |       1    |               1    |          1    |      1    |
| p10_decifrae_pyin_viterbi |    1    |        0.97 |        0.91 |       0.44 |       0.35 |        0.35 |      1    |      1    |     0.83 |     0.52 |       0.98 |               0.97 |          0.99 |      0.99 |
| p11_decifrae_harmsum      |    1    |        0.97 |        0.9  |       0.4  |       0.35 |        0.35 |      1    |      0.99 |     0.72 |     0.54 |       0.92 |               0.95 |          0.96 |      0.91 |
| p11a_grade10              |    1    |        0.97 |        0.89 |       0.45 |       0.35 |        0.35 |      1    |      0.98 |     0.79 |     0.35 |       0.91 |               0.98 |          0.94 |      0.9  |
| p12_decifrae_sem_viterbi  |    1    |        0.97 |        0.91 |       0.38 |       0.35 |        0.35 |      1    |      1    |     0.83 |     0.35 |       0.94 |               0.98 |          0.98 |      0.78 |
| p20_hfs                   |    1    |        1    |        0.64 |       0.64 |       0.64 |        0.64 |      0.77 |      0.64 |     0.64 |     0.64 |       0.99 |               0.68 |          1    |      1    |
| p20b_so_frot              |    0.99 |        0.99 |        0.99 |       0.99 |       0.97 |        0.92 |      0.99 |      0.99 |     0.98 |     0.9  |       0.99 |               0.99 |          0.99 |      0.99 |
| p21_envelope              |    1    |        1    |        0.64 |       0.64 |       0.35 |        0.35 |      1    |      0.88 |     0.64 |     0.35 |       1    |               0.48 |          1    |      0.94 |
| p30_anomalia              |    0.63 |        0.36 |        0.35 |       0.35 |       0.35 |        0.35 |      0.59 |      0.35 |     0.35 |     0.35 |       0.35 |               0.35 |          0.35 |      0.37 |
| p30b_ae                   |    0.6  |        0.35 |        0.35 |       0.35 |       0.35 |        0.35 |      0.35 |      0.35 |     0.35 |     0.35 |       0.35 |               0.35 |          0.35 |      0.43 |

## T3. Robustez no A2212 (por arquivo)
Origem: `out/experiments/20261014_193054_robustez_a2212.csv`, `out/experiments/20261014_210231_robustez_a2212.csv`

| pipeline                  |   limpo |   branco 10 |   rosa 10 |   rosa 5 |   marrom 0 |   percussivo 8 1.5 |
|:--------------------------|--------:|------------:|----------:|---------:|-----------:|-------------------:|
| b00_nivel                 |    0.73 |        0.73 |      0.73 |     0.72 |       0.68 |               0.33 |
| b01_forma_espectral       |    0.93 |        0.83 |      0.72 |     0.58 |       0.87 |               0.92 |
| p00_logmel_cnn            |    0.96 |        0.89 |      0.92 |     0.75 |       0.94 |               0.92 |
| p01_mfcc_cnn1d            |    0.87 |        0.76 |      0.79 |     0.74 |       0.84 |               0.83 |
| p02_grollmisch            |    0.95 |        0.58 |      0.69 |     0.53 |       0.66 |               0.95 |
| p03_etnext                |    0.9  |        0.77 |      0.57 |     0.47 |       0.58 |               0.89 |
| p03b_etnext_knn           |    0.77 |        0.67 |      0.54 |     0.47 |       0.48 |               0.73 |
| p04_classicas             |    0.98 |        0.9  |      0.91 |     0.8  |       0.88 |               0.95 |
| p04b_msaf                 |    0.34 |        0.33 |      0.34 |     0.33 |       0.34 |               0.33 |
| p04c_classicas_lr         |    0.97 |        0.89 |      0.85 |     0.69 |       0.91 |               0.97 |
| p05_melcnn                |    0.98 |        0.96 |      0.92 |     0.79 |       0.94 |               0.97 |
| p10_decifrae_pyin_viterbi |    0.74 |        0.76 |      0.72 |     0.64 |       0.57 |               0.75 |
| p11_decifrae_harmsum      |    0.75 |        0.76 |      0.72 |     0.64 |       0.57 |               0.75 |
| p11a_grade10              |    0.77 |        0.75 |      0.73 |     0.55 |       0.64 |               0.75 |
| p12_decifrae_sem_viterbi  |    0.73 |        0.74 |      0.73 |     0.61 |       0.56 |               0.74 |
| p20_hfs                   |    0.55 |        0.51 |      0.53 |     0.46 |       0.43 |               0.56 |
| p20b_so_frot              |    0.5  |        0.5  |      0.5  |     0.49 |       0.5  |               0.5  |
| p21_envelope              |    0.94 |        0.89 |      0.87 |     0.77 |       0.93 |               0.8  |
| p30_anomalia              |    0.7  |        0.67 |      0.69 |     0.66 |       0.68 |               0.7  |
| p30b_ae                   |    0.66 |        0.63 |      0.62 |     0.57 |       0.49 |               0.62 |

## T4. Comparações pareadas (diferença = b − a)
Origem: `out/experiments/20261014_213212_estatistica.csv`

Bootstrap pareado de grupos (15 no IDMT, 43 no A2212), 10 000 reamostragens, Holm sobre as 28 comparações.

| base   | cenario   | a                        | b                         | isola                                                 |   media_a |   media_b |   diferenca |   ic95_min |   ic95_max |   p_bootstrap_holm | classificacao   |
|:-------|:----------|:-------------------------|:--------------------------|:------------------------------------------------------|----------:|----------:|------------:|-----------:|-----------:|-------------------:|:----------------|
| idmt   | rosa 10   | p12_decifrae_sem_viterbi | p11_decifrae_harmsum      | Viterbi de rotação (baixo V3)                         |     0.997 |     0.994 |      -0.003 |     -0.009 |      0     |              1     | empate          |
| idmt   | rosa 10   | p11a_grade10             | p11_decifrae_harmsum      | grade de 2 cents no rastreador                        |     0.98  |     0.994 |       0.014 |     -0.003 |      0.034 |              1     | empate          |
| idmt   | rosa 10   | p11_decifrae_harmsum     | p10_decifrae_pyin_viterbi | termo pYIN na emissão                                 |     0.994 |     1     |       0.006 |      0     |      0.015 |              1     | empate          |
| idmt   | rosa 10   | p00_logmel_cnn           | p11_decifrae_harmsum      | Decifrae embarcável × CNN de referência (Strantzalis) |     0.716 |     0.994 |       0.278 |      0.067 |      0.533 |              0.34  | fragil          |
| idmt   | rosa 10   | p02_grollmisch           | p11_decifrae_harmsum      | Decifrae embarcável × baseline dos autores do IDMT    |     1     |     0.994 |      -0.006 |     -0.015 |      0     |              1     | empate          |
| idmt   | rosa 10   | p11_decifrae_harmsum     | p20b_so_frot              | só a rotação × p11 completa                           |     0.994 |     0.986 |      -0.009 |     -0.026 |      0.006 |              1     | empate          |
| idmt   | rosa 10   | b01_forma_espectral      | p21_envelope              | modulação do envelope × forma espectral               |     0.641 |     0.881 |       0.241 |      0.077 |      0.419 |              0.097 | fragil          |
| idmt   | branco 10 | p12_decifrae_sem_viterbi | p11_decifrae_harmsum      | Viterbi de rotação (baixo V3)                         |     0.907 |     0.899 |      -0.009 |     -0.044 |      0.027 |              1     | empate          |
| idmt   | branco 10 | p11a_grade10             | p11_decifrae_harmsum      | grade de 2 cents no rastreador                        |     0.89  |     0.899 |       0.009 |     -0.037 |      0.061 |              1     | empate          |
| idmt   | branco 10 | p11_decifrae_harmsum     | p10_decifrae_pyin_viterbi | termo pYIN na emissão                                 |     0.899 |     0.913 |       0.014 |      0     |      0.03  |              0.53  | fragil          |
| idmt   | branco 10 | p00_logmel_cnn           | p11_decifrae_harmsum      | Decifrae embarcável × CNN de referência (Strantzalis) |     0.641 |     0.899 |       0.258 |      0.075 |      0.45  |              0.097 | fragil          |
| idmt   | branco 10 | p02_grollmisch           | p11_decifrae_harmsum      | Decifrae embarcável × baseline dos autores do IDMT    |     0.925 |     0.899 |      -0.026 |     -0.154 |      0.11  |              1     | empate          |
| idmt   | branco 10 | p11_decifrae_harmsum     | p20b_so_frot              | só a rotação × p11 completa                           |     0.899 |     0.988 |       0.09  |      0.006 |      0.193 |              0.511 | fragil          |
| idmt   | branco 10 | b01_forma_espectral      | p21_envelope              | modulação do envelope × forma espectral               |     0.641 |     0.641 |       0     |      0     |      0     |              1     | empate          |
| idmt   | rosa 5    | p12_decifrae_sem_viterbi | p11_decifrae_harmsum      | Viterbi de rotação (baixo V3)                         |     0.835 |     0.725 |      -0.11  |     -0.232 |     -0.003 |              0.705 | fragil          |
| idmt   | rosa 5    | p11a_grade10             | p11_decifrae_harmsum      | grade de 2 cents no rastreador                        |     0.791 |     0.725 |      -0.067 |     -0.212 |      0.066 |              1     | empate          |
| idmt   | rosa 5    | p11_decifrae_harmsum     | p10_decifrae_pyin_viterbi | termo pYIN na emissão                                 |     0.725 |     0.829 |       0.104 |      0.054 |      0.161 |              0.003 | confirmado      |
| idmt   | rosa 5    | p00_logmel_cnn           | p11_decifrae_harmsum      | Decifrae embarcável × CNN de referência (Strantzalis) |     0.641 |     0.725 |       0.084 |     -0.164 |      0.319 |              1     | empate          |
| idmt   | rosa 5    | p02_grollmisch           | p11_decifrae_harmsum      | Decifrae embarcável × baseline dos autores do IDMT    |     0.925 |     0.725 |      -0.2   |     -0.382 |     -0.036 |              0.34  | fragil          |
| idmt   | rosa 5    | p11_decifrae_harmsum     | p20b_so_frot              | só a rotação × p11 completa                           |     0.725 |     0.977 |       0.252 |      0.11  |      0.409 |              0.003 | confirmado      |
| idmt   | rosa 5    | b01_forma_espectral      | p21_envelope              | modulação do envelope × forma espectral               |     0.357 |     0.641 |       0.284 |      0.103 |      0.534 |              0.097 | fragil          |
| a2212  | limpo     | p12_decifrae_sem_viterbi | p11_decifrae_harmsum      | Viterbi de rotação (baixo V3)                         |     0.728 |     0.751 |       0.023 |      0     |      0.051 |              0.63  | fragil          |
| a2212  | limpo     | p11a_grade10             | p11_decifrae_harmsum      | grade de 2 cents no rastreador                        |     0.77  |     0.751 |      -0.019 |     -0.053 |      0.014 |              1     | empate          |
| a2212  | limpo     | p11_decifrae_harmsum     | p10_decifrae_pyin_viterbi | termo pYIN na emissão                                 |     0.751 |     0.744 |      -0.007 |     -0.019 |      0     |              1     | empate          |
| a2212  | limpo     | p00_logmel_cnn           | p11_decifrae_harmsum      | Decifrae embarcável × CNN de referência (Strantzalis) |     0.963 |     0.751 |      -0.212 |     -0.328 |     -0.093 |              0.01  | confirmado      |
| a2212  | limpo     | p02_grollmisch           | p11_decifrae_harmsum      | Decifrae embarcável × baseline dos autores do IDMT    |     0.947 |     0.751 |      -0.195 |     -0.309 |     -0.086 |              0.003 | confirmado      |
| a2212  | limpo     | p11_decifrae_harmsum     | p20b_so_frot              | só a rotação × p11 completa                           |     0.751 |     0.5   |      -0.251 |     -0.395 |     -0.107 |              0.029 | confirmado      |
| a2212  | limpo     | b01_forma_espectral      | p21_envelope              | modulação do envelope × forma espectral               |     0.935 |     0.94  |       0.005 |     -0.067 |      0.081 |              1     | empate          |

## T5. Sequências: atraso, trocas espúrias e trocas perdidas
Origem: `out/experiments/20261014_212255_sequencias_idmt.csv`, `out/experiments/20261014_212304_sequencias_idmt.csv`, `out/experiments/20261014_212313_sequencias_idmt.csv`, `out/experiments/20261014_212322_sequencias_idmt.csv`

30 sequências por condição, permanências de 5–20 janelas de 3 s. argmax = sem HMM (a 'p13'); viterbi = HMM offline; online = filtro causal. 'perdidas' = trocas verdadeiras nunca detectadas, por sequência.

| condicao_sintetica   | pipeline                  | decodificacao   |   change_penalty |   acuracia |   atraso_mediano_s |   espurias_por_min |   perdidas |
|:---------------------|:--------------------------|:----------------|-----------------:|-----------:|-------------------:|-------------------:|-----------:|
| branco 10            | p00_logmel_cnn            | argmax          |              nan |       0.62 |               0    |               0.29 |       2.13 |
| branco 10            | p00_logmel_cnn            | online          |                2 |       0.62 |               0    |               0.29 |       2.13 |
| branco 10            | p00_logmel_cnn            | online          |                8 |       0.61 |               0.15 |               0.29 |       2.13 |
| branco 10            | p00_logmel_cnn            | viterbi         |                2 |       0.62 |               0    |               0.29 |       2.13 |
| branco 10            | p00_logmel_cnn            | viterbi         |                8 |       0.62 |               0    |               0.29 |       2.13 |
| branco 10            | p02_grollmisch            | argmax          |              nan |       0.94 |               0    |               0.04 |       0.3  |
| branco 10            | p02_grollmisch            | online          |                2 |       0.94 |               0    |               0.04 |       0.3  |
| branco 10            | p02_grollmisch            | online          |                8 |       0.94 |               0    |               0.04 |       0.3  |
| branco 10            | p02_grollmisch            | viterbi         |                2 |       0.94 |               0    |               0.04 |       0.3  |
| branco 10            | p02_grollmisch            | viterbi         |                8 |       0.94 |               0    |               0.04 |       0.3  |
| branco 10            | p10_decifrae_pyin_viterbi | argmax          |              nan |       0.93 |               0    |               1.18 |       0    |
| branco 10            | p10_decifrae_pyin_viterbi | online          |                2 |       0.93 |               0.1  |               0.41 |       0.1  |
| branco 10            | p10_decifrae_pyin_viterbi | online          |                8 |       0.86 |               2.05 |               0.14 |       0.27 |
| branco 10            | p10_decifrae_pyin_viterbi | viterbi         |                2 |       0.92 |               0    |               0.15 |       0.37 |
| branco 10            | p10_decifrae_pyin_viterbi | viterbi         |                8 |       0.89 |               0    |               0.11 |       0.63 |
| branco 10            | p11_decifrae_harmsum      | argmax          |              nan |       0.91 |               0    |               1.69 |       0    |
| branco 10            | p11_decifrae_harmsum      | online          |                2 |       0.91 |               0.15 |               0.31 |       0.2  |
| branco 10            | p11_decifrae_harmsum      | online          |                8 |       0.82 |               2.25 |               0.17 |       0.5  |
| branco 10            | p11_decifrae_harmsum      | viterbi         |                2 |       0.91 |               0    |               0.2  |       0.4  |
| branco 10            | p11_decifrae_harmsum      | viterbi         |                8 |       0.86 |               0    |               0.14 |       0.77 |
| branco 10            | p20b_so_frot              | argmax          |              nan |       0.99 |               0    |               0.35 |       0    |
| branco 10            | p20b_so_frot              | online          |                2 |       0.99 |               0    |               0.35 |       0    |
| branco 10            | p20b_so_frot              | online          |                8 |       0.99 |               0    |               0.35 |       0    |
| branco 10            | p20b_so_frot              | viterbi         |                2 |       0.99 |               0    |               0.35 |       0    |
| branco 10            | p20b_so_frot              | viterbi         |                8 |       0.99 |               0    |               0.35 |       0    |
| branco 10            | p21_envelope              | argmax          |              nan |       0.62 |               0    |               0.9  |       2.13 |
| branco 10            | p21_envelope              | online          |                2 |       0.62 |               0    |               0.29 |       2.13 |
| branco 10            | p21_envelope              | online          |                8 |       0.59 |               1.4  |               0.19 |       2.13 |
| branco 10            | p21_envelope              | viterbi         |                2 |       0.62 |               0    |               0.27 |       2.13 |
| branco 10            | p21_envelope              | viterbi         |                8 |       0.62 |               0    |               0.21 |       2.13 |
| limpo                | p00_logmel_cnn            | argmax          |              nan |       1    |               0    |               0    |       0    |
| limpo                | p00_logmel_cnn            | online          |                2 |       1    |               0    |               0    |       0    |
| limpo                | p00_logmel_cnn            | online          |                8 |       1    |               0    |               0    |       0    |
| limpo                | p00_logmel_cnn            | viterbi         |                2 |       1    |               0    |               0    |       0    |
| limpo                | p00_logmel_cnn            | viterbi         |                8 |       1    |               0    |               0    |       0    |
| limpo                | p02_grollmisch            | argmax          |              nan |       1    |               0    |               0    |       0    |
| limpo                | p02_grollmisch            | online          |                2 |       1    |               0    |               0    |       0    |
| limpo                | p02_grollmisch            | online          |                8 |       1    |               0    |               0    |       0    |
| limpo                | p02_grollmisch            | viterbi         |                2 |       1    |               0    |               0    |       0    |
| limpo                | p02_grollmisch            | viterbi         |                8 |       1    |               0    |               0    |       0    |
| limpo                | p10_decifrae_pyin_viterbi | argmax          |              nan |       1    |               0    |               0    |       0    |
| limpo                | p10_decifrae_pyin_viterbi | online          |                2 |       1    |               0    |               0    |       0    |
| limpo                | p10_decifrae_pyin_viterbi | online          |                8 |       0.98 |               0.15 |               0    |       0    |
| limpo                | p10_decifrae_pyin_viterbi | viterbi         |                2 |       1    |               0    |               0    |       0    |
| limpo                | p10_decifrae_pyin_viterbi | viterbi         |                8 |       1    |               0    |               0    |       0    |
| limpo                | p11_decifrae_harmsum      | argmax          |              nan |       1    |               0    |               0    |       0    |
| limpo                | p11_decifrae_harmsum      | online          |                2 |       1    |               0    |               0    |       0    |
| limpo                | p11_decifrae_harmsum      | online          |                8 |       0.98 |               0    |               0    |       0    |
| limpo                | p11_decifrae_harmsum      | viterbi         |                2 |       1    |               0    |               0    |       0    |
| limpo                | p11_decifrae_harmsum      | viterbi         |                8 |       1    |               0    |               0    |       0    |
| limpo                | p20b_so_frot              | argmax          |              nan |       0.99 |               0    |               0.35 |       0    |
| limpo                | p20b_so_frot              | online          |                2 |       0.99 |               0    |               0.35 |       0    |
| limpo                | p20b_so_frot              | online          |                8 |       0.99 |               0    |               0.35 |       0    |
| limpo                | p20b_so_frot              | viterbi         |                2 |       0.99 |               0    |               0.35 |       0    |
| limpo                | p20b_so_frot              | viterbi         |                8 |       0.99 |               0    |               0.35 |       0    |
| limpo                | p21_envelope              | argmax          |              nan |       1    |               0    |               0    |       0    |
| limpo                | p21_envelope              | online          |                2 |       1    |               0    |               0    |       0    |
| limpo                | p21_envelope              | online          |                8 |       0.96 |               1.75 |               0    |       0    |
| limpo                | p21_envelope              | viterbi         |                2 |       1    |               0    |               0    |       0    |
| limpo                | p21_envelope              | viterbi         |                8 |       1    |               0    |               0    |       0    |
| rosa 10              | p00_logmel_cnn            | argmax          |              nan |       0.68 |               0    |               0.23 |       1.83 |
| rosa 10              | p00_logmel_cnn            | online          |                2 |       0.68 |               0    |               0.23 |       1.83 |
| rosa 10              | p00_logmel_cnn            | online          |                8 |       0.67 |               0    |               0.23 |       1.83 |
| rosa 10              | p00_logmel_cnn            | viterbi         |                2 |       0.68 |               0    |               0.23 |       1.83 |
| rosa 10              | p00_logmel_cnn            | viterbi         |                8 |       0.68 |               0    |               0.23 |       1.87 |
| rosa 10              | p02_grollmisch            | argmax          |              nan |       1    |               0    |               0    |       0    |
| rosa 10              | p02_grollmisch            | online          |                2 |       1    |               0    |               0    |       0    |
| rosa 10              | p02_grollmisch            | online          |                8 |       1    |               0    |               0    |       0    |
| rosa 10              | p02_grollmisch            | viterbi         |                2 |       1    |               0    |               0    |       0    |
| rosa 10              | p02_grollmisch            | viterbi         |                8 |       1    |               0    |               0    |       0    |
| rosa 10              | p10_decifrae_pyin_viterbi | argmax          |              nan |       1    |               0    |               0    |       0    |
| rosa 10              | p10_decifrae_pyin_viterbi | online          |                2 |       1    |               0    |               0    |       0    |
| rosa 10              | p10_decifrae_pyin_viterbi | online          |                8 |       0.98 |               0.2  |               0    |       0    |
| rosa 10              | p10_decifrae_pyin_viterbi | viterbi         |                2 |       1    |               0    |               0    |       0    |
| rosa 10              | p10_decifrae_pyin_viterbi | viterbi         |                8 |       1    |               0    |               0    |       0    |
| rosa 10              | p11_decifrae_harmsum      | argmax          |              nan |       1    |               0    |               0.08 |       0    |
| rosa 10              | p11_decifrae_harmsum      | online          |                2 |       1    |               0    |               0    |       0    |
| rosa 10              | p11_decifrae_harmsum      | online          |                8 |       0.98 |               0.35 |               0    |       0    |
| rosa 10              | p11_decifrae_harmsum      | viterbi         |                2 |       1    |               0    |               0.01 |       0    |
| rosa 10              | p11_decifrae_harmsum      | viterbi         |                8 |       1    |               0    |               0.01 |       0    |
| rosa 10              | p20b_so_frot              | argmax          |              nan |       0.99 |               0    |               0.45 |       0    |
| rosa 10              | p20b_so_frot              | online          |                2 |       0.99 |               0    |               0.45 |       0    |
| rosa 10              | p20b_so_frot              | online          |                8 |       0.99 |               0    |               0.45 |       0    |
| rosa 10              | p20b_so_frot              | viterbi         |                2 |       0.99 |               0    |               0.45 |       0    |
| rosa 10              | p20b_so_frot              | viterbi         |                8 |       0.99 |               0    |               0.45 |       0    |
| rosa 10              | p21_envelope              | argmax          |              nan |       0.87 |               0.05 |               1.87 |       0.07 |
| rosa 10              | p21_envelope              | online          |                2 |       0.86 |               1.25 |               0.35 |       0.23 |
| rosa 10              | p21_envelope              | online          |                8 |       0.66 |               2.35 |               0.07 |       1.47 |
| rosa 10              | p21_envelope              | viterbi         |                2 |       0.86 |               0    |               0.16 |       0.77 |
| rosa 10              | p21_envelope              | viterbi         |                8 |       0.65 |               0    |               0.2  |       1.97 |
| rosa 5               | p00_logmel_cnn            | argmax          |              nan |       0.62 |               0    |               0.29 |       2.13 |
| rosa 5               | p00_logmel_cnn            | online          |                2 |       0.62 |               0    |               0.29 |       2.13 |
| rosa 5               | p00_logmel_cnn            | online          |                8 |       0.62 |               0    |               0.29 |       2.13 |
| rosa 5               | p00_logmel_cnn            | viterbi         |                2 |       0.62 |               0    |               0.29 |       2.13 |
| rosa 5               | p00_logmel_cnn            | viterbi         |                8 |       0.62 |               0    |               0.29 |       2.13 |
| rosa 5               | p02_grollmisch            | argmax          |              nan |       0.94 |               0    |               0.04 |       0.3  |
| rosa 5               | p02_grollmisch            | online          |                2 |       0.94 |               0    |               0.04 |       0.3  |
| rosa 5               | p02_grollmisch            | online          |                8 |       0.94 |               0    |               0.04 |       0.3  |
| rosa 5               | p02_grollmisch            | viterbi         |                2 |       0.94 |               0    |               0.04 |       0.3  |
| rosa 5               | p02_grollmisch            | viterbi         |                8 |       0.94 |               0    |               0.04 |       0.3  |
| rosa 5               | p10_decifrae_pyin_viterbi | argmax          |              nan |       0.82 |               0.1  |               2.84 |       0.03 |
| rosa 5               | p10_decifrae_pyin_viterbi | online          |                2 |       0.81 |               0.2  |               0.97 |       0.37 |
| rosa 5               | p10_decifrae_pyin_viterbi | online          |                8 |       0.82 |               1.45 |               0.31 |       0.47 |
| rosa 5               | p10_decifrae_pyin_viterbi | viterbi         |                2 |       0.82 |               0    |               0.51 |       0.63 |
| rosa 5               | p10_decifrae_pyin_viterbi | viterbi         |                8 |       0.82 |               0    |               0.22 |       0.9  |
| rosa 5               | p11_decifrae_harmsum      | argmax          |              nan |       0.72 |               0.2  |               3.68 |       0.03 |
| rosa 5               | p11_decifrae_harmsum      | online          |                2 |       0.73 |               0.3  |               1.58 |       0.47 |
| rosa 5               | p11_decifrae_harmsum      | online          |                8 |       0.73 |               2.05 |               0.59 |       0.57 |
| rosa 5               | p11_decifrae_harmsum      | viterbi         |                2 |       0.73 |               0.05 |               0.57 |       0.8  |
| rosa 5               | p11_decifrae_harmsum      | viterbi         |                8 |       0.73 |               0.25 |               0.35 |       1.1  |
| rosa 5               | p20b_so_frot              | argmax          |              nan |       0.98 |               0    |               0.52 |       0    |
| rosa 5               | p20b_so_frot              | online          |                2 |       0.99 |               0    |               0.39 |       0    |
| rosa 5               | p20b_so_frot              | online          |                8 |       0.99 |               0    |               0.39 |       0    |
| rosa 5               | p20b_so_frot              | viterbi         |                2 |       0.99 |               0    |               0.39 |       0    |
| rosa 5               | p20b_so_frot              | viterbi         |                8 |       0.99 |               0    |               0.39 |       0    |
| rosa 5               | p21_envelope              | argmax          |              nan |       0.62 |               0    |               0.29 |       2.13 |
| rosa 5               | p21_envelope              | online          |                2 |       0.59 |               1.25 |               0.29 |       2.13 |
| rosa 5               | p21_envelope              | online          |                8 |       0.49 |               7.3  |               0.22 |       2.37 |
| rosa 5               | p21_envelope              | viterbi         |                2 |       0.62 |               0    |               0.29 |       2.13 |
| rosa 5               | p21_envelope              | viterbi         |                8 |       0.49 |               0    |               0.09 |       3.03 |

## T6. Detecção de anomalia treinada só no normal
Origem: `out/experiments/20261014_184738_anomalia.csv`

| pipeline     | cenario       | anomalo   |   auc |   pauc |
|:-------------|:--------------|:----------|------:|-------:|
| p30_anomalia | a2212_arquivo | Bearing   | 0.926 |  0.651 |
| p30_anomalia | a2212_arquivo | Propeller | 0.963 |  0.867 |
| p30_anomalia | a2212_arquivo | todas     | 0.955 |  0.811 |
| p30_anomalia | a2212_m1m2    | Propeller | 0.944 |  0.851 |
| p30_anomalia | a2212_m1m2    | todas     | 0.944 |  0.851 |
| p30_anomalia | idmt_limpo    | broken    | 1     |  1     |
| p30_anomalia | idmt_limpo    | heavyload | 1     |  1     |
| p30_anomalia | idmt_limpo    | todas     | 1     |  1     |
| p30_anomalia | idmt_rosa10   | broken    | 1     |  1     |
| p30_anomalia | idmt_rosa10   | heavyload | 1     |  1     |
| p30_anomalia | idmt_rosa10   | todas     | 1     |  1     |
| p30b_ae      | a2212_arquivo | Bearing   | 0.989 |  0.956 |
| p30b_ae      | a2212_arquivo | Propeller | 0.915 |  0.778 |
| p30b_ae      | a2212_arquivo | todas     | 0.94  |  0.835 |
| p30b_ae      | a2212_m1m2    | Propeller | 0.92  |  0.86  |
| p30b_ae      | a2212_m1m2    | todas     | 0.92  |  0.86  |
| p30b_ae      | idmt_limpo    | broken    | 1     |  1     |
| p30b_ae      | idmt_limpo    | heavyload | 1     |  1     |
| p30b_ae      | idmt_limpo    | todas     | 1     |  1     |
| p30b_ae      | idmt_rosa10   | broken    | 1     |  1     |
| p30b_ae      | idmt_rosa10   | heavyload | 1     |  1     |
| p30b_ae      | idmt_rosa10   | todas     | 1     |  1     |

## T7. Bancada sintética (verdade conhecida), SNR 10 dB
Origem: `out/experiments/20261014_185415_bancada_sintetica.csv`

Mediana entre 10 sementes. erro_pct_med = erro mediano de f_rot (%); frac_ok5 = fração de quadros com erro < 5 %; taxa_r133 = fração travada em 4/3·f. Variantes sem sufixo = grade de 10 cents; `_2c` = 2 cents.

| variante   |   broken |   flicker |   interruption |   nominal |   sag |   swell |
|:-----------|---------:|----------:|---------------:|----------:|------:|--------:|
| argmax     |    33.33 |     36.36 |          26.57 |      0.09 | 19.99 |   10.06 |
| argmax_2c  |    33.34 |     35.37 |           0.03 |      0.04 |  0.02 |    0.04 |
| pyin       |    33.33 |     41.13 |          26.57 |      0.09 | 10.04 |    0.2  |
| pyin_2c    |    33.34 |     41    |           0.03 |      0.04 |  0.02 |    0.04 |
| viterbi    |    33.33 |     41.13 |          26.57 |      0.09 | 10.04 |    0.2  |
| viterbi_2c |    33.34 |     41    |           0.03 |      0.04 |  0.02 |    0.04 |

| variante   |   broken |   flicker |   interruption |   nominal |   sag |   swell |
|:-----------|---------:|----------:|---------------:|----------:|------:|--------:|
| argmax     |        0 |      0.27 |           0.03 |         1 |  0.34 |    0.52 |
| argmax_2c  |        0 |      0.29 |           0.98 |         1 |  0.96 |    0.98 |
| pyin       |        0 |      0    |           0    |         1 |  0.44 |    0.81 |
| pyin_2c    |        0 |      0    |           0.98 |         1 |  0.65 |    1    |
| viterbi    |        0 |      0    |           0    |         1 |  0.44 |    0.81 |
| viterbi_2c |        0 |      0    |           0.98 |         1 |  0.65 |    1    |

## T8. Rotação rastreada × velocidade comandada (A2212, com Viterbi)
Origem: `out/experiments/20261014_175806_tacometro_a2212.csv`

Grade de 10 cents (rodado antes da correção).

|   faixa_min_Hz | classe    | motor   |   n |   spearman |     p |
|---------------:|:----------|:--------|----:|-----------:|------:|
|             15 | Bearing   | M4      |   7 |     -0.214 | 0.645 |
|             15 | Healthy   | M1      |   8 |      0.548 | 0.16  |
|             15 | Healthy   | M2      |   8 |      0.719 | 0.045 |
|             15 | Healthy   | M3      |   6 |      0.829 | 0.042 |
|             15 | Propeller | M1      |   6 |     -0.232 | 0.658 |
|             15 | Propeller | M2      |   8 |     -0.204 | 0.629 |
|             60 | Bearing   | M4      |   7 |      0.464 | 0.294 |
|             60 | Healthy   | M1      |   8 |      0.405 | 0.32  |
|             60 | Healthy   | M2      |   8 |      0.929 | 0.001 |
|             60 | Healthy   | M3      |   6 |     -0.429 | 0.397 |
|             60 | Propeller | M1      |   6 |      0.943 | 0.005 |
|             60 | Propeller | M2      |   8 |      0.735 | 0.038 |
|            100 | Bearing   | M4      |   7 |      1     | 0     |
|            100 | Healthy   | M1      |   8 |     -0.286 | 0.493 |
|            100 | Healthy   | M2      |   8 |      0.905 | 0.002 |
|            100 | Healthy   | M3      |   6 |     -0.6   | 0.208 |
|            100 | Propeller | M1      |   6 |      0.928 | 0.008 |
|            100 | Propeller | M2      |   8 |      0.735 | 0.038 |

## T9. Custo estimado para o ESP32 (analítico, não medido)
Origem: `custo_mcu` nos `_meta.json` de 20261014_185859_validacao.csv, 20261014_203110_validacao_cnn.csv

| pipeline                  |   params |    ram_kb |    flash_kb |   mflops_s | nota                                                                                                                                                                                                                                                                                                                                                                                                               |
|:--------------------------|---------:|----------:|------------:|-----------:|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| b00_nivel                 |        9 |   0.1     |  nan        |    0.03    | RMS + pico por janela                                                                                                                                                                                                                                                                                                                                                                                              |
| b01_forma_espectral       |      114 |   9       |  nan        |    1.5872  | estimativa analítica                                                                                                                                                                                                                                                                                                                                                                                               |
| p03_etnext                |     7070 |   1.79297 |   27.3438   |    0.8     | estimativa analítica; n_SV ~200 suposto (depende do treino)                                                                                                                                                                                                                                                                                                                                                        |
| p03b_etnext_knn           |    70000 |   1.79297 |  273.438    |    0.8     | estimativa analítica; n_treino ~2000 suposto (depende da base)                                                                                                                                                                                                                                                                                                                                                     |
| p04_classicas             |    68099 |   7.23047 |  292.113    |    2.07485 | estimativa analítica; classificador dominado por kNN/SVM, que guardam o treino (n_treino≈1000 suposto); a variante embarcável é p04c_classicas_lr                                                                                                                                                                                                                                                                  |
| p04b_msaf                 |      113 |  96.0039  |    0.441406 |    0.62276 | estimativa analítica; RAM dominada pela FFT de 16 k pontos (~96 KB, cabe na SRAM do ESP32 sem folga); alternativa: Goertzel só nos 28 bins (~28·16k MAC/s, RAM < 1 KB, mas perde a normalização L1 sobre o espectro inteiro)                                                                                                                                                                                       |
| p04c_classicas_lr         |      163 |   5.23047 |   26.7383   |    2.00708 | estimativa analítica; custo dominado pela FFT de 512 pts (62,5 quadros/s)                                                                                                                                                                                                                                                                                                                                          |
| p10_decifrae_pyin_viterbi |       65 | 167.2     |  nan        |   15.79    | estimativa analítica; pYIN não embarcável                                                                                                                                                                                                                                                                                                                                                                          |
| p11_decifrae_harmsum      |       65 | 167.2     |  nan        |   15.79    | estimativa analítica; Viterbi com janela de 40 quadros                                                                                                                                                                                                                                                                                                                                                             |
| p11a_grade10              |       65 |  84.7     |  nan        |    9.22    | estimativa analítica; Viterbi com janela de 40 quadros                                                                                                                                                                                                                                                                                                                                                             |
| p12_decifrae_sem_viterbi  |       65 |  73.4     |  nan        |   11.94    | estimativa analítica; Viterbi com janela de 40 quadros                                                                                                                                                                                                                                                                                                                                                             |
| p20_hfs                   |     1682 | 194.7     |    6.6      |   18.52    | estimativa analítica; inclui o rastreador (15.79 MFLOP/s); N=2560 (f_o=25 Hz, FFT não potência de 2: no MCU, Goertzel nos K=240 bins ou FFT fixa + interpolação); kNN guarda os pontos de treino em 2-D (8 B por janela de treino, fora de 'params')                                                                                                                                                               |
| p20b_so_frot              |        6 | 167.2     |    0.1      |   15.79    | custo = rastreador (como p11); kNN guarda os pontos de treino em 3-D (12 B por janela de treino, fora de 'params')                                                                                                                                                                                                                                                                                                 |
| p21_envelope              |      228 |  24.2     |    1.4      |    3.14    | estimativa analítica, janela de 1 s (buffer de envelope 1024 floats por banda; 3 s triplica a RAM do buffer); params = LR (3 classes) + média/desvio do scaler                                                                                                                                                                                                                                                     |
| p30_anomalia              |     1122 |   8.2     |    4.4      |    1.67    | estimativa analítica; inv(Sigma) 32x32 em float32 = 4 kB; d2 de Mahalanobis ~2 kFLOP por janela                                                                                                                                                                                                                                                                                                                    |
| p30b_ae                   |    50760 |  12.2     |   49.6      |    4.8     | estimativa analítica; flash em int8 (×4 em float32); AE por quadro de 32 ms com contexto de 5 quadros                                                                                                                                                                                                                                                                                                              |
| p00_logmel_cnn            |     7703 |  10.4575  |    7.62     |    3.57    | 7703 parâmetros (artigo ≈7,7 kB flash int8 coerente); MACC analítico 517467 vs 501 428 do X-CUBE-AI. ESP32 (LX6 240 MHz) estimado escalando os ≈6.5 ciclos/MACC medidos no STM32 pela frequência: ≈14 ms/inferência (ESP-NN no ESP32-S3 tende a ser mais rápido; sem SIMD no LX6 pode ser 2× pior). RAM: 5,5 kB de ativações do artigo + buffers de entrada e FFT. Uma inferência por 1,024 s: carga < 2 % da CPU. |
| p01_mfcc_cnn1d            |     3553 |   7       |   38.2      |    2.38    | 3553 parâmetros (pesos int8 ≈ 3.5 kB; os 38,2 kB do EI incluem DSP e runtime). 112560 MACC por inferência, ~4,4× menos que a rota STM (501 428). ESP32: rede ≈ 3.1 ms escalando os ciclos/MACC do STM32 pela frequência; o gargalo reportado é o DSP (337 ms por janela no MCU do artigo, que com passo de 0,5 s ocuparia > 60 % da CPU), não a rede. Comparação: rota STM 5,5 kB RAM, 7,7 kB flash, 40,8 ms.      |
| p02_grollmisch            |   542083 |  49.3     | 2117.5      |   28.6     | estimativa analítica; pesos float32 (int8: ~1/4 do flash); RAM = buffer de entrada com sobreposição + FFT + ativações                                                                                                                                                                                                                                                                                              |
| p05_melcnn                |  2452483 | 373.5     | 9580        |   34.6     | estimativa analítica; flash float32 ~9,4 MB não cabe no ESP32 (int8 ~2,4 MB); RAM = duas maiores ativações + mel de 3 s; 1 inferência por janela de 3 s                                                                                                                                                                                                                                                            |
