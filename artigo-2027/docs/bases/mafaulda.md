# MaFaulDa — Machinery Fault Database (SMT/COPPE/UFRJ)

Escrito em 23/10/2026 a partir dos arquivos baixados. Os números de §2 a §8 saíram de
`scripts/inventario_mafaulda.py` (stdout guardado em `data/mafaulda/_metadados/inventario_saida.txt`;
medidas por arquivo em `data/mafaulda/_metadados/inventario_detalhes.csv`). Onde a documentação
diverge do medido, as duas versões estão registradas. **Há duas divergências graves: a taxa de
amostragem real é 51,2 kHz, e não 50 kHz (§3), e o microfone está defeituoso em 166 arquivos
concentrados em três classes (§3, §5).**

## 1. Origem, citação, licença, tamanho

- Página: http://www02.smt.ufrj.br/~offshore/mfs/page_01.html (índice em `.../mfs/index.html`,
  "Last change: June 16 2021"). Laboratório de Sinais, Multimídia e Telecomunicações (SMT),
  COPPE/Poli/UFRJ; contato indicado: Felipe M. L. Ribeiro. Cópias das páginas em
  `data/mafaulda/_metadados/`.
- **Licença / termos de uso: não encontrado.** Nem a página da base nem o índice falam de licença,
  termos ou forma de citação. O download é livre (sem login). O espelho do Kaggle não foi consultado
  (exige login).
- **Citação:** a base não tem artigo de referência próprio. Trabalhos de origem, do mesmo grupo:
  - Ribeiro, F. M. L.; Marins, M. A.; Netto, S. L.; da Silva, E. A. B. *Rotating machinery fault
    diagnosis using similarity-based models*. SBrT 2017, doi:10.14209/sbrt.2017.133. **Texto
    completo lido** (PDF da biblioteca da SBrT). Cita a base como "MaFaulDa - Machinery Fault
    Database, http://www02.smt.ufrj.br/~offshore/mfs/, 2016".
  - Marins, M. A.; Ribeiro, F. M. L.; Netto, S. L.; da Silva, E. A. B. *Improved similarity-based
    modeling for the classification of rotating-machine failures*. Journal of the Franklin
    Institute 355 (2018) 1913–1930. **Lidas a descrição da base e o protocolo** (PDF da página do
    autor, `IJ28.pdf`).
  - Marins, M. A. *Classificação de falhas em máquinas rotativas utilizando métodos de
    similaridade e random forest*. Projeto de Graduação, UFRJ. **Lido o capítulo 2 (bancada e
    medições)**. Ano: não conferido.
  - Citados como origem da aquisição, **não lidos**: de Lima, A. A. et al., *On fault
    classification in rotating machines using Fourier domain features and neural networks*,
    LASCAS (ano não conferido); Pestana-Viana, D. et al., *The influence of feature vector on the
    classification of mechanical faults using neural networks*, LASCAS 2016.
- Download: `database/mafaulda/full.zip`, Content-Length 12 896 655 242 bytes, Last-Modified
  08/04/2017, ETag `"58e8285a-300b3578a"`. Há também zips e tgz por subconjunto (normal,
  horizontal-misalignment, vertical-misalignment, imbalance, underhang, overhang); baixei só o
  `full.zip`, que contém todos.
- sha256 do `full.zip`: `fc62677c000b7a3e79db88502a542df23b8bf797454a9ebad520ad9d3a915cc7`
  (em `_metadados/full_zip.sha256`). Extraído para `data/mafaulda/full/` com verificação de CRC
  (unzip sem erro): 1951 CSV, 33 327 768 961 bytes; os 1951 tamanhos conferem um a um com a
  lista do zip (`_metadados/full_zip_listing.txt`). O zip foi apagado depois da conferência.
- Cache: `data/mafaulda/_cache/**.npy` (microfone e tacômetro, float32, N × 2; 3,7 GB), usado pelo
  loader.

## 2. Árvore e nomes

