# melcnn2025 — Mel-espectrograma + CNN leve no IDMT

## 1. Referência

H. Uzel, Y. Özüpak, F. Alpsalaz, E. Aslan, I. Zaitsev. "Acoustic-based fault diagnosis of electric motors
using Mel spectrograms and convolutional neural networks". *Scientific Reports* 16:3379 (online 23/12/2025).
DOI 10.1038/s41598-025-33269-z. PMC12835145: https://pmc.ncbi.nlm.nih.gov/articles/PMC12835145

## 2. Leitura

**Texto completo** (PMC).

## 3. Pré-processamento, parâmetros e arquitetura

- Base: IDMT, **2 378 wav** (774 good, 789 broken, 815 heavy load). Esse total coincide com o total de
  recortes de 3 s do IDMT (`train_cut/` + `test_cut/`, todas as condições juntas). O texto diz duração total de
  42 min 32 s, o que não bate com 2 378 × 3 s ≈ 119 min (inconsistência do artigo).
- Mel-espectrograma: FFT 2048, hop 512, 128 bandas mel, depois **redimensionado para 32 × 32**.
  As figuras falam em eixo de 0 a 16 000 amostras "provavelmente reamostrado" — a taxa usada na prática
  não está clara.
- CNN: Resize(32×32) → Normalization → Conv(32, 3×3) → Conv(64, 3×3) → MaxPool 2×2 → Conv(128, 3×3) →
  Dropout (0,25–0,5) → … → Dense → softmax; 2 403 861 parâmetros. Adam, entropia cruzada esparsa,
  10 épocas, lote 32, early stopping (paciência 5).
- Linhas de base: kNN (k = 5) + MFCC 39-d (13 + Δ + ΔΔ, librosa); MLP 128→64 + MFCC; ResNet-18
  (ImageNet), CRNN (Conv + BiGRU), Transformer de espectrograma (6 camadas, 384-d).
- Custo declarado: 48–50 ms/amostra no Raspberry Pi 4; "150 MB de memória".

## 4. Protocolo de split e risco de vazamento

Declaram: arquivos agrupados "por sessão de gravação e unidade de motor", depois split **70/15/15 por
arquivo**; além disso 5-fold estratificado sobre treino+validação, repetido com 10 sementes, teste fixo.
O agrupamento descrito é impossível como escrito: cada classe é **uma** unidade física, então agrupar por
unidade poria cada classe inteira num só lado. Como cada "arquivo" é um recorte de 3 s, o split por arquivo é
de fato um split por recorte: recortes contíguos da mesma gravação ficam em treino e teste.

**Risco de vazamento: alto.**

## 5. Números reportados (IDMT, teste do split 70/15/15)

| Modelo | Teste | 5-fold (média ± dp) |
|---|---|---|
| CNN proposta (32×32) | **99,7 %** | 99,5 ± 0,3 % |
| ResNet-18 | 98,4 % | 98,2 ± 0,5 % |
| CRNN | 98,9 % | 98,7 ± 0,4 % |
| Transformer | 98,1 % | 97,9 ± 0,6 % |
| MLP + MFCC | 93,2 % | 92,9 ± 0,6 % |
| kNN + MFCC | 91,8 % | 91,5 ± 0,7 % |

Matriz de confusão do teste: 122/116/119 acertos, 1 erro (heavy load → good). Sensibilidade: 256 mels 99,9 %,
FFT 4096 99,8 %. Robustez a ruído (AWGN a 20/10/0 dB): experimento descrito, **números não encontrados**.
Transferência para Paderborn (base de rolamentos de vibração/corrente, "aumentada com ruído acústico"):
88,5 % sem e 92,3 % com fine-tuning.

## 6. O que falta para reproduzir; veredito

Faltam: camadas finais exatas da CNN (o texto corta após o dropout); taxa de amostragem efetiva; como os
2 378 recortes foram agrupados; resultados de ruído. O artigo tem várias inconsistências internas
(duração total, "150 MB" para 2,4 M parâmetros, descrições de figuras incoerentes).

**Veredito: só inspiração.** O número de 99,7 % não é comparável. Aproveitável: a linha de base
kNN + MFCC 39-d (k = 5), que dá ~92 % mesmo com vazamento e é barata o bastante para o ESP32.
