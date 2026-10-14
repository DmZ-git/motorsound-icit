# peeters2019 — Revisão e comparação de estimação de velocidade instantânea sem tacômetro

## 1. Referência

Cédric Peeters, Quentin Leclère, Jérôme Antoni, Peter Lindahl, John Donnal, Steven Leeb, Jan Helsen.
"Review and comparison of tacholess instantaneous speed estimation methods on experimental vibration data".
*Mechanical Systems and Signal Processing*, v. 129, p. 407–436, 2019. DOI: 10.1016/j.ymssp.2019.02.031.

- Link pedido (emsg.mit.edu): https://emsg.mit.edu/dev_site/wp-content/uploads/2024/05/77_paper.pdf — retornou HTTP 403.
- Versão lida: manuscrito aceito (author's final manuscript), MIT DSpace, https://hdl.handle.net/1721.1/130338 (CC BY-NC-ND).
- Dados/estimativas publicados: https://data.mendeley.com/datasets/s8jrrnnj9g

## 2. Leitura

**Texto completo** (manuscrito aceito, extraído do PDF para texto). As figuras não foram vistas, só as legendas;
por isso os números de erro, que só aparecem em gráficos (Fig. 10, 20, 11, 21), **não foram extraídos**.

## 3. Método

É uma revisão com comparação experimental de sete métodos, todos aplicados a vibração (acelerômetro), não a som.
Synchrosqueezing foi deliberadamente excluído (citam dúvida da literatura, Iatsenko et al., sobre se ele aumenta a
confiabilidade em sinais ruidosos multicomponente). Os sete métodos:

1. **Demodulação de fase** (sinal analítico). Passa-banda ideal por FFT em torno de um harmônico de alta SNR, sinal
   analítico, `φ(t) = Im(log x_a(t))`, desenrolar a fase, `f_inst = (1/2π) dφ/dt`. Para sinais longos com grande
   variação, processam por janelas retangulares com 50 % de sobreposição e combinam com média ponderada por Hanning.
   Limite de variação de velocidade para o filtro do n-ésimo harmônico não invadir os vizinhos:
   `Δθ̇_max = f_shaft / (2n + 1)` (Eq. 4).
2. **Demodulação iterativa**: estima com harmônico baixo, reamostra no domínio angular, e demodula um harmônico mais
   alto com banda menor. Ganho satura após 1–2 iterações.
3. **TKEO** (Randall & Smith): `Ψ(x) = ẋ² − x ẍ ≈ ω² A²`; com envelope quadrático `se_x = x² + x̂²` e o envelope do
   derivado `se_ẋ`, obtém-se `ω(t) = sqrt(se_ẋ(t) / se_x(t))` (Eq. 13–14; a raiz quadrada se perdeu na extração de
   texto mas decorre de Eq. 13). Derivação feita por multiplicação por jω no espectro, junto com o passa-banda.
4. **MOPA** (multi-order probabilistic approach, Leclère et al. 2016). STFT; o espectro (branqueado, para atenuar
   ressonâncias) é tratado como pdf da velocidade: `p(Ω|H_i) = A(H_i ω)/ξ_i` em `[Ω_min, Ω_max]`, normalizado para
   área unitária; acima de Nyquist a pdf é uniforme (Eq. 18–19). As pdfs de todas as ordens H_i são **multiplicadas**.
   Prior de continuidade: a pdf no instante j é o produto das pdfs de j−K…j+K, cada uma convoluída com Gaussiana de
   desvio `σ_k = |γ k Δt|` (γ = aceleração típica). Estimativa final = **valor esperado** da pdf (não caminho MAP).
5. **ViBES** (MIT, inédito antes deste artigo). Usuário define faixas relativas `R = {r_i}` em torno de componentes
   fortes; a cada instante as faixas são escaladas pela estimativa anterior `f̂_{t−Δt}`; o módulo da STFT em cada
   faixa vira pdf, interpolada para a grade da faixa mais alta, e as pdfs são multiplicadas (independência); estimativa
   = valor esperado. Adaptações: (a) **regiões ilegais** Z (ex.: 58,5–61,5 Hz e 178,5–181,5 Hz, zumbido da rede) onde
   a faixa é descartada; (b) **modo espera/rastreio** com histerese sobre a variância da pdf composta
   (σ²_l = 0,2 Hz², σ²_h = 0,4 Hz² no gerador), saída vazia em espera e retorno a um palpite f_w.
6. **MOPA no cepstrograma** (STCT = IFFT de ln|STFT|), inspirado em Bonnardot (Surveillance 8). A escala fica
   invertida (quefrência ∝ 1/velocidade).
7. **Rastreamento de máximo + filtro de Vold-Kalman** (Schmidt et al. 2018). Custo por quadro
   `κ = −|STFT(n,k)|² + ρ1·max[0,(kΔf − f_max(n−1))² − Δf_c1²] + ρ2·max[0,(kΔf − f_IF,w(n))² − Δf_c2²]`, onde
   `f_IF,w` é a extrapolação de um polinômio de grau 1 ajustado por mínimos quadrados nos últimos N_m = 5 pontos;
   minimizado por força bruta. Depois um VKF (angular-displacement, ordem 2) como passa-banda variante no tempo, e
   demodulação de fase.

Harmônicos e erro de oitava: não há tratamento explícito de "oitava". O MOPA/ViBES lidam com ambiguidade de
harmônico pela multiplicação das pdfs de várias ordens (um pico só sobrevive se coerente em todas); os métodos de
um harmônico dependem da escolha manual do harmônico e de `ω_init`.

Causalidade: demodulação (filtro FFT ideal sobre janelas), TKEO, MOPA (usa ±K janelas) e VKF são **offline**.
ViBES e a parte de rastreamento de máximo são **causais** quadro a quadro (dependem só do passado), mas a etapa
VKF posterior não.

Custo: tempos medidos em CPU i5-5300U, um núcleo, mostrados só em figura (Fig. 11 e 21); o texto diz que o mapa de
custo do rastreamento de máximo é o principal custo no Surveillance 8 e que métodos de STFT exigem ≥ 3 parâmetros
só para a STFT. Tabela 1 lista os parâmetros de cada método (de 3 a 11).

## 4. Protocolo de avaliação

Três bases experimentais, sem aprendizado:

- **CMMNO 2014** (caixa de engrenagens de turbina eólica): 20 kHz, ~550 s, referência por encoder angular.
  STFT: janela uniforme 1 s, FFT 2 s, 95 % de sobreposição (Tabela 3 traz N_w = 5000, N_FFT = 10000). MOPA com as 10
  primeiras harmônicas de cada ordem fundamental da Tabela 2, K_w = 20, γ = 0,4 Hz/s, intervalo {15, 35} Hz.
- **Surveillance 8** (motor aeronáutico Safran): referência por tacômetro; harmônico 38 do eixo HP para demodulação,
  janela de 1 s, banda 100 Hz; MOPA com 60 harmônicos de HP e L1, {175, 230} Hz.
- **Gerador de navio** (Detroit Diesel 3-71, 5 kHz, ~200 s, com parada total e partida de 0 a 55 Hz em ~4 s):
  **sem referência**, só avaliação visual.

Métrica: erro absoluto médio e mediano em relação ao encoder/tacômetro; tempo de execução.
Risco de vazamento: não há treino, mas **os parâmetros foram ajustados à mão em cada base, olhando a própria base**
(ex.: o harmônico de 55 Hz do CMMNO foi escolhido "devido aos bons resultados obtidos com ele"; `ω_init` obtido por
inspeção visual do espectrograma). Os números são, portanto, otimistas para uso automático.

## 5. Números reportados

Valores numéricos de erro: **não encontrados no texto** (só nas Fig. 10 e 20). Conclusões qualitativas do texto:
no CMMNO o melhor é o MOPA, seguido de perto pela demodulação iterativa e pelo máximo+VKF; o ViBES fica "um pouco
pior" que o MOPA; o MOPA por cepstro é o pior em todas as bases. No Surveillance 8 os métodos de demodulação de fase
são os melhores (harmônicos 38 e 75 muito limpos); o TKEO tem mediana baixa mas média alta por um erro perto de
105 s. No gerador, MOPA rastreia contínuo mas dá valores sem sentido na parada; demodulação, TKEO e máximo+VKF
precisam ser rodados em duas partes. A taxa citada por Yoo et al. (2023) para MOPA no Surveillance 8 é outra
medida, de outro artigo.

## 6. Reimplementação

Para reimplementar MOPA, ViBES, TKEO e demodulação há equações e tabelas de parâmetros suficientes. Teria de ser
suposto: o método de branqueamento do MOPA ("deve ser escolhido conforme a aplicação"), detalhes do VKF (remetido a
referências), e o valor exato de ρ1, ρ2. Os números publicados não podem ser conferidos sem ler os gráficos e sem as
bases CMMNO/Surveillance 8.

**Veredito: só inspiração** para o MotorSound (é vibração, e a comparação de números depende de figuras e de bases
que não temos). Como fonte de *baselines* de rastreamento (demodulação de fase, MOPA, máximo com continuidade) é
**reproduzível** no nível do algoritmo.

## 7. Relevância para o MotorSound

(a) **Rastreador offline.** O MOPA é a formulação probabilística mais próxima do nosso Viterbi: emissão =
produto (soma de log) das pdfs `A(H_i ω)` sobre várias ordens, o que já pune erro de oitava/sub-harmônico porque um
candidato a f0 errado não acumula energia em todas as ordens. Dá para usar `log p(Ω|H_i)` normalizado como emissão do
nosso Viterbi e trocar o prior Gaussiano de continuidade (σ = γΔt) por transição de Viterbi equivalente; a diferença
de saída (valor esperado × caminho MAP) vira um experimento direto. A demodulação de fase / iterativa serve de
**refino sub-bin** depois do caminho grosso do Viterbi (o Viterbi resolve "qual harmônico", a fase resolve a
precisão). O MOPA e o rastreamento de máximo são *baselines* naturais para a tabela de comparação.

(b) **ESP32.** O ViBES é causal e barato: faixas escaladas pela estimativa anterior, produto de poucas pdfs e
valor esperado. Duas ideias valem diretamente: as **regiões ilegais** (descartar 50/60 Hz e harmônicos da rede,
fonte óbvia de falso rastreio em motor elétrico) e o **modo espera/rastreio com histerese na variância**, que é o
análogo do nosso estado de silêncio. O TKEO (`ω = sqrt(se_ẋ/se_x)`) é O(N) após um passa-banda e cabe no MCU, mas
sofre com ruído em alta frequência por causa da derivada.

(c) **Features normalizadas pela rotação.** O artigo motiva a reamostragem angular (ordem) a partir da velocidade
estimada como objetivo principal da estimação; não propõe features. O alerta útil: em mudanças rápidas de
velocidade os harmônicos borram na STFT e qualquer normalização por rotação herda o erro.

Para a novidade: o artigo cita Viterbi em TFR para velocidade (Xi et al. 2015, ficha `fsst_viterbi2015`) e HMM para
rastreamento de frequência (Barrett & Holdsworth 1993). "Viterbi para rotação" sozinho não é novidade.
