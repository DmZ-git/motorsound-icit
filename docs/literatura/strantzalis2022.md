# strantzalis2022 — Reconhecimento do estado de um motor DC com Edge AI

## 1. Referência

K. Strantzalis, F. Gioulekas, P. Katsaros, A. Symeonidis. "Operational State Recognition of a DC Motor
Using Edge Artificial Intelligence". *Sensors* 22(24):9658, 2022. DOI 10.3390/s22249658.
MDPI: https://www.mdpi.com/1424-8220/22/24/9658 (bloqueou o acesso automatizado: "Access Denied")
Texto lido em PMC: https://pmc.ncbi.nlm.nih.gov/articles/PMC9783357/

## 2. Leitura

**Texto completo** (versão PMC).

## 3. Pré-processamento, parâmetros e arquitetura

Base: IDMT-ISA-ELECTRIC-ENGINE. A Tabela 2 lista todas as condições (pure 15 min, talking 18, atmo
high/medium/low 9 cada, white noise 9, stress test 6), ou seja, **usaram treino e teste oficiais juntos**.
Dizem que "a duração de cada amostra é 30 s" (não bate com a base, que tem gravações longas e recortes de 3 s).

Implementação A (ferramentas ST, X-CUBE-AI):
- reamostragem 44,1 → 16 kHz (librosa); FFT de 1024, hop 512; 30 filtros mel; log-mel **30 × 32**
  (32 colunas ≈ 1,024 s); normalização z-score.
- CNN Keras: Conv2D(16, 3×3, ReLU) → MaxPool 2×2 → Conv2D(16, 3×3, ReLU) → MaxPool 2×2 → Flatten(576)
  → Dense(9, ReLU) → Dense(3, softmax). Lote 500, 10 épocas. Quantização int8 (TFLite) → X-CUBE-AI.
- Custo no STM32 B-L475E-IOT01A (Cortex-M4): 501 428 MACC, 40,767 ms por inferência,
  RAM 5,52 KB, flash 7,65 KB (7,8 KB em outro trecho).
- No MCU: microfone MEMS MP34DT01 a 16 kHz, janela rolante de 1024 amostras com 50 %; uma coluna do
  espectrograma a cada 512 amostras; uma inferência a cada 1024 ms.

Implementação B (Edge Impulse):
- janelas de 1 s com 0,5 s de sobreposição; MFCC com FFT 256, quadro 20 ms, 32 filtros mel, 13 coeficientes.
- CNN 1D: reshape → Conv1D(30, k=5, ReLU) → MaxPool(5) → Conv1D(10, k=5, ReLU) → MaxPool(5) → Flatten →
  Dense softmax. Adam, entropia cruzada, 90 épocas. Modelo quantizado.
- Custo: RAM pico 7,0 KB, ROM 38,2 KB, 16 ms de inferência (+337 ms de DSP, 36 ms de classificação no ciclo real).

## 4. Protocolo de split e risco de vazamento

Implementação A: **25 % de teste sorteado aleatoriamente** sobre o conjunto de espectrogramas; dos 75 %
restantes, 25 % aleatório para validação. O teste tem 31 902 espectrogramas = 25 % do total → ~127 600
espectrogramas, muito mais do que a duração total (~75 min ≈ 4 400 janelas disjuntas de 1,024 s) permite;
logo as janelas se sobrepõem fortemente (passo não descrito) e janelas quase idênticas caem em treino e teste.
Implementação B: split interno do Edge Impulse (não descrito).

Teste em tempo real: gravações da base "não usadas no treino" concatenadas com trocas de estado,
reproduzidas por alto-falantes ao lado da placa (pure, atmo medium, white noise e um stress test próprio com
fala, ruídos impulsivos e música). Quais trechos foram reservados não está descrito.

**Risco de vazamento: alto** (split aleatório de janelas sobrepostas, misturando todas as condições;
mais a confusão classe = unidade física).

## 5. Números reportados

- A (Keras, teste aleatório, 31 902 janelas): treino 99,91 %, validação 99,87 %; matriz da Tabela 3 dá
  31 859/31 902 = **99,87 %**. Modelo C no MCU, 1 871 amostras (texto diz 1 901): 100 %.
- B (Edge Impulse): acurácia **97,8 %** (Tabela 5); quantizado 93,3 % vs 93,38 % não otimizado (em outro trecho);
  matriz de teste da Tabela 6: 1 495/1 528 = 97,8 %.
- Tempo real (Tabelas 9 e 10, % de janelas por classe verdadeira): A — good 100 %, broken 89,26 %,
  heavy load 88,84 %; B — good 89,06 %, broken 94,74 %, heavy load 95,62 %.
- Tempo de resposta às trocas de estado (média): **A 4,55 s, B 3,83 s**; pior caso 9,07 s
  (A, good→heavy load no stress test próprio).

## 6. O que falta para reproduzir; veredito

Não descrito: passo entre espectrogramas; quais arquivos entraram (cut ou longos) e se a seleção do teste
foi por arquivo; quais trechos foram para o teste em tempo real; parâmetros internos do Edge Impulse.

**Veredito: só inspiração.** A arquitetura A é totalmente descrita e serve como "baseline embarcável" de
referência de custo (RAM/flash/MACC/latência), e o experimento de troca de estado com tempo de resposta é
exatamente a métrica de sequência do projeto (atraso de detecção). O número de acurácia não é comparável.
