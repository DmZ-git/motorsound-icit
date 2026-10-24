# MIMII (Hitachi, 2019) e MIMII DUE (Hitachi, 2021)

Ficha escrita em 23/10/2026 a partir dos arquivos baixados. Os números de §3 a §6 saem de
`scripts/inventario_mimii.py`, rodado sobre `data/mimii/extraido/` com `--resumo --amostra 30`
(semente 20261022) e, numa segunda execução, com `--hash`. As saídas estão copiadas em
`data/mimii/_metadados/inventario_inv.txt` e `inventario_hash.txt`; os JSON da API do Zenodo
ficam em `data/mimii/_metadados/api/` e os PDFs dos dois artigos em `data/mimii/_metadados/`.
Onde a documentação diverge do medido, as duas versões estão registradas. O índice por arquivo é
`out/indices/mimii.csv` (18 173 linhas, uma por wav).

## 1. Origem, citação, licença, tamanho

**MIMII** — Zenodo `https://zenodo.org/record/3384388`, DOI `10.5281/zenodo.3384388`, versão
"public 1.0" (20/09/2019). Licença **CC BY-SA 4.0** (campo `license` da API), acesso `open`,
download sem login. Citação: H. Purohit, R. Tanabe, K. Ichige, T. Endo, Y. Nikaido, K. Suefusa,
Y. Kawaguchi, "MIMII Dataset: Sound Dataset for Malfunctioning Industrial Machine Investigation
and Inspection", *Proc. DCASE 2019 Workshop*, pp. 209–213, 2019; arXiv:1909.09347.

A descrição do Zenodo avisa que a versão pública tem **só 4 dos 7 modelos por tipo (IDs 00, 02,
04, 06)**; os IDs 01, 03, 05 do artigo nunca saíram neste registro (foram usados no conjunto de
avaliação do DCASE 2020 Task 2, ver §9). Doze zips, 100 243 834 344 bytes (≈ 100,2 GB):

| zip | bytes | zip | bytes | zip | bytes |
|---|---|---|---|---|---|
| 6_dB_fan | 10 158 673 161 | 0_dB_fan | 10 411 902 283 | -6_dB_fan | 10 878 096 548 |
| 6_dB_pump | 7 659 077 508 | 0_dB_pump | 7 869 431 302 | -6_dB_pump | 8 236 951 723 |
| 6_dB_slider | 7 129 464 289 | 0_dB_slider | 7 508 958 315 | -6_dB_slider | 7 951 254 230 |
| 6_dB_valve | 6 915 951 837 | 0_dB_valve | 7 486 536 802 | -6_dB_valve | 8 037 536 346 |

**MIMII DUE** — Zenodo `https://zenodo.org/record/4740355`, DOI `10.5281/zenodo.4740355`, versão
1.01 (05/05/2021). Licença **CC BY-NC-SA 4.0** (não comercial; aceitável para artigo acadêmico,
mas diferente da do MIMII), aberto, sem login. Citação: R. Tanabe, H. Purohit, K. Dohi, T. Endo,
Y. Nikaido, T. Nakamura, Y. Kawaguchi, "MIMII DUE: Sound Dataset for Malfunctioning Industrial
Machine Investigation and Inspection with Domain Shifts due to Changes in Operational and
Environmental Conditions", arXiv:2105.02702, 2021 (lida a v3, 27/09/2021). A página diz que é
"entirely the same" que o dev + additional training do DCASE 2021 Task 2; o md5 de
`dev_data_fan.zip` é idêntico nos registros 4740355 e 4562016 (conferido na API). O registro do
DUE só tem treino das seções 03–05; o teste dessas seções está no registro 4884786 (DCASE 2021
Evaluation, CC BY-NC-SA 4.0) e os rótulos no 5257674 (Ground Truth, CC BY-NC-SA 4.0). Registro
4740355: 10 zips (dev_data e eval_data_*_train de fan, gearbox, pump, slider, valve), 9 428 046 039
bytes no total. (Totais somados dos JSON da API em `data/mimii/_metadados/api/`.)

