# Auditoria científica independente final (Agente F)

Agente F, 25/10/2026, branch `embarcado-icit`, HEAD `7459bd0`. Fiz tudo em modo de leitura. O único
arquivo que escrevi no repositório é este. Meus scripts estão em
`C:\Users\erick\AppData\Local\Temp\claude\c--Users-erick-dev-Decifrae-Artigo-Artigo3-MotorSound-\c30485f2-9096-4647-ba53-a61d9633dbba\scratchpad\auditF2\`
(`recalc.py`, `macros.py`). Não abri o `test/` do IDMT. Li só os CSVs de resultado do teste. Não
abri o ESP32. Rodei os testes de novo: `pytest embarcados/tests/unit embarcados/tests/equivalence/test_comparador.py`
deu **37 passed**.

## Vereditos

| portão | veredito | condição exata |
|---|---|---|
| A · bases e splits | **PASS** | nenhuma. As legendas declaram classe = motor (IDMT) e Bearing = M4. A integridade do teste oficial foi verificada (§4). |
| B · Python reproduzível | **PASS** | nenhuma. As métricas do teste, recalculadas a partir das predições por janela, batem com o CSV. |
| C · C completo, testes, equivalência | **CONDITIONAL PASS** | declarar a cobertura real da equivalência no chip (MA1, MA2): só 8 das 20 etapas de janela, nenhuma das 6 de sequência no comparador, só janelas de 1 s do A2212, e só os casos de borda de silêncio e clipping |
| D · viabilidade embarcada medida no chip | **CONDITIONAL PASS** | as medições existem, vêm do chip e batem com os CSVs. A viabilidade vale só para o `p11_c_lowmem` em janelas de 1 s, a RTF 2,65 (fora de tempo real), e o IDMT de 3 s não cabe. O texto já diz isso. Falta declarar que a latência exclui o init único (m2) e tirar do resumo a causa dada como fato (m1). |
| E · rastreabilidade | **CONDITIONAL PASS** | as 131 macros usadas têm fonte e nenhuma está `pending`. Os 22 números que reconstruí batem. Faltam as correções de redação MA1, MA2 e m1–m5. |

## Problemas críticos

Nenhum. Não achei vazamento, segunda leitura do teste, ajuste depois do teste, número fabricado
nem "measured on the ESP32" sem medição no chip.

## Problemas maiores

**MA1 · A cobertura da equivalência no chip aparece maior do que é.** O texto em `main.tex:49-50`
diz "its on-chip outputs agree with the reference within the frozen tolerances on all 79 windows compared",
e `main.tex:567-568` diz "matching the reference within the frozen tolerances". Os dados mostram o seguinte
(`results/csv/equivalencia_esp32_esp32_p11_c_lowmem_20261025-141829*.csv`):
- O hash das tolerâncias é o congelado, `d74dae80…`, o mesmo de `447a7c2`. Há 1 659 PASS, 0 FAIL e razão máxima de 0,187.
- No chip só 8 das 20 etapas de janela foram comparadas: `vit_path`, `vit_score`, `env_mod`,
  `features`, `z`, `cos`, `probs` e `decisao`. As outras 12 (`y_prep`, `stft_S`, `grade`, `sal`, `e_norm`,
  `custo`, `f_rot`, `P_mean`, `env_abs`, `env_dec`, `welch_f`, `welch_P`) e as 6 etapas de HMM do
  comparador estão `AUSENTE`. O HMM no chip só foi conferido pelo fluxo de 30 janelas (MA-ok abaixo).
- Só rodaram 79 dos 164 casos. São 30 limpos, 45 ruidosos e 4 de borda, todos de 1 s do A2212. Os 4 de borda
  são `silencio1s` e `clip_a2212`, cada um nos dois modelos. Os casos de borda `dc_ruido3s`, `rot8hz3s`,
  `rot90hz3s`, `silencio3s` e `clip_idmt` deram `@@ERRO ... sem memoria` (`esp32_equiv_esp32_20261025-141820.csv`).
  O caso DC, o único que falha em float32 no PC, nunca rodou no `p11_c_lowmem`, nem no PC: a
  `equivalencia_float-lowmem_20261025-135553` tem os mesmos 79 casos e 18 etapas, sem `stft_S` e `sal`, e
  nenhuma etapa de sequência.

O `main.tex:400-404` declara "the 8 final outputs ... (Viterbi path to decision)". Isso está certo,
mas não diz quais bordas entraram. **Correção:** no resumo, escrever "its final outputs (Viterbi
path to decision) agree ... on all 79 one-second A2212 windows that fit (75 real, 4 silence/clipping
edge cases)". No §Cost, acrescentar que o PC e o chip cobrem os mesmos 79 casos e que os casos de
borda de 3 s, incluindo o DC, não foram verificados no `p11_c_lowmem`. Na conclusão, usar a mesma
ressalva.

**MA2 · A frase "the same comparison on the chip" é falsa** (`main.tex:349-350`). No chip a
variante é outra (`p11_c_lowmem`, e não `p11_c_ref` float), o subconjunto é outro (79 de 164 janelas)
e as etapas também (8 de 26). **Correção:** "a comparison of the low-memory variant on the chip,
restricted to the final outputs of the 1-s windows, is reported in Section~\ref{sec:cost}".

## Problemas menores

- **m1** · `main.tex:51`: o resumo afirma "dominated by a table-free harmonic sum in emulated double precision".
  O domínio do rastreio está medido (88,6 % da mediana). A causa, o double emulado, é uma
  hipótese, e o próprio `:408` diz "we attribute". Não há tempo do `p11_c_lowmem` no PC que isole o
  efeito da tabela. Além disso, os 39 % no PC (`:407`) são da variante `p11_c_ref`, e por isso a
  comparação mistura variante e plataforma. Escrever "dominated by the harmonic sum, which we attribute to ...".
- **m2** · Latência no chip: `init_ms = 0` nas 205 linhas de `esp32_lat_esp32_20261025-143026.csv`.
  O `p11_lm_init` roda uma vez por tamanho (`firmware/main/main.c:235-237`) e o `g_lm_init_us` não
  está gravado em nenhum CSV ou JSON. A exclusão é legítima, mas `main.tex:404-406` e a legenda de
  `tables/latex/recursos.tex` precisam dizer "excluding the one-time initialisation".
- **m3** · `main.tex:504`: o texto diz "$p \geq 0.26$ for M1–M3", mas p(M2) = 0,256
  (`aud_rotacao_motor_20261025-140355.csv`). Escrever "$p > 0.25$".
- **m4** · A legenda de `tables/latex/desempenho_compacto.tex` diz, na versão regular, "CIs ... are given in the text".
  No texto só aparecem os ICs do A2212 por arquivo (clássicas e p11) e os do teste. Como a Fig. `balacc` só existe na WiP, os
  outros ICs não aparecem em lugar nenhum da regular. Escrever "for the A2212 file split in the text".
- **m5** · `main.tex:557`: "The independent audit predates the on-chip and test-set measurements" ficou
  desatualizado. Trocar para "an independent audit covered the on-chip and test-set results".
- **m6** · Fluxo no chip (`main.tex:519-521`): as 30 janelas levam cerca de 2,65 s cada (`us_total`). É uma
  reprodução a partir da flash, não um fluxo em tempo real. Acrescentar "processed back to back, not in real time".
- **m7** · Reprodutibilidade da bancada: `esp32_lat`, `esp32_equiv`, `esp32_seq` e `bancada.json` têm
  `sujo: true`. O campo `compilado` (01:36:51) é o `__TIME__` do `main.c`, não o do binário
  (`p11_bancada.bin` às 13:56:30, 220 384 B = `size_esp32_lowmem`). O C não mudou depois de `b4504aa`,
  então não vejo risco no resultado, mas convém gravar o sha256 do `.bin` no manifesto.
- **m8** · `paper/$n.aux`, `$n.log` e `$n.pdf` não estão rastreados: são um build perdido, de 4 pp e com 19 avisos. Não
  submeter nem commitar. Os PDFs válidos são `main.pdf` (6 pp, 0 avisos) e `main_wip.pdf` (4 pp, 0 avisos).
- **m9** · O `results/manifests/manifesto.json` não lista os artefatos do teste (`teste_idmt_*`,
  `numeros_teste.tex`). As macros têm fonte no comentário, mas o manifesto está incompleto. A Fig.
  `teste_idmt_condicoes_en.pdf` está com o caminho fixo no `main.tex:467` e não passa pelo `figuras.tex`.
- **m10** · `reports/c_equivalence.md:5` ainda diz "fixadas antes de qualquer saída do C existir para este
  agente". É verdade só para o agente D. O `main.tex` já tem a cronologia certa (`:278-280`). Isso não afeta o artigo.

## Verificações pedidas

**(1) "Measured on the ESP32".** Todas as ocorrências (`main.tex:103`, `:305-306`, `:394`,
`:397-409`, `:420-422`, `:519-521`, coluna (4) de `recursos.tex`) vêm de `bancada.json`,
`esp32_memoria_*141018`, `esp32_lat_*143026`, `esp32_seq_*143251` ou `equivalencia_esp32_*141829`,
todos produzidos pelo firmware no chip. As razões 5,9× e 3,0× (`:393`) misturam uma estimativa (a arena)
com uma medição (o pool) e estão marcadas como estimativa. "Radios off" se apoia em `CONFIG_BT_ENABLED=n`
e no Wi-Fi nunca iniciado (`bancada.json: wifi_bt`). Passa.

**(2) Embarcabilidade.** O texto bate com os dados. O chip é um ESP32-D0WD-V3 rev. 3.1, sem PSRAM
(`psram_init 0`). O pool tem 307,6 KiB, e a referência de 3 s (1 820 KiB) e a de 1 s (936 KiB) não cabem. O
IDMT de 3 s dá "did not fit". O `p11_c_lowmem` de 1 s cabe com pico de 303 308 B (96 %), heap mínimo de
7 584 B e pilha livre de 20 536 B. A RTF é 2,65. O texto não afirma tempo real em lugar nenhum: "*not* real time" está em `:406`,
"neither real-time operation nor low power" em `:548` e "2.65 times slower" no resumo e na conclusão.

**(3) Equivalência no chip.** As tolerâncias são as congeladas (sha `d74dae80…`, `tol_build: float`). A cobertura é
a descrita em MA1. O fluxo no chip foi conferido à parte: decisões online iguais às do Python em 30/30 janelas
e às do C no PC em 30/30, |Δp| ≤ 2,2e-5, atraso de 0 nas duas trocas.

**(4) Integridade do teste do IDMT.** Cronologia:
- `cc5e2bf` às 15:07:40 commita o congelamento (gerado às 15:07:39, a partir de `ac0f113`, árvore limpa) e `teste_idmt.py`;
- a avaliação começa às 15:07:49, dura 88 s e registra o commit `6f8caee`, um commit de 15:08:36 que só tem
  `.aux`/`.log` do LaTeX e resultados do ESP32, sem código;
- os resultados entram em `2dd8a47` às 15:09:45.

O `cmd_avaliar` confere os hashes e a configuração antes de abrir o teste. Os 11 sha256 do
congelamento batem com os arquivos de hoje. `git diff cc5e2bf HEAD` não mexe em `pipelines/`,
`motorsound/` nem `teste_idmt.py`; só mudaram os geradores de texto, figura e tabela. Só existe
um conjunto `teste_idmt_*`. Fora do `io.py`, só `teste_idmt.py` e `tests/test_splits.py` usam
`MOTORSOUND_ABRIR_TESTE`. A única mudança de tabela feita depois do teste foi na legenda.

A interpretação está certa. Os 27 testes são 9 condições × 3 classes, o treino só tem `pure`, e o texto diz
"transfer across sessions and noise of the same three motors, not state recognition" (`:481-482`)
e "class still equals motor there" (`:531`). "Leads only under white noise" se confirma: `whitenoise_low` é a
única condição em que a p11 supera as clássicas. **Passa.**

**(5) Auditoria anterior.**

| item | estado |
|---|---|
| M1 | aplicado em `:320-325`, `:335-344` e `RESULTADOS.md:18-19` |
| M2 | aplicado em `:278-283`, com o holdout 24/24 reproduzível e duas rodadas idênticas |
| M3 | aplicado em `:457-460` (0,052 / 0,068) |
| M4 | aplicado em `:500-507` (ver m3) |
| M5 | aplicado em `:461-462` |
| m1 | aplicado |
| m2 | aplicado em `:210` e `:228` (espdsp fica de fora) |
| m3 | aplicado em `:283-285` |
| m4 | aplicado ("at least", KiB) |
| m5 | aplicado em `:300-303` e na legenda de `recursos` |
| m6 | parcial (ver m5 desta auditoria) |
| m7 | aplicado em `:498` |
| m8 | resolvido |
| m9 | não se aplica (o texto não traz AUC do protocolo motor) |
| m10 | aplicado em `:506` |

**(6) Números sem fonte ou afirmações além dos dados.** O `checagem_latex.txt` dá 131 macros e 0 `pending`.
Os números do PDF compilado, por exemplo 2.65, 0.704, 0.595, 303,308 e 0.052, coincidem com as macros. Não achei
número sem fonte. As afirmações que vão além dos dados são MA1, MA2 e m1.

## Números reconstruídos (macro → valor no `numeros*.tex`/PDF → recalculado por F)

| # | macro | texto | recalculado (fonte) |
|---|---|---|---|
| 1 | `\tstBalCls` | 0.704 | 0,7040 (sklearn sobre `teste_idmt_janelas`) |
| 2 | `\tstBalDec` | 0.595 | 0,5951 |
| 3 | `\tstBalRmsNorm` | 0.476 | 0,4765 |
| 4 | `\tstBalClsLo/Hi` | [0.615, 0.789] | [0,615, 0,791] (meu bootstrap por gravação, estratificado, outra semente) |
| 5 | `\tstBalDecLo/Hi` | [0.471, 0.714] | [0,474, 0,714] |
| 6 | `\tstAucCls/Dec/RmsNorm` | 0.954 / 0.869 / 0.728 | 0,9537 / 0,8688 / 0,7285 |
| 7 | `\tstPredGoodDecBrokenPct/HeavyloadPct` | 72.6 / 45.7 | 72,6 / 45,7 |
| 8 | `\tstBalDecWhite` vs `\tstBalClsWhite` | 0.954 vs 0.869 | 0,954 vs 0,869; única condição com p11 > p04 |
| 9 | `\tstNWin/NRec/NCond` | 2021 / 27 / 9 | 2021 / 27 / 9 |
| 10 | `\VChipLatBldcMed/Pninetyfive/Max/N` | 2647.5 / 2647.6 / 2647.7 / 200 | 2647,50 / 2647,62 / 2647,67 / 200 (75 distintas, aquecimento de 5 excluído; total = soma das etapas, erro de 1e-12) |
| 11 | `\VChipRtfBldc` | 2.65 | 2,6475 |
| 12 | `\VChipBldcTrackPct` | 89 | 88,6 |
| 13 | `\VChipWsBldcB`, `\VChipPoolKib`, `\VChipWsPoolPct` | 303,308 / 308 / 96 | 303 308 / 307,56 / 96,3 |
| 14 | `\VChipHeapIntBldcB`, `\VChipStackBldc` | 7,584 / 20,536 | 7 584 / 20 536 |
| 15 | `\VChipRefOverPoolThree/One` | 5.9 / 3.0 | 5,92 / 3,04 |
| 16 | `\VChipEqCases`, `…Real`, `…Edge`, `…Fail`, `…MaxRatio`, `…Stages` | 79, 75, 4, 0, 0.19, 8 | 79 (30 + 45 + 4), 0 FAIL, 0,187, 8; sha `d74dae80` |
| 17 | `\VChipSeqSame/Windows`, `\VChipSeqDelay`, `\VChipSeqHmmUs` | 30/30, 0, 104 | 30/30, 0 e 0, 104 |
| 18 | `\VEqLmStages` | 18 | 18 (79 casos; faltam `stft_S` e `sal`) |
| 19 | `\audHoldoutN`, `…PathSame`, `…RatioMax`, `…DpMax` | 24, 24, 0.20, 8.7e-6 | 24, 24, 0,2002, 8,66e-6 |
| 20 | `\VDiagDropHPClsBldc/DecBldc` | 0.052 / 0.068 | 0,0516 / 0,0682 |
| 21 | `\audRotBldcSpearmanPMTwo` | 0.26 | 0,256 (M1 0,292, M3 0,266; ver m3) |
| 22 | `\VBuildLmBinKb`, `…DramKb/TotalKb`, `…IramKb/TotalKb` | 215, 35/176, 58/128 | 220 384 B = 215,2; 35 632 / 180 736; 59 099 / 131 072 |

## O que mudar antes de submeter

1. MA1: resumo (`:49-50`), §Cost (`:400-404`) e conclusão (`:567-568`) devem dizer "final outputs, 79 one-second A2212 windows,
   silence/clipping edge cases only".
2. MA2: corrigir "the same comparison on the chip" em `:349-350`.
3. m1, m2, m3, m4, m5 e m6: são uma frase cada, em `main.tex:51`, `:404-406` com a legenda de `recursos.tex`, `:504`, a legenda de
   `desempenho_compacto`, `:557` e `:519-521`.
4. Não submeter o `paper/$n.pdf`.

Com essas correções, os portões C, D e E passam a PASS. A versão regular de 6 pp está sustentada pelos dados.
