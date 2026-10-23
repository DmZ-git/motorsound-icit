# Tabelas da auditoria (geradas por `scripts/consolidar.py`)

Gerado em 23/10/2026 11:09. Não editar à mão. Acurácia balanceada (bacc) das predições concatenadas de todas as dobras; `acaso` = 1/(classes no teste).

## 1. Inventário (de `out/indices/<id>.csv`)

| base | arquivos | gravações | unidades | classes | horas | taxa (Hz) | máx. classes numa unidade | unidades com ≥2 classes | arquivos com f_alim | com rotação medida |
|---|---|---|---|---|---|---|---|---|---|---|
| imvacd | 256 | 96 | 8 | 8 | 0.72 | 44100, 48000 | 1 | 0 | 256 | 0 |
| kaist | 5 | 5 | 5 | 3 | 0.08 | 51200 | 1 | 0 | 0 | 0 |
| mafaulda | 1951 | 1951 | 11 | 10 | 2.65 | 51200 | 3 | 4 | 0 | 1951 |
| mimii | 18173 | 18173 | 14 | 2 | 50.48 | 16000 | 2 | 14 | 0 | 0 |
| toycar | 7200 | 7200 | 15 | 2 | 20.00 | 16000 | 2 | 9 | 0 | 0 |
| uoemd | 128 | 128 | 8 | 8 | 0.36 | 42000 | 1 | 0 | 64 | 0 |

## 2. Exp1 — controle de nível (RMS + pico → classe), tarefa `todas`

A última coluna treina e testa dentro de cada unidade que tem ≥2 classes (split por gravação) e dá mediana [mín–máx] entre unidades.

| base | aleatório | por gravação | por unidade (motor fora) | acaso | dentro da unidade | CSV |
|---|---|---|---|---|---|---|
| imvacd | 0.152 | 0.117 | 0.000 | 0.125 | — | `out/experiments/exp1_nivel_imvacd_20261023-084715.csv` |
| kaist | 0.500 | — | 0.167 | 0.333 | — | `out/experiments/exp1_nivel_kaist_20261022-235513.csv` |
| mafaulda | 0.432 | 0.430 | 0.052 | 0.100 | 0.888 [0.671–0.962] (n=4) | `out/experiments/exp1_nivel_mafaulda_20261023-110036.csv` |
| mimii | 0.520 | 0.518 | 0.506 | 0.500 | 0.772 [0.572–0.856] (n=8) | `out/experiments/exp1_nivel_mimii_20261023-101118.csv` |
| toycar | 0.639 | 0.642 | 0.636 | 0.500 | 0.511 [0.444–0.591] (n=6) | `out/experiments/exp1_nivel_toycar_20261023-101124.csv` |
| uoemd | 0.221 | 0.180 | 0.000 | 0.125 | — | `out/experiments/exp1_nivel_uoemd_20261022-235654.csv` |

Referência IDMT e A2212 (protocolo do Artigo3, não reanalisadas; `out/experiments/ref_artigo3_idmt_a2212_20261023-094832.csv`): acurácia balanceada com RMS normalizado.

| base | pipeline | aleatório | por gravação/blocos | por unidade (motor fora) | acaso |
|---|---|---|---|---|---|
| a2212 | b00_nivel | 0.564 | 0.562 | 0.542 | 0.333 |
| a2212 | b01_forma_espectral | 0.998 | 0.910 | 0.578 | 0.333 |
| idmt | b00_nivel | 0.992 | 0.991 | — | 0.333 |
| idmt | b01_forma_espectral | 1.000 | 1.000 | — | 0.333 |

## 3. Exp2 — rótulos permutados (média de 5 permutações, tarefa `todas`)

Permutação por segmento tem de dar o acaso em todo split. Permutação por gravação acima do acaso no split aleatório significa que as features reconhecem a gravação.

