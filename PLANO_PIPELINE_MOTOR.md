# Plano da pipeline — Artigo 3 (MotorSound, IEEE ICIT 2027)

Escrito em 14/10/2026 a partir da leitura do código em `Artigo1(cross-talk)/Decifra-Baixo`
e `Artigo2(Acordes)/Decifra-Violão`. Prazo de submissão: 26/10/2026, artigo regular de 6
páginas. Toda referência a função abaixo aponta para um arquivo que existe nesta pasta.

## 1. A ideia em uma frase

O motor é tratado como um instrumento de uma corda só: a camada contínua é o
`_viterbi_pitch` do baixo V3, adaptado para rastrear a rotação, e a camada discreta é o
`viterbi.decode` do violão, adaptado para decidir o estado de operação em vez do acorde.
O protocolo experimental (cache, validação, congelamento, estatística pareada) vem inteiro
do violão.

## 2. O que vem de cada módulo

### 2.1 Do baixo (`Decifra-Baixo/fase2_v1v2v3`)

| Função real | Arquivo | Uso no motor | Mudança necessária |
|---|---|---|---|
| `_viterbi_pitch` | `v3_viterbi.py` | **Rastreador da rotação.** Mesma emissão (`0,55·prox·vprob + 0,45·e_norm`, com energia em f e 0,5 em 2f), mesmo estado de silêncio, mesmas transições (passo, oitava, silêncio). | Estados viram uma grade de frequência de rotação em vez de semitons; o loop O(T·S²) precisa ser restrito a saltos vizinhos para caber no MCU. |
| `_VIT_W_OCTAVE` | `v3_viterbi.py` | Pune a troca entre f e 2f, que é o erro de oitava do motor (fundamental fraca, 2º harmônico forte). | Nenhuma; é o mesmo fenômeno. |
| Estado `SIL` | `v3_viterbi.py` | Motor parado ou sinal sem rotação detectável. | Nenhuma. |
| `librosa.pyin` + `voiced_prob` | `v3_viterbi.py` | Entrada de proximidade da emissão, como você queria. | `fmin`/`fmax` definidos pelo espectro do IDMT, não pelas cordas. |
| `_filtro_passa_banda` | `v2_polifonico.py` | Isolar a faixa da rotação antes do pYIN, como cada corda era isolada. | `sosfiltfilt` é não causal; no MCU vira `sosfilt` (um biquad em cascata). |
| `_spectral_gate` | `v2_polifonico.py` | Tirar o ruído de fundo (atmo, white noise) com piso por quantil. | Nenhuma no offline; no MCU, o quantil vira média móvel do piso. |
| `_validar_harmonicos` / `_HARMONIC_RATIO` | `v2_polifonico.py` | Feature de razão harmônica (energia nos harmônicos sobre o ruído). | Vira feature contínua em vez de limiar. |
| HPSS `margin=3.0` | `v3_viterbi.py` | Separar o tom do motor da fala e de impactos. | Só no offline; entra na ablação. Caro demais para o MCU. |
| `modulo_ruidos.py` (Lee & Shen 2011) | `fase1_modular` | **Bancada sintética com verdade conhecida.** Ver §4. | Volta ao domínio original: a frequência de referência deixa de ser a da nota e passa a ser a da rotação. |

O que **não** vem do baixo: as bandas por corda (`CORDAS_5`), a fusão de segmentos, os
filtros de onset e de corda errada, o DP de dedilhado e a tablatura. O motor tem uma fonte
só, então o problema de cross-talk do Artigo 1 não existe aqui.

### 2.2 Do violão (`Decifra-Violão`)

