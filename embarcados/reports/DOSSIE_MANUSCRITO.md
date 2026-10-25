# Dossiê de evidências para o manuscrito

> Gerado por `embarcados/src/python/gerar_dossie.py` em 20261025-153841 (commit 7459bd0). Não editar à mão. Todo valor sai de um arquivo citado; "não medido" e "não disponível" são literais.

## 1. Bases de dados e protocolo experimental

**ToyCar / DCASE 2022 Task 2: não usado.** Nenhum experimento deste projeto usa ToyCar ou qualquer dado DCASE. As únicas bases são IDMT-ISA-ELECTRIC-ENGINE e BLDC A2212 (regra em `CLAUDE.md` e `embarcados/CLAUDE.md`; inventário em `docs/DATASETS.md`). Se o manuscrito cita ToyCar, isso precisa sair.

Contagem por listagem de arquivos feita por este script (o `test/` do IDMT não foi listado nem lido):

| base | subconjunto | classe | arquivos | gravações de origem |
|---|---|---|---|---|
| IDMT | train_cut (recortes de 3 s de train/<classe>/pure.wav) | engine1_good | 105 | 1 |
| IDMT | train_cut (recortes de 3 s de train/<classe>/pure.wav) | engine2_broken | 124 | 1 |
| IDMT | train_cut (recortes de 3 s de train/<classe>/pure.wav) | engine3_heavyload | 128 | 1 |
| A2212 | todos (sem split oficial) | Bearing | 7 | 7 (M4) |
| A2212 | todos (sem split oficial) | Healthy | 22 | 22 (M1, M2, M3) |
| A2212 | todos (sem split oficial) | Propeller | 14 | 14 (M1, M2) |

Versões, formatos, sha256 dos zips, contagens do `test/`/`test_cut/` do IDMT (lidas só do cabeçalho em 14/10) e as particularidades (uma gravação por classe no `train/` do IDMT; classe = unidade física no IDMT; Bearing só no motor M4 do A2212; um arquivo A2212 a 44,1 kHz; saturação em Propeller/M1) estão em `docs/DATASETS.md` §1–2. **Arquivos excluídos:** nenhum arquivo inteiro; a sobra final < 3 s de cada `train/<classe>/pure.wav` não vira recorte (decisão dos autores da base).

**Divisão.** Desenvolvimento só em validação cruzada (o teste fica trancado por código em `motorsound/io.py`). O teste oficial do IDMT foi lido **uma única vez**, em 25/10/2026, com autorização do Erick e depois do congelamento commitado das configurações (§2, "Teste oficial do IDMT"). Os demais números são de validação:

- IDMT: blocos temporais. Cada `train/<classe>/pure.wav` em segmentos de 3 s agrupados em 5 blocos contíguos; deixa-um-bloco-de-fora, com o primeiro segmento de cada bloco descartado como guarda. Grupo = (gravação, bloco). Fonte: `docs/DATASETS.md` §3, `motorsound/splits.py`.
- A2212: `StratifiedGroupKFold` de 5 dobras agrupado por arquivo, segmentos de 1 s; e deixa-um-motor-de-fora (protocolo `motor`), que é o teste com **unidades não vistas**.
- Protocolo `aleatorio` (vazamento intencional, janelas da mesma gravação nos dois lados) só como auditoria da inflação.
- Seed dos splits: `20261014` (meta `out/experiments/20261014_185859_validacao_meta.json`). Todas as 20 pipelines rodaram nos mesmos cenários e partições: `[['idmt', 'blocos', True], ['idmt', 'aleatorio', True], ['idmt', 'blocos', False], ['a2212', 'arquivo', True], ['a2212', 'motor', True], ['a2212', 'aleatorio', True], ['a2212', 'arquivo', False]]`.

**Supervisionado, não one-class.** As pipelines são classificadores supervisionados com rótulo de estado (IDMT good/broken/heavyload; A2212 Healthy/Propeller/Bearing) no treino de cada dobra. As duas exceções são p30_anomalia e p30b_ae, treinadas só no normal (rótulos usados só na avaliação). Normalização, protótipos e pesos são ajustados só no treino de cada dobra. Se o manuscrito fala em "anomaly detection" one-class para a pipeline selecionada, isso não corresponde ao que foi feito.

