# Tabelas da auditoria (geradas por `scripts/consolidar.py`)

Gerado em 23/10/2026 08:40. Não editar à mão. Acurácia balanceada (bacc) das predições concatenadas de todas as dobras; `acaso` = 1/(classes no teste).

## 1. Inventário (de `out/indices/<id>.csv`)

| base | arquivos | gravações | unidades | classes | horas | taxa (Hz) | máx. classes numa unidade | unidades com ≥2 classes | arquivos com f_alim | com rotação medida |
|---|---|---|---|---|---|---|---|---|---|---|
| kaist | 5 | 5 | 5 | 3 | 0.08 | 51200 | 1 | 0 | 0 | 0 |
| uoemd | 128 | 128 | 8 | 8 | 0.36 | 42000 | 1 | 0 | 64 | 0 |

## 2. Exp1 — controle de nível (RMS + pico → classe), tarefa `todas`

| base | aleatório | por gravação | por unidade (motor fora) | acaso | CSV |
|---|---|---|---|---|---|
| kaist | 0.500 | — | 0.167 | 0.333 | `out/experiments/exp1_nivel_kaist_20261022-235513.csv` |
| uoemd | 0.221 | 0.180 | 0.000 | 0.125 | `out/experiments/exp1_nivel_uoemd_20261022-235654.csv` |

## 3. Exp2 — rótulos permutados (média de 5 permutações, tarefa `todas`)

Permutação por segmento tem de dar o acaso em todo split. Permutação por gravação acima do acaso no split aleatório significa que as features reconhecem a gravação.

| base | features | permutação | aleatório | por gravação | por unidade | acaso |
|---|---|---|---|---|---|---|
| kaist | forma | gravacao | 1.000 | — | 0.087 | 0.333 |
| kaist | forma | segmento | 0.319 | 0.343 | 0.343 | 0.333 |
| kaist | nivel | gravacao | 0.591 | — | 0.167 | 0.333 |
| kaist | nivel | segmento | 0.303 | 0.349 | 0.349 | 0.333 |
| uoemd | forma | gravacao | 0.467 | 0.164 | 0.187 | 0.125 |
| uoemd | forma | segmento | 0.120 | 0.123 | 0.126 | 0.125 |
| uoemd | nivel | gravacao | 0.169 | 0.133 | 0.142 | 0.125 |
| uoemd | nivel | segmento | 0.125 | 0.125 | 0.121 | 0.125 |

## 4. Exp3 — distância entre espectros médios (dB RMS, só forma, 50–7900 Hz)

| base | classe diferente, mesma unidade | unidade diferente, mesma classe | ambas diferentes | CSV |
|---|---|---|---|---|
| kaist | — | 6.16 (n=2) | 7.34 (n=8) | `out/experiments/exp3_espectros_kaist_20261022-235517.csv` |
| uoemd | — | — | 1.29 (n=28) | `out/experiments/exp3_espectros_uoemd_20261022-235709.csv` |

## 5. Exp4 — rastreador (regra pré-registrada: 0,40–1,05 × f_alim)

`mod. oitava` é diagnóstico a posteriori: conta como acerto também f_ref/2 e 2·f_ref.

| base | condição | arquivos | erro rel. mediano | frac. arquivos ≤2 % | frac. segmentos ≤2 % | f_alim/f_rot mediana | frac. ≤2 % mod. oitava | CSV |
|---|---|---|---|---|---|---|---|---|
| uoemd | todas | 64 | 0.251 | 0.469 | 0.469 | 1.471 | 0.938 | `out/experiments/exp4_rastreador_uoemd_20261022-235745.csv` |
| uoemd | 15 | 16 | 0.022 | 0.438 | 0.438 | 1.023 | 0.812 |  |
| uoemd | 30 | 16 | 0.504 | 0.000 | 0.000 | 2.018 | 0.938 |  |
| uoemd | 45 | 16 | 0.009 | 0.812 | 0.812 | 1.009 | 1.000 |  |
| uoemd | 60 | 16 | 0.013 | 0.625 | 0.625 | 1.014 | 1.000 |  |

## 6. Exp5 — separabilidade por unidade (forma espectral → unidade)

| base | tarefa | features | aleatório | por gravação | acaso | CSV |
|---|---|---|---|---|---|---|
| kaist | classe=rolamento_externo | forma | 1.000 | — | 0.500 | `out/experiments/exp5_unidade_kaist_20261022-235519.csv` |
| kaist | classe=rolamento_externo | forma+nivel | 1.000 | — | 0.500 | `out/experiments/exp5_unidade_kaist_20261022-235519.csv` |
| kaist | classe=rolamento_interno | forma | 1.000 | — | 0.500 | `out/experiments/exp5_unidade_kaist_20261022-235519.csv` |
| kaist | classe=rolamento_interno | forma+nivel | 1.000 | — | 0.500 | `out/experiments/exp5_unidade_kaist_20261022-235519.csv` |
| kaist | todas | forma | 1.000 | — | 0.200 | `out/experiments/exp5_unidade_kaist_20261022-235519.csv` |
| kaist | todas | forma+nivel | 1.000 | — | 0.200 | `out/experiments/exp5_unidade_kaist_20261022-235519.csv` |
| uoemd | todas | forma | 0.705 | 0.447 | 0.125 | `out/experiments/exp5_unidade_uoemd_20261022-235713.csv` |
| uoemd | todas | forma+nivel | 0.719 | 0.458 | 0.125 | `out/experiments/exp5_unidade_uoemd_20261022-235713.csv` |

## 7. Matrizes classe × unidade (arquivos)

### kaist

| classe            |   bancada_KAIST/rolamento_A_BPFI_0.3mm |   bancada_KAIST/rolamento_A_BPFI_1.0mm |   bancada_KAIST/rolamento_A_BPFO_0.3mm |   bancada_KAIST/rolamento_A_BPFO_1.0mm |   bancada_KAIST/rolamento_A_normal |
|:------------------|---------------------------------------:|---------------------------------------:|---------------------------------------:|---------------------------------------:|-----------------------------------:|
| normal            |                                      0 |                                      0 |                                      0 |                                      0 |                                  1 |
| rolamento_externo |                                      0 |                                      0 |                                      1 |                                      1 |                                  0 |
| rolamento_interno |                                      1 |                                      1 |                                      0 |                                      0 |                                  0 |

### uoemd

| classe                            |   motor_B_R |   motor_F_B |   motor_H_H |   motor_K_A |   motor_R_M |   motor_R_U |   motor_S_W |   motor_V_U |
|:----------------------------------|------------:|------------:|------------:|------------:|------------:|------------:|------------:|------------:|
| barras_rotor_quebradas            |           0 |           0 |           0 |          16 |           0 |           0 |           0 |           0 |
| desalinhamento_rotor              |           0 |           0 |           0 |           0 |          16 |           0 |           0 |           0 |
| desequilibrio_rotor               |           0 |           0 |           0 |           0 |           0 |          16 |           0 |           0 |
| desequilibrio_tensao_e_falta_fase |           0 |           0 |           0 |           0 |           0 |           0 |           0 |          16 |
| falha_enrolamento_estator         |           0 |           0 |           0 |           0 |           0 |           0 |          16 |           0 |
| rolamento_defeituoso              |           0 |          16 |           0 |           0 |           0 |           0 |           0 |           0 |
| rotor_empenado                    |          16 |           0 |           0 |           0 |           0 |           0 |           0 |           0 |
| saudavel                          |           0 |           0 |          16 |           0 |           0 |           0 |           0 |           0 |