```
data/mafaulda/full/
├── normal/<rot>.csv                                   49
├── imbalance/{6g,10g,15g,20g,25g,30g,35g}/<rot>.csv   333
├── horizontal-misalignment/{0.5mm,1.0mm,1.5mm,2.0mm}/<rot>.csv          197
├── vertical-misalignment/{0.51mm,0.63mm,1.27mm,1.40mm,1.78mm,1.90mm}/<rot>.csv  301
├── underhang/{cage_fault,ball_fault,outer_race}/{0g,6g,20g,35g}/<rot>.csv  558
└── overhang/{cage_fault,ball_fault,outer_race}/{0g,6g,20g,35g}/<rot>.csv   513
```

Exemplos reais: `normal/12.288.csv`, `imbalance/35g/56.7296.csv`,
`vertical-misalignment/1.27mm/12.0832.csv`, `underhang/ball_fault/20g/47.7184.csv`.
As pastas codificam **classe** (1º nível; rolamento = posição + tipo de defeito) e **severidade**
(2º/3º nível: massa em g, deslocamento em mm; nos rolamentos, a massa de desbalanceamento
adicionada). O nome do arquivo é a **rotação em Hz**: 246 valores distintos de 12,0832 a 62,2592,
todos múltiplos de 0,2048 Hz = 51 200/250 000 (resolução da DFT de 250 000 amostras a 51,2 kHz).
O nome é uma rotação *medida* pelo tacômetro, não um setpoint (§7). Não há repetição de
(classe, severidade, rotação).

Cada CSV: sem cabeçalho, vírgula, 250 000 linhas × 8 colunas, valores em texto com 5 algarismos
significativos (o menor passo entre valores do microfone é 2·10⁻⁵ V em todos os arquivos).

## 3. Taxa, canais, resolução (medidos)

- **Taxa: 51 200 Hz, não os 50 000 Hz declarados** (página, artigos e projeto de graduação dizem
  50 kHz e 5 s). Evidências medidas:
  1. Zumbido da rede (60 Hz no Brasil) e harmônicos 120/180 Hz: lidos com 51,2 kHz, aparecem em
     60·k Hz exatos (saliência mediana 40 dB acima do fundo local no microfone, 37 dB no
     tacômetro); lidos com 50 kHz, caem em 58,59/117,19/175,78 Hz. Em 1920 de 1951 arquivos o
     pico de rede no microfone é mais saliente na hipótese 51,2 kHz.
  2. O nome do arquivo bate com a frequência do tacômetro calculada a 51,2 kHz (razão mediana
     1,0000; com 50 kHz a razão seria 1,0240 = 51,2/50).
  3. Os nomes são múltiplos de 51 200/250 000 Hz.
  4. O módulo NI 9234 citado na página opera em 51,2 kHz/n; 50 kHz não é uma taxa disponível
     nele (dado do fabricante, não verificado nos arquivos).
  
  Consequência: cada arquivo dura **4,8828 s**, não 5 s, e todas as frequências calculadas com
  50 kHz ficam 2,34 % baixas. O loader usa `SR = 51_200` (`SR_DECLARADO = 50_000` fica registrado).
- Colunas (página): 1 tacômetro; 2–4 acelerômetro do mancal underhang (axial, radial,
  tangencial); 5–7 acelerômetro do mancal overhang; 8 microfone. Conferido pelo conteúdo:
  - **Coluna 1 (índice 0) = tacômetro**: trem de pulsos de −2,9 a 5,1 V, duty cycle mediano 0,134,
    frequência de pulsos igual ao pico de 1x dos acelerômetros (§7).
  - **Coluna 8 (índice 7) = microfone**: é a única coluna com quase nenhuma energia abaixo de
    200 Hz (onde os acelerômetros concentram 50–95 % da potência) e com espectro de banda larga
    até 20 kHz; tem passo de quantização próprio. Mas **~91 % da potência do canal é um tom
    espúrio em ~22,55 kHz** (fração mediana 0,909; 22,54–22,58 kHz), fora da banda do SM81
    (20 Hz–20 kHz). O mesmo tom aparece, mais fraco, nas colunas 2–4. A correlação do microfone com
    as colunas 2–4 (até −0,8) vem desse tom. O nível acústico útil deve ser medido abaixo de 20 kHz.
