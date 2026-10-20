# Contexto para estudo — Artigo 3 do Decifrae (MotorSound)

Documento para alimentar um chat de estudo. Reúne a situação do projeto, o que o orientador
pediu, as decisões técnicas já tomadas e o conteúdo a ser explicado. Gerado em 14/10/2026.

**Como ler este documento:** tudo que está marcado como *hipótese* ainda não foi verificado
nos dados reais. Os números de resultado que aparecem são dos módulos antigos (baixo e
violão), que já foram medidos. Da pipeline nova, **ainda não existe nenhum resultado em
dado real**; o código foi testado só em sinal sintético.

---

## Parte 1 — A situação

### 1.1 Quem é o Erick e o que é o Decifrae

Erick é estudante de Engenharia de Computação na UFMT (Campus Várzea Grande). O TCC dele é o
Decifrae, uma plataforma de transcrição musical (decifrae.com.br) com dois módulos de
pesquisa, cada um virando um artigo para o ACDSA 2027:

- **Artigo 1 — Baixo (cross-talk):** áudio de baixo → tablatura. Usa pYIN por corda +
  Viterbi + programação dinâmica para o dedilhado.
- **Artigo 2 — Violão (acordes):** áudio de violão → sequência de acordes. Usa chroma CQT
  sincronizado por batida + similaridade de cosseno + Viterbi.

Ele também tem experiência com o projeto BIMO (braço robótico, comparação de metaheurísticas
para cinemática inversa, artigo aceito no LARS 2026).

### 1.2 O Artigo 3 (MotorSound)

A ideia é levar a pipeline do Decifrae para outro domínio: **diagnosticar o estado de
operação de motores elétricos pelo som**, e no fim embarcar o resultado num microcontrolador
(ESP32) como um monitor preventivo de baixo custo.

- **Conferência alvo:** IEEE ICIT 2027 (International Conference on Industrial Technology),
  Florianópolis, 15–17/03/2027. Artigo regular de **6 páginas** incluindo referências.
- **Prazo de submissão: 26/10/2026**, prorrogado e marcado como prazo final (sem nova
  extensão). Work-in-progress (4 páginas) tem o mesmo prazo. Notificação em 11/12/2026,
  versão final e registro em 18/01/2027. Apresentação presencial obrigatória; ainda não foi
  decidido quem apresenta.
- **Trilhas que encaixam:** T6 (Signals, Image Processing, and Computational Intelligence) e
  T5 (Embedded Systems).
- **Fluxo planejado pelo Erick:** código em Python → testar na base e comparar com as análises
  existentes → revisar com o orientador → portar para C/C++ no microcontrolador.

### 1.3 A reunião com o orientador (15/10/2026)

Em reunião anterior, Erick e o orientador discutiram problemas ao tentar rodar a pipeline
antiga no novo problema:

1. **Que frequência e que transformada usar** — CQT (usada no violão) ou FFT/STFT.
2. **Como a divisão no tempo funcionaria** — o violão segmenta por batida (beat tracking), e
   motor não tem batida.
3. **Qual seria a nova pipeline.**

Para esta reunião, o orientador pediu que o Erick levasse:

- artigos que embasem academicamente a ideia;
- as soluções para os problemas da pipeline antiga no novo desafio;
- as mudanças na pipeline nova: o que foi feito e por quê;
- um código base claro, objetivo e fácil de entender;
- embasamento teórico ou revisão teórica para discutir melhorias.

O Erick precisa **entender muito bem** o que está sendo feito para conduzir a discussão, e não
só apresentar.

### 1.4 Estado atual do trabalho

- Um plano da pipeline foi escrito a partir da leitura do código real do baixo e do violão
  (`PLANO_PIPELINE_MOTOR.md`).
- Um esqueleto de código em Python existe (`motor_acoustic.zip`), com 7 testes passando
  **em sinal sintético**. Ele foi escrito antes da leitura do código do Decifrae e será
  ajustado conforme o plano.
