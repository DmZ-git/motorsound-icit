# ToyCar do DCASE 2022 Task 2 (subconjunto do ToyADMOS2, NTT)

Ficha escrita em 23/10/2026 a partir dos arquivos baixados. Os números de §3 a §8 vêm de
`scripts/inventario_toycar.py`, rodado sobre `data/toycar/` com `--resumo`, `--nivel`, `--hash`,
`--rotacao` e `--reuso`. As saídas estão em `data/toycar/_metadados/inventario_*.txt`. Os
metadados das fontes (JSON do Zenodo, texto dos artigos, slides e receita do ToyADMOS2, página e
resultados do DCASE) também estão em `data/toycar/_metadados/`. Quando a documentação diverge do
que foi medido, registro as duas versões.

## 1. Origem, citação, licença, tamanho

- **Base:** ToyCar do *DCASE 2022 Challenge Task 2* ("Unsupervised Anomalous Sound Detection
  for Machine Condition Monitoring Applying Domain Generalization Techniques").
  O áudio é um recorte do ToyADMOS2, remixado com ruído de fábrica e reamostrado a 16 kHz.
- **Registros no Zenodo:** os três são CC BY 4.0, públicos e sem login (licença conferida pela API).

  | conjunto | DOI | arquivo | MD5 (confere com a API) |
  |---|---|---|---|
  | development (15/03/2022) | 10.5281/zenodo.6355122 | `dev_ToyCar.zip`, 982,0 MB | `1ccca43751353e516ee7d6d55457c705` |
  | additional training (15/04/2022) | 10.5281/zenodo.6462969 | `eval_data_ToyCar_train.zip`, 786,9 MB | `fde0d2e49a542c5e3e38f9ecd8329496` |
  | evaluation (01/06/2022) | 10.5281/zenodo.6586456 | `eval_data_ToyCar_test.zip`, 157,2 MB | `3c52961e75d60623a40b9dcc47ec1009` |

  O total baixado é de 1,93 GB.
- **Rótulos do eval:** vêm do repositório `github.com/Kota-Dohi/dcase2022_evaluator` (licença MIT),
  arquivos `ground_truth_data/`, `ground_truth_domain/` (commit 9785ae5c, 12/07/2022) e
  `ground_truth_attributes/` (commit de8719b7, 07/12/2022). Estão copiados em
  `data/toycar/_rotulos_eval/`. Há um problema com o terceiro arquivo, descrito em §9.
- **Citações.** Li o texto completo das duas referências principais:
  - Harada, N.; Niizumi, D.; Takeuchi, D.; Ohishi, Y.; Yasuda, M.; Saito, S. *ToyADMOS2: Another
    dataset of miniature-machine operating sounds for anomalous sound detection under domain
    shift conditions*. Proc. DCASE Workshop 2021, pp. 1–5; arXiv:2106.02369. Li o HTML do arXiv.
    A tarefa citava "EUSIPCO 2021", mas o próprio artigo, o DCASE e o slide da NTT citam o DCASE
    Workshop 2021.
  - Dohi, K. et al. *Description and discussion on DCASE 2022 Challenge Task 2...*. Proc. DCASE
    2022 Workshop; arXiv:2206.05876v2. Li o PDF inteiro.
- **Outras fontes lidas inteiras:**
  - o slide `ToyADMOS2_details.pdf` (8 páginas);
  - `UsersManual.md` e `recipe_benchmark.xlsx` do GitHub `nttcslab/ToyADMOS2-dataset`;
  - a página da task e a página de resultados do DCASE 2022.
- **O ToyADMOS2 completo** (Zenodo 4580270) tem 169,4 GB e licença `other-nc` (não comercial),
  em MPEG-4 ALS a 48 kHz. **Não foi baixado.**

## 2. Árvore e nomes

```
data/toycar/
├── _zips/                       os 3 zips (MD5 conferido)
├── extraido/
│   ├── dev_ToyCar/ToyCar/{train,test}/*.wav          3600  + attributes_0{0,1,2}.csv
│   ├── eval_data_ToyCar_train/ToyCar/train/*.wav     3000  + attributes_0{3,4,5}.csv
│   └── eval_data_ToyCar_test/ToyCar/test/*.wav        600  (nomes cegos)
├── _rotulos_eval/               gt_data_0X, gt_domain_0X, gt_attributes_0X (+ README, LICENSE)
└── _metadados/                  JSON do Zenodo, textos/slides/receita, saídas do inventário
```

**Padrão dos nomes rotulados:**
`section_<SS>_<source|target>_<train|test>_<normal|anomaly>_<NNNN>_car_<ID>_spd_<VV>V_mic_<m>_noise_<n>.wav`.

Exemplos reais:

- `section_00_source_train_normal_0000_car_A1_spd_28V_mic_1_noise_1.wav`
- `section_00_target_test_anomaly_..._car_C1_...`
- `section_02_target_..._mic_2_noise_2.wav`
- `section_03_source_train_normal_0000_car_F1_spd_31V_mic_1_noise_6.wav`

O eval de teste tem nomes cegos: `section_03_0000.wav` … `section_05_0199.wav`.

**Atributos do nome:**

- `section`: 00–02 no dev, 03–05 no eval. Cada seção é um cenário de deslocamento de domínio:
  - 00 e 03: modelo do carro + ruído;
  - 01 e 04: velocidade + ruído;
  - 02 e 05: microfone + ruído.
- `source`/`target`: domínio.
- `train`/`test`: partição.
- `normal`/`anomaly`: classe.
- `NNNN`: índice dentro do grupo.
- `car`: ID do carrinho. No ToyADMOS2 é a coluna `CarID`, "ID of machine hardware recorded".
  A letra é a configuração (modelo) e o número é o exemplar físico.
- `spd`: tensão da fonte × 10 (28V = 2,8 V).
  A página do DCASE chama o campo de "speed_level_times_10", mas os valores são as tensões
  (2,8 a 4,0 V), não níveis 1 a 5. **Divergência.**
- `mic`: canal do microfone do ToyADMOS2. Os canais 1–3 são dinâmicos e os canais 4–5 são condensadores.
  A página do DCASE diz "we use only the first channel", mas a seção 02/target usa `mic_2`.
- `noise`: índice do ruído de fábrica misturado.

Os CSV de atributos do DCASE trazem `speed` no lugar de `spd`. Os do eval omitem o prefixo
`ToyCar/train/`. Fora isso, os 6 600 valores conferem com os nomes (0 divergências).

## 3. Taxa, canais, resolução (medidos)

Os 7 200 wav são todos iguais no formato: 16 000 Hz, 1 canal, PCM 16 bit, 10,000 s.
O ToyADMOS2 original é de 48 kHz e 12 s, segundo o artigo. O DCASE não diz como os 10 s foram
recortados dos 12 s: **não encontrado**. Nenhum arquivo de teste chega a |y| ≥ 0,999, ou seja, não há clipping.

## 4. Duração

São 7 200 arquivos × 10 s = 20,0 h: 18,33 h normais (6 600) e 1,67 h de anomalia (600).
Por seção:

- treino: 990 source + 10 target, todos normais;
- teste: 50 normais + 50 anomalias por domínio.

Isso bate com a documentação.

## 5. Matriz classe × unidade física

**Como o ToyADMOS2 fez as anomalias.** As fontes são o artigo, o slide p. 5–6 e a receita.

- **Cinco configurações (A–E).** Cada uma combina chassi, motor, pinhão, redução, mancal e pneu:

  | configuração | chassi | motor | redução |
  |---|---|---|---|
  | A | AR | kit standard | 4,2:1 |
  | B | VZ | Torque-Tuned 2 | 3,5:1 |
  | C | FM-A | Torque-Tuned 2 | 3,5:1 |
  | D | AR | Light-Dash | 5:1 |
  | E | FM-A | kit standard | 4:1 |

- **Dois exemplares físicos por configuração.** A receita lista os `CarID` A1, A2, …, E1, E2,
  ou seja, 10 carrinhos.
- **As anomalias saem de peças danificadas trocadas no próprio carrinho.** As peças são o eixo
  (empenado), a engrenagem (deformada ou derretida) e a roda (danificada), cada uma em 3 graus
  (Lo/Mid/High). O slide mostra a peça normal e as três danificadas lado a lado.
- **Pela receita, cada exemplar tem gravações normais e anômalas.** O exemplar 1 recebe sobretudo
  eixo e engrenagem deformada; o exemplar 2 recebe engrenagem derretida e roda.

Logo, **a unidade física é o atributo `car`**, e a anomalia é feita no mesmo carro.

**Matriz medida, só no teste (dev).** Só o teste tem anomalias.

| seção/domínio | carro | anomalia | normal |
|---|---|---|---|
| 00 source | A1 | 13 | 9 |
| 00 source | A2 | 12 | 16 |
| 00 source | E1 | 17 | 7 |
| 00 source | E2 | 8 | 18 |
| 00 target | C1 | 24 | 25 |
| 00 target | C2 | 26 | 25 |
| 01 source e target | A1 | 25 + 25 | 24 + 25 |
| 01 source e target | A2 | 25 + 25 | 26 + 25 |
| 02 source e target | A1 | 25 + 25 | 25 + 24 |
| 02 source e target | A2 | 25 + 25 | 25 + 26 |

- **Dev.** São 6 carrinhos (A1, A2, C1, C2, E1, E2). **Todos têm normal e anomalia** no teste.
  A1 e A2 dominam: 2 946 de 3 600 arquivos, e aparecem nas três seções.
  A seção 00 tem treino target de só 10 arquivos (C1 6, C2 4).
- **Eval (seções 03–05).** O treino usa os carros F1, F2, G1, G2, H1 e H2. **Essas letras não
  existem no ToyADMOS2 público**, que só tem A–E segundo a receita, o slide e a lista de arquivos do
  Zenodo 4580270. Quais carrinhos físicos são, e se são novos ou renomeados: **não encontrado**.
  - Seção 04: só F1/F2.
  - Seções 03 e 05: F (source), G (target, 5 + 5 arquivos), H.
  - Os 600 wav de teste do eval não têm `car` utilizável (ver §9). No índice, `unidade` é
    `desconhecida_sec0X`.
- **Não há confusão classe = unidade.** No teste dev, anomalia e normal estão balanceadas dentro
  de cada carro, e dentro de cada (seção, domínio, tensão).
- **Ressalva: há poucas unidades.** São só 6 carros no dev, e na prática 2 deles (A1, A2) carregam
  quase tudo. Separar por carro tira quase todos os dados. O DCASE separa por seção e domínio, não por carro.

## 6. Matriz classe × condição

No teste do dev, cada célula tem normal e anomalia em número parecido:

| seção | source | target |
|---|---|---|
| 00 | carros A1, A2, E1, E2; 28V e 34V; mic 1; noise 1 | carros C1, C2; 28V e 34V; mic 1; noise 2 |
| 01 | A1, A2; 28V e 40V; mic 1; noise 1 | A1, A2; 31V e 37V; mic 1; noise 2 |
| 02 | A1, A2; 34V e 40V; mic 1; noise 1 | A1, A2; 34V e 40V; **mic 2**; noise 2 |

- **Tensões e ruído.** A combinação (seção, domínio, tensão) tem sempre 24–26 por classe.
  O ruído é sempre 1 no source e 2 no target, nas três seções. Ou seja, o domínio vem sempre
  junto com uma troca de ruído.
- **O eval (03–05)** usa ruído 6 em todo o treino, e só mic 1.
  O teste do eval só traz classe e domínio (gt_data e gt_domain), 50/50 por domínio.
- **Normal e anomalia do mesmo carro saem em condições compatíveis:** mesma tensão, mesmo mic,
  mesmo ruído, mesma seção e domínio.
- **Datas e sessões não aparecem** em nenhum nome nem CSV. Se normais e anomalias foram gravados em
  dias diferentes: **não encontrado**.

**Nível (RMS sem DC, todos os 1 200 wav de teste)**

- A mediana de anomalia − normal por (seção, domínio, carro) vai de −2,5 a +1,9 dB, sem sinal
  consistente.
- Um detector trivial que usa o RMS como score dá AUC entre 0,24 e 0,62 por seção e domínio.
  No dev fica perto de 0,5 (0,39–0,62). No eval há AUC < 0,5 em 03/target (0,24) e 04/source
  (0,27): ali as anomalias são *mais silenciosas*.
- **Não há atalho de nível que favoreça a anomalia.** O RMS por arquivo fica entre −36 e −22 dBFS.

## 7. Verdade de rotação

- `spd` é **setpoint**, não medida: é a tensão de uma fonte estabilizada ligada direto ao motor
  CC (artigo e slide p. 3/5: "To control operating speed, five voltage levels, 2.8–4.0 V").
- **Não há rpm, tacômetro nem corrente** nos arquivos. Os níveis do ToyADMOS2 são
  1 = 2,8 V, 2 = 3,1 V, 3 = 3,4 V, 4 = 3,7 V e 5 = 4,0 V.
- A rotação de um motor CC sem carga é aproximadamente proporcional à tensão. A constante de
  cada motor (kit standard, Torque-Tuned 2, Light-Dash) e a carga do rolo do "speed checker" não
  estão publicadas. Por isso a relação tensão → rpm do motor ou da roda (pela redução de §5) **não
  pode ser calculada**.
- **Conferência (estimativa):** procurei o pico dominante em 20–1000 Hz (Welch, ~1 Hz) nos normais
  de treino source do dev. Ele fica em **130,9 Hz em todos os carros e em todas as tensões**
  (mediana; 2,8, 3,4 e 4,0 V). Ele não acompanha a tensão, então não é rotação. Hipótese não
  confirmada: componente do ruído 1 ou do equipamento.
- `f_alim_hz` e `rot_esperada_hz` ficam vazios no índice. `velocidade` = `spd` bruto e
  `carga` = tensão em V.

## 8. Gravação de origem

- Cada wav é um clipe de 10 s com ruído somado.
- **Hash do áudio:** os 7 200 arquivos são distintos.
- No ToyADMOS2, cada gravação tem 12 s e 5 canais simultâneos. Como o DCASE escolheu e recortou
  os clipes **não está documentado**. Em especial, ficam em aberto duas perguntas:
  - se um wav `mic_2` (02/target) e um `mic_1` (02/source) podem vir do mesmo evento gravado;
  - se a mesma gravação limpa foi reaproveitada entre as seções 00, 01 e 02, que usam os mesmos A1/A2 e tensões.
- **Teste de reuso (`--reuso`).** Usei a correlação cruzada normalizada máxima, em qualquer atraso,
  entre 40 arquivos de um grupo e todos os do outro. Arquivos independentes ficam em torno de 0,05–0,07.
  - Comparando seções diferentes com o mesmo carro e a mesma tensão, o melhor par de cada arquivo
    tem mediana 0,08–0,13. Mas 0–10 % dos arquivos têm um par acima de 0,5 (máximo 0,67).
  - **O controle com carros diferentes dá o mesmo resultado:** A1×A2 chega a 0,63 e A1×E2 a 0,55,
    com 7 % dos arquivos acima de 0,5.
  - Logo, a coincidência vem de **trechos de ruído compartilhados**. O mixer do ToyADMOS2 sorteia
    o ruído "randomly picked with replacement" (UsersManual). Não é a mesma gravação do carro.
  - Não encontrei indício de reuso do sinal limpo, mas o teste não o exclui.
  - Há risco de vazamento: o mesmo trecho de ruído pode cair no treino e no teste.
- `gravacao_id` = `toycar/<dev|eval>/<nome sem extensão>`, um por wav. É a melhor aproximação
  disponível. Para validação honesta, agrupar por `unidade` (carro) é mais seguro.

## 9. Split do DCASE, resultados, vazamento

**Protocolo.** Treina-se só com normais: 990 source + 10 target por seção. Testa-se com
100 normais + 100 anomalias por seção. A métrica é a AUC por domínio e a pAUC (p = 0,1) por seção
(Dohi et al.). No eval, o domínio e os atributos de teste são ocultos.

**Rótulos do eval.** `gt_data` (classe) e `gt_domain` (domínio) do evaluator estão no mesmo commit
de 12/07/2022 e são os do ranking oficial. Já o `gt_attributes` (dez/2022) traz um
`file_name_with_attributes` que **discorda** dos outros dois em ~50 % dos arquivos: classe e
domínio cruzados ficam em proporções de acaso, nas três seções.

Conferi pelo áudio (script ad hoc, kNN k = 5 em espectro Welch, treinado no eval train rotulado):

| seção | `car` previsto × `gt_attributes` | domínio previsto × `gt_domain` | domínio previsto × `gt_attributes` |
|---|---|---|---|
| 03 | 0,195 | 0,785 | ~0,5 |
| 04 | 0,390 | 0,59 | ~0,5 |
| 05 | 0,175 | 0,845 | ~0,5 |

A seção 04 tem só F1/F2 e o deslocamento é de velocidade. Conclusão: **`gt_domain` está alinhado
com o áudio; `gt_attributes` não** (parece permutado). O índice usa `gt_data`/`gt_domain` e
**deixa car/spd/mic do eval de teste como desconhecidos**.

**Resultados publicados para o ToyCar** (Dohi et al., Tabela 1/2; página de resultados):

- **Baseline AE, dev:**

  | | sec 00 | sec 01 | sec 02 |
  |---|---|---|---|
  | AUC source | 86,42 | 89,85 | 98,84 |
  | AUC target | 41,48 | 41,93 | 26,50 |
  | pAUC | 51,31 | 54,08 | 52,79 |

  Para a sec 02, a página da task dá pAUC 52,96 % e 52,79 % como média aritmética, enquanto a
  Tabela 1 do artigo põe 52,79 na sec 02. **Divergência.**
- **Baseline MobileNetV2, dev:** AUC source 47,40/62,02/74,19; target 56,40/56,38/45,64.
- **Eval (página de resultados, 83 submissões):**
  - AE: AUC 61,18, pAUC 60,21 (source 82,04 / target 58,22);
  - MobileNetV2: 42,79 / 53,44;
  - 1º no ranking geral (Liu_CQUPT): 88,45 / 81,83;
  - maior AUC no ToyCar: 97,17 (Wei_HEU_task2_2);
  - mediana das submissões: 76,33.
- Os relatórios técnicos das equipes **não foram lidos**.

**Riscos de vazamento e de leitura errada:**

1. **O domínio coincide com o ruído.** Nas três seções dev, o ruído é 1 no source e 2 no target.
   Um modelo pode aprender o ruído em vez do carro.
2. **A1/A2 se repetem nas seções 00–02.** Treinar numa seção e testar noutra não é generalização
   entre unidades.
3. **Os trechos de ruído se repetem entre arquivos** (§8).
4. **O AE com AUC target < 50 %** mostra que a mudança de domínio pesa mais que a anomalia.
5. **Classificadores de seção ou atributo** (MobileNetV2 e boa parte do top 10) usam
   carro/velocidade/ruído como alvo auxiliar.

## 10. Consequências para o protocolo

1. É a primeira base auditada em que **anomalia e normal vêm do mesmo carrinho físico**, na mesma
   condição, e sem atalho de nível. Isso serve para (5), (6) e (8).
2. Mas há só **6 carros rotulados no dev**, e só **2 (A1, A2) com volume**. Deixar um carro de fora
   é possível só em 00/source (A1, A2, E1, E2) e com poucos arquivos. Toda tabela deve informar
   por carro.
3. **A velocidade é só tensão nominal** (5 níveis; o dev usa 2,8–4,0 V). Não há rpm, então não
   serve para (4) nem para (ordens) com verdade de rotação.
4. **O eval de teste não tem carro:** serve só para classe/domínio. Não usar o `gt_attributes` do evaluator.
5. **Controlar o ruído:** domínio = ruído no dev, e há ruído repetido entre arquivos. Agrupar por carro e relatar por domínio.
