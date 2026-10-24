# Auditoria do repositório: referência Python da p11 × porte C do colaborador

Agente A, 23/10/2026, entre 22h22 e 22h40 (hora local do PC). Somente leitura fora deste
arquivo. Nenhum arquivo de `data/idmt/test` ou `test_cut` foi aberto; a única leitura de dados
foi `soundfile.info` (cabeçalho) dos 43 wav do A2212 e dos 3 wav de `data/idmt/train`.

Ambiente da verificação: `.venv\Scripts\python.exe`, numpy 2.5.4, scipy 1.18.1 (comando
`python -c "import numpy,scipy;print(numpy.__version__,scipy.__version__)"`). Os fatos de tipo
(dtype) e dimensão citados abaixo saem de um script de sondagem com ruído sintético
(`<scratchpad>\audit_probe.py`, sem dados reais), cuja saída está transcrita em §2.0.

Convenção: `py:` = arquivo Python e linha; `c:` = `embarcados/src/c/colaborador_original/decifrae_core.c`;
`h:` = o `.h` correspondente.

---

## 1. Árvore de `embarcados/`

Instantâneo às **22:22:28** e repetido às **22:27:11** de 23/10/2026 (idêntico nas duas vezes;
outros agentes trabalham em paralelo, então a árvore pode ter mudado depois disso):

| arquivo | bytes | modificado | o que é |
|---|---:|---|---|
| `.gitignore` | 252 | 22:21:18 | ignora `gold/`, `results/c_out/`, `build*/`, `firmware/build/`, `*.f32/.f64/.i32`, binários MSVC |
| `CLAUDE.md` | 5573 | 22:21:12 | contrato da pasta: referência dourada somente leitura, regras de protocolo, 4 categorias de medição, prazos |
| `PROMPT_TESTES_EMBARCADO.md` | 11001 | 22:21:12 | tarefa em 8 etapas (auditoria, ouro, tolerâncias, porte, ESP32, resultados, artigo, portões) com os agentes A–F |
| `reports/BLOQUEIOS.md` | 1666 | 22:21:57 | B1 sem ESP32, B2 sem ESP-IDF, B3 estrutura diferente; MSVC disponível |
| `reports/CONTRATO_INTERFACES.md` | 5342 | 22:21:45 | etapas com id, formato dos vetores de ouro, artefato do modelo, regra do score do Viterbi, builds float/double |
| `src/c/colaborador_original/decifrae_core.c` | 9547 | 22:21:12 | porte do colaborador (ver abaixo) |
| `src/c/colaborador_original/decifrae_core.h` | 4269 | 22:21:12 | cabeçalho do porte |
| `reports/repository_audit.md` | — | — | este arquivo |

**Procedência do porte do colaborador.** Conferido contra o commit `e00f2a9` ("Codigos em C",
2 arquivos, 255 + 109 linhas):

```
$ git show e00f2a9:Embarcado/decifrae_core.c | diff --strip-trailing-cr - embarcados/src/c/colaborador_original/decifrae_core.c && echo "c identico"
c identico
$ (idem para .h)
h identico
sha256  348304592966a79052972090ed4e24425b16e9b8918be69a04094a03068c268c  decifrae_core.c
sha256  3097d5654c81b37e3eb4ea458b48e0a214613592c31df5afff2917298d41b2fb  decifrae_core.h
```

O `.c` contém quatro funções públicas (confirmadas pelo `dumpbin`, §5):
`decifrae_default_config` (c:42–53, só preenche a struct), `decifrae_build_features`
(c:55–164, monta as 13 features a partir de saídas do front-end que **outro código** precisa
fornecer), `decifrae_cosine_predict` (c:166–204) e `decifrae_hmm_viterbi` (c:206–255, Viterbi
genérico com matriz de transição e prior fornecidos pelo chamador). O próprio cabeçalho declara
(h:39–41): *"rastrear(), stft_mag(), and _envelope_mod() are not included in the supplied
snippet. They must be ported separately"*. Ou seja, **o front-end inteiro (STFT, grade, soma
harmônica, emissão, Viterbi de rotação, envelope) não existe no C**.

---

## 2. Etapa a etapa: referência Python × C do colaborador

### 2.0 Saída da sondagem (fatos de tipo e tamanho na referência)

```
J 1201 15.0 60.0 k_oct 600
seg 3.0 N 48000 T 39
seg 1.0 N 16000 T 8
y_prep dtype float32 0.09999999573449281
S (4097, 39) float32 f float64 1.953125
hann sym 0.0 0.0
rfft dtype on f32: complex64
sal float64 (39, 1201)
max pos h48 1474.56 F-1 4096
custo (39, 1202) float64
hilbert dtype complex64
env abs float32 sosfilt out float64 sos shape (2, 6)
dec len 3000
welch sig (x, fs=1.0, window='hann_periodic', nperseg=None, noverlap=None, nfft=None, detrend='constant', return_onesided=True, scaling='density', axis=-1, average='mean')
welch B 513 0.9765625 bins in 2..100 100
A2212 welch B 501 1.0 99
feat float64 (13,)
round half even [0. 2. 2.]
P float32 tot float32 logP float32 cent float64 nband 2043 first bin 6 last 2048
```