- O Claude Code está rodando a **fase exploratória** na pasta do projeto: inventário das
  bases, fichas de literatura, núcleo comum, ~13 pipelines (reproduções da literatura e
  variantes do Decifrae), bancada de ruídos e um relatório de decisão. Os resultados ainda
  não saíram.

---

## Parte 2 — As bases de dados

### 2.1 IDMT-ISA-ELECTRIC-ENGINE (base principal)

Grollmisch, Abeßer, Liebetrau, Lukashevich, *Sounding Industry: Challenges and Datasets for
Industrial Sound Analysis*, EUSIPCO 2019 (Fraunhofer IDMT). Disponível no Zenodo e no Kaggle.

- 3 unidades idênticas de um motor **BLDC 2ACT 42BLF01** (4000 RPM, 24 V).
- 2378 arquivos wav, 44,1 kHz, 32 bit, mono. 774 good, 789 broken, 815 heavyload.
- Pastas `train/`, `test/`, `train_cut/`, `test_cut/`.
- Condições de fundo: pure, talking, white noise, atmo (fábrica) low/medium/high, stress test.

**Como as classes foram produzidas (ponto crítico):**

| Classe | Como foi gerada |
|---|---|
| good | motor a 60 % da tensão de alimentação |
| broken | tensão chaveando entre 15 % e 75 % **a cada 18 ms** |
| heavyload | 60 % da tensão **com carga extra** |

Nenhuma classe é um defeito mecânico real. São condições de alimentação e carga. Isso é bom
para a ideia (a rotação e a modulação dela separam as classes, e é isso que um rastreador de
pitch mede), mas o artigo precisa dizer abertamente que o sistema distingue condições de
operação, não falhas físicas.

### 2.2 BLDC A2212 (base secundária)

Estacio, Bravo Montenegro, Rengifo Rodas, *Dataset of audio signals from brushless DC motors
for predictive maintenance*, Data in Brief 50:109569, 2023. Mendeley Data
(doi 10.17632/j4yr5fmhv4.1).

- 43 wav de ~10 s a 16 kHz, quatro motores A2212 (de drone/aeromodelo).
- Classes: saudável, hélice quebrada, rolamento com falha. Velocidade variada entre gravações.
- Motores 1 e 2: saudável e hélice. Motor 3: só saudável. **Motor 4: só rolamento.**

Por que entra: tem defeitos mecânicos reais, o que responde à crítica de que o IDMT só mede
velocidade. Limitação séria: o rolamento está confundido com a identidade do motor 4. A
comparação limpa é saudável × hélice nos motores 1 e 2.

### 2.3 O que a literatura já obteve nessas bases

| Trabalho | Método | Resultado | Observação |
|---|---|---|---|
| Strantzalis et al., Sensors 22(24):9658, 2022 | log-mel 30×32 + CNN de 2 camadas, quantizada, num STM32 (Cortex-M4) | ~99,9 % no Python; 7,65 KB de flash, 5,52 KB de RAM, 40,8 ms de inferência | Sob ruído forte (stress test) a confiança cai para 54–61 % em algumas classes; troca de estado detectada em 4–9 s. Split aleatório de 25 % sobre features de recortes. |
| Grollmisch et al., EUSIPCO 2019 | baselines com rede neural | acurácia alta | Os próprios autores dizem que os baselines são muito sensíveis a mudanças na configuração de gravação. |
| ETNeXt, Scientific Reports 2026 | wavelet sym4 7 níveis + histograma ternário + seleção NCA/Chi2 + kNN/SVM | alta, com validação cruzada 10-fold | Usou IDMT e A2212, recortes de 3 s. |

**Leitura importante:** acurácia alta não é contribuição nessa base. O espaço aberto está em
robustez a ruído, tempo de detecção, interpretabilidade, custo embarcado e na honestidade do
protocolo (ver "vazamento", Parte 5.6).

---