## 2. Comparação das 20 pipelines

**Lista.** As 20 configurações, a origem (artigo de referência ou Decifrae), o que muda em cada uma e as suposições estão em `docs/PIPELINES.md` (uma linha por pipeline) e no cabeçalho de cada arquivo em `pipelines`. Nenhuma ficou como "reprodução": todas dependem de parâmetros que os artigos omitem.

**Métrica:** acurácia por janela (fração de janelas com classe prevista = rótulo), agregada sobre as dobras de validação; `n` = número de janelas avaliadas. Plataforma: PC (Python). Fontes: `out/experiments/20261014_185859_validacao.csv`, `out/experiments/20261014_203110_validacao_cnn.csv`; tabela histórica equivalente em `artigo-ICIT/RESULTADOS_tabelas.md` T1 (gerada por `experimentos/tabelas_resultados.py`). Validação, não teste.

| pipeline | tipo | idmt blocos | n idmt blocos | idmt aleatorio | n idmt aleatorio | a2212 arquivo | n a2212 arquivo | a2212 motor | n a2212 motor | a2212 aleatorio | n a2212 aleatorio |
|---|---|---|---|---|---|---|---|---|---|---|---|
| b00_nivel | baseline | 0.991 | 345 | 0.992 | 357 | 0.728 | 430 | 0.693 | 430 | 0.730 | 430 |
| b01_forma_espectral | baseline | 1.000 | 345 | 1.000 | 357 | 0.935 | 430 | 0.723 | 430 | 0.998 | 430 |
| p00_logmel_cnn | inspirada | 1.000 | 345 | 1.000 | 357 | 0.965 | 430 | 0.507 | 430 | 0.998 | 430 |
| p01_mfcc_cnn1d | inspirada | 1.000 | 345 | 1.000 | 357 | 0.874 | 430 | 0.542 | 430 | 0.891 | 430 |
| p02_grollmisch | inspirada | 1.000 | 345 | 1.000 | 357 | 0.947 | 430 | 0.730 | 430 | 1.000 | 430 |
| p03_etnext | inspirada | 1.000 | 345 | 1.000 | 357 | 0.905 | 430 | 0.381 | 430 | 0.981 | 430 |
| p03b_etnext_knn | inspirada | 1.000 | 345 | 1.000 | 357 | 0.770 | 430 | 0.379 | 430 | 0.953 | 430 |
| p04_classicas | inspirada | 1.000 | 345 | 1.000 | 357 | 0.979 | 430 | 0.777 | 430 | 0.998 | 430 |
| p04b_msaf | inspirada | 0.875 | 345 | 0.894 | 357 | 0.344 | 430 | 0.326 | 430 | 0.374 | 430 |
| p04c_classicas_lr | inspirada | 0.997 | 345 | 1.000 | 357 | 0.974 | 430 | 0.749 | 430 | 0.995 | 430 |
| p05_melcnn | inspirada | 1.000 | 345 | 1.000 | 357 | 0.977 | 430 | 0.495 | 430 | 1.000 | 430 |
| p10_decifrae_pyin_viterbi | decifrae | 1.000 | 345 | 1.000 | 357 | 0.744 | 430 | 0.528 | 430 | 0.793 | 430 |
| p11_decifrae_harmsum | decifrae | 1.000 | 345 | 1.000 | 357 | 0.751 | 430 | 0.528 | 430 | 0.788 | 430 |
| p11a_grade10 | decifrae | 0.997 | 345 | 1.000 | 357 | 0.770 | 430 | 0.560 | 430 | 0.798 | 430 |
| p12_decifrae_sem_viterbi | decifrae | 0.997 | 345 | 0.997 | 357 | 0.728 | 430 | 0.533 | 430 | 0.793 | 430 |
| p20_hfs | inspirada | 1.000 | 345 | 1.000 | 357 | 0.549 | 430 | 0.386 | 430 | 0.691 | 430 |
| p20b_so_frot | inspirada | 0.988 | 345 | 0.989 | 357 | 0.500 | 430 | 0.379 | 430 | 0.612 | 430 |
| p21_envelope | inspirada | 1.000 | 345 | 1.000 | 357 | 0.940 | 430 | 0.784 | 430 | 0.993 | 430 |
| p30_anomalia | inspirada | 0.632 | 345 | 0.633 | 357 | 0.702 | 430 | 0.628 | 430 | 0.716 | 430 |
| p30b_ae | inspirada | 0.603 | 345 | 0.633 | 357 | 0.660 | 430 | 0.428 | 430 | 0.763 | 430 |