### Subconjunto baixado (19 953 566 887 bytes ≈ 19,95 GB, abaixo do teto de 20 GB)

Critério: `fan` primeiro (é motor, e o MIMII DUE lista "over voltage" como anomalia do ventilador;
o MIMII lista "voltage change"); SNR 6 dB porque é o nível em que a máquina domina o ruído e a
análise espectral do motor é mais limpa; `pump` em 6 dB como segundo tipo com motor; o MIMII DUE
só de `fan`, com as seis seções e o rótulo da avaliação. Slider e valve ficaram de fora (valve
não tem motor girando de modo contínuo; slider é movimento linear repetido).

| zip | registro | bytes | md5 (Zenodo = medido) | sha256 medido |
|---|---|---|---|---|
| 6_dB_fan.zip | 3384388 | 10 158 673 161 | 0890f7d3c2fd8448634e69ff1d66dd47 | 7d274e8e1615b4ec9a8c8417d663a42d179ac38494d368af4d278a70de8ecbe4 |
| 6_dB_pump.zip | 3384388 | 7 659 077 508 | a09ba6060c10fc09cd4c8770213b0b9f | 2e17955bca7c7f9054da09d99ab9f24d18a4dd252d6d863878b525285a3f2a4e |
| dev_data_fan.zip | 4740355 | 1 072 486 301 | 2fd8c081d1adf559372763f204fd154f | 6206d6bc5a43b5e405f2341b2687526d078ecf39079c482ef6c2cf38b17a32c2 |
| eval_data_fan_train.zip | 4740355 | 759 752 515 | 57afe982988072fa2c96faefd5516a73 | d8ba5bc03c852a3e026f0140e7db27d79f27c9632bc74cb5577788261d71d9c2 |
| eval_data_fan_test.zip | 4884786 | 303 544 739 | 933cae49ac0fe9c3cb759c386b21cd11 | 878455e96f91c8c4b5fead75c27e0c69924c702aec7153d3f8f0e5dbb50136d8 |
| ground_truth_data.zip | 5257674 | 32 663 | af8bbeee280b860bdf177ee341d51f11 | 9b8cbcd7f7c1613613bbd749e498ea83eb23c4f64bbff59fdf400241563c6961 |

Todos os md5 conferiram. Contagem de wav extraídos = contagem do zip em todos (fan 5 550; pump
4 205; DUE dev 4 209; DUE eval train 3 009; DUE eval test 1 200). Os zips foram apagados depois
da conferência, por falta de disco; os hashes acima permitem reconferir um novo download. Os zips
descomprimidos ocupam ≈ 26 GB (fan 14,2 GB e pump 10,8 GB segundo o diretório do zip; wav PCM não
comprime). Prefixo `due_` nas pastas locais só distingue a origem.

## 2. Árvore e nomes

```
data/mimii/extraido/
├── mimii_6_dB_fan/fan/id_{00,02,04,06}/{normal,abnormal}/NNNNNNNN.wav     (8 dígitos, de 0)
├── mimii_6_dB_pump/pump/id_{00,02,04,06}/{normal,abnormal}/NNNNNNNN.wav
├── due_dev_data_fan/fan/
│   ├── train/section_SS_{source,target}_train_normal_NNNN_<atributos>.wav     SS = 00..02
│   ├── source_test/section_SS_source_test_{normal,anomaly}_NNNN.wav
│   └── target_test/section_SS_target_test_{normal,anomaly}_NNNN.wav
├── due_eval_data_fan_train/fan/train/section_SS_..._train_normal_NNNN_<atributos>.wav  SS = 03..05
├── due_eval_data_fan_test/fan/{source,target}_test/section_SS_<dom>_test_NNNN.wav (sem rótulo)
└── due_ground_truth/ground_truth_data/ground_truth_<tipo>_section_SS_<dom>_test.csv
                                       linhas "section_03_source_test_0000.wav,1"  (1 = anomalia)
```

