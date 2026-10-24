# Metodologia (artigo ICIT 2027, `embarcados/`)

> Gerado por `embarcados/src/python/gerar_artigo_textos.py` a partir de `paper/numeros.json`
> (commit 9df3b2c). Não editar à mão. Os valores entre parênteses vêm das macros de
> `paper/numeros.tex`, que trazem arquivo e coluna de origem. O histórico da fase exploratória
> está em `artigo-ICIT/METODOLOGIA.md`, `RESULTADOS.md` e `DECISAO.md` (somente leitura).

## Pipeline e analogia

A pipeline avaliada é a `p11_decifrae_harmsum`, herdada do transcritor de baixo e do
classificador de acordes do projeto Decifrae. O motor é tratado como um "instrumento de uma corda
só": a frequência de rotação faz o papel da fundamental, rastreada por uma soma de harmônicos e
decodificada por um Viterbi com estado de silêncio e custo de salto de oitava. A analogia motiva a
representação e não afirma que o som do motor seja uma nota musical nem que o desempenho em música
se transfira para máquinas.

Cada janela de análise (3 s no IDMT, 1 s no A2212, ambas a 16 kHz)
é normalizada em RMS (alvo 0,1) e processada sozinha. A STFT usa janela de Hann
simétrica de 8 192 amostras e salto de 1 024, sem centralização (39 quadros
em 3 s e 8 em 1 s). Para cada uma das 1 201 frequências
candidatas, numa grade logarítmica de 15 a 60 Hz com passo de 2 cents,
a saliência soma 48 harmônicos com interpolação linear entre bins e é normalizada por
quadro. O custo de emissão é −log(0,45·ẽ + 10^{-3}), e o estado de silêncio recebe um
custo que depende da energia do quadro relativa ao quadro mais forte da janela. O Viterbi admite
saltos de até 50 estados (100 cents) com custo de 0,55 por
semitom, saltos de oitava de ±600 estados com custo 7,0 e entrada ou saída do
silêncio com custo 3,2. Todos esses valores foram lidos do código Python da referência por
import (`PreDecifrae`, `motorsound.rotacao`, `_envelope_mod`), não de comentário nem do C.

Do caminho de rotação e do espectro médio saem 13 features, na ordem da referência:
mediana e IQR de f_rot, fração de quadros em silêncio, saliência média no caminho, razão harmônica,
energia relativa de quatro grupos de ordens harmônicas, frequência e proeminência do pico de
modulação do envelope (Hilbert, Butterworth de ordem 4 em 200 Hz com
`sosfilt` causal, dizimação por 16, Welch, pico entre 2 e 100 Hz),
centróide e planura espectral. As features são padronizadas com média e desvio do treino e
comparadas por cosseno a um protótipo por classe; um softmax com temperatura 0,1 dá as
probabilidades. Nas sequências, um HMM com penalidade de troca fixa e um estado extra "N" roda como
filtro forward causal, uma janela por vez (6 penalidades varridas, de 0,5 a
16; log-probabilidade do estado N igual a -50).

## Porte em C

O porte segue o Python linha a linha e compila sem alteração no PC e no ESP-IDF. Ele nunca guarda
o espectrograma inteiro: cada quadro é janelado, transformado por uma FFT real, acumulado no
espectro médio e reduzido à sua linha de saliência. O Viterbi precisa da janela inteira, porque o
custo do silêncio usa a energia máxima da janela; por isso a saliência normalizada e os
ponteiros de retorno ficam guardados. O envelope de Hilbert precisa de uma FFT do tamanho da
janela, que não é potência de dois, e o porte inclui uma FFT de raiz mista. Toda a memória é uma
arena fornecida pelo chamador, cujo tamanho sai da mesma função de layout que distribui os
ponteiros. O tipo real é escolhido na compilação (float32 para o microcontrolador, double para a
verificação), e as decisões que mudam inteiros (grade, medianas, arredondamento das posições
harmônicas) são feitas em double nos dois builds, reproduzindo o arredondamento meio-para-par e o
desempate pelo primeiro índice da referência. A única aproximação usada é a variante nomeada
`p11_c_sosd`, que mantém em double o estado do filtro do envelope dentro do build float32.

## Bases, protocolos e métricas