## Parte 3 — As pipelines antigas (o que já existe e funciona)

### 3.1 Baixo — versões V1, V2 e V3

| Versão | O que faz |
|---|---|
| V1 | Um pYIN no sinal inteiro, banda B0–G4. Segmentação ingênua. |
| V2 | HPSS (margem 3,0) → 5 filtros passa-banda Butterworth SOS de ordem 8, um por corda → spectral gate (piso por quantil 0,10 × 1,2) → filtro de Wiener → pYIN por corda → limpeza heurística (voiced_prob, histerese de RMS, validação harmônica, correção de oitava) → DP de dedilhado → tablatura. |
| V3 | V2 com a limpeza heurística substituída por **Viterbi num HMM de pitch por corda**. |

**HMM da V3 (`v3_viterbi.py::_viterbi_pitch`):**

- Estados: silêncio + um estado por semitom, da corda solta até +24.
- Emissão: `score = 0,55 · prox · voiced_prob + 0,45 · e_norm`, onde
  `prox = exp(−(cents/70)²)` mede a distância entre o f0 do pYIN e o estado, e
  `e_norm` é a energia do espectro na fundamental mais metade da energia no 2º harmônico.
  Custo = −log(score).
- Transição: ficar = 0; passo de d semitons = 0,55·d; salto de oitava (±12, ±24) = 7,0;
  entrar ou sair do silêncio = 3,2.

**Resultado em áudio real (`Corda_Sol_G.mp3`, só a corda G tocada; nota abaixo de G2 é
impossível):** notas impossíveis 40,9 % (V1) → 14,8 % (V2) → 13,0 % (V3). Achado: a V2 já
resolve o erro de oitava pela estrutura de bandas; o resíduo da V3 é a mesma nota aparecendo
em bandas de cordas vizinhas (cross-talk), que é o tema do Artigo 1.

### 3.2 Violão — pipeline de acordes

```
áudio 22 050 Hz → HPSS (margem 3,0)
  parte percussiva → beat tracking → bordas dos segmentos
  parte harmônica  → chroma CQT (36 bins/oitava) → mediana temporal (21)
                   → agregação por batida (mediana)
→ cosseno contra 60 templates binários de acorde (+ penalidade de complexidade 0,06)
→ softmax (temperatura 0,1) → Viterbi em log com 61 estados (60 acordes + N)
   change_penalty = 2,0 fixo; força N quando RMS < 20 % da mediana da faixa
→ junta vizinhos iguais → absorve segmentos < 0,8 s
```

- **Lição de projeto:** uma matriz de transição estocástica normal dilui a penalidade de troca
  por todos os estados e faz o decodificador "travar" num acorde. A correção foi um custo de
  troca fixo, independente do número de estados.
- **Arquitetura:** extração cara (HPSS, beat track, CQT) separada da decisão barata (gate,
  emissão, Viterbi), para varrer hiperparâmetros sobre um cache.
- **Protocolo:** validação e teste separados por músico; configuração congelada antes de
  abrir o teste; comparações pareadas com bootstrap por faixa, Wilcoxon e correção de Holm.
- **Resultados de referência:** Tempo Perdido 78,8 % → 98,2 %; Boate Azul (root)
  84,7 % → 94,1 %.

---

## Parte 4 — Os problemas da reunião anterior e as soluções

### 4.1 CQT ou FFT? → STFT linear

1. **O som do motor é um pente harmônico linear.** Rotação f e harmônicos 2f, 3f, 4f… são
   igualmente espaçados em Hz. A STFT tem bins igualmente espaçados, então o pente cai
   regular sobre a grade e "somar energia nos harmônicos" vira somar bins em posições
   regulares. A CQT tem bins em escala logarítmica (fração de semitom), feita para notas
   musicais, que são espaçadas por razão (2^(1/12)). No motor essa escala não traz vantagem e
   complica a soma harmônica.