- 1 canal acústico (mono), Shure SM81, valor em volts. Sensibilidade e ganho: não encontrados.
- **Saturação / falha do microfone:** nos arquivos bons não há saturação. O pico |AC| máximo por
  classe fica entre 0,7 e 7,3 V, e o máximo de amostras no valor extremo é ≤ 17 por arquivo. Mas
  **166 arquivos têm o microfone defeituoso**, com espectro quase branco (flatness > 0,5 contra
  ~0,16–0,25 nos bons) e/ou RMS robusto de 0,1 a 12,6 V (contra 25–66 mV nos bons), impulsos
  espúrios de até ±200 V e offset DC de até −4,5 V. Um SM81 não produz ±200 V, então é falha de
  cabo, conector ou aquisição. 45 arquivos tocam o mesmo mínimo, −204,12 V (valor de clipping).
  Onde estão (classe × severidade):
  desbalanceamento 20 g (5), 25 g (2), 30 g (37 de 47), 35 g (38 de 45);
  rolamento_underhang_gaiola 6 g (14), 20 g (37), 35 g (33). Nenhum nas outras classes.
  Na ordem de gravação, o defeito começa no meio da varredura de rotação e segue até o fim.
  Por exemplo, imbalance/30g está bom nos 10 primeiros arquivos e defeituoso a partir de
  22,7 Hz. O defeito parece disparar com vibração alta. O índice marca esses arquivos no `obs`
  ("MICROFONE DEFEITUOSO").

## 4. Duração

Todos os arquivos têm 250 000 amostras = 4,8828 s (5 s pela documentação).

| classe | arquivos | duração (s) |
|---|---|---|
| normal | 49 | 239,3 |
| desbalanceamento | 333 | 1 626,0 |
| desalinhamento_horizontal | 197 | 961,9 |
| desalinhamento_vertical | 301 | 1 469,7 |
| rolamento_underhang_gaiola | 188 | 918,0 |
| rolamento_underhang_esfera | 186 | 908,2 |
| rolamento_underhang_pista_externa | 184 | 898,4 |
| rolamento_overhang_gaiola | 188 | 918,0 |
| rolamento_overhang_esfera | 137 | 668,9 |
| rolamento_overhang_pista_externa | 188 | 918,0 |
| **total** | **1 951** | **9 526,4 (2,65 h)** |

As contagens por classe e por severidade batem com a página (49/197/301/333; underhang
188/184/186; overhang 188/188/137; total 1951) e com a Tabela 2.2 do projeto de graduação.

## 5. Matriz classe × unidade física

**Há uma bancada só** (SpectraQuest MFS-ABVT, motor de 1/4 cv, um eixo, um rotor, dois
mancais, um microfone). Os arquivos não trazem número de série nem identificador de peça. A
unidade do índice sai das pastas e dos artigos:

| classe | unidade no índice | arquivos |
|---|---|---|
| normal | bancada_mafaulda | 49 |
| desalinhamento_horizontal | bancada_mafaulda (motor deslocado) | 197 |
| desalinhamento_vertical | bancada_mafaulda (motor deslocado) | 301 |
| desbalanceamento | bancada_mafaulda+massa_{6,10,15,20,25,30,35}g | 333 (45–49 por massa) |
| rolamento_{underhang,overhang}_gaiola | rolamento_defeituoso_gaiola | 188 + 188 |
| rolamento_{underhang,overhang}_esfera | rolamento_defeituoso_esfera | 186 + 137 |
| rolamento_{underhang,overhang}_pista_externa | rolamento_defeituoso_pista_externa | 184 + 188 |

- **Rolamentos:** segundo Marins et al. (2018), "the ABVT manufacturer provided three defective
  bearings, each one with a distinct defective element ... placed one at a time in two different
  positions". Ou seja, **um rolamento físico por tipo de defeito**, o mesmo nas posições
  underhang e overhang. Nos arquivos isso não pode ser verificado. **Tipo de defeito = peça.** Não
  há segunda peça com o mesmo defeito, nem a mesma peça sem defeito. Um classificador de
  "gaiola × esfera × pista" pode estar reconhecendo três rolamentos individuais (folgas,
  montagem), e não o defeito. A troca de posição (underhang ↔ overhang) é a única variação de
  montagem da mesma peça e permite testar "treina numa posição, testa na outra".
