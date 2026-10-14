# fsst_viterbi2015 — Synchrosqueezing com deslocamento de frequência + Viterbi para velocidade instantânea

## 1. Referência

Songtao Xi, Hongrui Cao, Xuefeng Chen, Xingwu Zhang, Xiaoliang Jin.
"A frequency-shift synchrosqueezing method for instantaneous speed estimation of rotating machinery".
*Journal of Manufacturing Science and Engineering* (ASME), v. 137, n. 3, art. 031012, jun. 2015.
DOI: 10.1115/1.4029824.

- Link: https://scholar.xjtu.edu.cn/en/publications/a-frequency-shift-synchrosqueezing-method-for-instantaneous-speed/
- Página do autor (mesmo resumo): https://faculty.xjtu.edu.cn/hrcao/zh_CN/article/279236/content/11706.htm
- PDF de acesso aberto: não encontrado (ASME, pago).

## 2. Leitura

**Só o resumo.** Texto completo não acessado.

## 3. Método (segundo o resumo)

Três etapas: (1) uma **transformada synchrosqueezing com deslocamento de frequência** (FSST) é aplicada ao sinal de
vibração para obter a representação tempo-frequência; (2) o **algoritmo de Viterbi** extrai a frequência instantânea
*deslocada* dessa representação; (3) a frequência deslocada é convertida de volta na frequência instantânea do sinal
medido, e dela a velocidade instantânea.

- Como é feito o deslocamento de frequência (multiplicação por exponencial complexa? quanto?): não encontrado.
  A motivação declarada é ganhar resolução em frequência e concentração TF, "especialmente para componentes de
  alta frequência".
- Wavelet-mãe/janela, escalas, parâmetros: não encontrado.
- Função de custo do Viterbi (emissão, penalidade de transição, número de estados): não encontrado.
- Tratamento de harmônicos e erro de oitava: não encontrado (o resumo fala de uma única frequência instantânea).
- Causalidade: Viterbi sobre o TFR inteiro sugere **offline**; não confirmado.
- Custo computacional: não encontrado.

## 4. Protocolo de avaliação

Simulações numéricas e experimentos (o resumo não diz qual máquina, se havia tacômetro, nem a métrica).
Comparação contra o synchrosqueezing padrão. Sem aprendizado, sem risco de vazamento no sentido de ML.

## 5. Números reportados

Nenhum número no resumo. Afirmações qualitativas: resolução em frequência "muito maior", melhor concentração TF,
estimativa de IF mais precisa que o SST, e menor sensibilidade a ruído, principalmente em componentes de alta
frequência. Valores numéricos: não encontrado.

## 6. Reimplementação

Teria de ser suposto praticamente tudo: o operador de deslocamento, a transformada base, o custo do Viterbi e os
parâmetros. **Veredito: só inspiração.**

Observação de contexto: Peeters et al. (2019) citam este trabalho como exemplo de FSST + Viterbi e deixam
synchrosqueezing de fora da comparação por haver dúvida, na literatura (Iatsenko et al.), sobre se ele melhora a
confiabilidade do rastreio em sinais ruidosos multicomponente em relação à STFT.

## 7. Relevância para o MotorSound

(a) **Rastreador offline.** É o antecedente direto mais citado de "Viterbi sobre representação TF para velocidade
de máquina rotativa". Isso define o que **não** é novidade no MotorSound: Viterbi para rotação já existe desde pelo
menos 2015. O que pode ser novidade, e precisa ser defendido contra esta referência, é o conjunto que vem da
transcrição musical: estado de silêncio (motor parado / sem harmônico confiável), custo explícito de oitava, emissão
multi-harmônica e aplicação a áudio de microfone. A ideia do deslocamento de frequência (trazer uma região de alta
frequência para onde a resolução é melhor) pode ser testada de forma barata com zoom-FFT/heterodinagem em torno de um
harmônico alto, como refino depois do caminho Viterbi.

(b) **ESP32.** Synchrosqueezing é caro (reatribuição por bin e quadro) e o resumo não dá custo; não é candidato para
o MCU. A heterodinagem + decimação, se for o que o "deslocamento" faz, é barata e embarcável.

(c) **Features normalizadas pela rotação.** Nada específico.