2. **O chroma do violão destrói a informação.** Chroma soma todas as oitavas em 12 classes.
   Para acorde isso é desejável; para o motor, 40 Hz e 80 Hz viram a mesma coisa, e a
   diferença entre good e heavyload é justamente a altura.
3. **O sinal é estacionário, então janela longa é permitida.** Resolução Δf = taxa/N.
   A 16 kHz com N = 4096: Δf ≈ 3,9 Hz, janela de 256 ms. Na música, janela longa borra
   mudanças de nota; no motor quase nada muda dentro do arquivo. Interpolação parabólica no
   pico refina abaixo de um bin.

*Hipótese a confirmar no espectro real:* 60 % de 4000 RPM ≈ 2400 RPM ≈ 40 Hz mecânicos; com
carga, talvez ~34 Hz. A diferença (~6 Hz ≈ 1,5 bin) é separável, mas por pouco. Em motor
BLDC, a frequência dominante pode ser a de comutação (rotação × número de pares de polos), e
não a mecânica; o espectro vai dizer.

A CQT pode entrar como variante na ablação, mas a escolha principal tem justificativa física.

### 4.2 Beat tracking → janelas fixas + HMM

No violão, o beat tracking respondia: *qual é a unidade de tempo em que o acorde é
constante?* Acordes mudam nas batidas.

No motor não há batida, o estado é constante por longos trechos, e uma troca é um evento raro
(o motor começando a falhar). A unidade de tempo vira uma **escolha de projeto**:

- janela curta → detecta a troca mais rápido, mas a decisão é ruidosa;
- janela longa → decisão estável, alarme atrasado.

Ponto de partida: **1 s com 50 % de sobreposição** (mesmo tamanho de Strantzalis 2022, o que
facilita comparar; contém ~40 rotações a 40 Hz).

O papel que a batida tinha (dizer onde algo muda) passa para o **Viterbi de estados do
violão**: ele só troca de estado quando a evidência acumulada supera o `change_penalty`. O
`min_dur` vira o tempo mínimo de confirmação antes do alarme. Surge uma métrica que não
existia no violão: **atraso de detecção**.

### 4.3 A nova pipeline

O motor é tratado como **um instrumento de uma corda só**:

- a camada contínua é o HMM de pitch do baixo V3, adaptado para rastrear a **rotação**;
- a camada discreta é o Viterbi do violão, adaptado para decidir o **estado de operação**.

```
wav 44,1 kHz → reamostrar 16 kHz → passa-altas 15 Hz (1 biquad) → normalização por RMS
│
├─ ETAPA CARA (cacheável)
│   spectral gate (do baixo V2)
│   STFT Hann 4096 / salto 1024 (64 ms entre quadros), magnitude comprimida (raiz)
│   saliência por soma harmônica: S(f) = Σ_h w_h·A(h·f) − λ·Σ_h w_h·A((h−½)·f)
│       8 harmônicos, w_h = 0,85^(h−1), λ = 0,5
│   (opcional, só offline) pYIN + voiced_prob como no baixo V3
│   Viterbi da rotação: estado de silêncio, custo por salto, custo de "oitava" f↔2f
│   refinamento parabólico → f0(t)
│   features por janela de 1 s (18 no total, 4 grupos):
│       rotação: f0 mediana, dispersão (IQR), inclinação, saliência e contraste do ridge
│       harmônicos: razão de energia nos harmônicos 1–4 e total
│       modulação: pico do espectro de modulação do envelope (frequência e força)
│       espectro: centróide, planicidade, energia em 4 bandas largas
│
└─ ETAPA BARATA
    classificador: cosseno contra protótipos de classe (do violão) ou regressão logística
    softmax → Viterbi de estados (offline) ou filtro forward (online, no MCU)
        change_penalty fixo; força "N" quando o RMS é baixo (motor parado)
    absorve estados curtos (min_dur) → alarme
```

**O que vem de onde e por quê:**

