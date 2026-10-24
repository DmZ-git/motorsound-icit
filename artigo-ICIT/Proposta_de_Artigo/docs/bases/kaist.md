# Ficha da base `kaist`

Escrita em 22/10/2026 a partir dos arquivos baixados. Quando a documentação diverge do que
foi medido, as duas versões estão registradas. Os números vêm da saída de
`scripts/inventario_kaist.py` (comando na §10), salvo os marcados "documentação" ou "artigo".

## 1. Origem, citação, licença, tamanho

- **Citação**: W. Jung, S.-H. Kim, S.-H. Yun, J. Bae, Y.-H. Park, "Vibration, acoustic,
  temperature, and motor current dataset of rotating machine under varying operating
  conditions for fault diagnosis", *Data in Brief* 48 (2023) 109049.
  DOI do artigo: 10.1016/j.dib.2023.109049 (PMC10036499, acesso aberto).
- **Dados**: Mendeley Data, "Vibration, Acoustic, Temperature, and Motor Current Dataset of
  Rotating Machine Under Varying Load Conditions for Fault Diagnosis", versão 6
  (publicada em 08/02/2023), DOI 10.17632/ztmf3m7h5x.6,
  https://data.mendeley.com/datasets/ztmf3m7h5x/6 . Instituição: KAIST (Daejeon, Coreia).
- **Licença**: CC BY 4.0, segundo a API do Mendeley e o artigo. **Não exige login**: a API
  pública e os links de download funcionam sem autenticação.
- **Tamanho total declarado**: 4 254 773 960 B em 3 zips:

| Arquivo no Mendeley | Bytes | sha256 (Mendeley = local) | Baixado? |
|---|---|---|---|
| `acoustic.zip` | 47 205 730 | `5dd1371ccec00729498e2d151df13593f6400fdced8895481ff6a8a945a5962f` | sim, em `raw/acoustic.zip` |
| `current,temp.zip` | 1 547 790 068 | `7caa22152ff1fd673a6a3a9614f95fabd2890c6aea0860b40a1f5b27eabb59d6` | sim, em `raw/current_temp.zip` |
| `vibration.zip` | 2 659 778 162 | `df73fa80371a2ad3c983825044447b7772fcefd1db2ac409b2b298b533e4659f` | não; só o diretório central e os cabeçalhos, lidos por HTTP Range |

Os sha256 locais batem com os do Mendeley. Os arquivos extraídos e seus sha256:

| Arquivo | sha256 |
|---|---|
| `acoustic/0Nm_BPFI_03.mat` | `d5bdb08a3ca0e4d05d670ad3dce2cda22a67beebd6548b7fe332929ae9c30f24` |
| `acoustic/0Nm_BPFI_10.mat` | `15f62152a69011ceaec4f518d287b67a5f15d0198be5d65076b38618db5184a7` |
| `acoustic/0Nm_BPFO_03.mat` | `30c769bda061f714b01f51e2e8e7d876e4c5d311480335bb9d97876cc6eb106b` |
| `acoustic/0Nm_BPFO_10.mat` | `db846869053997bacd23773a8a077251183b2ebae0cc3844d1fde8cfe9f97849` |
| `acoustic/0Nm_Normal.mat` | `2b3ff16a9fb43fa929988484e1e390d31e1da0241c90b0d1279eb8ef29010066` |
| `current/0Nm_BPFI_03.tdms` | `3dbdf50a5d7e4f98e1e1bef23d9b107aa4ab566d0933559633dfa756d4d7bcf0` |
| `current/0Nm_BPFI_10.tdms` | `06aee97ebc67df0b0f8b994ff517f8a4391ed19fb8ecf7e2b540cacb3042cea5` |
| `current/0Nm_BPFO_03.tdms` | `51e0fd8bb844138f40319ffacda53892f81903cda4bb2052d2db746c34537471` |
| `current/0Nm_BPFO_10.tdms` | `5dbac9865d0a1ed1cdf44f44a586cda2165c85ce8f18673ce0b7e6cfe2ec2a2b` |
| `current/0Nm_Normal.tdms` | `ade3d91107fda795908818072f7fb39da8be2fb4326eedd96d3232293947ae29` |

Do `current_temp.zip` foram extraídos só os 5 TDMS de mesmo nome dos acústicos. Os outros
40 continuam dentro do zip.