| base | features | permutação | aleatório | por gravação | por unidade | acaso |
|---|---|---|---|---|---|---|
| imvacd | forma | gravacao | 0.598 | 0.112 | 0.129 | 0.125 |
| imvacd | forma | segmento | 0.128 | 0.121 | 0.122 | 0.125 |
| imvacd | nivel | gravacao | 0.173 | 0.126 | 0.134 | 0.125 |
| imvacd | nivel | segmento | 0.123 | 0.124 | 0.126 | 0.125 |
| kaist | forma | gravacao | 1.000 | — | 0.087 | 0.333 |
| kaist | forma | segmento | 0.319 | 0.343 | 0.343 | 0.333 |
| kaist | nivel | gravacao | 0.591 | — | 0.167 | 0.333 |
| kaist | nivel | segmento | 0.303 | 0.349 | 0.349 | 0.333 |
| mafaulda | forma | gravacao | 0.166 | 0.101 | 0.097 | 0.100 |
| mafaulda | forma | segmento | 0.099 | 0.097 | 0.100 | 0.100 |
| mafaulda | nivel | gravacao | 0.109 | 0.099 | 0.096 | 0.100 |
| mafaulda | nivel | segmento | 0.101 | 0.101 | 0.102 | 0.100 |
| mimii | forma | gravacao | 0.514 | 0.498 | 0.501 | 0.500 |
| mimii | forma | segmento | 0.502 | 0.500 | 0.500 | 0.500 |
| mimii | nivel | gravacao | 0.502 | 0.496 | 0.499 | 0.500 |
| mimii | nivel | segmento | 0.500 | 0.498 | 0.499 | 0.500 |
| toycar | forma | gravacao | 0.530 | 0.505 | 0.504 | 0.500 |
| toycar | forma | segmento | 0.500 | 0.500 | 0.497 | 0.500 |
| toycar | nivel | gravacao | 0.504 | 0.495 | 0.494 | 0.500 |
| toycar | nivel | segmento | 0.501 | 0.501 | 0.502 | 0.500 |
| uoemd | forma | gravacao | 0.467 | 0.164 | 0.187 | 0.125 |
| uoemd | forma | segmento | 0.120 | 0.123 | 0.126 | 0.125 |
| uoemd | nivel | gravacao | 0.169 | 0.133 | 0.142 | 0.125 |
| uoemd | nivel | segmento | 0.125 | 0.125 | 0.121 | 0.125 |

## 4. Exp3 — distância entre espectros médios (dB RMS, só forma, 50–7900 Hz)

| base | classe diferente, mesma unidade | unidade diferente, mesma classe | ambas diferentes | CSV |
|---|---|---|---|---|
| imvacd | — | — | 1.42 (n=28) | `out/experiments/exp3_espectros_imvacd_20261023-084738.csv` |
| kaist | — | 6.16 (n=2) | 7.34 (n=8) | `out/experiments/exp3_espectros_kaist_20261022-235517.csv` |
| mafaulda | 0.55 (n=6) | 0.71 (n=21) | 0.65 (n=93) | `out/experiments/exp3_espectros_mafaulda_20261023-104729.csv` |
| mimii | 1.54 (n=8) | 3.15 (n=56) | 2.94 (n=56) | `out/experiments/exp3_espectros_mimii_20261023-100951.csv` |
| toycar | 1.06 (n=6) | 2.39 (n=81) | 2.59 (n=66) | `out/experiments/exp3_espectros_toycar_20261023-091523.csv` |
| uoemd | — | — | 1.29 (n=28) | `out/experiments/exp3_espectros_uoemd_20261022-235709.csv` |

## 5. Exp4 — rastreador (regras pré-registradas)

UOEMD e IM-VACD: faixa 0,40–1,05 × f_alim, referência = rotação síncrona f_alim (2 polos), logo o erro inclui o escorregamento. MaFaulDa: faixa única da base (0,9 × mín a 1,1 × máx da rotação medida), referência = tacômetro.

`mod. oitava` é diagnóstico a posteriori: conta como acerto também f_ref/2 e 2·f_ref.