| Peça | Origem | Por quê |
|---|---|---|
| HMM de pitch com estado de silêncio e custo de oitava | baixo V3 | mesmo problema: rastrear uma nota com harmônicos fortes e fundamental fraca |
| Passa-banda SOS, spectral gate, validação harmônica | baixo V2 | isolar a faixa da rotação e tirar ruído de fundo |
| Soma harmônica com penalidade inter-harmônica | nova (literatura de pitch) | dispensa o pYIN no MCU; a penalidade em (h−½)·f elimina o erro de sub-oitava |
| Espectro de modulação do envelope | nova | o "broken" é uma modulação (chaveamento a cada 18 ms), um "tremolo" |
| Viterbi de estados com `change_penalty` fixo e `force_n` | violão | consolidar a decisão no tempo, sem o bug do decodificador travado |
| Cosseno contra protótipos | violão | barato no MCU; os templates viram médias de classe aprendidas |
| Extração cara × decisão barata, cache, congelamento, estatística pareada | violão | protocolo já validado |
| Distúrbios de Lee & Shen (2011) | baixo, fase 1 (`modulo_ruidos.py`) | eles vêm da qualidade de energia elétrica (sag, swell, flicker); o "broken" do IDMT é exatamente um distúrbio de alimentação |

**O que fica de fora e por quê:** bandas por corda e filtros de cross-talk (o motor é uma
fonte só); DP de dedilhado e tablatura (não há braço); chroma (dobra oitavas); beat tracking
(não há batida); os 60 templates de acorde.

**Para o microcontrolador:** saem o pYIN e o HPSS (caros). O rastreador embarcado usa só a
energia espectral na emissão. Estimativa analítica: ~131 kB de RAM e ~4 % de CPU num ESP32 a
240 MHz. *É estimativa, não medição.*

---

## Parte 5 — Revisão teórica (o que precisa ser entendido)

### 5.1 Fourier, STFT e resolução

- Transformada de Fourier discreta; bins igualmente espaçados em Δf = fs/N.
- STFT: janelas deslizantes; compromisso tempo-frequência (Δt · Δf ≥ constante).
- Janela de Hann e vazamento espectral; zero-padding **não** aumenta resolução real, só
  interpola.
- Interpolação parabólica do pico (estimar frequência entre bins).

### 5.2 CQT e chroma

- CQT: bins logarítmicos, Q constante (largura proporcional à frequência); janelas longas nos
  graves e curtas nos agudos.
- Chroma: soma da energia por classe de altura, ignorando a oitava.
- Por que serve para acorde e não para rotação.

### 5.3 Estimação de frequência fundamental

- Série harmônica; fundamental ausente ou fraca.
- Soma harmônica e produto harmônico; erro de oitava e de sub-oitava; penalidade
  inter-harmônica.
- YIN (função diferença, normalização cumulativa) e pYIN (limiares probabilísticos + HMM).

### 5.4 HMM e Viterbi

- Estados, emissões, transições; cadeia de Markov oculta.
- Viterbi como programação dinâmica em log-espaço; backtracking.
- Diferença entre Viterbi (melhor caminho, offline) e filtro forward (crença causal, online).
- Transição "pegajosa" e penalidade de troca; por que diluir a penalidade trava o
  decodificador.
- Os dois usos aqui: HMM contínuo (rotação, muitos estados) e HMM discreto (estado do motor,
  poucos estados).

### 5.5 Sinais de máquinas rotativas

- Frequência de rotação, harmônicos, frequência de comutação em BLDC (pares de polos).
- Modulação de amplitude e de frequência; envelope e espectro de modulação.
- Estimação de velocidade sem tacômetro (tacholess) e rastreamento de ordem.

### 5.6 Avaliação honesta

- Vazamento: recortes da mesma gravação em treino e teste inflam a acurácia.
- Split agrupado por gravação; validação separada do teste; congelamento.
- Bootstrap pareado, Wilcoxon, correção de Holm (o protocolo do violão).
- Métricas por condição de ruído; atraso de detecção; trocas espúrias.