**Custo:** só **estimativa analítica** (parâmetros, RAM, flash, MFLOP/s) no campo `custo_mcu` de `out/experiments/20261014_185859_validacao_meta.json` e do meta da CNN; tabela em RESULTADOS_tabelas.md T9. **Nenhuma das 20 teve custo medido** (nem no PC por pipeline, nem no ESP32); só a p11 teve medição, ver §5.

**Seleção da p11.** A p11_decifrae_harmsum **não** foi escolhida por um critério formal de compromisso desempenho × custo. Ela é a versão embarcável da pipeline Decifrae (o tema do trabalho), definida antes da comparação. A comparação mostra que ela **perde** para CNNs e features clássicas no A2212 (relatório de decisão `artigo-ICIT/DECISAO.md`). O manuscrito não pode dizer que ela foi selecionada como o melhor compromisso.

**Diagnóstico refeito para o artigo do ESP32** (acurácia balanceada macro, IC95 por bootstrap de gravações, 4 variantes; fonte `embarcados/results/csv/diagnostico_20261023_222831.csv`, script `embarcados/src/python/diagnostico.py`; os ICs estão no CSV):

| variante | a2212 aleatorio | a2212 arquivo | a2212 motor | a2212 motor_sem_M4 | idmt aleatorio | idmt blocos |
|---|---|---|---|---|---|---|
| b00_rmspico_bruto | 0.625 | 0.608 | 0.503 | 0.754 | 0.997 | 0.993 |
| b00_rmspico_norm | 0.564 | 0.562 | 0.542 | 0.814 | 0.992 | 0.991 |
| p04_classicas | 0.998 | 0.981 | 0.613 | 0.92 | 1.0 | 1.0 |
| p11_decifrae_harmsum | 0.849 | 0.816 | 0.456 | 0.685 | 1.0 | 1.0 |

**Teste oficial do IDMT** (lido uma vez; congelamento em `embarcados/configs/experiments/congelado_teste_idmt_20261025-150739.json`, commitado antes do resultado; script `embarcados/src/python/teste_idmt.py`; resultado `embarcados/results/csv/teste_idmt_20261025-150749.csv`; janelas em `embarcados/results/predictions/teste_idmt_janelas_*.csv`; curvas ROC/PR em `results/csv/teste_idmt_curvas_*.csv`). Treino em todo o `train/` (pure), teste em todas as janelas de 3 s do `test/`. Acurácia balanceada agregada (IC95 por bootstrap de gravações):

| variante | valor | ic95_lo | ic95_hi | n_janelas | n_gravacoes |
|---|---|---|---|---|---|
| b00_rmspico_norm | 0.476 | 0.349 | 0.606 | 2021 | 27 |
| b00_rmspico_bruto | 0.5 | 0.36 | 0.638 | 2021 | 27 |
| p04_classicas | 0.704 | 0.615 | 0.789 | 2021 | 27 |
| p11_decifrae_harmsum | 0.595 | 0.471 | 0.714 | 2021 | 27 |

AUC macro um-contra-o-resto:

| variante | valor |
|---|---|
| b00_rmspico_norm | 0.728 |
| b00_rmspico_bruto | 0.753 |
| p04_classicas | 0.954 |
| p11_decifrae_harmsum | 0.869 |

Por condição (acurácia balanceada; cada condição tem 3 gravações, uma por classe):