| base | condição | arquivos | erro rel. mediano | frac. arquivos ≤2 % | frac. segmentos ≤2 % | f_alim/f_rot mediana | frac. ≤2 % mod. oitava | CSV |
|---|---|---|---|---|---|---|---|---|
| imvacd | todas | 256 | 0.017 | 0.566 | 0.564 | 1.018 | 0.656 | `out/experiments/exp4_rastreador_imvacd_20261023-084932.csv` |
| imvacd | 15 | 96 | 0.024 | 0.323 | 0.317 | 1.025 | 0.375 |  |
| imvacd | 20 | 32 | 0.023 | 0.438 | 0.438 | 1.024 | 0.500 |  |
| imvacd | 25 | 32 | 0.012 | 0.812 | 0.812 | 1.012 | 0.812 |  |
| imvacd | 30 | 96 | 0.012 | 0.771 | 0.771 | 1.012 | 0.938 |  |
| mafaulda | todas | 1951 | 0.000 | 0.967 | 0.966 | — | 0.982 | `out/experiments/exp4_rastreador_mafaulda_20261023-110820.csv` |
| mafaulda | sem canal defeituoso | 1785 | 0.000 | 0.999 | 0.998 | — | 0.999 |  |
| mafaulda | só canal defeituoso | 166 | 0.002 | 0.627 | 0.620 | — | 0.801 |  |
| uoemd | todas | 64 | 0.251 | 0.469 | 0.469 | 1.471 | 0.938 | `out/experiments/exp4_rastreador_uoemd_20261022-235745.csv` |
| uoemd | 15 | 16 | 0.022 | 0.438 | 0.438 | 1.023 | 0.812 |  |
| uoemd | 30 | 16 | 0.504 | 0.000 | 0.000 | 2.018 | 0.938 |  |
| uoemd | 45 | 16 | 0.009 | 0.812 | 0.812 | 1.009 | 1.000 |  |
| uoemd | 60 | 16 | 0.013 | 0.625 | 0.625 | 1.014 | 1.000 |  |

## 6. Exp5 — separabilidade por unidade (forma espectral → unidade)

| base | tarefa | features | aleatório | por gravação | acaso | CSV |
|---|---|---|---|---|---|---|
| imvacd | todas | forma | 0.835 | 0.439 | 0.125 | `out/experiments/exp5_unidade_imvacd_20261023-084743.csv` |
| imvacd | todas | forma+nivel | 0.841 | 0.444 | 0.125 | `out/experiments/exp5_unidade_imvacd_20261023-084743.csv` |
| kaist | classe=rolamento_externo | forma | 1.000 | — | 0.500 | `out/experiments/exp5_unidade_kaist_20261022-235519.csv` |
| kaist | classe=rolamento_externo | forma+nivel | 1.000 | — | 0.500 | `out/experiments/exp5_unidade_kaist_20261022-235519.csv` |
| kaist | classe=rolamento_interno | forma | 1.000 | — | 0.500 | `out/experiments/exp5_unidade_kaist_20261022-235519.csv` |
| kaist | classe=rolamento_interno | forma+nivel | 1.000 | — | 0.500 | `out/experiments/exp5_unidade_kaist_20261022-235519.csv` |
| kaist | todas | forma | 1.000 | — | 0.200 | `out/experiments/exp5_unidade_kaist_20261022-235519.csv` |
| kaist | todas | forma+nivel | 1.000 | — | 0.200 | `out/experiments/exp5_unidade_kaist_20261022-235519.csv` |
| mafaulda | classe=desbalanceamento | forma | 0.623 | 0.393 | 0.143 | `out/experiments/exp5_unidade_mafaulda_20261023-110152.csv` |
| mafaulda | classe=desbalanceamento | forma+nivel | 0.688 | 0.451 | 0.143 | `out/experiments/exp5_unidade_mafaulda_20261023-110152.csv` |
| mafaulda | todas | forma | 0.562 | 0.396 | 0.091 | `out/experiments/exp5_unidade_mafaulda_20261023-110152.csv` |
| mafaulda | todas | forma+nivel | 0.658 | 0.488 | 0.091 | `out/experiments/exp5_unidade_mafaulda_20261023-110152.csv` |
| mimii | classe=abnormal | forma | 0.986 | 0.981 | 0.125 | `out/experiments/exp5_unidade_mimii_20261023-101009.csv` |
| mimii | classe=abnormal | forma+nivel | 0.987 | 0.983 | 0.125 | `out/experiments/exp5_unidade_mimii_20261023-101009.csv` |
| mimii | classe=normal | forma | 1.000 | 1.000 | 0.125 | `out/experiments/exp5_unidade_mimii_20261023-101009.csv` |
| mimii | classe=normal | forma+nivel | 1.000 | 1.000 | 0.125 | `out/experiments/exp5_unidade_mimii_20261023-101009.csv` |
| mimii | todas | forma | 0.996 | 0.996 | 0.125 | `out/experiments/exp5_unidade_mimii_20261023-101009.csv` |
| mimii | todas | forma+nivel | 0.997 | 0.996 | 0.125 | `out/experiments/exp5_unidade_mimii_20261023-101009.csv` |
| toycar | classe=anomalia | forma | 0.801 | 0.752 | 0.167 | `out/experiments/exp5_unidade_toycar_20261023-091657.csv` |
| toycar | classe=anomalia | forma+nivel | 0.796 | 0.753 | 0.167 | `out/experiments/exp5_unidade_toycar_20261023-091657.csv` |
| toycar | classe=normal | forma | 0.886 | 0.859 | 0.083 | `out/experiments/exp5_unidade_toycar_20261023-091657.csv` |
| toycar | classe=normal | forma+nivel | 0.881 | 0.859 | 0.083 | `out/experiments/exp5_unidade_toycar_20261023-091657.csv` |
| toycar | todas | forma | 0.872 | 0.868 | 0.083 | `out/experiments/exp5_unidade_toycar_20261023-091657.csv` |
| toycar | todas | forma+nivel | 0.874 | 0.868 | 0.083 | `out/experiments/exp5_unidade_toycar_20261023-091657.csv` |
| uoemd | todas | forma | 0.705 | 0.447 | 0.125 | `out/experiments/exp5_unidade_uoemd_20261022-235713.csv` |
| uoemd | todas | forma+nivel | 0.719 | 0.458 | 0.125 | `out/experiments/exp5_unidade_uoemd_20261022-235713.csv` |