Usamos só o `train/` do IDMT (3 gravações de 3 motores,
357 janelas; o teste oficial não foi aberto) e o A2212 inteiro
(43 arquivos de 4 motores, 22 Healthy,
14 Propeller e 7 Bearing; 430 janelas).
Todos os métodos usam exatamente as mesmas janelas e os mesmos folds. No IDMT comparamos o split
aleatório por janela (vazamento intencional, 18 grupos) e o split por blocos
contíguos (15 grupos), que ainda não separa classe de motor porque há uma gravação
por classe. No A2212 comparamos o split aleatório, o split agrupado por arquivo
(43 grupos), o motor fora e o motor fora sem o fold do M4
(36 grupos), porque o rolamento só existe no motor M4. A tabela
`tables/latex/bases_protocolos.tex` resume o que cada protocolo permite concluir.

Os baselines são o de nível (RMS e pico com regressão logística, com e sem normalização de RMS) e o
de features clássicas p04 (32 estatísticas de ZCR, RMS, centróide e MFCC com voto
suave), inspirado no resumo de Vanaraj et al. (ficha `docs/literatura/vanaraj2026.md`), que só foi lido no resumo. As métricas são
acurácia balanceada, F1 macro, ROC-AUC macro um-contra-todos com os scores contínuos e
sensibilidade por classe. Os intervalos de 95 % vêm de um bootstrap que reamostra
grupos (blocos ou arquivos), estratificado por classe, com 2 000 reamostragens (semente
20261023). No protocolo de motor fora a unidade independente é o motor, e com
4 motores não há intervalo honesto nessa escala; o intervalo por arquivo é otimista.

## Protocolo de equivalência

A referência instrumentada gerou vetores de ouro de todas as etapas em 164 janelas:
60 limpas da validação das duas bases, 90 ruidosas (5
perturbações: ruído branco em três SNRs e os distúrbios sintéticos de afundamento e cintilação) e
14 casos de borda (silêncio, clipping, rotação sintética fora da faixa e nível constante
com ruído mínimo). As etapas do HMM foram conferidas em 20 sequências de probabilidade,
6 penalidades e 2 durações mínimas (240 configurações). As
tolerâncias de cada etapa vêm de uma análise de arredondamento de primeira ordem e de uma emulação
numérica independente de cada build; foram commitadas no commit `447a7c2` (2026-10-24 09:10,
sha256 `d74dae800ceb…`) antes da comparação formal (2026-10-24 09:11), e o hash vai em cada linha do
resultado. As duas emulações passaram em todas as linhas (6 852 e 6 852
de 6 852) antes de o C ser comparado. Etapas contínuas são comparadas pelo erro
máximo relativo à escala da etapa; saídas discretas (caminho de rotação, decisão, estados do HMM)
têm de coincidir exatamente, salvo empate demonstrado com os valores da própria referência. Uma
falha numa etapa marca as seguintes como "a jusante". Uma ressalva de cronologia: o porte em C foi
depurado contra os vetores de ouro pelo agente C antes do congelamento (commit do porte anterior
ao das tolerâncias); o "pré-registro" vale para a comparação formal do agente D.

## Categorias de custo

Separamos quatro categorias, também nos nomes das macros. A estimativa analítica (`VEst…`) é o
tamanho da arena devolvido pela função de layout do C (`p11_host ws <n>`). A medição no PC
(`VPc…`) cronometra 200 janelas por base depois de 5 de aquecimento, em ciclo sobre
75 janelas distintas, dentro do executável e em torno das chamadas do núcleo
(MSVC 19,44 x64 \slash{}O2 \slash{}fp:precise, Windows 11, 16 CPUs lógicas); a referência Python é cronometrada nas mesmas
janelas. O build do firmware (`VBuild…`) é o `idf.py size` para ESP32 e ESP32-S3 (ESP-IDF
v5,5.6, GCC 14,2.0), só estático. A medição no ESP32 (`VChip…`) manda as janelas pela
serial e mede com o temporizador do chip só em torno do processamento, junto com as marcas de heap
e de pilha. Só essa última categoria pode ser chamada de "medido no ESP32", e ela ainda está
pendente. Energia não é medida e não é estimada por datasheet.