### 5.7 Embarcado

- Restrições de RAM, flash e CPU de um ESP32; FPU de precisão simples.
- Filtros causais (`sosfilt`) versus não causais (`sosfiltfilt`).
- Custo de FFT (~N log N), da soma harmônica e do Viterbi com saltos limitados.

---

## Parte 6 — Artigos que embasam a ideia

**Estado de leitura:** só Strantzalis 2022 foi lido por inteiro. Os demais foram lidos pelo
resumo. As referências clássicas da Parte 6.4 vêm de conhecimento geral e os dados
bibliográficos devem ser conferidos antes de citar.

### 6.1 Bases e o que já foi feito nelas
- Grollmisch et al., *Sounding Industry*, EUSIPCO 2019 — origem do IDMT; baselines sensíveis
  a mudança de configuração.
- Strantzalis, Gioulekas, Katsaros, Symeonidis, *Operational State Recognition of a DC Motor
  Using Edge AI*, Sensors 22(24):9658, 2022 — CNN embarcada no STM32; principal comparação.
- Celik et al., *ETNeXt*, Scientific Reports 16, 2026 — wavelet + kNN/SVM em IDMT e A2212.
- *Acoustic-based fault diagnosis of electric motors using Mel spectrograms and CNNs*, 2025
  (PMC12835145).
- Estacio et al., Data in Brief 50:109569, 2023 — base A2212.

### 6.2 Rastreamento de rotação (sustenta o uso de pitch tracking + Viterbi)
- Peeters et al., *Review and comparison of tacholess instantaneous speed estimation methods
  on experimental vibration data*, MSSP 129:407–436, 2019 — rastreamento de harmônicos no
  espectrograma é a família dominante.
- *A frequency-shift synchrosqueezing method for instantaneous speed estimation of rotating
  machinery*, J. Manuf. Sci. Eng. 137(3), 2015 — usa **Viterbi** para extrair a frequência
  instantânea.
- Yoo et al., *Tacholess instantaneous speed estimation considering the characteristics of
  vibration harmonics*, PHM Society 2023 — combinar vários harmônicos.
- Wheat, *Correcting domain shifts in electric motor vibration data for unseen operating
  conditions*, arXiv 2504.10661, 2025 — Harmonic Feature Space, normalizar pela rotação.

### 6.3 Acústico e embarcado
- Vanaraj et al., *AI-powered acoustic analysis for non-intrusive fault detection in BLDC
  motors of EVs*, Proc. IMechE Part A, 2026 — features clássicas (ZCR, RMS, centróide, MFCC).
- Glowacz, *Acoustic-Based Fault Diagnosis of Commutator Motor*, Electronics 7:299, 2018.
- *TinyML for Acoustic Anomaly Detection in IoT*, ICECCME 2025 (arXiv 2603.26135) — ESP32.

### 6.4 Referências clássicas para a teoria (conferir dados antes de citar)
- de Cheveigné & Kawahara, YIN, J. Acoust. Soc. Am., 2002.
- Mauch & Dixon, pYIN, ICASSP 2014.
- Rabiner, tutorial de HMM, Proceedings of the IEEE, 1989.
- Brown, cálculo da CQT, J. Acoust. Soc. Am., 1991.
- Klapuri, estimação de múltiplas f0 por soma de amplitudes harmônicas, ISMIR 2006.
- Noll, harmonic product/sum spectrum, 1969.
- FitzGerald, HPSS por filtro de mediana, DAFx 2010.
- Müller, *Fundamentals of Music Processing* (livro).
- Randall, *Vibration-based Condition Monitoring* (livro) — envelope, rastreamento de ordem.
- Lee & Shen, 2011 — modelos de distúrbios de qualidade de energia (usados no
  `modulo_ruidos.py`).

### 6.5 Como cada grupo sustenta o argumento