- **Desbalanceamento:** cada severidade é uma massa (ou combinação de massas) diferente presa ao
  rotor. Não se sabe se é uma peça ou várias. Massa = severidade.
- **Desalinhamento:** a peça é a mesma bancada; muda o deslocamento do motor.
- **Normal:** os rolamentos originais da bancada, gravados só na sessão de 03/10/2014 (§6).

O que permite: generalização entre rotações dentro da mesma configuração; deixar uma
severidade de fora; transferir entre posições do mesmo rolamento; estudar ordens e rastrear
rotação com verdade de tacômetro. O que **não** permite: generalizar entre máquinas, entre
microfones ou entre peças com o mesmo defeito. Deixar uma unidade de fora é impossível, porque
tirar o rolamento "esfera" tira a classe inteira.

## 6. Matriz classe × condição e sessões

Condição = severidade (pasta) × rotação (nome). Cada (classe, severidade) é **uma varredura de
rotação**: 20 a 51 arquivos, de ~12 Hz a ~62 Hz, um arquivo por rotação.

| classe | severidades (arquivos) | rotação nominal (Hz) |
|---|---|---|
| normal | — (49) | 12,29–61,44 |
| desbalanceamento | 6 g (49), 10 g (48), 15 g (48), 20 g (49), 25 g (47), 30 g (47), 35 g (45) | 12,08–62,05; 30/35 g até 59,4/56,7 |
| desalinhamento_horizontal | 0,5 (50), 1,0 (49), 1,5 (49), 2,0 mm (49) | 12,29–62,05 |
| desalinhamento_vertical | 0,51 (51), 0,63 (50), 1,27 (50), 1,40 (50), 1,78 (50), 1,90 mm (50) | 12,08–62,26 |
| rolamento_underhang_* | 0 g / 6 g / 20 g / 35 g; gaiola 49/48/49/42, esfera 50/49/49/38, pista 49/49/49/37 | 12,9–62,05; 35 g até 50,8–59,0 |
| rolamento_overhang_* | 0 g / 6 g / 20 g / 35 g; gaiola 49/49/49/41, esfera 49/43/25/20, pista 49/49/49/41 | 12,3–61,8; esfera 20/35 g só até 36,0/32,2 |

Divergências: nas pastas de rolamento as massas são **0, 6, 20 e 35 g**, como na Tabela 2.2 do
projeto de graduação. O texto de Marins et al. (2018) e o texto do projeto dizem "6, 10 e 20 g".
A página chama os defeitos de rolamento de "inner and outer" na introdução. A tabela da página e as
pastas usam gaiola (cage), esfera (ball) e pista externa (outer race), e Marins et al. (2018) dizem
"outer track, rolling elements, and inner track". **Não há pista interna nos arquivos.** No
projeto de graduação, as contagens da tabela "Mancal Não Invertido" batem com a pasta
`overhang/`, mas o texto define "não invertido" como o mancal entre rotor e motor (underhang).
Vale a pasta.

**Sessões** (data/hora de cada membro no zip, preservadas na extração):
- 03/10/2014: normal (12:34–13:52) e desbalanceamento 6, 10, 15, 20 e 25 g (14:18–15:49).
- 06/10/2014: desbalanceamento 30 e 35 g (10:12–10:38), desalinhamento vertical (10:43–12:45) e
  desalinhamento horizontal (12:58–14:31).
- Rolamentos: todos com data 07/04/2017 16:05–16:54, a ~1,4 s por arquivo, menos que os 4,9 s de
  cada gravação. São datas de cópia/empacotamento, não de aquisição. **Data de aquisição dos
  rolamentos: não determinada.**