O artigo cita ainda três outros conjuntos de dados (10.17632/vxkj334rzv.7, x3vhp8t6hg.7,
j8d8pfkvj2.7). São a "parte 2": rolamentos com velocidade variável, só vibração, corrente e
rpm, **sem áudio**. Não foram baixados.

## 2. Árvore e padrão de nomes

```
data/kaist/
├── _meta/      dataset.json, root_files.json, version6.json (API Mendeley);
│               paper_PMC10036499.xml + paper.txt (artigo completo, Europe PMC)
├── raw/        acoustic.zip, current_temp.zip         (zips originais, não modificados)
├── acoustic/   0Nm_Normal.mat, 0Nm_BPFI_03.mat, 0Nm_BPFI_10.mat, 0Nm_BPFO_03.mat, 0Nm_BPFO_10.mat
└── current/    os 5 .tdms de mesmo nome
```

O padrão é `<carga>Nm_<condição>[_<severidade>].mat`. A condição é `Normal`, `BPFI`
(pista interna) ou `BPFO` (pista externa). A severidade é `03` = 0,3 mm ou `10` = 1,0 mm.
**O áudio tem só 5 arquivos, todos com 0 Nm**: não há áudio de desalinhamento,
desbalanceamento, defeito de 3,0 mm nem cargas de 2 ou 4 Nm. A descrição do Mendeley
("3 cargas, desalinhamento, desbalanceamento") vale para vibração e corrente, não para o
áudio. O artigo (Tabela 2 e §3.1) confirma: o áudio foi gravado só sem carga "to avoid the
noise from air-cooled brakes". Na vibração e na corrente há 45 arquivos (15 condições × 3
cargas). Na vibração, quatro nomes vêm com o erro de grafia `2Nm_Unbalalnce_*`.

## 3. Formato, taxa de amostragem e resolução (medidos)

- **Formato real**: MAT v5 ("MATLAB 5.0 MAT-file, Platform: PCWIN"), exportado pelo Siemens
  Simcenter **Test.Lab** ("Data saved by Test.Lab in MKS"). Cada arquivo contém uma struct
  `Signal` com `x_values` (eixo do tempo implícito: `start_value`, `increment`,
  `number_of_values`), `y_values.values` (float64, em Pa) e `function_record` (metadados do
  run). **Divergência**: a documentação fala em "two columns 'Time Stamp' and 'values'", mas
  não existe coluna de tempo.
- **Canais**: 1. Canal 3, "Point1", grupo "Acoustic", unidade Pa. É o microfone PCB 378B02
  "nearby bearing housing A" (artigo). O loader devolve esse canal como float32.
- **sr = 51 200 Hz** (increment = 1,953125e-5 s) em todos. Bate com o artigo.
- **Resolução**: o menor passo entre valores distintos é 2,4319e-05 a 2,4378e-05 Pa. Pelo
  pico, a faixa usada fica entre ~15,9 e ~18,1 bits. Exportado em float64.
- **Saturação: não há.** Só 1 ou 2 amostras por arquivo chegam a ≥ 99,9 % do pico. O pico
  máximo é 3,31 Pa. A média é ~0, com |média| ≤ 3,4e-4 Pa.

## 4. Duração por arquivo e por classe

| Classe | Arquivos | Duração total |
|---|---|---|
| normal | 1 | 60,0 s |
| rolamento_externo (BPFO 0,3 e 1,0 mm) | 2 | 120,0 s |
| rolamento_interno (BPFI 0,3 e 1,0 mm) | 2 | 120,0 s |
| **Total** | **5** | **300,0 s** |

Cada arquivo tem 3 072 000 amostras, ou seja, 60,000 s, como no artigo.

### Níveis (amostra de cada classe, arquivo inteiro)

| Arquivo | RMS (Pa) | dB SPL | Pico (Pa) | Fator de crista | RMS em janelas de 1 s |
|---|---|---|---|---|---|
| 0Nm_Normal | 0,1476 | 77,4 | 0,876 | 5,93 | 0,131–0,166 |
| 0Nm_BPFO_03 | 0,1410 | 77,0 | 0,754 | 5,35 | 0,126–0,167 |
| 0Nm_BPFO_10 | 0,8953 | 93,0 | 3,055 | 3,41 | 0,868–0,910 |
| 0Nm_BPFI_03 | 0,4184 | 86,4 | 1,725 | 4,12 | 0,411–0,461 |
| 0Nm_BPFI_10 | 1,0250 | 94,2 | 3,308 | 3,23 | 1,010–1,048 |