## 7. Matrizes classe × unidade (arquivos)

### imvacd

| classe                            |   motor_B_R |   motor_F_B |   motor_H_H |   motor_K_A |   motor_R_M |   motor_R_U |   motor_S_W |   motor_V_U |
|:----------------------------------|------------:|------------:|------------:|------------:|------------:|------------:|------------:|------------:|
| barras_rotor_quebradas            |           0 |           0 |           0 |          32 |           0 |           0 |           0 |           0 |
| desalinhamento_rotor              |           0 |           0 |           0 |           0 |          32 |           0 |           0 |           0 |
| desequilibrio_rotor               |           0 |           0 |           0 |           0 |           0 |          32 |           0 |           0 |
| desequilibrio_tensao_e_falta_fase |           0 |           0 |           0 |           0 |           0 |           0 |           0 |          32 |
| falha_enrolamento_estator         |           0 |           0 |           0 |           0 |           0 |           0 |          32 |           0 |
| rolamento_defeituoso              |           0 |          32 |           0 |           0 |           0 |           0 |           0 |           0 |
| rotor_empenado                    |          32 |           0 |           0 |           0 |           0 |           0 |           0 |           0 |
| saudavel                          |           0 |           0 |          32 |           0 |           0 |           0 |           0 |           0 |

### kaist

| classe            |   bancada_KAIST/rolamento_A_BPFI_0.3mm |   bancada_KAIST/rolamento_A_BPFI_1.0mm |   bancada_KAIST/rolamento_A_BPFO_0.3mm |   bancada_KAIST/rolamento_A_BPFO_1.0mm |   bancada_KAIST/rolamento_A_normal |
|:------------------|---------------------------------------:|---------------------------------------:|---------------------------------------:|---------------------------------------:|-----------------------------------:|
| normal            |                                      0 |                                      0 |                                      0 |                                      0 |                                  1 |
| rolamento_externo |                                      0 |                                      0 |                                      1 |                                      1 |                                  0 |
| rolamento_interno |                                      1 |                                      1 |                                      0 |                                      0 |                                  0 |

### mafaulda

