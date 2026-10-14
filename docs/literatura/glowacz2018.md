# glowacz2018 — Acoustic-Based Fault Diagnosis of Commutator Motor (MSAF-15-MULTIEXPANDED-8-GROUPS)

## 1. Referência

Adam Glowacz. "Acoustic-Based Fault Diagnosis of Commutator Motor". *Electronics* (MDPI), vol. 7,
n. 11, art. 299, 2018 (recebido 04/10/2018, publicado 05/11/2018). DOI: 10.3390/electronics7110299.
Link: https://www.mdpi.com/2079-9292/7/11/299

## 2. Leitura

**Texto completo** (PDF da MDPI, 21 páginas, extraído com pdftotext). A extração do layout foi parcialmente
corrompida em algumas figuras; as tabelas de resultado e a descrição do método foram lidas sem ambiguidade.

## 3. Pré-processamento, parâmetros e classificador

- Motor: motor de comutador (universal), 500 W, 3000 rpm, 230 V, 50 Hz, 1,84 kg.
- Cinco estados: saudável; bobina do rotor rompida; bobinas do estator em curto; dente quebrado na
  roda dentada (sprocket); trem de engrenagens danificado.
- Aquisição: microfone ZALMAN ZM-MIC1 / gravador digital, WAVE, mono, **44 100 Hz**. Resolução em bits: não encontrada.
- Pré-processamento: trilha cortada em arquivos de **1 s**; normalização de amplitude; janela de
  **32 768 amostras**; FFT → espectro de **16 384 bins** (resolução ≈ 1,35 Hz; o tipo de janela não é dito).
- Extração **MSAF-15-MULTIEXPANDED-8-GROUPS**:
  1. espectro médio por estado a partir de conjuntos de treino;
  2. diferenças absolutas entre os espectros de todos os pares de estados (10 pares para 5 estados);
  3. em cada diferença, tomar as 15 maiores componentes (MSAF-15);
  4. repetir para vários conjuntos de treino (6 conjuntos) e contar quantas vezes cada frequência aparece;
  5. manter as frequências que aparecem acima do limiar TCoF-TS. O texto escreve "TCoF-TS = 0.125
     (3/18 = 0.125)", o que é inconsistente (3/18 = 0,167); o valor efetivo é ambíguo;
  6. agrupar em até 8 grupos de componentes "essenciais" (critério de agrupamento descrito só por exemplo).
  - Resultado: **28 frequências** (Hz): 48, 50, 79, 81, 97, 101, 128, 157, 159, 1469, 1471, 1672, 1926,
    1927, 1934, 1935, 1939, 1942, 1953, 1957, 1958, 1961, 1978, 2038, 2039, 2042, 2059, 2547.
  - Vetor de features = amplitudes do espectro normalizado nessas 28 frequências.
- Classificadores:
  - NN (vizinho mais próximo) com **distância de Manhattan**;
  - NM (média mais próxima: protótipo = média dos vetores de treino da classe), Manhattan;
  - SOM de **144 nós** (MATLAB), outros hiperparâmetros não encontrados;
  - BNN (MLP com backprop) **28–100–5**, saída one-hot; função de ativação, taxa e épocas: não encontradas.

## 4. Protocolo de split e risco de vazamento

- Treino: **30 amostras de 1 s** (6 por estado, 6 "conjuntos de treino"). Teste: **500 amostras de 1 s**
  (as percentagens inteiras por classe sugerem 100 por estado; o texto não diz explicitamente).
- Não é dito se treino e teste vêm de gravações diferentes, nem de quantas gravações/motores físicos.
  Há um motor por estado (implícito pelas fotos); cada estado é, portanto, também uma unidade física
  ou uma sessão diferente.
- **Risco de vazamento: médio-alto.** Os segmentos de 1 s saem do "corte da trilha" e nada indica que
  treino e teste foram tirados de trechos/gravações separados. Pior: a seleção de frequências é feita
  sobre diferenças de espectros entre estados que são unidades/sessões diferentes, então as 28 linhas
  podem codificar a identidade da sessão (rotação exata, ruído de fundo) e não a falha. O fato de o
  treino ser pequeno (30 amostras) e o teste grande reduz o sobreajuste do classificador, mas não o
  vazamento por sessão.

## 5. Números reportados (base própria, não pública)

Eficiência total de reconhecimento TERCM (média das 5 eficiências por classe):

| Classificador | Saudável | Rotor | Estator | Sprocket | Engrenagem | TERCM |
|---|---|---|---|---|---|---|
| NN | 90 | 87 | 94 | 100 | 100 | **94,2 %** |
| NM | 89 | 91 | 93 | 100 | 100 | **94,6 %** |
| SOM | 87 | 86 | 81 | 90 | 98 | 88,4 % |
| BNN | 85 | 88 | 82 | 91 | 99 | 89,0 % |

## 6. Custo embarcado

Não há MCU. O autor só cita custo de hardware de "laptop + microfones por US$ 300". Estimativa nossa
(não do artigo): uma FFT de 32 768 pontos em float exige ~128 KB só para o buffer complexo, o que cabe
no ESP32 com PSRAM mas não folgado na SRAM interna; o classificador NM é trivial (5 protótipos × 28
floats = 560 B).

## 7. Para reproduzir / veredito

Base não pública. Para o método seria preciso supor: tipo de janela; como 44 100 amostras viram uma
janela de 32 768 (truncamento?); regra exata do passo "8 grupos"; o valor real do TCoF-TS; quais
segmentos formam cada "conjunto de treino". **Veredito: só inspiração** (não reproduzível nem na base original).

## 8. Relevância para o MotorSound

É o caso mais barato possível de "pico espectral + protótipo", muito próximo da nossa ideia de
reaproveitar o rastreador harmônico do Decifrae: em vez de frequências fixas, usar as harmônicas
da rotação rastreada. Entraria como variante de `p04_classicas` (`p04b_msaf`) ou como baseline:

- **extrair**: FFT de 1 s (no IDMT, 44,1 kHz; na A2212, 16 kHz → 16 384 pontos), magnitude normalizada.
- **treinar**: selecionar K bins (K = 28 ou K escolhido na validação) pelas maiores diferenças entre
  espectros médios de classe, **calculados só no treino de cada dobra**; classificador NM (Manhattan).
- **prever**: distância aos protótipos → softmax(−d) como pseudo-probabilidade.
- **custo_mcu**: FFT de 16 k pontos + 28 leituras + 3–5 distâncias; RAM dominada pela FFT.
- Teste crítico: com o split por gravação, as frequências selecionadas mudam entre dobras? Se mudam
  muito, é sinal de que estavam aprendendo a sessão, não a falha. Vale como figura no artigo.