Consequência importante: com numpy 2.x, `np.fft.rfft` sobre float32 devolve **complex64** e
`scipy.signal.hilbert` sobre float32 devolve **complex64**. A referência dourada, portanto, já
calcula a STFT, o espectro médio `P` e a planura em **float32**; a soma harmônica, a emissão, o
Viterbi de rotação, o envelope pós-filtro, o Welch e o classificador estão em **float64**. As
tolerâncias de equivalência precisam partir disso (um C todo em double não é "mais fiel" nas
etapas em float32).

### 2.1 Tabela de status

Legenda: **completo** = mesma conta; **parcial** = parte da conta, ou conta igual com
diferença numérica declarada; **diferente** = conta diferente da referência; **ausente** = não
existe no C.

| # | etapa | referência Python | C do colaborador | status |
|---|---|---|---|---|
| 1 | leitura e reamostragem | py `motorsound/io.py:121–132` (`sf.read(dtype="float32", always_2d=True)` l.125; média dos canais l.126; `librosa.resample(..., res_type="soxr_hq")` l.127–129); recorte `runner.py:90–91` com `int(round(t0*sr))` | — | **ausente** (fora do escopo do núcleo por contrato, §1 do CONTRATO) |
| 2 | normalização RMS | py `runner.py:70–75`: `y=float32`; `r=float(sqrt(mean(y**2))+1e-12)`; `y*(0.1/r)` (resultado float32) | — | **ausente** |
| 3 | passa-banda | py `rotacao.py:35–37`; único uso em `rotacao.py:184`, dentro de `if usar_pyin:` (l.181). A p11 usa `usar_pyin=False` (`p11_decifrae_harmsum.py:10`, `_decifrae.py:40`) | — | **não se aplica**: a p11 **não filtra** o sinal antes da STFT; nada a portar |
| 4 | STFT | py `rotacao.py:40–48`: Hann simétrico `np.hanning(n_fft)` em float32 (l.41; extremos 0,0 e 0,0); zero-pad só se `len<n_fft` (l.42–43); `n=1+(len−n_fft)//hop`, sem centralização (l.44); `abs(rfft)` complex64 → float32 (l.46, l.48); `freqs=rfftfreq` float64 (l.47) | — | **ausente** (h:39) |
| 5 | grade | py `rotacao.py:30–32`: `n=floor(1200·log2(fmax/fmin)/passo)+1`; `fmin·2^(k·passo/1200)` em float64 | `grid_hz` é **entrada** float32 (h:46); só usado na busca binária c:94–99 | **ausente** (o C não gera a grade) |
| 6 | soma harmônica | py `rotacao.py:51–67`: 48 harmônicos peso 1; `pos=h·f/df`; inclui só `pos<F−1` (l.61; com f≤60 Hz, 48·60/1,953=1474,6 < 4096, logo todos entram); interpolação linear (l.62–64) acumulada em float64; `e_norm=sal/(max_j sal + 1e-12)` (l.66) | `e_norm` é **entrada** float32 (h:45) | **ausente** |
| 7 | emissão | py `rotacao.py:70–99` com `f0=None`: `score=1,0·0,45·e_norm` (l.90; o termo 0,55·prox·vprob é zero); `custo=−log(score+1e-3)` (l.92); SIL: `rel=e_q/(e_q.max()+1e-12)` com `e_q=sal.max(axis=1)` (l.179, l.96, l.198), `custo_SIL=−log((1−clip(rel,0,1))+1e-3)` (l.97–98) | — | **ausente**. Atenção para o porte: `e_ref=e_q.max()` é o máximo **do segmento inteiro**, logo o custo do SIL no quadro t depende de quadros futuros |
| 8 | Viterbi de rotação | py `rotacao.py:102–159`: mínimo custo; `delta0=custo[0]` sem prior (l.116); `psi` int32 (T, J+1) (l.117); saltos `o=−50..+50` varridos em ordem crescente com `<` estrito (l.114, 123–130); depois `+k_oct`, `−k_oct` (l.131–138); depois vindo do SIL (l.139–142); linha do SIL com `<=` a favor de ficar em SIL e `j_best=argmin(prev[:J])` (l.146–152); final `argmin(delta)` (l.155); SIL → −1 (l.158) | — (`max_jump` e `use_viterbi` existem só na struct, h:23–24, c:51–52; nenhuma função os usa) | **ausente** |
| 9 | `f_rot` | py `rotacao.py:204`: `grade[path]`, NaN no SIL | entrada `f_rot_hz` float32, NaN = SIL (h:32, h:44) | **ausente** (consumido, não produzido) |
| 10 | features 0–2 (mediana, IQR, fração SIL) | py `_decifrae.py:68–71, 95`: `nanmedian`, `nanpercentile([75,25])` (linear), `1−vivos.mean()`; 0,0 se não há quadro vivo | c:67–87: ordena por inserção (c:18–29), percentil linear `pos=(n−1)q` (c:31–40) em float32 | **completo** na conta; float32 no C × float64 no Python |
| 11 | feature 3 (saliência no caminho) | py `_decifrae.py:73–75`: `searchsorted` (lado esquerdo) em grade float64; quadro SIL conta 0 na média sobre T | c:89–104: busca binária `grid[mid] < fr` (= lado esquerdo), soma em double, divide por `n_frames` | **completo** se o chamador passar `grid_hz` e `f_rot_hz` arredondados do mesmo double (senão a busca pode cair no índice vizinho) |
| 12 | `P` e features 4–8, 11, 12 | py `_decifrae.py:77–94`: `S` recalculado por `stft_mag` (l.77) e `P=(S**2).mean(axis=1)` em **float32** (l.78); `tot=P[banda].sum()+1e-20` float32 (l.81); `pos=np.round(k·f_med/df)` (meio para par, float64) (l.84); só `pos<len(P)−1`, sem limite inferior (l.85); grupos `10·log10(·/tot+1e-10)` float64 (l.91); centróide `(f·P).sum()/(P.sum()+1e-20)` (l.93); planura `exp(mean(log(P[banda]+1e-20)))/(P[banda].mean()+1e-20)` com log e média em **float32** (l.94) | c:106–162: `P` é **entrada** (`mean_power`, h:49); `lroundf(k·f_med/df)` em float32, meio para longe do zero (c:130); exige `pos>=1` e `pos<n_bins−1` (c:131); grupos com `log10f` float32 (c:148); centróide com `f=i·df` float32 e somas double (c:113, 117, 155); planura com `log(double p + 1e-20)` e média aritmética `(Σp+1e-20)/n_band` (c:120, 124, 157) em vez de `mean(P)+1e-20`; rejeita `p<0` ou não finito (c:115) | **parcial / diferente**: ver D4–D9 |
| 13 | `_envelope_mod` (features 9, 10) | py `_decifrae.py:50–60`: `abs(hilbert(y))` FFT de N pontos, complex64 (l.51); `butter(4,200,fs=sr,sos)` (l.52); `sosfilt` com estado inicial zero, saída float64, dizimação `[::sr//1000]`=`[::16]` sem antialias adicional (l.53); **subtrai a média** (l.54); `welch(env,1000,nperseg=min(len,1024))` com defaults (hann periódica, noverlap=nperseg//2, detrend constante, densidade) (l.55); máscara 2–100 Hz (l.56); `argmax` (primeiro índice) (l.59); `10·log10(Ppico/(mediana+1e-20)+1e-12)` (l.60); se máscara vazia, (0,0) (l.57–58) | c:151–153 só copia `modulation_peak_hz` e `modulation_peak_prominence_db` recebidos | **ausente** (h:39) |
| 14 | z-score | py `classificador.py:30`: `(x−mu)/sd`, com `sd=std+1e-9` de `fit` (l.22) | — | **ausente**: `decifrae_cosine_predict` usa a feature crua (c:174–188); h:76–78 diz que o protótipo "MUST use the exact feature scaling", mas a função não aplica `mu`/`sd` à entrada |
| 15 | cosseno + softmax | py `classificador.py:29–37`: `Z/(‖Z‖+1e-9)` (l.31); `M` já normalizado com `+1e-9` no `fit` (l.26); `Z @ M.T` (l.32); `e/max(temp,1e-6)` com temp=0,1 (`_decifrae.py:47`, l.35); softmax com máximo subtraído, sem epsilon no denominador (l.36–37); `prever` espalha nas colunas `clf.classes` (`_decifrae.py:104–108`) | c:174–203: `dot/(‖x‖·‖r‖)` só se `denom > 1e-20`, senão 0 (c:190–191); renormaliza o protótipo; `temperature>0` exigido (c:172); softmax igual em float32 (c:196–202); `n_classes ≤ 16` | **diferente** (D1–D3) |
| 16 | log-emissões do HMM | py `hmm.py:26–41`: `log(p+1e-12)` (l.36); estado N extra com `n_logprob` (l.37); ramo `p^(1/temp)` se `temp≠1` (l.32–35); `force_n` (l.38–40) | c:222, c:238: `logf(max(p, 1e-30))`; sem estado N, sem temperatura, sem `force_n` | **diferente** (D10–D12) |
| 17 | HMM Viterbi | py `hmm.py:44–67`: `log_T` cheio de `−change_penalty`, diagonal 0 (l.54–55); `delta0=log_em[0]` (l.58); `argmax` sobre a origem, primeiro índice (l.61); final `argmax` (l.64) | c:206–255: matriz `transition_log` e `initial_log` genéricas do chamador; `>` estrito (= primeiro índice) em c:232 e c:246; float32; backpointer (T, S) do chamador | **parcial**: a recursão é a mesma e o desempate coincide; reproduz `decode` se o chamador montar `log_T` com `change_penalty`, `initial_log=0` e acrescentar a coluna N, mas o log da emissão difere (D10) e tudo é float32 |
| 18 | filtro forward online | py `hmm.py:70–85`: `logaddexp`, normalização por quadro, `argmax` causal | — | **ausente** (D13). É a decodificação que o próprio `sequencias.py` (l.11–12) chama de "o que roda no MCU" |
| 19 | `absorb_short` e conversões | py `hmm.py:96–141` (`_merge`, `absorb_short`, `rotulos_para_segmentos`, `segmentos_para_rotulos` com `int(round(·))`) | — | **ausente**; mas **não é exercitado** nos experimentos: `experimentos/sequencias.py:55` chama `avaliar` sem `min_dur`, e o default é `0.0` (`motorsound/sequencias.py:51`), logo `sequencias.py:60` nunca entra no ramo |

### 2.2 Lista de divergências C × Python (todas conferidas no código)

- **D0 (escopo).** Faltam no C: STFT, grade, soma harmônica, emissão, Viterbi de rotação,
  `P_mean`, envelope/Hilbert/Butterworth/Welch, z-score, filtro online, `absorb_short`
  (h:39–41 o admite). O C cobre só ~3 das 19 etapas da tabela, e nenhuma das caras.
- **D1.** Cosseno sem z-score: c:174–188 usa `features` cru; Python padroniza (classificador.py:30).
  Sem o z-score as features em Hz (centróide ~10³) dominam o cosseno. É erro de algoritmo, não
  de precisão.
- **D2.** Denominador do cosseno: c:190–191 `denom > 1e-20 ? dot/denom : 0` com `‖x‖·‖r‖`;
  Python `Z/(‖Z‖+1e-9)` e protótipo pré-normalizado com `+1e-9` (classificador.py:26, 31).
  Diferença relativa ~1e-9 em casos normais; comportamento distinto para vetor quase nulo.
- **D3.** Temperatura: c:172 rejeita `temperature<=0`; Python usa `max(temp,1e-6)` (l.35).
  Irrelevante com T=0,1.
- **D4.** Arredondamento da posição do harmônico: `lroundf` (meio para longe do zero, float32,
  c:130) × `np.round` (meio para par, float64, _decifrae.py:84). Sondagem:
  `np.round([0.5,1.5,2.5]) = [0,2,2]`. Além do desempate, a conta em float32 pode cruzar o .5
  num caso em que a de float64 não cruza.
- **D5.** Limite inferior: C exige `pos>=1` (c:131); Python não testa (l.85), e `pos=0` indexaria
  `P[−1]`. Inerte na p11 (f_med ≥ 15 Hz ⇒ pos ≥ 8), mas é diferença de código.
- **D6.** Planura: C usa `log` double de `p` (c:120) e média aritmética `(Σp + 1e-20)/n_band`
  (c:124, 157); Python faz `log` e `mean` em **float32** (l.94; sondagem `logP float32`) e soma
  `1e-20` depois da média. Aqui a referência é menos precisa que o C.
- **D7.** Grupos em `log10f` float32 (c:148) × `np.log10` float64 (l.91).
- **D8.** Centróide com `f = i·df` em float32 (c:113) × `rfftfreq` float64 (l.47, l.93).
- **D9.** Ordem de soma: C acumula `P` em double sequencial (c:116–121); Python soma `P[banda]`
  em float32 (soma por pares do numpy) (l.81).
- **D10.** Log da emissão do HMM: `logf(max(p,1e-30))` (c:222, 238) × `log(p+1e-12)` (hmm.py:36).
  Para `p=0`: −69,08 no C, −27,63 no Python. Muda o caminho quando há probabilidades
  muito pequenas (comuns com softmax a T=0,1).
- **D11.** Estado N ausente: o C não acrescenta a coluna `n_logprob`; nos experimentos
  `n_logprob=−50` (sequencias.py:50) e o N existe na matriz (C+1 estados).
- **D12.** Sem ramo de temperatura `p^(1/temp)` (hmm.py:32–35) e sem `force_n` (l.38–40). Nos
  experimentos `temp=1.0` e `force_n=None`, então não são exercitados, mas o código diverge.
- **D13.** `filtro_online` ausente (hmm.py:70–85).
- **D14.** Precisão: todo o C é float32 (`float` em todas as interfaces, h:44–53, 82–104), com
  acumuladores double em c:90, 107–110, 139, 146. A referência mistura float32 (STFT, `P`,
  planura, `y_prep`, `abs(hilbert)`) e float64 (o resto). O contrato (§5) já prevê build com
  `P11_REAL=double`; o colaborador não tem essa macro.
- **D15.** Parâmetros fixos no código: `48` harmônicos em c:127–129 e c:140 ignoram
  `cfg->n_harmonics`; a banda 10–4000 Hz está em literal (c:118); `max_jump` e `use_viterbi` não
  são usados por função alguma.
- **D16.** Validação de entrada: o C devolve erro para `p<0` ou não finito (c:115), feature não
  finita (c:176) e protótipo não finito (c:186); o Python propaga NaN. Diferença de contrato,
  não de valor.

---

## 3. Parâmetros extraídos do código Python (com arquivo:linha)

Nada aqui vem de comentário nem do C. A última coluna compara com o que o `.h`/`.c` declara.

| parâmetro | valor | onde | C do colaborador |
|---|---|---|---|
| fs | 16000 | `_decifrae.py:34` (`PreDecifrae.sr`); reamostragem `io.py:127–129` | 16000 (c:44) ✓ |
| n_fft | 8192 | `_decifrae.py:38` | 8192 (c:48) ✓ (não usado) |
| hop | 1024 | `_decifrae.py:39` | 1024 (c:49) ✓ (não usado) |
| janela | `np.hanning(n_fft)` simétrica, float32 | `rotacao.py:41` | não declarada |
| centralização / pad | nenhuma; pad à direita só se `len<n_fft` | `rotacao.py:42–44` | não declarada |
| F (bins) | 4097, df = 1,953125 Hz | `rotacao.py:46–47` (sondagem) | `n_bins`, `df_hz` como entrada |
| T (quadros) | 39 para 3 s (N=48000); 8 para 1 s (N=16000) | `rotacao.py:44` (sondagem) | — |
| faixa | 15–60 Hz | `_decifrae.py:35–36` | 15, 60 (c:45–46) ✓ (não usados) |
| passo | 2 cents | `_decifrae.py:37` | 2 (c:47) ✓ (não usado) |
| J | 1201 (`floor(1200·log2(4)/2)+1`) | `rotacao.py:31` (sondagem) | não declarado |
| nº de harmônicos | 48 (`np.arange(1, N_HARM+1)`) | `_decifrae.py:28, 65` | 48 (c:50) ✓; literal 48 em c:127–140 |
| pesos dos harmônicos | 1 (`np.ones_like`) | `rotacao.py:55–56` | — |
| ε da normalização da saliência | `1e-12` | `rotacao.py:66` | — |
| pesos da emissão | 0,55 (pYIN, zerado na p11) e 0,45 (energia) | `rotacao.py:90` | — |
| ε do custo | `+1e-3` dentro do `−log` (energia e SIL) | `rotacao.py:92, 98` | — |
| ε do `rel` do SIL | `1e-12` | `rotacao.py:96` | — |
| VIT_SIGMA_CENTS | 70,0 (só com pYIN; inerte na p11) | `rotacao.py:23` | — |
| VIT_W_STEP | 0,55 por semitom → `0,55·|Δ|·2/100` por estado | `rotacao.py:24, 115` | — |
| VIT_W_OCTAVE | 7,0 | `rotacao.py:25` | — |
| VIT_W_SIL | 3,2 | `rotacao.py:26` | — |
| VIT_W_EMIT | 1,0 | `rotacao.py:27, 90` | — |
| max_salto | 50 estados (= 1 semitom) | `_decifrae.py:42`; uso `rotacao.py:114` | 50 (c:51) ✓ (não usado) |
| k_oct | `int(round(1200/2)) = 600` | `rotacao.py:113` (sondagem) | — |
| usar_pyin / usar_viterbi | False / True | `p11_decifrae_harmsum.py:10`, `_decifrae.py:40–41` | `use_viterbi=1` (c:52) ✓ (não usado) |
| banda das features | 10 ≤ f ≤ 4000 Hz (bins 6..2048, 2043 bins) | `_decifrae.py:80` (sondagem) | literal em c:118 ✓ |
| grupos de ordens | (1,4), (5,12), (13,24), (25,48) | `_decifrae.py:29` | c:143–144 ✓ (índices 0-based) |
| ε das features | `tot+1e-20` (l.81); grupos `+1e-10` (l.91); centróide `+1e-20` (l.93); planura `+1e-20` dentro do log e no denominador (l.94) | `_decifrae.py` | `DECIFRAE_EPS=1e-20f` (c:7), `1e-10f` (c:148) ✓ nos valores, ≠ no ponto de aplicação da planura (D6) |
| Hilbert | `ss.hilbert(y)` de N pontos (N = segmento inteiro) | `_decifrae.py:51` | — |
| Butterworth | ordem 4, 200 Hz passa-baixa, SOS (2 seções), `sosfilt` zi=0 | `_decifrae.py:52–53` | — |
| dizimação | `sr // 1000 = 16` (taxa 1 kHz) | `_decifrae.py:53` | — |
| remoção de média | `env − env.mean()` | `_decifrae.py:54` | — (o CONTRATO, l.30, diz "sem média" para `env_dec`; ver nota abaixo) |
| Welch | fs=1000, `nperseg=min(len,1024)`; defaults scipy 1.18: `window='hann_periodic'`, `noverlap=nperseg//2`, `detrend='constant'`, `scaling='density'`, `average='mean'` | `_decifrae.py:55` (assinatura na sondagem) | — |
| Welch, tamanhos | 3 s: len 3000, nperseg 1024, B=513, Δf=0,977 Hz, 100 bins em 2–100 Hz; 1 s: len 1000, nperseg 1000, B=501, Δf=1 Hz, 99 bins | sondagem | — |
| faixa de modulação | 2 ≤ f ≤ 100 Hz | `_decifrae.py:56` | — |
| ε da proeminência | `median+1e-20`, `+1e-12` dentro do log10 | `_decifrae.py:60` | — |
| sd do z-score | `std(axis=0)+1e-9` (ddof=0) | `classificador.py:22` | ausente |
| ε dos cossenos | `‖M‖+1e-9`, `‖Z‖+1e-9` | `classificador.py:26, 31` | `denom > 1e-20` (c:191) ✗ |
| T do softmax | 0,1 (`temp_cos`); piso `1e-6` | `_decifrae.py:47`; `classificador.py:35` | "0.1 matches PostDecifrae" (h:79) ✓ |
| seg_s | IDMT 3,0 s; A2212 1,0 s | `config.py:37–38`; escolha em `runner.py:164` | — |
| rms_alvo | 0,1, ε `1e-12` | `config.py:34`; `runner.py:73–74` | — |
| change_penalty | `PostConfig` 2,0 (`config.py:21`); **nos experimentos varrido** em `[0.5, 1, 2, 4, 8, 16]` (`experimentos/sequencias.py:25, 55`) | | genérico (matriz) |
| n_logprob | `PostConfig` −2,5 (`config.py:22`) e default de `hmm.decode` −2,5 (`hmm.py:44`); **o que roda nos experimentos é −50,0**, default de `sequencias.avaliar` (`motorsound/sequencias.py:50`), porque `experimentos/sequencias.py:55` não passa o argumento | | ausente |
| temp do HMM | 1,0 (`sequencias.py:50`; `config.py:20`) | | ausente |
| ε do log do HMM | `log(p+1e-12)`; preenchimento −50 | `hmm.py:36, 29` | `clamp 1e-30` (c:222, 238) ✗ |
| min_dur | 0,0 em `PostConfig` (`config.py:25`) e em `avaliar` (`sequencias.py:51`); nenhum experimento passa outro valor (grep por `min_dur` em `experimentos/`: só `motorsound/`) | | ausente |
| sequências | `n_seq=30`, `seed=0` (`experimentos/sequencias.py:35–36`); permanência 5–20 janelas IDMT, 3–10 A2212 (l.50); split `blocos`/`arquivo` (l.41) | | — |

Nota sobre `env_dec`: o CONTRATO_INTERFACES.md (linha 30) define `env_dec` como "dizimado por
16, sem média". O código (l.54) subtrai a média antes do Welch. Entendo "sem média" como "vetor
gravado antes da subtração", mas a frase admite a leitura oposta; sugiro que a sessão principal
deixe explícito. Como o Welch usa `detrend='constant'`, a subtração não altera `welch_P`
exceto pelo arredondamento.

Nota sobre as bases (cabeçalhos lidos com `soundfile.info`): o A2212 tem 43 arquivos,
**estéreo** PCM_16, 42 a 16 kHz e 1 a 44,1 kHz (`Healthy/M1/1400`); o `train/` do IDMT tem 3
arquivos mono PCM_24 a 44,1 kHz. Isso bate com `docs/DATASETS.md:46` e `:116–118`, mas não com o
`CLAUDE.md` do Artigo3 ("16 kHz", "32 bit"). Para o MCU: no A2212 a entrada do núcleo é a média
de dois canais, e em 1 arquivo há reamostragem.

---

## 4. Ordem das 13 features

Conferida em `_decifrae.py:95` (`[f_med, iqr, 1 − vivos.mean(), sal, razao, *grupos, fm, prom, cent, plan]`)
e em c:80–159:

| índice | lista pedida | Python (l.95) | C | situação |
|---|---|---|---|---|
| 0 | rotação mediana | `f_med` (l.70) | c:80 | ✓ |
| 1 | IQR | `iqr` (l.71) | c:81–82 | ✓ |
| 2 | fração de silêncio | `1 − vivos.mean()` | c:87 | ✓ |
| 3 | saliência média no caminho | `sal` (l.75) | c:104 | ✓ |
| 4 | razão harmônica | `razao` (l.90) | c:141 | ✓ |
| 5–8 | energia relativa 1–4, 5–12, 13–24, 25–48 (dB) | `*grupos` (l.91, GRUPOS l.29) | c:143–149 | ✓ |
| 9 | freq. do pico de modulação | `fm` (l.92) | c:152 (entrada) | ✓ |
| 10 | proeminência | `prom` (l.92) | c:153 (entrada) | ✓ |
| 11 | centróide | `cent` (l.93) | c:155 | ✓ |
| 12 | planura | `plan` (l.94) | c:157–161 | ✓ |

**Sem divergência de ordem.** Ressalvas de definição: a feature 4 conta só a energia em
`k·f_med ± 1 bin` (soma de 3 bins), não a energia nas ordens do caminho quadro a quadro; e as
features 4–8 usam a mediana `f_med`, não `f_rot` por quadro. Ambas são iguais no C.

---

## 5. Build no PC e firmware

**Arquivo:** `C:\Users\erick\AppData\Local\Temp\claude\c--Users-erick-dev-Decifrae-Artigo-Artigo3-MotorSound-\c30485f2-9096-4647-ba53-a61d9633dbba\scratchpad\audit_build\build.bat`

```bat
set PATH=C:\Program Files (x86)\Microsoft Visual Studio\Installer;%PATH%
call "C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\VC\Auxiliary\Build\vcvars64.bat" >nul
cl /nologo /O2 /W4 /c "...\embarcados\src\c\colaborador_original\decifrae_core.c" /Fo"%~dp0decifrae_core.obj"
cl /nologo /O2 /W4 /std:c11 /c "...\decifrae_core.c" /Fo"%~dp0decifrae_core_c11.obj"
dumpbin /nologo /symbols decifrae_core.obj | findstr External | findstr decifrae
```

Executado com `cmd /c "<caminho do .bat>"` a partir do PowerShell. Saída:

```
=== build /O2 /W4 (C default) ===
decifrae_core.c
exit=0
=== build /O2 /W4 /std:c11 ===
decifrae_core.c
exit=0
=== dumpbin symbols ===
017 00000000 SECT6  notype ()    External     | decifrae_default_config
018 00000000 SECT4  notype ()    External     | decifrae_build_features
019 00000000 SECT5  notype ()    External     | decifrae_cosine_predict
01A 00000000 SECT7  notype ()    External     | decifrae_hmm_viterbi
```

Compilador: `Microsoft (R) C/C++ Optimizing Compiler Versão 19.44.35228 para x64`. **Zero
avisos em /W4**, nos dois modos. Observação: sem a linha `set PATH=...Installer`, o
`vcvars64.bat` imprime `'vswhere.exe' não é reconhecido como um comando interno` (ocorreu num
segundo .bat usado só para ler a versão), embora o `cl` ainda funcione. Só compilação (`/c`);
não há `main` nem teste, então nada foi executado.

**Firmware (confirmação do BLOQUEIOS.md):**

```
PS> Get-Command gcc,clang,idf.py,xtensa-esp32-elf-gcc -ErrorAction SilentlyContinue  → (nenhum)
C:\Espressif exists=False
C:\Users\erick\esp exists=False
C:\Users\erick\.espressif exists=False
PS> Get-PnpDevice -PresentOnly -Class Ports
Porta de comunicação (COM1) ACPI\PNP0501\0
```

Não há projeto ESP-IDF em `embarcados/` (árvore em §1), não há IDF instalado e não há ESP32
conectado (só a COM1 da placa-mãe). B1 e B2 confirmados às 22h30 de 23/10.

---

## 6. Memória da p11 como está na referência — ESTIMATIVA ANALÍTICA

> **ESTIMATIVA ANALÍTICA** (categoria 1 do CLAUDE.md de `embarcados/`). Conta de
> bytes = elementos × tamanho do tipo, com F=4097, J=1201 e T, N da sondagem. Não é medição no
> PC, nem `idf.py size`, nem medição no ESP32. Ignora temporários do numpy (por exemplo, a
> matriz `y[idx]` de T×8192 em `stft_mag`, l.45–46) e overhead de alocador.

Buffers vivos de um segmento na referência, com os tipos que ela usa de fato:

| buffer | tipo | IDMT 3 s (N=48000, T=39) | A2212 1 s (N=16000, T=8) |
|---|---|---:|---:|
| `y` | float32 N | 187,5 KiB | 62,5 KiB |
| `S` (F×T) | float32 | 624,2 KiB | 128,0 KiB |
| `sal` (T×J) | float64 | 365,9 KiB | 75,1 KiB |
| `e_norm` (T×J) | float64 | 365,9 KiB | 75,1 KiB |
| `custo` (T×(J+1)) | float64 | 366,2 KiB | 75,1 KiB |
| `psi` (T×(J+1)) | int32 | 183,1 KiB | 37,6 KiB |
| `hilbert(y)` | complex64 N | 375,0 KiB | 125,0 KiB |
| saída do `sosfilt` | float64 N | 375,0 KiB | 125,0 KiB |
| **soma** | | **2842,9 KiB** | **703,3 KiB** |

Um porte "ingênuo" em float32 que mantenha a estrutura da referência (S inteiro, `e_norm`
float32, `psi` int32, Hilbert de N complexos, `y`) fica em **≈1553 KiB (3 s)** e **≈391 KiB
(1 s)**. Isso não cabe na SRAM interna de um ESP32 (ordem de 520 KB no total, menos o que o
IDF usa; dado de conhecimento geral, não verificado em chip), então exigiria PSRAM ou outra
estrutura.

**Porte por quadro** (mesma conta, buffers reorganizados): fixo ≈ 142 KiB = janela de 8192
float32 (32 KiB) + FFT real in-place (32 KiB) + tabela de Hann (32 KiB, ou calculada) + coluna
|S| (16 KiB) + `P_mean` acumulado (16 KiB) + saliência de um quadro (4,7 KiB) + `delta`/`novo`
(9,4 KiB). Três coisas **não** ficam por quadro sem mudar o algoritmo:

1. **Backtracking do Viterbi** exige `psi` de T×(J+1); em int16 (J+1=1202 cabe) são
   91,6 KiB (3 s) ou 18,8 KiB (1 s).
2. **Custo do SIL** usa `e_ref = max_t e_q[t]` do segmento inteiro (rotacao.py:179, 198, 96):
   o Viterbi só pode rodar depois do último quadro, logo `e_norm` precisa ficar guardado
   (T×J float32 = 183 KiB / 37,5 KiB) — ele também é necessário para a feature 3 depois do
   backtracking. Usar o máximo corrente (causal) é uma **variante nomeada**, não equivalência.
3. **Hilbert de N pontos** (l.51) é não causal no segmento; por quadro só com Hilbert FIR ou
   outra aproximação, também **variante nomeada**. Manter a referência exige N complex64
   (375 KiB / 125 KiB) — ou N float32 de `y` mais FFT in-place.

Para o A2212 (1 s), um porte equivalente fica em torno de 142 + 19 + 38 + 125 ≈ 324 KiB
(estimativa); para o IDMT (3 s), ≈ 142 + 92 + 183 + 375 ≈ 792 KiB, o que torna PSRAM ou
variantes nomeadas inevitáveis no segmento de 3 s.

---

## 7. Plano priorizado até 26/10/2026

O porte do colaborador compila limpo mas cobre só a parte barata do fim da cadeia. O
caminho crítico é o front-end, que não existe em C. Em ordem:

1. **(23–24/10, bloqueante) Vetores de ouro** por etapa com o wrapper (agente B), nos tipos da
   §2.0, com `n_logprob=−50`, `temp=1`, `change_penalty` da varredura, `min_dur=0`.
2. **(24/10) Front-end em C, equivalente à referência**: `stft_mag` (Hann simétrico, sem
   centralização), grade, soma harmônica com interpolação linear, emissão com `e_ref` do
   segmento, Viterbi de rotação com a ordem de varredura literal (−50..+50, +600, −600, SIL;
   `<` estrito; `<=` na linha SIL), `P_mean`. Depois, `_envelope_mod` (Hilbert por FFT de N,
   SOS dos coeficientes gerados em Python, `sosfilt` zi=0, dizimação 16, média, Welch
   reproduzindo os defaults do scipy 1.18). Precisa de uma FFT real de tamanho arbitrário
   (N=48000 e 16000 não são potências de 2) ou de uma escolha explícita.
3. **(24/10) Corrigir o fim da cadeia** com mudanças mínimas, cada uma registrada: z-score com
   `mu`/`sd` do `.bin` (D1), denominador `+1e-9` (D2), `log(p+1e-12)` e estado N com `n_logprob`
   (D10, D11), `round` meio-para-par (D4), planura como no Python (D6), `filtro_online` (D13).
   `absorb_short` pode ficar para depois: não é exercitado nos experimentos.
4. **(24/10, antes de comparar) Tolerâncias commitadas** (agente D), separando float32 × double
   (`P11_REAL`), e métricas discretas (`vit_path`, rótulos do HMM) por concordância exata.
5. **(25/10 até 12h) Equivalência no PC** com MSVC, primeiro em double, depois float32. Esse é
   o resultado que sustenta o artigo **sem** ESP32.
6. **(25/10) Decisão regular × WiP.** Com B1/B2 abertos, só há categorias 1 e 2 (estimativa e
   PC). Recomendo WiP de 4 pp se não houver ESP32 + IDF até a manhã de 25/10, e dizer no texto
   que nada foi medido no ESP32. Se a autorização do IDF sair, `idf.py size` (categoria 3) é
   barato e vale mais que tentar medir tempo às pressas.
7. **(25–26/10) Memória e custo** recalculados a partir do C real (por quadro, int16 no `psi`),
   com as variantes nomeadas (máximo corrente no SIL, Hilbert FIR) comparadas à referência só se
   houver tempo.