| condicao | b00_rmspico_bruto | b00_rmspico_norm | p04_classicas | p11_decifrae_harmsum |
|---|---|---|---|---|
| _todas | 0.5 | 0.476 | 0.704 | 0.595 |
| atmo_high | 0.333 | 0.343 | 0.486 | 0.381 |
| atmo_low | 0.713 | 0.465 | 0.986 | 0.828 |
| atmo_medium | 0.333 | 0.338 | 0.658 | 0.514 |
| stresstest | 0.463 | 0.382 | 0.626 | 0.602 |
| talking_1 | 0.333 | 0.352 | 0.648 | 0.431 |
| talking_2 | 0.333 | 0.333 | 0.629 | 0.414 |
| talking_3 | 0.357 | 0.338 | 0.643 | 0.524 |
| talking_4 | 0.333 | 0.333 | 0.619 | 0.419 |
| whitenoise_low | 0.93 | 0.94 | 0.869 | 0.954 |

Classe = motor também no teste; o resultado mede transferência entre sessões e ruído dos mesmos três motores, não reconhecimento de estado.

Baselines realmente executadas: b00_nivel (RMS + pico, normalizado e bruto) e b01_forma_espectral. Ablações executadas: p11a (grade 10 cents), p12 (sem Viterbi de rotação), p20b (só rotação), HMM × argmax ("p13", em `experimentos/sequencias.py`).

## 3. Configuração exata da pipeline selecionada (p11_decifrae_harmsum)

**Não há CQT.** A p11 usa STFT (n_fft 8192, hop 1024, Hann) e uma soma harmônica numa grade logarítmica de frequência; não existe CQT em nenhuma etapa. Ordem real (contrato em `embarcados/reports/CONTRATO_INTERFACES.md`, mapeamento Python → C em `embarcados/reports/c_port.md` §2):

1. Pré-processamento: mono, reamostragem para 16000 Hz (só o arquivo A2212 a 44,1 kHz precisa), normalização RMS para 0.1; janelas de 3 s (IDMT) ou 1 s (A2212).
2. STFT: n_fft 8192, hop 1024, Hann simétrico, |X|.
3. Soma harmônica de 48 harmônicos numa grade de 15.0–60.0 Hz com passo de 2.0 cents (1201 pontos); emissão −log(0,45·e_norm + 1e-3) mais estado de silêncio.
4. Viterbi de rotação: salto máximo 50 pontos, custo de passo 0.55, salto de oitava ±600 pontos com custo 7.0, custo de silêncio 3.2 → rotação f_rot.
5. Envelope de modulação: Hilbert sem padding, Butterworth ordem 4 a 200 Hz em SOS (`sosfilt`, causal), decimação ×16 para 1000 Hz, Welch.
6. 13 features (rotação, saliência, razões harmônicas, ganhos por grupo de harmônicos, f_mod e proeminência, centroide, planura); tabela em `embarcados/tables/latex/parametros_features.tex`.
7. Decisão por cosseno: z-score com média/desvio do treino, cosseno com o protótipo (média dos z do treino) de cada classe, softmax com temperatura T = 0,1.
8. HMM de estados (opcional, por sequência): Viterbi em log com `change_penalty` fixo (não há matriz de transição treinada), filtro online e absorção de trechos curtos; parâmetros cp = 2, n_logprob = −50, temp = 1 nos experimentos de fluxo (`embarcados/src/python/seq_pc.py`).

Constantes do filtro geradas a partir do scipy da referência em `embarcados/src/c/p11_constantes.h`. Código de referência: `pipelines/p11_decifrae_harmsum.py`, `pipelines/_decifrae.py`, `motorsound/rotacao.py`, `motorsound/classificador.py`, `motorsound/hmm.py`.

Bibliotecas: Python 3.12.13, numpy 2.5.4 (meta `embarcados/results/csv/bench_pc_20261024-091911_meta.json`); scipy na versão registrada no cabeçalho de `embarcados/src/c/p11_constantes.h`. Precisão: referência em float32/float64 como no numpy; C em dois builds, `float` (o do MCU) e `double` (só PC).

## 4. Consistência entre Python e C

**Objetivo: equivalência dentro de tolerância, por etapa**, não equivalência exata. As tolerâncias foram congeladas em `configs/embedded/tolerancias.json` antes da comparação formal, mas **depois** de o desenvolvedor do C ter depurado o porte contra os mesmos vetores de ouro (cronologia na auditoria `embarcados/reports/independent_scientific_audit.md`, M2).