Exemplos reais: `mimii_6_dB_fan/fan/id_00/abnormal/00000000.wav`,
`due_dev_data_fan/fan/train/section_01_source_train_normal_0000_strenght_1_big_ambient.wav`,
`due_dev_data_fan/fan/train/section_02_target_train_normal_0000_strength_1_Fact_D_ambient.wav`,
`due_dev_data_fan/fan/target_test/section_00_target_test_anomaly_0000.wav`.
A grafia **`strenght`** aparece nos nomes de treino de origem das seções 01 e 02 (erro do
original; o script aceita as duas). Os índices do MIMII são contíguos de 0 a n−1 em toda pasta
(sem buracos). O nome do MIMII não traz nada além de tipo, ID e classe.

## 3. Taxa, canais, resolução (medidos)

| base | sr | canais | formato | arquivos |
|---|---|---|---|---|
| MIMII (fan, pump, 6 dB) | 16 000 Hz | 8 | PCM_16 | 9 755 |
| MIMII DUE (fan) | 16 000 Hz | 1 | PCM_16 | 8 418 |

Bate com a documentação (16 kHz, 16 bit; arranjo circular TAMAGO-03 de 8 microfones no MIMII;
"only the single-channel audio recorded with the first microphone of the array" no DUE).
**Canal adotado: 0** (microfone "No. 1"). Motivos: é o único usado no baseline do artigo do MIMII
("We used only the first channel"), no DCASE 2020 Task 2 ("we used only the first channel") e o
único distribuído no DUE, o que deixa as duas bases e a literatura comparáveis. Na amostra, a
correlação de forma de onda entre canais 0 e 1 vai de 0,52 a 0,97 (mediana por grupo) e a
diferença de RMS entre os 8 canais de 0,5 a 4,8 dB: os canais não são cópias, e trocar de canal
muda o nível. `scripts/lib/load_mimii.py::carregar` devolve o canal 0 em float32.

## 4. Duração

Todo arquivo tem exatamente 10,000 s (desvio 0 nas duas bases).

| base | unidade | normal (n / min) | anômalo (n / min) |
|---|---|---|---|
| MIMII | fan_id_00 | 1 011 / 168,5 | 407 / 67,8 |
| MIMII | fan_id_02 | 1 016 / 169,3 | 359 / 59,8 |
| MIMII | fan_id_04 | 1 033 / 172,2 | 348 / 58,0 |
| MIMII | fan_id_06 | 1 015 / 169,2 | 361 / 60,2 |
| MIMII | pump_id_00 | 1 006 / 167,7 | 143 / 23,8 |
| MIMII | pump_id_02 | 1 005 / 167,5 | 111 / 18,5 |
| MIMII | pump_id_04 | 702 / 117,0 | 100 / 16,7 |
| MIMII | pump_id_06 | 1 036 / 172,7 | 102 / 17,0 |
| DUE | fan_sec_00 … fan_sec_05 (cada) | 1 203 / 200,5 | 200 / 33,3 |

Totais: MIMII 7 824 normais (21,73 h) e 1 931 anômalos (5,36 h); DUE 7 218 normais (20,05 h) e
1 200 anômalos (3,33 h). As contagens do MIMII batem exatamente com a Tabela 1 do artigo para os
IDs 00/02/04/06 (ex.: fan normal 1011+1016+1033+1015 = 4 075; pump anômalo 143+111+100+102 = 456).

## 5. Matriz classe × unidade física