| Função real | Arquivo | Uso no motor | Mudança necessária |
|---|---|---|---|
| `decode` (Viterbi em log, `change_penalty` fixo) | `viterbi.py` | **HMM de estados de operação.** 3 classes + estado N (sem motor/sinal fraco). | Nenhuma no algoritmo. O `change_penalty` independente do número de estados é a correção do bug do decodificador travado; mantém. |
| `force_n` por RMS | `viterbi.py` + `pipeline.decode` | Gate de silêncio relativo (`rel_gate`, `abs_floor`). | Nenhuma. |
| `_absorb_short` / `min_dur` | `pipeline.py` | Duração mínima de um estado antes de virar alarme. | Nenhuma. |
| `CosineChordClassifier` | `classifier.py` | **Classificador por cosseno contra protótipos de classe** (um vetor médio de features por estado, aprendido no treino). | Templates deixam de ser binários e passam a ser centróides; o cosseno continua, e é barato no MCU. |
| `extract_features` / `decode` separados | `pipeline.py` | Mesma fronteira: extração cara (STFT, ridge, features) cacheada; decisão barata varrida. | Renomear `Features` para campos do motor. |
| `PreChromaConfig` / `PostChromaConfig` | `config.py` | `PreConfig` (sr, n_fft, hop, faixa de f0, HPSS on/off) e `PostConfig` (temp, change_penalty, n_logprob, rel_gate, min_dur). | Trocar os campos; a estrutura fica. |
| `cache.py` | `cache.py` | Cache de features por arquivo com verificação de config. | Trocar a chave de `pre`. |
| `noise.py` | `noise.py` | Ruído branco/rosa a SNR alvo e transientes percussivos para estressar além das condições do IDMT. | Remover `detune` (sem sentido aqui). |
| `stats.py` | `stats.py` | Bootstrap pareado por **gravação**, Wilcoxon, Holm, regra confirmado/empate/frágil. | Trocar a unidade de "faixa" para "gravação de origem" e os pares. |
| `congelar.py` + `docs/CONGELAMENTO.md` | — | Congelar as configurações antes de ler o teste. | Nenhuma no processo. |
| Regras do `CLAUDE.md` | — | Busca só na validação, teste lido uma vez, um CSV por experimento, nenhum número digitado à mão. | Copiar para um `CLAUDE.md` do Artigo 3. |

O que **não** vem do violão: chroma (dobra as frequências módulo oitava e apaga a
informação de rotação), beat tracking (motor não tem batida; as janelas são fixas), os 60
templates de acorde e o `mir_eval`.

## 3. A pipeline resultante

```
wav 44,1 kHz ─▶ 16 kHz ─▶ passa-altas (biquad) ─▶ normalização RMS
        │
        ├─[PRÉ, cacheável]──────────────────────────────────────────────┐
        │  spectral gate (baixo V2)                                      │
        │  passa-banda na faixa da rotação (baixo V2, causal)            │
        │  pYIN + voiced_prob (baixo V3)                                 │
        │  STFT 4096/1024 ─▶ emissão do baixo V3 (prox·vprob + energia)   │
        │  Viterbi de rotação com estado SIL e custo de oitava (baixo V3)│
        │  features por janela de 1 s: f0, dispersão, razão harmônica,   │
        │  pico de modulação do envelope, forma espectral                │
        └────────────────────────────────────────────────────────────────┘
        │
        └─[PÓS, barata]────────────────────────────────────────────────┐
           cosseno contra protótipos de classe (violão classifier)      │
           softmax(temp) ─▶ Viterbi de estados com change_penalty fixo  │
           e force_n por RMS relativo (violão viterbi/pipeline)         │
           _absorb_short(min_dur) ─▶ alarme                             │
           └────────────────────────────────────────────────────────────┘
```

No ESP32 a parte PÓS roda inteira. Da parte PRÉ saem o pYIN e o HPSS: o rastreador
embarcado usa só a energia espectral na emissão (o termo `0,45·e_norm`), e a ablação
mede quanto isso custa em relação à versão com pYIN.

## 4. Bancada sintética: o módulo de ruídos volta para casa

