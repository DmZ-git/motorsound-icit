# etnext2026 — ETNeXt (wavelet + padrões ternários + NCA/Chi2 + kNN/SVM)

## 1. Referência

B. Celik, E. Taskin, A. Akbal *et al.* "ETNeXt: integrated feature engineering and classification framework
for BLDC motor fault detection". *Scientific Reports* 16, 11820 (2026). DOI 10.1038/s41598-026-37590-z.
https://www.nature.com/articles/s41598-026-37590-z (também PMC13065822). Recebido 10/09/2025, publicado 03/03/2026.
Código/dados indicados num link do Google Drive no próprio artigo.

## 2. Leitura

**Texto completo** (HTML da Nature). Tabelas numéricas (3, 4, 6, 7, 8) são imagens/links e não foram abertas
uma a uma; os números abaixo vêm do texto corrido.

## 3. Pré-processamento, parâmetros e arquitetura

- Base principal: **não é o pacote oficial do IDMT**, é o repack do Kaggle "Electrical Motor Operational State
  Sound Data" (usuário pythonafroz): 2 021 wav de 3 s (669 good, 665 broken, 687 heavy load). O artigo diz
  44,1 kHz/32 bit e, na mesma frase, "cada wav é um segmento de 3 s a 48 kHz" (contraditório). A composição
  do repack (quais condições, se vem de `train_cut/` + `test_cut/`) não é descrita; o IDMT completo tem
  2 378 recortes (774/789/815), então o repack é um subconjunto de origem não documentada.
- Extrator: DWT de 7 níveis, wavelet sym4, usando os coeficientes de aproximação de cada nível. Em cada
  nível, janelas sobrepostas de 16 amostras; funções signum, ternária superior e ternária inferior geram 3
  histogramas de 256 bins → 768 atributos por nível → **5 376 atributos por segmento de 3 s**
  (só signum: 1 792). Limiar da função ternária: não encontrado no texto lido.
- Seleção: NCA e Chi2, 5 execuções cada, 35 atributos por subconjunto → 10 subconjuntos + 28 combinações
  por voto = 38 vetores candidatos; escolhe-se o melhor por algoritmo guloso **avaliado com o próprio CV**.
- Classificadores: Fine kNN e Cubic SVM (MATLAB 2023b, Classification Learner).
- Ablação de wavelet: coif4/5, db4/8, sym4/8, níveis 7 e 9 → sym4 nível 7.
- Experimento secundário: cocleagrama (gammatone, `gtgram`) de segmentos de 1 s → ResNet50 pré-treinada →
  Chi2 (500) → NCA (50) → SVM/kNN, 10-fold estratificado.
- Custo declarado: vetor bruto ~21 KB, 35 atributos ~140 bytes; SVM ~50–200 KB (< 500 vetores de suporte),
  kNN ~280 KB; "cabe em Raspberry Pi"; para MCU sugere ponto fixo ou SVM linear.

## 4. Protocolo de split e risco de vazamento

**10-fold CV sobre os segmentos de 3 s**, sem agrupamento por gravação. Os autores afirmam que "não há
sobreposição entre treino e teste, o que descarta vazamento" — isso vale para segmentos, não para gravações:
recortes contíguos da mesma gravação caem em dobras diferentes. Além disso, a seleção de atributos e a
escolha do melhor dos 38 vetores usam o mesmo CV que gera o número reportado (viés de seleção).
A2212: gravações de 10 s cortadas em segmentos de 3 s, também 10-fold (agrupamento não mencionado).

**Risco de vazamento: alto** (IDMT e A2212). No A2212, além disso, rolamento = motor 4; o texto ainda diz
"três motores A2212" (são quatro).

## 5. Números reportados

- IDMT (repack Kaggle, 10-fold): **Cubic SVM 100 %**, Fine kNN 99,80 %; média 99,85 % ± 0,24 (Tabela 7),
  G-mean 99,79 %, MCC 99,65 %; todas as dobras > 99,5 %. Teste z entre SVM e kNN: p = 0,0443.
- A2212 (segmentos de 3 s, 10-fold): **SVM 99,95 %, kNN 99,16 %** (conclusão).
- Cocleagrama + ResNet50 (IDMT): kNN 99,95 %, SVM 97,97 %.

## 6. O que falta para reproduzir; veredito

Faltam: composição do repack Kaggle; limiar ternário; detalhes de normalização; número de vizinhos do
"Fine kNN" e parâmetros do "Cubic SVM" são os padrões do MATLAB (k = 1; kernel polinomial grau 3), o que é
suposição; contagem de segmentos do A2212.

**Veredito: só inspiração.** O extrator (DWT + histogramas de padrões locais) é barato e embarcável e pode
virar uma pipeline "inspirada" avaliada no split oficial. Os 100 % não são comparáveis; segundo a regra do
CLAUDE.md (> 99 % → procurar vazamento), o vazamento aqui é estrutural.