Entradas idênticas: o Python de referência e o C recebem o mesmo `audio.f32` de cada caso do ouro (gerador `embarcados/src/python/gerar_ouro.py`, relatório `embarcados/reports/gold_vectors.md`). Comparador: `embarcados/tests/equivalence/comparar.py`.

Resultados por etapa, com maior erro e razão erro/tolerância: `embarcados/reports/c_equivalence_resultados_pc.md` (gerado de CSV), CSVs:

- `double`: `embarcados/results/csv/equivalencia_double_20261024-091106_resumo.csv` — itens PASS 4720, PASS_EMPATE 0, FAIL 0, em 39 linhas de etapa.
- `float`: `embarcados/results/csv/equivalencia_float_20261024-091122_resumo.csv` — itens PASS 4714, PASS_EMPATE 6, FAIL 2, em 39 linhas de etapa.
- `float-sosd`: `embarcados/results/csv/equivalencia_float-sosd_20261024-091222_resumo.csv` — itens PASS 4720, PASS_EMPATE 0, FAIL 0, em 39 linhas de etapa.
- variante `p11_c_lowmem` (float, só janelas de 1 s): `embarcados/results/csv/equivalencia_float-lowmem_20261025-135553_resumo.csv` — PASS 1422, FAIL 0.
- **no ESP32 físico**, `p11_c_lowmem`: `embarcados/results/csv/equivalencia_esp32_esp32_p11_c_lowmem_20261025-141829_resumo.csv` — FAIL 0; decisão 79/79 casos; razão erro/tolerância máx. 0.187. Só as saídas finais são comparadas (o firmware não devolve as etapas intermediárias, que aparecem como ausentes).
- teste fora da amostra (24 janelas fora do ouro, PC): `embarcados/results/csv/aud_holdout_20261025-140558_resumo.csv`.

Divergências conhecidas: nos 2 casos sintéticos de borda DC + ruído do build float, o pico do espectro de modulação está no piso de arredondamento float32 nas duas implementações; f_mod difere e o z-score falha; **a decisão de classe coincide**. A auditoria (M1) pede para contar isso como FAIL de borda, não como empate. Concordância de decisão: etapa `decisao` no relatório acima (casos do ouro).

Fluxo contínuo (Python × C no PC, filtro online do HMM):
- `embarcados/results/csv/seq_pc_20261024-092103_meta.json`: base idmt, decisões iguais Python × C = True, max |Δp| = 3.17e-06, detecções [{'troca_em': 8, 'de': 0, 'para': 1, 'atraso_janelas': 0}, {'troca_em': 16, 'de': 1, 'para': 2, 'atraso_janelas': 0}].
- `embarcados/results/csv/seq_pc_20261024-184829_meta.json`: base a2212, decisões iguais Python × C = True, max |Δp| = 2.68e-05, detecções [{'troca_em': 10, 'de': 0, 'para': 1, 'atraso_janelas': 0}, {'troca_em': 20, 'de': 1, 'para': 2, 'atraso_janelas': 0}].

## 5. Execução no ESP32 físico

Manifesto: `embarcados/results/manifests/bancada.json` (lido do chip por `esp_chip_info` pelo firmware; sessões anteriores em `results/manifests/bancada_*.json`).

- Chip: **ESP32** clássico (revisão 301, 2 núcleos), CPU 240.0 MHz, flash 4.0 MB, PSRAM 0.0 MB. **Não é ESP32-S3.** O esptool identificou o chip como ESP32-D0WD-V3 durante a gravação (saída de console, não registrada em arquivo).
- Placa portadora: placa de desenvolvimento com conversor USB-serial CH9102 e botões BOOT/EN (ou seja, **é uma placa estilo DevKit**). Fabricante e modelo exatos da placa: **não registrados**.
- SDK: ESP-IDF v5.5.6; compilador do IDF (xtensa-esp32-elf-gcc da toolchain do IDF v5.5.6); flags do núcleo `-O2 -ffp-contract=off P11_REAL=float P11_LOWMEM`; variante `p11_c_lowmem`; compilado em Oct 25 2026 01:36:51. Projeto e configuração: `embarcados/firmware` (`sdkconfig.defaults`, `main/CMakeLists.txt`). Wi-Fi e BT desligados.
- Execução: processor-in-the-loop. O áudio chega pela serial (ou pela flash); o tempo é medido com `esp_timer_get_time()` só em torno do processamento, sem aquisição.

