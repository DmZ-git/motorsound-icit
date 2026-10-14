# grollmisch2019 — Sounding Industry (artigo da base IDMT-ISA-ELECTRIC-ENGINE)

## 1. Referência

S. Grollmisch, J. Abeßer, J. Liebetrau, H. Lukashevich. "Sounding Industry: Challenges and
Datasets for Industrial Sound Analysis". *27th European Signal Processing Conference (EUSIPCO)*,
A Coruña, 2019. ISBN 978-9-0827-9703-9.
PDF: https://new.eurasip.org/Proceedings/Eusipco/eusipco2019/Proceedings/papers/1570526697.pdf
Base: https://www.idmt.fraunhofer.de/en/publications/datasets/isa-electric-engine.html (Zenodo 10.5281/zenodo.7551261)

## 2. Leitura

**Texto completo** (PDF de 4 páginas, extraído com pdftotext em 14/10/2026).

## 3. Pré-processamento, parâmetros e arquitetura

- Taxa: mantida em 44,1 kHz. O artigo diz 32 bit; na verificação local os wav de `train/`/`test/` são PCM 24 bit.
- Entrada: quadros de magnitude da STFT, janela de Hann, 50 % de sobreposição, fase descartada.
  A coluna "Input" da Tabela I é "metade do tamanho final da janela": para a base do motor,
  Input = 2048, logo **janela = 4096 amostras (~93 ms), hop = 2048 (~46 ms)**. Escala linear (não há log nem mel).
- Normalização: divisão pelo máximo dos dados de treino, para [0, 1]; o mesmo valor é aplicado a validação e teste.
- Classificador por quadro: DNN totalmente conectada **2048 → 256 → 64 → 16 → 3**, ReLU nas ocultas, softmax na saída,
  dropout 0,2 (entrada), 0,5 (ocultas), 0,2 (antes da saída). Inicialização Glorot uniforme.
- Treino: Adam, lr = 0,001, entropia cruzada categórica, **1000 épocas**, Keras/TensorFlow.
- Hiperparâmetros (camadas, unidades, dropout, tamanho da janela) escolhidos por otimização bayesiana
  (github.com/fmfn/BayesianOptimization).
- Decisão por arquivo (A_file): voto majoritário sobre as decisões por quadro.

## 4. Protocolo de split e risco de vazamento

- Treino **só com a condição `pure`** (pasta `train/`); todas as condições com ruído só no teste (split oficial).
- Validação: 10 % aleatório dos dados de treino, 90 % para treinar; repetido 5 vezes com embaralhamento diferente
  ("cross-validation folds"); o número reportado é a média das 5 repetições.
- Como `train/` tem **uma gravação por classe**, a validação aleatória é por quadro, dentro da mesma gravação:
  a validação vaza (quadros vizinhos, com 50 % de sobreposição, dos dois lados). Isso afeta a escolha de
  hiperparâmetros, não o teste.
- Teste: gravações distintas (`test/`), sem sobreposição temporal com o treino.

**Risco de vazamento: médio.** Não há vazamento de segmento entre treino e teste. Mas cada classe é uma
unidade física diferente (engine1_good, engine2_broken, engine3_heavyload), então o teste mede "reconhecer
o motor" tanto quanto "reconhecer o estado". A validação interna é otimista (quadros da mesma gravação).

## 5. Números reportados (Tabela II, IDMT, teste oficial)

| Condição de teste | A_frame | A_file |
|---|---|---|
| talking | 95,1 % | 96,7 % |
| white_noise | 99,3 % | 100 % |
| atmo_low | 100 % | 100 % |
| atmo_medium | 99,9 % | 100 % |
| atmo_high | 94,7 % | 100 % |
| stress test | 82,3 % | 86,7 % |

Acaso = 33,3 %. Os autores observam que os sons dos motores "variam mais do que em casos reais" e que o
stress test (ganho alterado, clipping, batidas na caixa) derruba a acurácia abaixo de 90 %.

## 6. O que falta para reproduzir; veredito

Não descrito: tamanho de lote; se há early stopping (diz 1000 épocas fixas); espaço de busca da otimização
bayesiana; se o voto do A_file foi feito sobre os arquivos longos de `test/` ou sobre os recortes de 3 s
(`test_cut/`) — os valores de A_file com 1/3 e 2/3 de passo (96,7 %, 86,7 %) sugerem recortes, mas isso não
está escrito; como as quatro gravações `talking_1..4` foram agregadas numa linha só (não encontrado).

**Veredito: reproduzível** (com suposições menores: lote, A_file sobre `test_cut/`, média das 5 sementes).
É a linha de base natural do projeto e a única referência com o split oficial e números por condição.