| Afirmação do artigo | Quem sustenta |
|---|---|
| Som de motor é um sinal harmônico cuja fundamental é a rotação | Peeters 2019; Yoo 2023; Randall |
| Rastrear essa fundamental com Viterbi é método estabelecido | FSST+Viterbi 2015; Peeters 2019 |
| Técnicas de transcrição musical (pYIN, HMM) são o mesmo problema | Mauch & Dixon 2014; Klapuri 2006; Decifrae |
| Diagnóstico acústico embarcado é viável e relevante | Strantzalis 2022; TinyML 2025 |
| Há espaço em robustez e protocolo, não em acurácia bruta | Grollmisch 2019; Strantzalis 2022 (stress test) |

---

## Parte 7 — O código base

Duas peças existem, ambas na pasta `Artigo3(MotorSound)`:

1. **`motor_acoustic.zip`** — esqueleto em Python, testado só em sinal sintético:
   `config.py` (parâmetros), `io.py` (leitura das bases), `preprocess.py` (passa-altas, RMS,
   janelas), `ridge.py` (soma harmônica + Viterbi + refino), `features.py` (18 features),
   `hmm.py` (Viterbi de estados e filtro online), `evaluate.py` (métricas),
   `mcu_budget.py` (estimativa para o ESP32), scripts de exploração do espectro e de
   benchmark, e 7 testes.
2. **`PLANO_PIPELINE_MOTOR.md`** — como o esqueleto muda para reaproveitar o código real:
   o Viterbi de rotação passa a seguir o `_viterbi_pitch` do baixo; o HMM de estados vira
   cópia do `viterbi.decode` do violão; a configuração ganha o formato `PreConfig`/`PostConfig`.

Resultado em sinal sintético (fundamental propositalmente fraca, rampa de 35 a 55 Hz): os três
rastreadores (soma harmônica + Viterbi, argmax por quadro, pYIN) acertam 100 % dentro de 2 %
até 0 dB de SNR; a −6 dB o pYIN declara tudo como "sem voz". **O teste é fácil demais para
mostrar o ganho do Viterbi**; a diferença real deve aparecer nas condições ruidosas do IDMT.

---

## Parte 8 — Decisões para levar ao orientador

1. **Tema central do artigo.** Candidatos:
   (a) transferência de técnicas de transcrição musical para monitoramento de máquinas;
   (b) auditoria de vazamento — quanto da acurácia publicada nessas bases vem do split;
   (c) robustez e generalização entre condições (features físicas × CNN);
   (d) tacômetro acústico de baixo custo;
   (e) compromisso atraso de detecção × falso alarme com HMM online;
   (f) custo no ESP32 × acurácia;
   (g) detecção de anomalia treinada só no estado normal;
   (h) distúrbios de qualidade de energia (Lee & Shen) como bancada acústica;
   (i) calibração automática de hiperparâmetros (ligação com o BIMO).
2. **STFT como principal e CQT como variante?**
3. **Janela de 1 s com 50 % de sobreposição** como ponto de partida?
4. **pYIN só offline e soma harmônica no MCU** — aceitável perder o pYIN no embarcado?
5. **Bases:** IDMT como principal, A2212 como secundária com a limitação declarada?
6. **Prazo de 26/10:** artigo regular (6 páginas) ou WiP (4 páginas)? Medir no ESP32 antes do
   prazo ou deixar para a versão final de janeiro?
7. **Quem apresenta** em Florianópolis (presencial obrigatório).

## Parte 9 — Cuidados para não errar na reunião

- Não afirmar resultado da pipeline nova: ainda não há resultado em dado real.
- Os valores de ~40 Hz e ~34 Hz são estimativas a confirmar no espectro.
- Não dizer que o IDMT tem defeitos mecânicos: são condições de tensão e carga.
- A acurácia alta da literatura provavelmente tem vazamento de split; dizer "provavelmente",
  até a auditoria medir.
- Só Strantzalis 2022 foi lido por inteiro.