| classe                            |   bancada_mafaulda |   bancada_mafaulda+massa_10g |   bancada_mafaulda+massa_15g |   bancada_mafaulda+massa_20g |   bancada_mafaulda+massa_25g |   bancada_mafaulda+massa_30g |   bancada_mafaulda+massa_35g |   bancada_mafaulda+massa_6g |   rolamento_defeituoso_esfera |   rolamento_defeituoso_gaiola |   rolamento_defeituoso_pista_externa |
|:----------------------------------|-------------------:|-----------------------------:|-----------------------------:|-----------------------------:|-----------------------------:|-----------------------------:|-----------------------------:|----------------------------:|------------------------------:|------------------------------:|-------------------------------------:|
| desalinhamento_horizontal         |                197 |                            0 |                            0 |                            0 |                            0 |                            0 |                            0 |                           0 |                             0 |                             0 |                                    0 |
| desalinhamento_vertical           |                301 |                            0 |                            0 |                            0 |                            0 |                            0 |                            0 |                           0 |                             0 |                             0 |                                    0 |
| desbalanceamento                  |                  0 |                           48 |                           48 |                           49 |                           47 |                           47 |                           45 |                          49 |                             0 |                             0 |                                    0 |
| normal                            |                 49 |                            0 |                            0 |                            0 |                            0 |                            0 |                            0 |                           0 |                             0 |                             0 |                                    0 |
| rolamento_overhang_esfera         |                  0 |                            0 |                            0 |                            0 |                            0 |                            0 |                            0 |                           0 |                           137 |                             0 |                                    0 |
| rolamento_overhang_gaiola         |                  0 |                            0 |                            0 |                            0 |                            0 |                            0 |                            0 |                           0 |                             0 |                           188 |                                    0 |
| rolamento_overhang_pista_externa  |                  0 |                            0 |                            0 |                            0 |                            0 |                            0 |                            0 |                           0 |                             0 |                             0 |                                  188 |
| rolamento_underhang_esfera        |                  0 |                            0 |                            0 |                            0 |                            0 |                            0 |                            0 |                           0 |                           186 |                             0 |                                    0 |
| rolamento_underhang_gaiola        |                  0 |                            0 |                            0 |                            0 |                            0 |                            0 |                            0 |                           0 |                             0 |                           188 |                                    0 |
| rolamento_underhang_pista_externa |                  0 |                            0 |                            0 |                            0 |                            0 |                            0 |                            0 |                           0 |                             0 |                             0 |                                  184 |

### mimii

| classe   |   fan_id_00 |   fan_id_02 |   fan_id_04 |   fan_id_06 |   fan_sec_00 |   fan_sec_01 |   fan_sec_02 |   fan_sec_03 |   fan_sec_04 |   fan_sec_05 |   pump_id_00 |   pump_id_02 |   pump_id_04 |   pump_id_06 |
|:---------|------------:|------------:|------------:|------------:|-------------:|-------------:|-------------:|-------------:|-------------:|-------------:|-------------:|-------------:|-------------:|-------------:|
| abnormal |         407 |         359 |         348 |         361 |          200 |          200 |          200 |          200 |          200 |          200 |          143 |          111 |          100 |          102 |
| normal   |        1011 |        1016 |        1033 |        1015 |         1203 |         1203 |         1203 |         1203 |         1203 |         1203 |         1006 |         1005 |          702 |         1036 |

### toycar

| classe   |   car_A1 |   car_A2 |   car_C1 |   car_C2 |   car_E1 |   car_E2 |   car_F1 |   car_F2 |   car_G1 |   car_G2 |   car_H1 |   car_H2 |   desconhecida_sec03 |   desconhecida_sec04 |   desconhecida_sec05 |
|:---------|---------:|---------:|---------:|---------:|---------:|---------:|---------:|---------:|---------:|---------:|---------:|---------:|---------------------:|---------------------:|---------------------:|
| anomalia |      113 |      112 |       24 |       26 |       17 |        8 |        0 |        0 |        0 |        0 |        0 |        0 |                  100 |                  100 |                  100 |
| normal   |     1357 |     1364 |       31 |       29 |      129 |      390 |     1201 |     1203 |       10 |       10 |      288 |      288 |                  100 |                  100 |                  100 |

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