Entre as classes a **diferença de nível é grosseira**: os defeitos de 1,0 mm estão ~16–17 dB
acima do normal, e o BPFI 0,3 mm está ~9 dB acima. Dentro de cada arquivo, o RMS por segundo
varia pouco. Um classificador só de nível já separa três grupos ({Normal, BPFO_03}, BPFI_03,
{BPFO_10, BPFI_10}). Normal e BPFO_03 têm o mesmo nível e diferem só no espectro.

dB SPL por banda (Hz), da saída do script:

| Arquivo | 20–250 | 250–1k | 1k–2k | 2k–4k | 4k–8k | 8k–16k | 16k–25,6k |
|---|---|---|---|---|---|---|---|
| Normal | 61 | 56 | 70 | 71 | 71 | 57 | 44 |
| BPFO_03 | 61 | 66 | 68 | 71 | 69 | 59 | 42 |
| BPFO_10 | 63 | 81 | 91 | 88 | 75 | 67 | 56 |
| BPFI_03 | 63 | 64 | 76 | 85 | 76 | 69 | 61 |
| BPFI_10 | 62 | 78 | 83 | 94 | 74 | 63 | 52 |

## 5. Matriz classe × unidade física (item principal)

A bancada é **única**: um motor de indução trifásico Siemens de 4 polos e 3 HP, um
torquímetro, uma caixa multiplicadora (×2,07), os mancais A e B, os discos do rotor e um freio
de histerese (artigo, §3.1). O rolamento com defeito semeado é montado **no mancal A**, e o
microfone fica junto do mancal A.

| Classe | Arquivo | Unidade física (rolamento no mancal A) | Motor / bancada |
|---|---|---|---|
| normal | 0Nm_Normal | rolamento sadio (1 peça) | bancada única |
| rolamento_externo | 0Nm_BPFO_03 | rolamento com trinca de 0,3 mm na pista externa | idem |
| rolamento_externo | 0Nm_BPFO_10 | rolamento com trinca de 1,0 mm na pista externa | idem |
| rolamento_interno | 0Nm_BPFI_03 | rolamento com trinca de 0,3 mm na pista interna | idem |
| rolamento_interno | 0Nm_BPFI_10 | rolamento com trinca de 1,0 mm na pista interna | idem |

- **Motores**: 1. Um protocolo "deixar um motor de fora" é impossível.
- **Rolamentos**: pela Fig. 6 do artigo, cada severidade é uma peça diferente (fotos
  separadas por tamanho de trinca). O artigo não diz explicitamente se o rolamento "normal" é
  uma peça própria ou se o rolamento do mancal B é trocado; isso fica como "não encontrado".
  Cada rolamento aparece numa única gravação de 60 s.
- **O que a base permite**: 1 classe sadia contra 4 rolamentos defeituosos. Um teste de
  generalização entre severidades é possível só dentro de cada tipo de defeito (treinar com
  0,3 mm e testar com 1,0 mm, e vice-versa), com uma única peça de cada lado.
- **O que não permite**: deixar um motor de fora; ter mais de uma peça por classe-severidade;
  deixar uma peça "normal" de fora (só existe uma). Qualquer divisão treino/teste dentro da
  classe normal corta a mesma gravação de 60 s.

## 6. Matriz classe × sessão / condição

Carga = 0 Nm e rotação nominal = 3010 rpm em todos os arquivos. Os metadados do Test.Lab
(`absolute_time` e `run_name`) dão a ordem das gravações:

| # | Início (absolute_time) | Arquivo | run_name | Intervalo desde o fim da anterior |
|---|---|---|---|---|
| 1 | 2021-07-05 14:48:36 | 0Nm_Normal | 210705_Normal_5cm | — |
| 2 | 2021-07-05 15:15:58 | 0Nm_BPFO_03 | 210705_Outer_0.3mm_5cm | 1582 s |
| 3 | 2021-07-05 15:37:01 | 0Nm_BPFO_10 | 210705_Outer_1mm_5cm | 1203 s |
| 4 | 2021-07-05 16:03:30 | 0Nm_BPFI_03 | 210705_Inner_0.3mm_5cm | 1529 s |
| 5 | 2021-07-05 16:26:00 | 0Nm_BPFI_10 | 210705_Inner_1mm_5cm | 1290 s |