**Toda unidade tem normal e anômalo**, nas duas bases (tabela de §4). No MIMII a unidade é o
model ID: o artigo diz "Each type of machine includes seven individual machines. Individual
machines may be of a different product model", isto é, um ID = uma máquina física. No DUE a
unidade é a seção: "each section roughly corresponds to a single product", com a exceção
documentada de **fan seção 01, que contém dois produtos do mesmo fabricante** (o tamanho, `big`
no source e `small` no target, é a mudança de domínio). Portanto o viés "classe = motor" do IDMT
**não ocorre**: dá para comparar normal e anômalo dentro da mesma máquina, e separar treino/teste
por unidade. Ressalvas: (i) a relação entre os ventiladores do MIMII (id_XX) e os do DUE
(section_XX) não é documentada; não encontrado se são as mesmas máquinas (o DUE foi gravado em
cabine de isolamento a 0,6 m, o MIMII "in a reverberant environment" a 50 cm, então ao menos a
gravação é outra); (ii) são só 4 unidades por tipo no MIMII e 6 seções no DUE.

## 6. Matriz classe × condição/sessão

**MIMII.** A única condição é o SNR, idêntico para normal e anômalo (aqui, 6 dB). O artigo diz:
"each machine sound was recorded in a separate session"; o ruído de fábrica foi gravado à parte,
com o mesmo arranjo, e misturado assim: calcula-se a potência média *a* de **todos os segmentos
do modelo**; para cada segmento sorteia-se um trecho de ruído *j* e ajusta-se sua potência para
SNR = 10·log10(*a*/*b_j*). Logo o ruído tem a mesma potência em normal e anômalo da mesma unidade
(o SNR é por modelo, não por arquivo), mas o nível da máquina não é normalizado. O artigo não diz
se normal e anômalo foram gravados no mesmo dia ou sessão; a frase sobre "separate session" é
ambígua (sessão por máquina). O **tipo de anomalia por arquivo não é rotulado**: só a pasta
`abnormal`; a Tabela 2 lista exemplos por tipo (fan: "Unbalanced, voltage change, clogging,
etc."; pump: "Leakage, contamination, clogging, etc.") e a conclusão promete "additional
meta-data related to different anomalies" numa versão futura (não encontrada no registro).

**MIMII DUE (fan).** Cada seção tem domínio source e target, independente da classe:

| seção | source train | source test N/A | target train | target test N/A | atributo source → target (nome do arquivo de treino) | descrição no artigo |
|---|---|---|---|---|---|---|
| 00 | 1 000 | 100/100 | 3 | 100/100 | strength_1_ambient → strength_2_ambient | variação da força do vento |
| 01 | 1 000 | 100/100 | 3 | 100/100 | strenght_1_big_ambient → strength_1_small_ambient | dois produtos, tamanhos diferentes |
| 02 | 1 000 | 100/100 | 3 | 100/100 | strenght_1_ambient → strength_1_Fact_D_ambient | ruído de outra fábrica |
| 03 | 1 000 | 100/100 | 3 | 100/100 | strength_1_ambient → strength_2_ambient | força do vento |
| 04 | 1 000 | 100/100 | 3 | 100/100 | strength_1_ambient → strength_2_ambient | força do vento |
| 05 | 1 000 | 100/100 | 3 | 100/100 | strength_1_temp_min → strength_1_temp_max | temperatura do vento |

Seções 03–05: o teste vem sem rótulo e a divisão 100/100 sai do ground truth. O artigo dá
"Test ... 200" para essas seções (sem quebrar normal/anômalo); o medido é 100/100 por domínio.
O nome do arquivo de **teste** não traz atributo; no CSV, `velocidade` do teste é a do treino do
mesmo domínio, marcada `velocidade_inferida_do_dominio` em `obs`. `strength_1/2` é um ajuste de
força do vento, não rpm. A mistura de ruído do DUE segue a mesma receita, com *a* = potência média
da seção e SNR −9 dB (fan 00, 02) e −12 dB (fan 01) segundo a Tabela 1 (a extração do PDF dessa
tabela sai embaralhada; os valores das seções 03–05 não foram lidos com segurança). Anomalias do
fan no DUE: "Wing damage, unbalanced, clogging, and over voltage" (Tabela 2), também sem rótulo
por arquivo.

**Nível medido (canal 0, 30 arquivos por grupo, RMS em dBFS, mediana [p10, p90]).**

| unidade | normal | anômalo | AUC só do RMS |
|---|---|---|---|
| fan_id_00 | −46,2 [−46,7, −45,7] | −43,3 [−47,2, −40,4] | 0,789 |
| fan_id_02 | −44,8 [−45,0, −44,7] | −46,4 [−47,0, −44,5] | 0,263 |
| fan_id_04 | −45,5 [−45,8, −45,2] | −43,8 [−45,4, −43,2] | 0,906 |
| fan_id_06 | −45,0 [−45,2, −44,6] | −41,9 [−49,9, −40,9] | 0,533 |
| pump_id_00 | −47,6 [−47,9, −45,5] | −40,2 [−49,2, −36,8] | 0,700 |
| pump_id_02 | −45,8 [−46,1, −45,3] | −44,8 [−45,4, −39,6] | 0,937 |
| pump_id_04 | −45,2 [−46,5, −44,1] | −45,4 [−46,7, −41,9] | 0,461 |
| pump_id_06 | −46,2 [−48,9, −43,6] | −43,3 [−47,6, −37,7] | 0,762 |

No MIMII o normal de cada unidade tem nível muito estável (faixa p10–p90 de ≈ 0,3–1 dB nos
ventiladores) e o anômalo é mais disperso e, na maioria das unidades, mais alto. Um "detector"
que só olha o RMS chega a AUC 0,91 (fan_id_04) e 0,94 (pump_id_02); 0,26 em fan_id_02 equivale a
0,74 com o sinal invertido. Isso pode ser física legítima (desbalanceamento e sobretensão mudam a
potência sonora) ou diferença de sessão, e não há como separar as duas coisas com o que a base
publica. Entre unidades o normal varia ≈ 3 dB (fan −46,2 a −44,8; pump −47,6 a −45,2).
No DUE o RMS é quase constante dentro da seção (p10–p90 de 0,1–0,3 dB nas seções 00–02, ≈ 2 dB
nas 03–05) e igual entre classes (fan_sec_00 normal −36,6, anômalo −36,4); mesmo assim a AUC do
RMS vai de 0,13 (fan_sec_01 target, isto é 0,87 invertido) a 0,83 (fan_sec_02 source). A
constância do nível sugere normalização por arquivo no DUE, o que não está documentado. Nenhum
arquivo satura (fração de amostras em ±1 = 0). Com 30 contra 30 o erro-padrão da AUC é ≈ 0,07;
os valores servem de alerta, não de estimativa final.

## 7. Verdade de rotação

**Não existe** em nenhuma das duas bases: nem rpm, nem tacômetro, nem frequência de alimentação,
nem modelo do motor ou número de pás. Não encontrado nos artigos, no Zenodo nem na página do
DCASE. O DUE só dá o nível `strength_1/2` (força do vento) por arquivo de treino, que é um
ordinal, não uma rotação. A relação esperada para um ventilador seria frequência de passagem de
pás = (rpm/60)·n_pás, com rpm ligado à frequência da rede (o laboratório da Hitachi em Kokubunji
fica na região de 50 Hz do Japão, mas o local da gravação não é dito) — nada disso é verificável
aqui. Medido na amostra: o pico mais forte da PSD média entre 30 e 1 000 Hz fica em 64–72 Hz em
quase todas as seções do DUE e em fan_id_00, e em 238–468 Hz nos outros ventiladores do MIMII;
não há como afirmar que algum deles é rotação ou passagem de pás. `f_alim_hz` e
`rot_esperada_hz` ficam vazios no índice.

## 8. Gravação de origem e `gravacao_id`

O artigo do MIMII diz que "10-second sound segments were recorded" e que a Tabela 1 conta
"segments"; não diz se cada segmento é uma gravação independente ou um recorte de uma gravação
longa, nem a ordem temporal. Os índices são contíguos de 0 a n−1 por pasta, mas isso não prova
continuidade; testar continuidade entre arquivos consecutivos não é viável porque cada segmento
recebeu um trecho de ruído sorteado independentemente. No DUE, idem ("clips"). Não há duplicatas
exatas de áudio (sha1 do canal 0) entre os 18 173 arquivos. Definição adotada:
`gravacao_id` = um por arquivo (`mimii:<tipo>_id_XX:<classe>:<índice>` e
`mimii_due:<tipo>:<nome sem extensão>`), porque é o máximo que os dados permitem. **Para
validação, a unidade independente segura é a máquina (`unidade`)**; dentro de uma unidade, os
segmentos de mesma classe devem ser tratados como possivelmente correlacionados (mesma sessão).

## 9. Trabalhos que usam a base, splits e vazamento

- **Purohit et al. 2019 (arXiv 1909.09347) — lido inteiro.** Por tipo e ID, todos os anômalos
  vão para teste, o mesmo número de normais sorteados vai para teste e o restante normal vai para
  treino (split por arquivo, dentro da unidade, não por sessão). Autoencoder de 5 quadros de
  log-mel 64 (frame 1024, hop 512), só canal 1. AUC média (6/0/−6 dB): fan 0,94/0,84/0,70;
  pump 0,81/0,74/0,68; valve 0,67/0,61/0,53; slider 0,90/0,80/0,70. Por ID a 6 dB: fan 00 0,75;
  02 0,99; 04 0,92; 06 0,99; pump 00 0,84; 02 0,45; 04 0,99; 06 0,94.
- **Tanabe et al. 2021 (arXiv 2105.02702, MIMII DUE) — lido inteiro (v3).** Treino só normal;
  um modelo por tipo nas seções 00–02. AUC fan source/target: AE 0,667/0,697 (00), 0,674/0,500
  (01), 0,642/0,662 (02); MobileNetV2 0,436/0,533, 0,783/0,781, 0,742/0,604.
- **DCASE 2020 Task 2 (página da tarefa lida; do artigo de descrição, Koizumi et al., arXiv
  2006.05822, só o resumo).** Usa MIMII + ToyADMOS, canal 1, IDs 00/02/04/06 no
  desenvolvimento e 01/03/05 (não públicos no Zenodo) na avaliação, ou seja, split **por
  máquina** entre dev e eval; dentro de cada ID, ~1 000 normais de treino e 100–200 normais e
  anômalos de teste. Baseline AE no dev, AUC fan 54,41/73,40/61,61/73,92 % (IDs 0/2/4/6), pump
  67,15/61,53/88,33/74,55 %. A página proíbe usar o MIMII público como dado externo, porque
  os dados da tarefa "are part of these datasets" e o anômalo do teste vazaria.
- **DCASE 2021 Task 2 (página lida).** É o MIMII DUE + ToyADMOS2; seções 00–02 dev, 03–05
  avaliação, com split por seção.
- Estudo de viés, atalho ou vazamento específico do MIMII: **não encontrado** (busca na web).

**Riscos de vazamento para o nosso uso.** (1) O split do artigo é por arquivo dentro da mesma
máquina e sessão: AUC alta pode vir de diferença de sessão/nível (ver AUC só do RMS em §6). (2)
Qualquer protocolo "classificar normal × anômalo" precisa separar treino e teste **por unidade**
(leave-one-machine-out), o que é possível porque toda unidade tem as duas classes. (3) MIMII 6 dB,
0 dB e −6 dB são a mesma máquina com outro ruído: nunca misturar SNRs entre treino e teste da
mesma unidade. (4) Não se sabe se fan_sec_XX do DUE são as mesmas máquinas dos fan_id_XX do
MIMII; não usar um como teste do outro sem declarar isso. (5) O tipo de anomalia não é rotulado,
então "voltage change" não pode ser isolado por arquivo.