Dentro de cada pasta, a ordem de gravação acompanha a rotação (Spearman mediano 0,957 entre mtime
e rotação nas pastas de 2014): cada severidade é uma varredura crescente, gravada em 11–80 min.
Consequências: (i) **o normal só existe na sessão de 03/10**, junto com o desbalanceamento leve.
Desalinhamentos e desbalanceamento pesado são de outra sessão, e os rolamentos de uma sessão
desconhecida. "Normal × falha" mistura falha e sessão. (ii) Dentro da varredura, rotação e
tempo (aquecimento, deriva) andam juntos.

**Nível acústico** (RMS robusto = mediana do RMS < 20 kHz em janelas de 0,1 s; mediana por
classe; arquivos bons): normal 47,7 mV; desalinhamento horizontal 53,2; vertical 46,7;
desbalanceamento 50,6; overhang gaiola/esfera/pista 42,3/42,1/45,8; **underhang
gaiola/esfera/pista 35,8/33,7/38,8**. O nível sobe com a rotação em todas as classes (normal:
40,5 mV abaixo de 20 Hz → 50,6 mV acima de 50 Hz). Os rolamentos underhang são ~20–30 % mais
silenciosos que o normal (esfera e pista externa em todas as faixas de rotação; gaiola também, exceto onde entram arquivos com microfone defeituoso). Pode ser a posição do rolamento ou a
sessão; os dados não separam as duas. Com o tom de 22,55 kHz incluído, o RMS total (145–191 mV)
mede o tom, não o som da máquina.

## 7. Verdade de rotação

- **Tacômetro gravado** (coluna 1, Monarch MT-190), trem de pulsos com duty cycle ~13 %.
- **Pulsos por volta: 1.** O pico do espectro do acelerômetro underhang radial na faixa
  [0,3; 1,3]·f_pulsos cai em f_pulsos (±2 %) em 96,4 % dos arquivos e perto de 0,5·f_pulsos em
  0,5 %. Os harmônicos do pulso (duty curto) explicam o pico em 2× que às vezes domina o espectro
  do próprio tacômetro.
- Estimativa: bordas de subida com histerese (40/60 % da faixa entre os percentis 5 e 95) e período
  refratário de 0,5/f0 (f0 = pico do espectro do tacômetro entre 5 e 70 Hz); rotação = mediana de
  1/intervalo. Em 173 arquivos o tacômetro é ruidoso ((p95−p5)/mediana > 0,05), sobretudo
  `underhang/ball_fault` (77) e `overhang/outer_race` 20/35 g. Nesses, sem o período refratário,
  a mediana das bordas chegava a 2× ou mais. O índice usa então o pico do espectro do tacômetro
  (resolução 0,049 Hz), e o `obs` registra isso.
- **Relação com o nome do arquivo:** nome / rotação medida (`rot_esperada_hz`): mediana 1,0000,
  p5 0,9966, p95 1,0036. |nome − medida| ≤ 0,2048 Hz (1 passo do nome) em 99,8 % dos arquivos,
  máximo 0,27 Hz. Ajuste: medida = 0,99986·nome + 0,004 Hz. **O nome é a rotação do eixo em Hz,
  estimada pelo grupo a partir do tacômetro com a taxa correta de 51,2 kHz** e quantizada em
  0,2048 Hz. Quem calcular a rotação com 50 kHz terá 2,34 % a menos (razão nome/medida = 1,024).
- Rotação dentro do arquivo: dispersão (p95−p5)/mediana de 0,8 % (mediana). A velocidade é
  constante; não há rampas.
- Faixa medida: 12,07–62,26 Hz (724–3 736 rpm). A documentação diz 737–3 686 rpm e
  "700–3600 rpm".
- `f_alim_hz` fica vazio: o motor é CC ("1/4 CV DC" na página) e não há inversor nem tensão
  registrada.

## 8. Gravação de origem

Cada CSV é uma aquisição separada de 4,88 s. Nenhum par de arquivos tem canal acústico idêntico
(hash), e as datas de 2014 mostram de ~15 s a ~1,6 min entre arquivos consecutivos.
`gravacao_id = mafaulda/<pasta>/<rotação>`, por exemplo `mafaulda/imbalance/35g/56.7296`. Mas as
gravações de uma pasta não são independentes entre si: formam uma varredura contínua da mesma
montagem, em minutos, com rotações vizinhas a 0,2–1,2 Hz. **Para separar treino e teste, o
grupo deve ser a pasta (classe × severidade) ou a sessão**, nunca o arquivo isolado.