**Memória, medida no chip.** Variante de referência `p11_c_ref` (`embarcados/results/csv/esp32_memoria_esp32_20261024-182937.csv`): o workspace não cabe —
  - a2212, janela de 16000 amostras: precisa 958480 B contíguos, maior bloco livre 110592 B, PSRAM 0 B.
  - idmt, janela de 48000 amostras: precisa 1863984 B contíguos, maior bloco livre 110592 B, PSRAM 0 B.

Variante `p11_c_lowmem` (pool de blocos, só janelas de 1 s): pool disponível no chip 245376 B de DRAM + 69568 B de IRAM, blocos [[69568, 1], [114624, 0], [110528, 0], [14272, 0], [5952, 0]]; heap livre após o pool 7048 B. Tentativas registradas (as primeiras falharam na alocação ou no acesso à IRAM, até as correções do firmware):
  - `embarcados/results/csv/esp32_memoria_esp32_20261024-182937.csv`: a2212: coube=0; idmt: coube=0
  - `embarcados/results/csv/esp32_memoria_esp32_20261024-213702.csv`: a2212: coube=0; idmt: coube=0
  - `embarcados/results/csv/esp32_memoria_esp32_20261024-223440.csv`: a2212: coube=0; idmt: coube=0
  - `embarcados/results/csv/esp32_memoria_esp32_20261025-012527.csv`: a2212: coube=0; idmt: coube=0
  - `embarcados/results/csv/esp32_memoria_esp32_20261025-013338.csv`: a2212: coube=0; idmt: coube=0
  - `embarcados/results/csv/esp32_memoria_esp32_20261025-135427.csv`: a2212: coube=0; idmt: coube=0
  - `embarcados/results/csv/esp32_memoria_esp32_20261025-141018.csv`: a2212: coube=1, ws=303308 B, heap mín. 7584 B; idmt: coube=0

**Tamanho do firmware (build, `idf.py size`, não é medição em execução):** esp32: `embarcados/results/firmware/size_esp32_20261024-101544.txt`, esp32_lowmem: `embarcados/results/firmware/size_esp32_lowmem_20261025-135632.txt`.

**Tempo de processamento, medido no ESP32** (`embarcados/results/csv/esp32_lat_esp32_20261025-143026.csv`; variante `p11_c_lowmem`, janelas de 1 s do A2212, aquecimento excluído; `esp_timer_get_time()` só em torno do processamento, sem aquisição nem transmissão), ms por janela:

| n | min | media | mediana | p95 | max |
|---|---|---|---|---|---|
| 200.0 | 2647.4 | 2647.5 | 2647.5 | 2647.6 | 2647.7 |

Mediana por etapa (ms): prep 1.8, rastreio 2345.5, viterbi 90.7, sal_caminho 0.0, envelope 199.3, features 10.1, cosseno 0.0, init 0.0.
Fator de tempo real (mediana do total / 1000 ms): 2.65 — **não é tempo real** nesta placa. A janela de 3 s do IDMT não cabe.

**Fluxo contínuo, medido no ESP32** (`embarcados/results/csv/esp32_seq_esp32_20261025-143251.csv`): 30 janelas lidas da flash, detecções [{'troca_em': 10, 'de': 0, 'para': 1, 'atraso_janelas': 0}, {'troca_em': 20, 'de': 1, 'para': 2, 'atraso_janelas': 0}]; decisões iguais às do Python (`embarcados/results/csv/seq_pc_20261024-184829.csv`): 30/30.

**Energia: não medida** (não há sensor de corrente). O manuscrito não pode afirmar tempo real nem baixo consumo no ESP32.

Para comparação, **medição no PC** (não ESP32) por janela, ms, sem aquecimento (`embarcados/results/csv/bench_pc_20261024-091911.csv`; máquina e compilador no meta; o C roda num processo novo por janela e `total_sem_init_ms` exclui a inicialização — ver auditoria m5):

