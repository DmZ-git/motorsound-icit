# Auditoria científica independente (Agente F, Etapa 8)

Agente F, 24/10/2026, branch `embarcado-icit`, HEAD `aba67c5`. Nenhum veredito abaixo se apoia no
que outro agente ou relatório disse: os números foram recalculados, os scripts foram rodados de
novo em cópias que gravam no rascunho do auditor
(`C:\Users\erick\AppData\Local\Temp\claude\c--Users-erick-dev-Decifrae-Artigo-Artigo3-MotorSound-\c30485f2-9096-4647-ba53-a61d9633dbba\scratchpad\auditoria_F\`,
daqui em diante `F\`) e o código foi lido. Nada foi escrito no repositório além deste arquivo.
O `test/` do IDMT não foi lido e nenhum ESP32 foi aberto.

## Vereditos

| portão | veredito | condição |
|---|---|---|
| A · bases e splits | **CONDITIONAL PASS** | declarar a confusão (IDMT classe = motor = gravação; A2212 Bearing = M4) na legenda da tabela de desempenho e da Fig. `balacc`, nas duas versões (regular e WiP) |
| B · Python reproduzível | **PASS** | — |
| C · C completo, compilando, testes, equivalência | **CONDITIONAL PASS** | reescrever o "empate" do float32 (não é empate; ver M1) e declarar no `main.tex` a cronologia porte → tolerâncias (M2); o resultado vale **só no PC** |
| D · viabilidade embarcada medida no chip | **FAIL** (bloqueado, não reprovado) | não há nenhuma medição no chip; lista exata do que falta em §D |
| E · rastreabilidade | **CONDITIONAL PASS** | todos os números conferidos batem com a fonte; corrigir as afirmações interpretativas M1, M3, M4, M5 e as menores m1–m6 |

Recomendação: **WiP (4 pp)**, a menos que a bancada no chip (§D) rode e passe antes do prazo. Pela
regra do PROMPT, regular exige C e D aprovados; D não tem medição.

## Problemas críticos

Nenhum. Não encontrei vazamento não declarado, acesso ao teste, número fabricado nem divergência
entre texto e fonte. Os problemas abaixo são de interpretação e de redação, e todos têm correção
barata.

## Problemas maiores

**M1 · O "empate" do float32 não é empate.** O texto diz "Welch spectrum has a near-tie"
(`paper/main.tex:44`), "two candidate peaks whose reference powers differ by 9,4e-16"
(`:296-299`), "a numerical tie" (`:479`) e "PASS only through a demonstrated tie" (`:313`).
Rodei o comparador no meu build limpo e li os vetores (`F\f_empate.py`). No caso
`borda_dc_ruido3s_midmt`, na banda de 2 a 100 Hz, a referência tem P[52] = 1,25e-15 e P[90] = 3,09e-16,
ou seja, o pico da referência é **4×** o bin que o C escolhe. O C float tem P ≈ 5e-14 nesses bins,
40× acima da referência (erro relativo ao máximo da banda: 45). Os dois lados estão no piso de
arredondamento float32. A regra `regra_fmod` (`tests/equivalence/comparar.py:309-321`) aceita o
empate se `P_ref[ref] − P_ref[C] ≤ 2·tw`, com `tw` absoluto (= rtol_max de borda 3e-3 × máximo
global de welch_P, 2·tw = 6,3e-11). Isso fica cinco ordens de grandeza acima de toda a banda, então
qualquer bin passaria como "empate". O resultado verdadeiro é que, nesse sinal mal condicionado,
f_mod e a proeminência são **indeterminados na resolução float32**, para a referência e para o C.
A decisão de classe coincide (p = 0,912 contra 0,896). Com isso, a frase "25 de 26 etapas" só vale
contando `env_mod` e `features` como PASS pela exceção. Sem ela, as etapas `env_mod`, `features` e
`z` falham nos 2 casos DC. Correção: trocar "near-tie / demonstrated tie / numerical tie" por algo
como "the modulation spectrum of this synthetic DC-plus-noise input lies at the float32 noise floor
(about 1e-7 of the stage maximum) in both implementations, so the peak location is undetermined;
f_mod differs by 37.1 Hz, z fails, the decision is unchanged". No comparador, chamar o status de
`INDETERMINADO_PISO` (ou contá-lo como FAIL de borda) em vez de `PASS_EMPATE`. O mesmo vale para
`reports/c_equivalence_resultados_pc.md` e `paper/RESULTADOS.md`.

**M2 · O "pré-registro" das tolerâncias é mais fraco do que o `main.tex` diz.** Cronologia
reconstruída de `git log` e dos mtimes:
- vetores de ouro em `gold/casos` às 22:34 de 23/10 (commit `a547593` às 22:36);
- fontes C entre 22:28 e 22:42, `build-float/p11_host.exe` e `build-double/p11_host.exe` às 22:42:53–55;
- CSVs do estudo de tolerâncias (só emulação) às 22:28–22:36, e `tolerancias.json` gravado pela
  última vez às **22:46:35**, quatro minutos depois do executável C final;
- commits: porte `9e8e472` às 09:07:38 de 24/10, tolerâncias `447a7c2` às 09:10:44, comparação
  formal às 09:11 (`equivalencia_float_20261024-091122.csv`).

O agente C depurou o C contra o mesmo ouro (`tests/unit/depurar_gold.py`, `reports/c_port.md` §7:
"vit_path e a classe prevista coincidem com o ouro nos 164 casos") antes de as tolerâncias serem
congeladas. A frase "Tolerâncias congeladas … antes de qualquer saída do C existir"
(`reports/c_equivalence_resultados_pc.md:3`; `c_equivalence.md:5`) é falsa como está escrita. O
`paper/METODOLOGIA.md:93-96` já faz a ressalva certa; o `main.tex` (`:258-262`, `:307`, `:500`)
não faz. Não há sinal de tolerância ajustada ao C. `emular_build.py` e `estudo_tolerancias.py` não
leem `c_out` nem chamam `p11_host`. A maior razão erro/tolerância nos PASS do C é 0,55 (double) e
0,34 (float). O FAIL em z ficou visível. Para separar a questão, fiz um **teste fora da amostra**
(`F\f_holdout.py`): 24 janelas sorteadas fora do ouro (12 IDMT `train/`, 12 A2212), que o agente C
nunca viu. Resultado com as tolerâncias congeladas: caminho de Viterbi idêntico 24/24, decisão
24/24, nenhuma feature fora da tolerância (razão máx. 0,20 float, 0,05 double), |Δp| ≤ 8,7e-6.
Correção: no `main.tex`, escrever "committed before the formal comparison; the C port had already
been debugged against the same golden vectors by its developer", citar este teste fora da
amostra, e trocar "pre-registered" por "frozen before the formal comparison".

**M3 · A queda "motor fora" no Discussion é quase toda por construção.** `main.tex:470-473` diz
que tirar um motor "cost the classical features 0.368 and the pipeline 0.360", contra a inflação de
0,017–0,033 do split aleatório, e usa isso para medir o confundimento por motor. Mas no protocolo
`motor` a recall de Bearing é 0 por construção, o que limita a acurácia balanceada a ≈ 2/3:
0,613 = (2 × 0,920)/3. Comparação igual por igual (`F\`, janelas Healthy/Propeller de M1–M3, os mesmos
modelos): clássicas 0,972 (arquivo) → 0,920 (motor), queda de 0,052; p11 0,753 → 0,685, queda de 0,068;
RMS normalizado 0,843 → 0,814. O efeito do motor existe, mas tem o tamanho da inflação do split
aleatório, não 10×. Correção: reescrever a frase com as quedas igual por igual (ou com
`A2212 w/o M4` contra o arquivo restrito a H/P) e dizer que a queda de 0,36 é dominada pela classe
Bearing, que fica sem treino.

**M4 · A frase da rotação no A2212 contradiz os próprios números.** `main.tex:429-433` diz "the
tracked rotation does not follow the speed label", com Spearman 0,02 em 43 arquivos. Recalculei de
`p11_features_20261023_222831.csv`: global 0,018, mas por motor 0,303 (M1, 14 arquivos), 0,302 (M2, 16),
0,543 (M3, 6) e **1,000 (M4, 7)**. A correlação global perto de zero vem de diferenças de nível entre
motores (efeito de Simpson), não da falta de relação dentro do motor. Correção: "pooled over motors
the rank correlation is 0.02, within motors it is positive (0.30–1.00, 6–16 files each); the
absolute value is not comparable across motors and 3.7 % of windows sit at the search ceiling, so it
should not be read as shaft speed".

**M5 · "falls below the level baseline" sem ressalva de IC.** A frase em `main.tex:410-411` é
verdadeira nos pontos: motor fora, p11 0,456 contra RMS normalizado 0,542 e bruto 0,503; sem M4,
0,685 contra 0,814 e 0,754. Os ICs se sobrepõem nos dois casos (p11 sem M4 [0,593, 0,774] contra
[0,720, 0,887] e [0,623, 0,873]), e no protocolo `motor` o IC é por arquivo, otimista, como o próprio
texto diz. A parte "never outperforms the classical features" está correta (no IDMT empata em 1,000).
Correção: "its point estimate also falls below the level baseline (overlapping intervals)".

## Problemas menores

- **m1** · Legendas de `tables/latex/desempenho_compacto.tex`, `desempenho.tex` e da Fig. `balacc`
  não declaram Bearing = M4 nem IDMT classe = motor. Isso viola a regra 3 de dados do `CLAUDE.md`
  da raiz. Na versão WiP, a tabela `bases_protocolos` (que declara) sai do documento (`\ifwip\else`).
- **m2** · `main.tex:208-212` (regular) e `:196-197` (WiP): "the only approximation is the named
  variant p11_c_sosd". `sosd` não é aproximação: é mais precisa e mais próxima do scipy, que promove
  para float64. Além disso existe a variante `p11_c_espdsp` (commit `9df3b2c`,
  `firmware/main/CMakeLists.txt`), que troca a FFT e **nunca foi comparada** com a referência. Se for
  medida no chip, precisa entrar no texto e passar pelo comparador.
- **m3** · As tolerâncias de borda são 3× o erro medido pela emulação nos mesmos sinais
  (`c_equivalence.md` §2.8) e algumas são largas: prom 3,5 dB e grupos 0,15 dB no float. O relatório
  de D declara isso; o `main.tex` não. Uma frase basta.
- **m4** · A razão 5,5× (`main.tex:343-346`) divide a arena (1 863 984 B, estimativa, no heap em
  tempo de execução) pela região DIRAM **total** do `idf.py size` (341 760 B, que já inclui 39,5 KB de
  código e 16 KB de dados). Em bytes a conta está certa (5,45; 958 480/341 760 = 2,80). Mas mistura MB
  decimal com KiB no texto e compara com o total, não com o heap livre (≤ 286 253 B restantes). Escrever
  "at least 5.5×" e usar uma unidade só. O texto também omite os 16 KiB de IRAM (100 % usados) do S3.
- **m5** · Bancada PC (`src/python/bench_pc.py:74-83`): o C roda num processo novo por janela com
  `n_rep = 1`, então o "aquecimento" não aquece o C. O total do C exclui `ws_init`, que o Python não
  tem como excluir. As duas coisas se compensam em parte; o texto deve dizer isso e que são 75 janelas
  distintas em ciclo (a tabela `recursos.tex` diz "200 timed windows").
- **m6** · Textos datados: "An independent audit of the comparison is also pending"
  (`main.tex:305-306`) e "have not yet been audited independently" (`:493-494`), e o mesmo em
  `RESULTADOS.md:26`. Atualizar para citar esta auditoria e as condições dela.
- **m7** · Octave errors: o limiar 1,5× está certo, porque todas as janelas good acima dele estão a
  2,00–2,02× da mediana (8,6 %). Vale escrever "(all at 2.0×)". "within 1 % of the upper limit"
  (f0 ≥ 0,99·60 Hz) confere: 3,7 % de 430.
- **m8** · `manifesto.json`: a tabela `bases_protocolos` lista a fonte duplicada
  `embarcados/paper/numeros.json.` (com ponto final). É cosmético.
- **m9** · No protocolo `motor`, a AUC junta scores de modelos de folds diferentes e, no fold M4,
  p_Bearing = 0 é imposto. É a prática comum, mas vale uma nota.
- **m10** · A grade de 2 cents e a faixa de 15–60 Hz foram escolhidas olhando o `train/` do IDMT
  (`motorsound/rotacao.py:168-173`). O texto declara a faixa (`main.tex:433-434`) mas não a grade.

## A · Bases e splits — evidência

- **Folds disjuntos** (`F\f_splits.py`, sobre `results/predictions/p11_folds_20261023_222831.csv`):
  A2212 por arquivo tem interseção de arquivos treino ∩ validação 0 nos 5 folds. Motor fora tem
  interseção de motores 0 nos 4 folds. IDMT blocos: nos 5 folds, distância mínima de 2 índices entre
  janela de treino e de validação da mesma gravação (a guarda de `motorsound/splits.py:42-54`
  funciona) e 0 sobreposições temporais.
- **Ouro e modelos**: os 75 casos não-borda de cada modelo (`idmt_bloco0`, `a2212_arq0`) estão todos
  na validação do fold e nenhum no treino do modelo exportado. O treino do `.json` do ouro é idêntico
  ao treino do fold no CSV (`gerar_ouro.py:327-360`).
- **Ajuste só no treino**: `CossenoPrototipos.fit` (μ, σ, protótipos) recebe `X[f.treino]`
  (`runner.py:182`, `diagnostico.py:213`). `StandardScaler` fica dentro de `make_pipeline` em p04 e b00.
  A normalização RMS é por janela, sem estatística de treino.
- **Teste do IDMT**: `runner.segmentos_de` usa `io.gravacoes("idmt", "train")` (`runner.py:156`). O
  `io.py:37-44,79-80` levanta `TesteTrancado` sem `MOTORSOUND_ABRIR_TESTE=1`. Nenhum `rec_id` com
  "test" nos folds. Grep em `embarcados/src`, `tests` e `firmware`: só comentários de "nunca o
  test/". Nada nos metas.
- **Problemas conhecidos**: declarados no texto (`main.tex:224-229`, `:400-416`, `:460-468`) e na
  tabela `bases_protocolos`. Falta nas legendas de desempenho (m1).

## B · Python reproduzível — evidência

`F\f_metricas.py` lê só `results/predictions/diagnostico_janelas_20261023_222831.csv` e recalcula com
scikit-learn (implementação diferente da de `diagnostico.py`) a acurácia balanceada, o F1 macro e a
AUC OvR macro para as 24 combinações (4 variantes × 6 protocolos). A diferença máxima contra
`results/csv/diagnostico_20261023_222831.csv` é **2,2e-16**. O bootstrap de `diagnostico.py:143-169`
reamostra **grupos** com reposição, estratificado pela classe do grupo (`rng.choice(gs, size=len(gs))`
sobre ids de grupo). Com um bootstrap meu, por grupo e com outra semente, a p11 no A2212 por arquivo
deu [0,750, 0,881] contra [0,747, 0,881] do CSV, e sem M4 [0,595, 0,770] contra [0,593, 0,774]. Para
comparar, reamostrando janelas o IC seria [0,783, 0,848], estreito demais. A referência não mudou
entre o commit do experimento (`9081500`, ancestral de HEAD) e HEAD (`git diff 9081500 HEAD --
motorsound pipelines` vazio).

## C · Porte C e equivalência — evidência

1. **Build**: `build_host.bat` termina com "BUILD OK", mas os executáveis estavam atualizados
   (mtime 23/10 22:42). Por isso fiz um build limpo em `F\build-{float,double}`
   (`F\build_limpo.bat`, MSVC 14.44, mesmo CMakeLists). Os três alvos compilam nos dois builds sem
   aviso de compilador (só avisos de comprimento de caminho do CMake).
2. **Testes**: `pytest embarcados/tests/unit embarcados/tests/equivalence/test_comparador.py` dá
   **37 passed** em 8,6 s.
3. **Comparador** (`F\f_comparar.py`: `gerar_saidas` com o meu build e `rodar(..., saida_csv=F\csv)`):
   double tem **6 852/6 852 PASS**; float tem 6 844 PASS, 6 PASS_EMPATE e **2 FAIL** (z em
   `borda_dc_ruido3s_midmt` e `_ma2212`). Isso reproduz exatamente os CSVs do D (sha256 das tolerâncias
   `d74dae800ceb…` nas linhas).
4. **(i) hash**: `git show 447a7c2:embarcados/configs/embedded/tolerancias.json | sha256sum` e o
   arquivo atual dão os dois `d74dae800cebfe4d670c33ba2c0a72639ad1e04e95bcbd9a904be13c294c1b5d`. Há um
   único commit no arquivo, que não mudou depois.
5. **(ii) cronologia**: ver M2. Compromete a força da palavra "pré-registro", não o resultado. O teste
   fora da amostra (24/24) mostra que a concordância não depende de o C ter sido depurado nesses
   vetores.
6. **(iii) bordas**: tolerâncias = 3× o erro da **emulação** (não do C) nos mesmos sinais. A
   evidência é mais fraca e o relatório de D declara isso; faltam no artigo (m3).
7. **(iv)**: ver M1. Os 2 FAIL em z são reais e o "empate" não é empate; a decisão é a mesma.
8. **(v) checagem pontual sem o comparador** (`F\f_pontual.py`): 10 casos (4 normais, entre limpos e
   ruidosos, e 6 de borda: clip, silêncio, 8 Hz, DC), chamando `p11_host caso` diretamente e comparando
   com `_d.extrair(_preparar(audio))`, `rastrear` e um cosseno montado com os modelos de
   `p11_modelos_*.json` (não com o `.bin` do ouro). Resultado: caminho idêntico em 10/10 nos dois
   builds e decisão idêntica em 10/10. Nos normais, a feature relativa máxima é 3,7e-5 (float) e
   2,9e-7 (double), com |Δp| ≤ 3,5e-7. Nas bordas, a planura do caso de 8 Hz difere 26 % (float) e
   3,9 % (double): valor minúsculo, no piso float32 da própria referência. No caso DC, f_mod difere,
   que é o M1. Mais o teste fora da amostra do M2.
9. **(vi) leitura do C** (`src/c/p11_core.c`, `p11_decisao.c`, `p11_fft.c`):
   - porte por quadro: reorganiza a memória, a conta é a mesma, e P_mean é somado na ordem do numpy
     sobre o eixo não contíguo. **Equivalente.**
   - índice direto no lugar de `searchsorted`: f_rot = grade[path] e a grade é estritamente crescente,
     então `searchsorted(side='left')` devolve o próprio índice. **Equivalente.**
   - tabela de interpolação: é a mesma fórmula pré-calculada (`i0` em uint16 ≤ 1474, `a` em
     p11_real), com o mesmo resultado sem a tabela. **Equivalente** (é cache, não aproximação).
   - psi int16: 1 202 estados < 32 767, sem perda. **Equivalente.**
   - soma em pares: reproduz a `pairwise_sum` do numpy, ou seja, aproxima-se da referência.
     **Equivalente.**
   - Viterbi (`p11_core.c:299-390`): mesma ordem de varredura (−50…+50, +600, −600, SIL), `<` estrito,
     `<=` na coluna SIL, argmin no primeiro índice. **Fiel.**
   - FFT própria de raiz mista no lugar do pocketfft: é outra implementação da mesma transformada;
     a diferença numérica fica coberta pela tolerância. Aceitável.
   - Precisão mista no build float (índices, grade e mediana em double): **declarada** no texto; não é
     troca de algoritmo.
   - Variantes que precisam de nome: `P11_SOS_DOUBLE` (`p11_c_sosd`) já tem; `P11_USAR_ESP_DSP`
     (`p11_c_espdsp`) tem nome mas **não foi comparada** (m2).

## D · Viabilidade embarcada — evidência e o que falta

Hoje existem só as categorias 1–3. Recalculei a partir de `C:\Espressif\build_p11_*\size.txt`:
ESP32 DRAM 15 104 B = 15 KiB, IRAM 59 087 B = 58 KiB e binário 0x342a0 = 209 KiB; S3 DIRAM 55 507 B =
54 KiB de 341 760 B = 334 KiB e binário 0x3a2c0 = 233 KiB. Todos batem com `numeros.tex`. A arena
(1 863 984 B / 958 480 B) vem de `paper/fontes/ws_float_*.json`, gerado por `p11_host ws`. Não há
`results/manifests/bancada.json` nem `esp32_*`, e as 27 macros `VChip*` estão `[pending]`. **Sem
medição no chip, este portão não pode passar.**

Falta exatamente:
1. `pil_esp32.py info` → `bancada.json` (modelo do chip, PSRAM detectada e tamanho, 240 MHz confirmados).
2. `pil_esp32.py equiv` com os 164 casos, mais `comparar.py --build esp32_<alvo> --tol-build float`
   (equivalência no chip).
3. `pil_esp32.py lat --n 200 --aquecimento 5` nas duas bases (latência por etapa, RTF, heap, pilha).
4. `pil_esp32.py seq` (fluxo contínuo e atraso de detecção).
5. Se `p11_c_espdsp` for usada: comparar antes com a referência.

Revisão de `firmware/main/main.c` e `src/python/pil_esp32.py`. Nada invalida a medição, mas há riscos
a corrigir antes de medir:
- **Risco de bloqueio, ESP32-S3**: o build tem `CONFIG_SPIRAM_MODE_QUAD=1` e
  `CONFIG_SPIRAM_SPEED_40M` (`C:\Espressif\build_p11_esp32s3\config\sdkconfig.h`). Os módulos
  S3 N8R8/N16R8, os mais comuns, têm PSRAM **octal**. Com `CONFIG_SPIRAM_IGNORE_NOTFOUND=y`, a PSRAM
  não é detectada, o `heap_caps_malloc` de 1,86 MB falha e toda janela devolve `@@ERRO`. Conferir o
  módulo e, se for octal, usar `CONFIG_SPIRAM_MODE_OCT=y` e 80 MHz. A velocidade da PSRAM muda a
  latência e deve ir para `bancada.json`.
- O tempo está bem delimitado: `esp_timer_get_time()` só em volta das chamadas do núcleo
  (`main.c:166-181`), o áudio é lido para `g_audio` antes de `t0` (sem cópia dentro do tempo), e o
  modelo e a serial ficam fora. Certo.
- O total e o RTF **excluem `ws_init`** (`pil_esp32.py:203`), mas o firmware chama `p11_ws_init` em
  toda janela (`main.c:168`). A função reconstrói a tabela de 345 KB com divisões em double, que são
  emuladas em software no Xtensa. A exclusão é legítima só se o init sair do laço (uma vez por `n`) ou
  for reportado à parte no resumo; hoje `init_ms` está no CSV mas não no JSON de resumo.
- O total também exclui o passo do HMM: no `lat` ele não roda, e no `seq` é medido à parte
  (`us_hmm`). Declarar.
- O protocolo serial não tem verificação de integridade (CRC) do áudio recebido. Um byte perdido
  desalinha o `le_exato` ou trava em `portMAX_DELAY`; a equivalência pegaria um erro, mas não o
  diagnosticaria. Sugestão: devolver o CRC32 do áudio em `@@RES`. `cmd_model` aloca `nbytes` antes de
  validar.
- O heap reportado é o **mínimo livre desde o boot** (`heap_caps_get_minimum_free_size`), não o pico
  de uso da p11. Para "pico de heap", gravar o heap livre antes da primeira alocação e subtrair.
  A pilha (`uxTaskGetStackHighWaterMark`, em bytes no ESP-IDF) está certa.
- O aquecimento de 5 janelas é aplicado, mas as 200 medições ciclam 75 janelas distintas; declarar.
- `seq` usa áudio quantizado em int16 (`pil_esp32.py:254-267`) e grava o mesmo áudio desquantizado
  para a referência. Está certo, desde que a comparação PC × chip use esse arquivo e não o ouro float.
- O detector de atraso (`pil_esp32.py:275-284`) não conta trocas espúrias por minuto, uma métrica
  pedida no `CLAUDE.md` da raiz.

## E · Rastreabilidade — números reconstruídos

Cada um recalculado da fonte indicada em `numeros.tex` (scripts em `F\`):

| # | número no texto | fonte | recalculado |
|---|---|---|---|
| 1 | bal. acc. clássicas A2212 arquivo 0,981 [0,950, 0,998] | predições | 0,9812 (IC próprio coerente) |
| 2 | p11 A2212 arquivo 0,816 [0,747, 0,881] | predições | 0,8162; IC próprio [0,750, 0,881] |
| 3 | RMS bruto A2212 arquivo 0,608 | predições | 0,6084 |
| 4 | RMS norm. IDMT blocos 0,991 | predições | 0,9913 |
| 5 | clássicas motor 0,613; p11 0,456 | predições | 0,6134; 0,4565 |
| 6 | sem M4: 0,920 / 0,814 / 0,685 | predições | 0,9201 / 0,8136 / 0,6847 |
| 7 | recall Healthy p11 arquivo 0,577 | CSV diag | 0,577 |
| 8 | inflação 0,033 / 0,017; queda 0,360 / 0,368 | CSV diag | 0,8494−0,8162; 0,9985−0,9812; ok (ver M3) |
| 9 | f_rot mediana 25,76 / 27,99 / 24,65 Hz; IQR máx. 0,20 | `p11_features` | 25,76 / 27,99 / 24,65; 0,20 |
| 10 | 8,6 % good > 1,5× mediana | `p11_features` | 8,6 % (todas a 2,0×) |
| 11 | 3,7 % das janelas A2212 ≥ 0,99·60 Hz | `p11_features` | 3,7 % de 430 |
| 12 | Spearman 0,02; por motor 0,30/0,30/0,54/1,00 | features + predições | 0,018; 0,303/0,302/0,543/1,000 (ver M4) |
| 13 | C 13,29 ms (p95 15,00); Py 95,3; 7,2×; 41/40/17 % | `bench_pc_…091911.json` | 13,290; 14,997; 95,32; 7,17; 41/40/17 |
| 14 | C 3,18 ms / Py 22,1 ms (1 s) | idem | 3,176; 22,06 |
| 15 | float 25/26 etapas, z 2/164, decisão 164/164 | CSV float | 25 PASS (contando PASS_EMPATE) + 1 FAIL; 164 (ver M1) |
| 16 | env_dec mediana 1,2e-5 → 1,2e-7 (sosd) | CSVs float e sosd | 1,232e-5; 1,174e-7 |
| 17 | 9,4e-16, 6,3e-11, 37,1 Hz | CSV float | reproduzido no meu build |
| 18 | arena 1,86 / 0,96 MB; sem tabela 1,52 / 0,61 | `fontes/ws_float_*.json` | 1 863 984 / 958 480; 1 518 096 / 612 592 B |
| 19 | 5,5× e 2,8× a DIRAM de 334 KiB | `size.txt` S3 | 5,45; 2,80; 341 760 B (ver m4) |
| 20 | DRAM 15 KiB + IRAM 58 KiB (ESP32); DIRAM 54 KiB (S3); binários 209 / 233 KiB | `size.txt` | 14,75→15; 57,7→58; 54,2→54; 208,7 / 232,7 |
| 21 | seq 24 janelas, atraso 0 / 0, Δp 3,2e-6 | `seq_pc_…092103` | 24; 0/0; 3,17e-6 |
| 22 | emulações 6 852/6 852 | metas emul | idem |

Manifesto: 380 macros (353 `ok`, 27 `pending`, todas `VChip*`). O sha256 de todas as fontes das
macros e dos 18 artefatos (tabelas, figuras e textos) bate com os arquivos atuais; nenhuma fonte falta.
Nenhum número do chip aparece fora de `[pending]` (o `\ifchipmedido` está falso e as 16 macros de
chip usadas no `main.tex` ficam `[pending]`). Os logs do LaTeX dão 6 pp (regular) e 4 pp (WiP). A
versão regular já tem 6 pp sem os resultados do chip; incluí-los vai exigir cortes.

Afirmações obrigatórias no `main.tex`:
- analogia sem dizer que motor é nota (`:79-81`): ok;
- classificação ≠ predição (`:101-102`, `:468`): ok;
- motores pequenos de laboratório (`:103`): ok;
- unidade conhecida ≠ motor novo (`:103-105`, `:460-464`): ok;
- quatro categorias de custo separadas (`:93-94`, `:271-280`, macros por prefixo): ok;
- energia, microfone I2S e bancada como trabalho futuro (`:490-493`): ok;
- p11 não supera os baselines, logo contribuição estreita (abstract `:56-58`, `:503-505`): ok;
- nenhum "equivalent" sem ressalva: só "not unconditional equivalence" (`:307`).

Citações: as 17 entradas do `references.bib` têm ficha em `docs/literatura/`. As duas lidas só no
resumo (`vanaraj2026`, `fsst_viterbi2015`/`xi2015`) são citadas só pelo que o resumo sustenta, e a
primeira com "we only read the abstract". Conferi nas fichas: split por segmento em `melcnn2025`,
`ertarin2025`, `etnext2026` e `tekin2025`; split aleatório de espectrogramas e tempo de resposta em
`strantzalis2022`; ausência de classificador e faixas de velocidade por classe em `estacio2023`;
média mais próxima (NM) em `glowacz2018`; ausência de implantação em `tinyml_acoustic2025` (que usa
UrbanSound8K, não som de máquina, o que vale mencionar).

## O que precisa mudar antes da submissão

1. Reescrever as frases do "empate" (M1) em `main.tex:44`, `:296-303`, `:313`, `:324` e `:479`, em
   `RESULTADOS.md` e em `c_equivalence_resultados_pc.md`.
2. Ressalva de cronologia mais o teste fora da amostra no `main.tex`; trocar "pre-registered" (M2).
3. Corrigir as frases da queda por motor (M3), da rotação no A2212 (M4) e do "falls below" (M5).
4. Legendas de desempenho e da Fig. `balacc` com Bearing = M4 e IDMT classe = motor (m1).
5. Corrigir "only approximation" e mencionar ou excluir `p11_c_espdsp` (m2); tolerâncias de borda
   (m3); unidades da razão de RAM (m4); bancada PC (m5); remover "audit pending" (m6).
6. Para a versão regular: rodar §D no chip, depois de conferir o modo da PSRAM do S3 e de tirar
   `ws_init` do laço ou reportá-lo. Sem isso, submeter a WiP.