## 9. Trabalhos que usam a base

- **Spadini, T.; Nose-Filho, K.; Suyama, R.** *Intelligent fault diagnosis of type and severity in
  low-frequency, low bit-depth signals*. arXiv:2411.06299v1, 2024 (UFABC). **Texto completo
  lido** (PDF do arXiv). Usa **só o microfone**, com 42 classes (tipo × severidade = as 42 pastas).
  Separação **aleatória de 20 % dos arquivos para teste**, antes do aumento de dados. Os autores
  insistem que nenhum segmento do mesmo arquivo vai para os dois lados, e usam validação cruzada
  estratificada de 10 dobras no treino. Melhor resultado: acurácia 99,54 % e F-beta 99,52 %
  (XGBoost, 8 kHz, 8 bits); só MFCC + deltas: 97,83 %. **Risco de vazamento alto**: com divisão
  aleatória por arquivo, cada arquivo de teste tem vizinhos de rotação (0,2–1 Hz) da mesma
  varredura no treino. A classe "42" é a pasta, ou seja, uma montagem e uma sessão. Além disso,
  166 arquivos com microfone defeituoso estão só em 7 das 42 pastas, e o nível difere por sessão
  e posição. O número mede reconhecimento de varredura/sessão; generalização para outra montagem
  não foi testada.
- **Ribeiro et al., SBrT 2017** (texto completo): usa os 8 sinais (o microfone entra nas
  features de espectro em fr, 2fr, 3fr e nas estatísticas), com os sinais normalizados à
  variância unitária. Rotação obtida da DFT do tacômetro. Divisão aleatória estratificada 90/10
  por cenário (arquivo), mais 10 dobras no treino. Acurácia 96,4 %. Mesmo risco de vazamento
  entre rotações vizinhas da mesma varredura.
- **Marins et al., J. Franklin Inst. 2018** (descrição e protocolo lidos): mesmos 8 sinais,
  divisão aleatória estratificada 90/10 dos cenários, 98,5 %. Mesmo risco.
- Citados por Spadini et al., **não lidos**: Pestana-Viana et al. (LASCAS 2016, MLP, 95,8 %),
  Rocha (Haar + XGBoost, 98,7 %, 10 dobras) e um trabalho com RF 99,84 % em 10 dobras (ref. [35]
  deles). Uso do microfone, split e números: não verificados.
- Trabalho de conclusão da Univ. de Boumerdès (2024, *Machine learning-based fault diagnosis of
  rotating machinery using acoustic data*): só o título e o resumo da busca; diz usar a MaFaulDa
  com som. Split e números: não encontrado.

## 10. Consequências para o protocolo

1. Usar `SR = 51 200` Hz. Qualquer frequência (ordens, BPFO = 2,998×, BSF, FTF) calculada com
   50 kHz sai 2,34 % baixa.
2. Excluir ou tratar à parte os 166 arquivos com microfone defeituoso (`obs` do índice). Sem
   isso, "desbalanceamento 30/35 g" e "underhang gaiola" são reconhecíveis pela falha do canal.
3. Filtrar abaixo de 20 kHz antes de medir nível ou extrair features (tom de 22,55 kHz).
4. Separar por pasta (varredura) ou por sessão, nunca por arquivo. Para falha × normal, lembrar
   que o normal só existe na sessão de 03/10/2014 e tem 49 arquivos.
5. Tipo de defeito de rolamento = uma peça. Declarar isso e, se possível, treinar numa posição e
   testar na outra.
6. Para rastrear rotação, a base é muito boa: 1951 arquivos com 1 pulso/volta e rotação
   constante de 12 a 62 Hz. O nome do arquivo já é a rotação medida (±0,2 Hz); nos 173 arquivos
   com tacômetro ruidoso, usar o pico espectral.