| impl | base | n | min | media | mediana | p95 | max |
|---|---|---|---|---|---|---|---|
| c_float_pc | a2212 | 200 | 2.736 | 3.222 | 3.176 | 3.839 | 4.452 |
| c_float_pc | idmt | 200 | 11.264 | 13.337 | 13.29 | 14.997 | 15.799 |
| python_ref | a2212 | 200 | 20.905 | 22.125 | 22.057 | 23.094 | 25.338 |
| python_ref | idmt | 200 | 90.513 | 95.669 | 95.322 | 100.168 | 110.219 |

## 6. Tabelas, gráficos e arquivos de comprovação

Já existem (dados + script de origem no manifesto `embarcados/results/manifests/manifesto.json`, gerados por `embarcados/src/python/gerar_artigo_tudo.py`):

- `embarcados/figures/png/arquitetura.png`
- `embarcados/figures/png/diag_balacc_20261023_222831.png`
- `embarcados/figures/png/diag_balacc_en_20261023_222831.png`
- `embarcados/figures/png/diag_confusao_p11_20261023_222831.png`
- `embarcados/figures/png/diag_confusao_p11_en_20261023_222831.png`
- `embarcados/figures/png/diag_espectro_20261023_222831.png`
- `embarcados/figures/png/diag_espectro_en_20261023_222831.png`
- `embarcados/figures/png/equivalencia_erro_etapas.png`
- `embarcados/figures/png/equivalencia_erro_etapas_en.png`
- `embarcados/figures/png/esp32_lat_ram_esp32_20261025-143026_en.png`
- `embarcados/figures/png/teste_idmt_condicoes_en.png`
- `embarcados/figures/png/teste_idmt_roc_en.png`
- `embarcados/tables/latex/bases_protocolos.tex`
- `embarcados/tables/latex/desempenho.tex`
- `embarcados/tables/latex/desempenho_compacto.tex`
- `embarcados/tables/latex/equivalencia.tex`
- `embarcados/tables/latex/equivalencia_compacta.tex`
- `embarcados/tables/latex/parametros_features.tex`
- `embarcados/tables/latex/recursos.tex`
- `out/figuras/pareto_custo.png` (desempenho × custo **estimado**, fase exploratória)

Situação dos itens pedidos:

- Tabela comparativa das 20 pipelines: **pronta** (§2 e RESULTADOS_tabelas.md T1), em validação.
- Desempenho × custo: **pronta só com custo estimado** (pareto_custo.png); com custo medido é inviável (§5).
- Matriz de confusão: **pronta em validação** para a p11 (diag_confusao_p11_*.png; contagens `cm_pred_*` no CSV de diagnóstico).
- ROC/PR **no teste oficial do IDMT**: **pronto** (pontos em `results/csv/teste_idmt_curvas_*.csv`, ROC em `figures/png/teste_idmt_roc_en.png`; PR só em CSV). O A2212 não tem conjunto de teste separado, então lá só há AUC de validação (`roc_auc_ovr_macro` no CSV de diagnóstico).
- Matriz de confusão **no teste**: **pronta** (`cm_pred_*` em `results/csv/teste_idmt_2*.csv`).
- Python × C com linha de identidade e distribuição dos erros: **parcial**; existe equivalencia_erro_etapas.png (erro por etapa contra a tolerância). Um gráfico de dispersão Python × C não foi feito, mas os dados existem (`gold/` + `results/c_out/`, não versionados por tamanho).
- Distribuição dos tempos no ESP32: **dados prontos** (§5, CSV por janela); a figura do artigo mostra a mediana por etapa (`figures/png/esp32_lat_ram_*`), não a distribuição (a variação entre janelas é < 1 ms).

Código: Python de referência em `pipelines/`, `motorsound/`; C em `embarcados/src/c/`; firmware em `embarcados/firmware/`; testes em `embarcados/tests/`. Manifesto do ouro (partições e casos): `embarcados/gold/manifest.json` (não versionado; gerado por `gerar_ouro.py`). Registros de compilação: `results/firmware/size_*.txt`; testes unitários: `tests/unit/test_p11.py`.