O `creation_time` interno é 9 h depois (2021/07/05 23:48 a 2021/07/06 01:26), o que bate com
o fuso da Coreia (UTC+9). Então `absolute_time` deve estar em UTC.

- **Classe = sessão = rolamento = horário.** Cada classe-severidade é um único run contínuo
  de 60 s. Entre um run e outro há 20–26 min, compatíveis com desmontar e trocar o rolamento.
  Todo o áudio foi gravado numa tarde, na ordem Normal → Externo → Interno. Qualquer deriva
  (temperatura da bancada, ruído ambiente, remontagem, aperto) está **confundida com a
  classe**. Um indício concreto é o tom de ~1454 Hz, provável engrenamento: 1454/24,25 ≈ 60
  dentes, e 60/29 = 2,069 bate com a razão de 2,07. Esse tom aparece com nível diferente em
  cada run, e no BPFO_10 o pico da banda 1440–1470 Hz cai em 1467,15 Hz.
- `_5cm` no run_name provavelmente é a distância do microfone. Não está documentado ("não
  encontrado").
- **O áudio não é simultâneo com a vibração nem com a corrente.** Os TDMS de corrente e
  temperatura foram criados em 25/06 (Normal), 28/06 (BPFI) e 30/06/2021 (BPFO), por outro
  sistema (NI FlexLogger, cDAQ-9185). Os MAT de vibração (cabeçalhos lidos por HTTP Range)
  foram criados nos mesmos dias de junho. O áudio é de 05/07/2021. A documentação sugere uma
  aquisição única ("collected from the accelerometers and microphone by the SIEMENS SCADAS"),
  mas os arquivos mostram campanhas diferentes.

## 7. Verdade de rotação

- **Não há tacômetro nem encoder na parte 1**, onde está o áudio. Os arquivos `rpm_*.csv` do
  artigo são da parte 2, outra base, sem áudio. Por isso `rot_esperada_hz` fica vazia.
- **Nominal**: 3010 rpm no eixo dos mancais. O artigo dá fs = 50,17 Hz.
- **Alimentação**: o motor é de 4 polos, com placa de 380 V / 60 Hz / 1770 rpm, mas opera
  **por inversor** abaixo de 60 Hz. Na corrente (outra sessão), o pico entre 20 e 80 Hz fica
  em **48,50–48,55 Hz** nos 5 TDMS (resolução de Welch: 0,05 Hz). Com 4 polos, a velocidade
  síncrona do motor é 24,26 Hz.
- **Estimativa no próprio áudio**, pelo envelope de 2 a 10 kHz, que **não é verdade direta**:
  eixo dos mancais a 50,20 Hz (3012 rpm) nos 5 arquivos, e linha do eixo do motor a 24,25 Hz
  (razão 2,0701 = caixa de 2,07). O escorregamento é ~0,1 % (sem carga). No BPFI_10, a linha
  do motor ficou fraca e o pico caiu na borda da banda (22,00 Hz). Nesse arquivo a razão
  calculada pelo script não vale.
- **Frequências de defeito**: com a geometria declarada (d = 7,90 mm, D = 38,5 mm, N = 9) e
  fs = 50,20 Hz, a teoria dá BPFO = 179,55 Hz e BPFI = 272,25 Hz. Medido: BPFO = 183,40 Hz
  no BPFO_10 (+44 dB, com harmônicos 366,8 e 550,15) e BPFI = 268,25 Hz no BPFI_10 (+35 dB,
  com bandas laterais de ±fs). A soma 183,4 + 268,25 = 451,65 Hz ≈ 9 × 50,20, o que é
  coerente com N = 9 e com um d/D efetivo diferente do declarado (~2 % de desvio).
- **Divergência**: o arXiv 2110.03348, do mesmo grupo, diz "motor rotates at 1505 rpm,
  doubled through the gearbox". O Data in Brief diz ×2,07. A medição (2,0701) confirma o
  Data in Brief.

## 8. Identificação da gravação de origem (`gravacao_id`)

`gravacao_id` = `run_name` do Test.Lab, gravado dentro do `.mat`
(`Signal.function_record.TL_export_properties_annotation.run_name`), por exemplo
`210705_Normal_5cm`. Cada arquivo é exatamente um run contínuo de 60 s, então arquivo =
gravação = rolamento = classe-severidade. Segmentos tirados de um mesmo arquivo nunca podem
cair em lados diferentes de uma divisão treino/teste. Com 1 gravação por classe-severidade,
**não existe divisão sem vazamento de sessão para classificar as 5 condições**.

## 9. Trabalhos que usam a base (áudio)

| Trabalho | O que li | Uso do áudio / divisão | Número reportado | Risco de vazamento |
|---|---|---|---|---|
| Jung et al. 2023, *Data in Brief* 48:109049 (artigo da base) | **texto completo** (XML do PMC) | descreve a base, sem experimento | — | — |
| W. Jung, J. Bae, Y.-H. Park, "Acoustic signal based non-contact ball bearing fault diagnosis using adaptive wavelet denoising", arXiv 2110.03348 (2021) | **texto completo** | 5 classes; janelas de 1 s com 10 % de sobreposição; 3 840 de treino e 960 de teste | 91,8 % (atributos + denoising); 52,1 % no áudio cru | alto: janelas da mesma gravação nos dois lados. **Divergência**: 4 800 janelas = 960 por classe, impossível com 60 s públicos (~66 janelas). O artigo diz "collected every two minutes", então usou mais áudio do que o publicado |
| H. Jo, Y. Yoo, M. Dai, S.-W. Ban, *Sensors* 2025 (PMC12655894), ensemble softmax multi-sensor | trechos por busca de palavras, não o texto inteiro | "Case 2" = KAIST, vibração + acústico; segmentos de 0,0343 s; 600 por classe, 300 de treino e 300 de validação + teste, aleatório, 10 repetições | 99,98 % | alto: segmentos aleatórios da mesma gravação de 60 s |
| S. Ding, G. Zhou, X. Wang, W. Li, *Entropy* 2025 (PMC12468436), CAVF-Net | trechos por busca de palavras | acústico + vibração reamostrados para 25,6 kHz, com ruído misturado; janelas de 2 s com passo de 1 s; 8:2 | 95,42 % fundido; 79,10 % só acústico | alto: janelas sobrepostas e divisão aleatória; além disso, "alinha temporalmente" áudio e vibração que **não são simultâneos** (§6) |
| M. M. Jensen et al., arXiv 2609.16314 (2026) | trechos por busca de palavras | **exclui o áudio** "due to incomplete availability across operating conditions" | — | — |

A Europe PMC lista 16 trabalhos que citam o artigo da base. Fora os acima, não verifiquei
se outros usam o áudio. Li et al. 2026 (PMC13419292) não menciona KAIST no texto completo.
Para os demais, a verificação de uso do áudio fica como "não encontrado".

## 10. Reprodução

```
.venv/Scripts/python.exe -I scripts/inventario_kaist.py data/kaist out/indices/kaist.csv --vib-remoto
```

O comando gera `out/indices/kaist.csv` (5 linhas, total de 300,0 s) e imprime todas as medidas
desta ficha. O loader é `scripts/lib/load_kaist.py`: `carregar()` lê o canal acústico,
`carregar_tacometro()` devolve None, `carregar_corrente()` lê o TDMS e `metadados()` lê os
dados do run. Precisa de `nptdms`, instalado no `.venv`. Observação: o `.venv` está em Python
3.12, não 3.14 como diz o `CLAUDE.md`.

## 11. Problemas encontrados

1. Áudio só em 0 Nm, com 5 condições (Normal, BPFO/BPFI de 0,3 e 1,0 mm), e
   **1 gravação de 60 s por condição**: 300 s no total.
2. Classe totalmente confundida com run, rolamento e horário.
3. Diferença de nível de até ~17 dB entre classes, um atalho trivial.
4. Áudio não simultâneo com corrente e vibração: dias diferentes, sistemas diferentes.
5. Nos TDMS, o nome interno dos arquivos BPFI está trocado: `0Nm_BPFI_03.tdms` traz
   `LogFile_0Nm_Inner_1mm`, e `0Nm_BPFI_10.tdms` traz `LogFile_0Nm_Inner_03mm`. A troca não
   afeta o áudio, cujos `run_name` batem com os nomes dos arquivos.
6. Os TDMS BPFO têm só 1 fase de corrente gravada (as outras têm 1 amostra).
7. `0Nm_Normal.tdms` tem 300 s; o artigo diz 120 s para o normal.
8. A documentação descreve colunas "Time Stamp/values" que não existem no `.mat`.