O `modulo_ruidos.py` adaptou para música os distúrbios de qualidade de energia de Lee &
Shen (2011): sag, swell, interruption, flicker, harmônicos, notch, spike. A classe "broken"
do IDMT foi gerada chaveando a tensão de alimentação entre 15 % e 75 % a cada 18 ms, que é
um flicker/sag de alimentação. Ou seja, o módulo que você adaptou da elétrica para a música
descreve exatamente a perturbação do dataset. A proposta é gerar o tom do motor com a
rotação acompanhando a tensão perturbada e usar isso como bancada com verdade conhecida
para testar o rastreador e o atraso de detecção de transição. Isso também dá uma linha de
continuidade entre os três artigos, o que ajuda a narrativa.

Ressalva: como no violão, a bancada sintética valida o comportamento, não entra como
resultado principal.

## 5. Variantes e comparações (rascunho para pré-registrar)

| Variante | O que é |
|---|---|
| `B0` | Log-mel 30×32 + CNN pequena (reprodução do Strantzalis 2022) |
| `B1` | Features sem rastreador (só forma espectral) + regressão logística |
| `A` | Rastreador **sem** Viterbi (argmax por quadro) + features + cosseno, sem HMM |
| `B` | Rastreador com Viterbi do baixo + features + cosseno, sem HMM |
| `C` | `B` + HMM de estados do violão (sistema completo offline) |
| `C_mcu` | `C` sem pYIN e sem HPSS (o que roda no ESP32) |

| Par | O que isola |
|---|---|
| A → B | o Viterbi de rotação (baixo V3) |
| B → C | o HMM de estados (violão), em atraso de detecção e trocas espúrias |
| C → C_mcu | o custo de tirar pYIN e HPSS para embarcar |
| B0 → C_mcu | o sistema embarcado contra a CNN de referência |

Métricas: acurácia e F1 por janela, por condição de ruído (pure, talking, white, atmo
low/medium/high, stress test), atraso mediano de detecção, trocas espúrias por minuto e
RAM/latência no ESP32.

## 6. Dados

**IDMT-ISA-ELECTRIC-ENGINE**: o zip está em `Artigo3(MotorSound)/` duas vezes (o
`(1).zip` é duplicata do mesmo tamanho; pode apagar um). Usar o split train/test oficial.
Para seguir o protocolo do violão é preciso tirar uma validação **de dentro do train**,
agrupada por gravação de origem, e só abrir o test no fim, depois do congelamento.

**BLDC A2212** (Estacio et al., Data in Brief 2023): ainda não está na pasta. 43 arquivos.
O rolamento só aparece no motor 4, então o split é por arquivo e a limitação vai escrita no
artigo.

## 7. O que muda no esqueleto entregue antes

O esqueleto `motor_acoustic.zip` foi escrito sem acesso a este código. Ajustes:

| Arquivo do esqueleto | Ação |
|---|---|
| `ridge.py` | Manter a soma harmônica só como emissão alternativa; o Viterbi passa a seguir `_viterbi_pitch` (estado SIL, custo de oitava, emissão mista com pYIN). |
| `hmm.py` | Substituir por uma cópia de `viterbi.decode` do violão, com `force_n`. O filtro forward online fica como versão MCU. |
| `config.py` | Reorganizar em `PreConfig` / `PostConfig` no molde do violão. |
| `features.py`, `evaluate.py`, `mcu_budget.py` | Manter. |
| `run_benchmark.py` | Reescrever no molde do `runner.py`: um CSV por experimento, validação separada do teste. |
| Novo | `stats.py` adaptado, `congelar.py`, `cache.py`, `CLAUDE.md` do Artigo 3, bancada Lee & Shen. |

## 8. Pendências que dependem de você

1. Descompactar o `IDMT-ISA-ELECTRIC-ENGINE.zip` dentro de `Artigo3(MotorSound)/`. O
   arquivo tem 1,5 GB e não dá para trazer inteiro para o ambiente de análise; descompactado,
   dá para trazer alguns wav por classe e rodar o `explore_spectrum.py`.
2. Baixar a base A2212 (Mendeley `j4yr5fmhv4`) para a mesma pasta.
3. Decidir se o Artigo 3 vira um repositório git como os outros dois (recomendo, pelo
   oráculo de regressão).
