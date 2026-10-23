# Outras bases públicas com som de motor ou máquina rotativa

Auditoria feita em 22/10/2026. Nenhum dado foi baixado. Consultei só páginas, metadados de API (Zenodo, Mendeley, DataCite) e o texto dos artigos. A coluna "Fonte" diz o que li em cada caso. Quando uma informação não foi encontrada, está escrito "não encontrado" ou "não verificado".

Estas bases já têm ficha própria e por isso só aparecem citadas aqui: UOEMD-VAFCVS (Ottawa, Mendeley `msxs4vj48g`), KAIST/Jung et al. 2023 (Mendeley `ztmf3m7h5x`), MIMII e MIMII DUE. Como referência também entram IDMT-ISA-ELECTRIC-ENGINE e BLDC A2212 (Estacio 2023, Mendeley `j4yr5fmhv4`).

Caminhos do artigo e o que cada um exige:

- **(2)** distúrbios de alimentação audíveis: falhas de tensão reais.
- **(4)** tacômetro acústico em rampas: velocidade conhecida.
- **(5)** vários motores com um microfone: unidades diferentes em condições compatíveis.
- **(6)** auditoria de avaliação: unidades suficientes para separar "motor fora".
- **(8)** anomalia sem rótulo com features harmônicas: muito dado normal, várias unidades e ruído real.
- **(ordens)** features de ordem: falhas mecânicas e elétricas reais, com rotação conhecida ou rastreável.

---

## 1. Tabela-resumo

Legenda: "Rot." é a verdade de rotação. "Elét." indica falha elétrica ou de tensão. "Login" indica se o download exige conta.

| Base | Máquina | Unid. físicas | Mic (taxa) | Rot. | Elét. | Ruído real | Tamanho | Licença | Login |
|---|---|---|---|---|---|---|---|---|---|
| **MaFaulDa** (UFRJ) | bancada SpectraQuest MFS, motor 1/4 cv | 1 | sim, Shure SM81 (50 kHz) | **tacômetro analógico gravado** | não | não (laboratório) | 12,9 GB (full.zip) | não encontrado na página | não |
| **UORED-VAFCLS** (Ottawa) | motor monofásico + rolamento de teste | 1 motor; 20 rolamentos trocados | sim, PCB 130F20 (42 kHz) | **sensor Hall → RPM gravado** (1750 rpm fixo) | não | não | 3,87 GB | CC BY 4.0 | não |
| **IM-VACD** (Ottawa) | 8 motores trifásicos Marathon D396 (os mesmos do UOEMD) | 8, mas 1 motor por classe | sim, 3 celulares (42 kHz declarado) | só setpoint (15/20/25/30 Hz) | **sim**: enrolamento do estator, desequilíbrio de tensão, barras quebradas | parcial (celular no laboratório) | 3,68 GB | CC BY 4.0 | não |
| **ToyADMOS2** (NTT) | ToyCar (mini 4WD, motor CC) e ToyTrain (HO) | ToyCar: 5 configurações; ToyTrain: vários modelos | sim, 5 (car) e 8 (train) canais | **tensão de fonte estabilizada** (5 níveis, 2,8–4,0 V) e nível de velocidade | não (tensão só controla velocidade) | ruído real tocado por alto-falantes | 169 GB | "other-nc" (não comercial) | não |
| **ToyADMOS** (NTT) | carro, esteira e trem de brinquedo | vários casos (número não verificado) | 4 mics (48 kHz) | não | não | ruído de fábrica misturado | ~180 GB comprimido, >440 GB extraído | LICENSE.pdf (não lido) | não |
| **DCASE 2020 T2** | ToyCar, ToyConveyor, fan, pump, slider, valve | 6–7 "Machine IDs" por tipo (3–4 no dev + outras no eval) | 16 kHz, 1 canal | não | não | ruído misturado | dev 8,1 GB | CC BY-NC-SA 4.0 | não |
| **DCASE 2021 T2** | fan, gearbox, pump, slider, ToyCar, ToyTrain, valve | 6 seções por tipo (seção ≈ machine ID) | 16 kHz, 1 canal (clipes de 10 s) | não, só domínio | não | ruído misturado | dev 7,5 GB | CC BY-NC-SA 4.0 | não |
| **DCASE 2022 T2 (MIMII DG + ToyADMOS2)** | fan, gearbox, bearing, slider, ToyCar, ToyTrain, valve | 6 seções por tipo, uma por tipo de deslocamento de domínio (não necessariamente unidades) | 16 kHz, 1 canal | **atributo no nome do arquivo**: bearing `vel`, slider `vel`, gearbox `volt`, ToyCar/ToyTrain `spd` | gearbox: atributo de tensão de operação, não falha | ruído de fábrica misturado | dev 6,2 GB | CC BY 4.0 (Zenodo) | não |
| **DCASE 2023 T2** | dev: os mesmos 7 tipos; eval: ToyDrone, ToyNscale, ToyTank, Vacuum, bandsaw, grinder, shaker | 1 seção por tipo | 16 kHz, 1 canal | atributos `vel`, `volt`, `spd` (drone/tank/vacuum: modelo e nível de velocidade) | não | misturado | dev 2,2 GB, add. 2,0 GB, eval 0,4 GB | CC BY 4.0 | não |
| **DCASE 2024 T2** | dev: os mesmos 7; eval: 3DPrinter, AirCompressor, **BrushlessMotor**, HairDryer, HoveringDrone, RoboticArm, Scanner, ToothBrush, ToyCircuit | 1 seção por tipo | 16 kHz, 1 canal | atributos só em fan/bearing/valve/ToyCar (dev) e 3DPrinter/hairdryer/robotic arm/scanner/ToyCircuit (eval); **BrushlessMotor sem atributos** | não | misturado | dev 2,2 GB, add. 2,0 GB, eval 0,4 GB | CC BY-NC-SA 4.0 | não |
| **DCASE 2025 T2** | tipos novos no eval, com dados extras de máquina limpa e de ruído | 1 seção por tipo | não verificado | não verificado | não verificado | misturado | não verificado | não verificado | não |
| **KAIST drone** (Yi, Choi, Lee) | 3 quadricópteros de modelos diferentes | 3 | 2 mics, gravado a 48 kHz e entregue a 16 kHz | não | "motor cap amassado" (mecânico) | ruído real do campus misturado (SNR 10–15 dB) | 7,6 GB | CC BY 4.0 | não |
| **QU-DMBF** (Qatar Univ.) | máquina A (CA trifásica + inversor) e máquina B (motor CC) | 2 | sim (4,096 kHz) | não encontrado | não | não | não encontrado (Google Drive) | não encontrada | não (Google Drive) |
| **SUBF v2.0** (UET Taxila) | motor de indução 0,25 cv + rolamento trocado | 1 | BOYA BY-M1 (10 kHz) | não (1440 rpm nominal) | não | não informado | 18 h (6480 clipes de 10 s) | não verificada | sim (Kaggle) |
| **GUET acoustic ball bearing** | bancada HZXT-008, rolamento 6200 | 1 bancada | 3 mics (32,768 kHz) | setpoint (800/1000/1200 rpm) | não | não | ~10,6 GB por classe × 5 ≈ 53 GB | CC BY 4.0 | não |
| **FSTF Mechanical Lab** | rotor GUNT PT500, SKF 6004 | 1 | app de celular (44,1 kHz) | setpoint (3 velocidades) | não | sim (ambiente real) | 77 MB | CC BY 4.0 | não |
| **HB-Bearing** (Yanshan) | motor YL601-4 movendo esteira de carvão | 1 motor sob teste (outros ligados ao lado) | gravador de câmera (16 kHz) | 1200 rpm fixo | não | **sim** (vozes e máquinas) | não verificado | CC BY 4.0 (DataCite) | **sim** (assinatura IEEE DataPort) |
| **AHU Parabolic Mirror** | motor assíncrono + inversor | 1 | espelho parabólico + mic direto (20 kHz) | 3 velocidades (valores não informados) | não | não | não verificado | CC BY 4.0 (DataCite) | **sim** (IEEE DataPort) |
| **Gdańsk AC motor voltage & audible noise** (Pałczyńska 2022) | acionamento CA naval com conversor de frequência (modelo de laboratório) | não encontrado | mic condensador ½" + medidor Sonopan | não encontrado | **tensão do conversor medida** (distúrbios do PWM, não falhas) | não | não encontrado | CC BY 4.0 (DataCite) | não verificado (página em JS) |
| **DREGON** (Inria) | 1 quadricóptero | 1 | 8 mics | **velocidade de cada rotor registrada** | não | não (salas) | não verificado | não verificada | não verificado |
| **UPV UAV (projeto MODERA)** | 1 quadricóptero em câmara anecoica | 1 | 6 mics a 2 m | só throttle (133 combinações) | não | não (anecoica) | 9,7 GB | CC BY-SA 4.0 | não |
| **ADCP** (Lee et al. 2025) | 1 motor KDE 5215XF + hélice | 1 | BOYA BY-PM700 (48 kHz) | só throttle | não | não | não encontrado | não encontrada | download não encontrado |
| **Paderborn KAt** | motor síncrono + rolamento 6203 | 1 bancada | **não tem** | velocidade e torque medidos | só correntes (não é falha elétrica) | não | — | CC BY-NC 4.0 | não |
| **Voiceprint bearing** (Mendeley `gprfs9hybk`) | motor de indução 3 kW | 1 | 44,1 kHz, mas **só imagens** | não | não | não | 0,97 GB | CC BY 4.0 | não |
| **DT-UOIMSD** (Ottawa) | gêmeo digital do UOEMD | sintético | canal "Acoustic" sintético | setpoint | sintético | não | não verificado | CC BY 4.0 | não |
| **Oviedo bomba centrífuga** (Zenodo 18500146) | bomba centrífuga com cavitação | 1 | só espectros/scripts | não | não | não | 0,07 GB | CC BY 4.0 | não |
| **Compressor de ar IITK** (Verma et al. 2016) | compressor alternativo de 1 estágio | 1 | 16 kHz na versão do MathWorks | não | não | oficina real | não verificado | não verificada | não verificado |

---

## 2. Promissoras (valem uma ficha completa)

### 2.1 MaFaulDa — Machinery Fault Database (UFRJ)

- **Link:** https://www02.smt.ufrj.br/~offshore/mfs/page_01.html (download sem login: `database/mafaulda/full.zip`, Content-Length 12.896.655.242 bytes, conferido com HEAD).
- **Citação:** a base não tem artigo de referência oficial. A lista awesome-bearing-dataset aponta como "artigo comumente citado" o trabalho do SBrT 2017, doi:10.14209/sbrt.2017.133. Não abri esse artigo.
- **Máquina:** SpectraQuest MFS ABVT, motor de 1/4 cv. Uma bancada só.
- **Classes:** 1951 sequências de 5 s a 50 kHz (250 mil amostras cada). São 49 normais, uma por rotação fixa entre 737 e 3686 rpm em passos de ~60 rpm, mais desbalanceamento (6–35 g), desalinhamento horizontal e vertical, e falhas de rolamento (pista externa, esfera, gaiola) nos mancais underhang e overhang.
- **Sensores:** tacômetro analógico, 2 acelerômetros triaxiais e o microfone Shure SM81 (20–20.000 Hz).
- **Falha elétrica:** não tem. Ruído real: não, é laboratório.
- **Fonte:** li a página do projeto (lista de sensores, tabela de sequências e faixa de rpm). Conferi o tamanho com HEAD. A página não fala de licença.
- Encaixe em cada caminho:
  - **(2)** não serve, porque não há distúrbio de tensão.
  - **(4)** serve bem para validar o tacômetro acústico em regime: são dezenas de rotações com tacômetro gravado. Não há rampas, cada arquivo tem velocidade fixa.
  - **(5)** e **(6)** não servem, porque há uma unidade só.
  - **(8)** é fraco, porque só há 49 arquivos normais.
  - **(ordens)** é a melhor base pública que achei para ordens mecânicas, porque a rotação é medida por tacômetro e cada falha aparece em muitas velocidades. Não tem falha elétrica.

### 2.2 ToyADMOS2 (NTT) e a entrada mais leve pelo DCASE 2022/2023

- **Link:** https://github.com/nttcslab/ToyADMOS2-dataset e doi:10.5281/zenodo.4580270.
- **Citação:** Harada, N. et al., "ToyADMOS2: Another dataset of miniature-machine operating sounds for anomalous sound detection under domain shift conditions", arXiv:2106.02369, 2021. Li o texto completo em HTML.
- **ToyCar:** mini 4WD com motor CC ligado a uma **fonte estabilizada em 5 tensões (2,8; 3,1; 3,4; 3,7; 4,0 V)** para controlar a velocidade. Há 5 configurações de chassi e peças (A–E), 5 microfones (3 dinâmicos e 2 condensadores) e clipes de 12 s. São mais de 35 mil amostras × 5 canais e mais de 8 mil anomalias em 300 padrões, com 3 profundidades de dano.
- **ToyTrain:** trem HO com 5 níveis de velocidade e 8 canais.
- **Ruído:** ambiental real, tocado por 4 alto-falantes e regravado.
- **Tamanho e licença:** 169 GB no Zenodo, licença "other-nc", não comercial. Conferi pela API do Zenodo.
- **DCASE 2022 dev:** traz parte desses dados a 16 kHz em clipes de 10 s, já com atributos no nome do arquivo (`car_<modelo>_spd_<nível>_mic_<n>_noise_<n>`). Tem 6,2 GB e licença CC BY 4.0. Fonte: página da task e API do Zenodo.
- Encaixe em cada caminho:
  - **(2)** não serve. A tensão é um setpoint de velocidade, não um distúrbio. Ainda assim é a única base em que a tensão de alimentação do motor é controlada e conhecida.
  - **(4)** é parcial: a velocidade conhecida é um nível ou tensão, sem rpm medido nem rampas.
  - **(5)** e **(6)** servem razoavelmente: são 5 configurações físicas de carro em condições iguais, e no DCASE há 6 ou mais seções por tipo.
  - **(8)** serve bem: muito dado normal, várias unidades e ruído real, ainda que tocado por alto-falante.
  - **(ordens)** é limitado: as falhas são mecânicas e a rotação é só nominal.

### 2.3 IM-VACD — Induction Motor Vibration and Acoustic Cellular Device (Ottawa)

- **Link:** https://data.mendeley.com/datasets/yc8yhg5xjd/2 (doi:10.17632/yc8yhg5xjd.2).
- **Citação:** a lista awesome-bearing-dataset indica doi:10.1016/j.ymssp.2026.114922. Não abri o artigo.
- **Motores:** os **mesmos 8 motores Marathon D396** do UOEMD, com falhas feitas pela SpectraQuest.
- **Classes:** saudável, desbalanceamento do rotor, desalinhamento, **falha de enrolamento do estator**, **desequilíbrio de tensão**, rotor empenado, **barras quebradas** e rolamento.
- **Condições:** 15/20/25/30 Hz constantes, com e sem carga, 10 s por arquivo.
- **Sensores:** 3 celulares (iPhone 13, Galaxy S6, Galaxy A50). O áudio é declarado a 42 kHz; a lista awesome-bearing relata um arquivo a 48 kHz.
- **Tamanho e licença:** 3,68 GB, CC BY 4.0.
- **Fonte:** API pública do Mendeley (descrição completa) e a lista awesome-bearing-dataset.
- **Atenção:** a descrição fala em "constant and variable speed", mas a codificação dos arquivos só lista velocidades constantes. Também **cada classe é um motor diferente**, então classe e unidade se confundem, como no UOEMD.
- Encaixe em cada caminho:
  - **(2)** serve: tem desequilíbrio de tensão e falha de estator reais, gravados com microfone barato. Complementa o UOEMD com outro tipo de microfone.
  - **(4)** não serve, porque só há setpoint, sem tacômetro e sem rampa.
  - **(5)** é parcial: são 8 unidades, mas cada uma com uma falha.
  - **(6)** serve como teste de transferência de microfone sobre as mesmas unidades do UOEMD.
  - **(8)** é fraco, porque há um motor saudável só.
  - **(ordens)** tem falhas elétricas e mecânicas reais, com rotação conhecida pelo setpoint do inversor.

### 2.4 UORED-VAFCLS (Ottawa), reserva

- **Link:** https://data.mendeley.com/datasets/y2px5tg92h (versão 5).
- **Citação:** Sehri, M.; Dumond, P.; Bouchard, M., "University of Ottawa constant load and speed rolling-element bearing vibration and acoustic fault signature datasets", *Data in Brief*, 2023, doi:10.1016/j.dib.2023.109327. Li as seções 2 e 3 do texto completo no PMC10331275.
- **Máquina:** um motor monofásico a 1750 rpm nominais. O rolamento de teste é trocado a cada ensaio: são 20 rolamentos (NSK 6203ZZ e FAFNIR 203KD), cada um em 3 estados (saudável, falha em desenvolvimento, falhado), com degradação acelerada.
- **Classes:** pista interna, pista externa, esfera e gaiola, 5 rolamentos de cada.
- **Sensores:** microfone PCB 130F20 a 2 cm do mancal, acelerômetro e **sensor Hall com engrenagem de 2 dentes convertido em RPM**, mais célula de carga e termopares. Tudo a 42 kHz, 10 s por conjunto, 60 conjuntos.
- **Tamanho e licença:** 3,87 GB, CC BY 4.0 (API do Mendeley).
- Encaixe em cada caminho:
  - **(4)** serve como checagem de rpm em regime fixo.
  - **(ordens)** serve para rolamentos com RPM medido e falhas naturais em desenvolvimento.
  - **(5)**, **(6)** e **(8)** não servem, porque o motor é um só.
  - **(2)** não serve.

---

## 3. Descartadas, com o motivo

- **DCASE 2020/2021 T2.** É MIMII, MIMII DUE e ToyADMOS reempacotados a 16 kHz (já cobertos). Não há verdade de rotação nem atributos úteis. A licença é CC BY-NC-SA 4.0. Ficam só como referência para **(8)**. Fonte: página da task 2020, descrições no Zenodo e API.
- **DCASE 2022–2025 T2 (fora o que entrou em 2.2).** O formato nativo de 16 kHz bate com o alvo, e há atributos `vel` (bearing, slider) e `volt` (gearbox). Mesmo assim:
  - desde 2023 há **uma seção por tipo**, o que mata **(5)** e **(6)**;
  - o ruído é **misturado** depois da gravação;
  - o **BrushlessMotor** de 2024 vem **sem atributos**;
  - no gearbox, `volt` é uma tensão de operação, não um distúrbio.

  Ainda assim pode valer como base externa para **(8)**. Fonte: páginas 2022/2023/2024, arXiv:2406.07250 (HTML), API do Zenodo. Os detalhes de 2025 não foram verificados além do resumo do arXiv:2506.10097.
- **ToyADMOS (2019).** Tem 4 mics a 48 kHz e cerca de 540 h normais, mas não tem velocidade nem falha elétrica, e é enorme (>440 GB). O ToyADMOS2 cobre melhor o caso de domínio. Fonte: README no GitHub e resumo do artigo (arXiv:1908.03299).
- **KAIST drone (Yi, Choi, Lee, ICSV29; arXiv:2304.11708; Zenodo 7779574, CC BY 4.0, 7,6 GB).** Tem 3 unidades a 16 kHz, falhas de hélice cortada e de tampa de motor amassada em cada um dos 4 motores, e ruído do campus **somado** (SNR 10–15 dB). Não tem rpm nem falha elétrica, e o motor é de drone com hélice. Serve no máximo para um teste lateral de **(5)**. Fonte: Zenodo e texto do arXiv (extraí os trechos de amostragem e de falhas).
- **QU-DMBF.** São 2 máquinas (CA com inversor e CC), 19 configurações de rolamento e 13,5 h, mas o **som está a 4,096 kHz** (banda só até 2 kHz). Não encontrei tacômetro nem licença, e a distribuição é por Google Drive. Citação: Kiranyaz, S.; Devecioglu, O. C. et al. (ordem de autores do arXiv), "Exploring Sound vs Vibration for Robust Fault Detection on Rotating Machinery", *IEEE Sensors Journal*, doi:10.1109/JSEN.2024.3405889 (arXiv:2312.10742). Li o texto completo e o README do GitHub `OzerCanDevecioglu/...`.
- **SUBF v2.0 (UET Taxila).** Tem 1 motor a 1440 rpm, só rolamento (pista interna e externa), 10 kHz e Kaggle com login. A citação indicada é *Digital Signal Processing* 2025, doi:10.1016/j.dsp.2024.104776, que não abri. Todos os dados vêm da lista awesome-bearing-dataset; **não abri a página do Kaggle**.
- **GUET multi-condition acoustic ball bearing.** Tem 1 bancada, 3 velocidades e 3 cargas, 3 mics a 32,768 kHz e 50 min por condição, mas o formato é **.bkc proprietário (B&K)** e o total chega a ~53 GB. Só rolamento. Fonte: API do Mendeley (`j99rf8rsrz`, `w9d6g35kcy`) e a lista awesome-bearing. A citação indicada é doi:10.1016/j.dib.2026.112919, não aberta.
- **FSTF Mechanical Lab (Mendeley `n9y9c7xrz3`).** Tem 1 rotor, 6 casos × 3 velocidades e 77 MB gravados por celular. É pequena demais e a velocidade é só setpoint. Fonte: API do Mendeley e a lista awesome-bearing.
- **HB-Bearing (IEEE DataPort, doi:10.21227/f6y0-ef20).** O ruído de fundo é real e a taxa é de 16 kHz, mas há 1 motor só, rotação fixa e download por assinatura. Pode servir de teste de robustez em **(8)**, se houver acesso. Fonte: a lista awesome-bearing apenas.
- **AHU Parabolic Mirror (IEEE DataPort, doi:10.21227/jm8w-zw54).** Tem 1 bancada, só rolamento e exige assinatura. Fonte: a lista awesome-bearing.
- **Gdańsk "AC motor voltage and audible noise dataset" (Pałczyńska, 2022, doi:10.34808/x55q-sz53_dyr_roz31, CC BY 4.0).** É a única base que achei que **correlaciona distúrbio elétrico (tensão do conversor) e ruído audível**, o que interessa a **(2)**. Porém os distúrbios vêm do PWM, não de falhas da rede, e a máquina não parece ter falhas. Não consegui ler a página (MOST Wiedzy é em JS); o que sei vem do DataCite e do resumo da busca. **Vale uma checagem manual rápida** antes de descartar de vez.
- **DREGON (Strauss et al., IROS 2018).** Tem microfones e **velocidade de cada rotor registrada**, mas é 1 drone, sem falhas e com ruído de hélice dominando. Serviria só para uma prova de conceito de **(4)**. Fonte: página do autor (loria.fr) e resumo da busca. A licença e a taxa não foram verificadas, porque dregon.inria.fr devolveu 403.
- **UPV UAV/MODERA (Zenodo 17457069, CC BY-SA 4.0, 9,7 GB).** É 1 UAV com throttle sem rpm e sem falhas. Fonte: API do Zenodo.
- **ADCP (arXiv:2503.00790).** É 1 motor, só throttle, e não encontrei o download. Fonte: texto completo do arXiv.
- **Paderborn KAt.** **Confirmado: não tem microfone.** São correntes do motor e vibração, mais velocidade, torque, carga e temperatura. Fonte: página oficial "Data Sets and Download".
- **Voiceprint bearing (Mendeley `gprfs9hybk`).** Só traz imagens tempo-frequência, sem o áudio bruto. Fonte: API do Mendeley.
- **DT-UOIMSD (Mendeley `7pzg74tpn3`).** É sintético, um gêmeo digital do UOEMD. Fonte: API do Mendeley.
- **Bomba de Oviedo (Zenodo 18500146).** Só traz espectros e scripts. Fonte: API do Zenodo. A base de bomba centrífuga do IEEE DataPort (doi:10.21227/8cqr-jr43) foi vista só no resultado de busca e exige login. Não verifiquei.
- **Compressor de ar IITK (Verma et al., IEEE Trans. Reliability 2016).** É 1 compressor alternativo, sem rpm e sem falha elétrica. Fonte: só o resumo da busca (página IITK e exemplo do MathWorks).
- **NASA SALT (hélices).** Tem rpm medido e microfones de campo distante, mas não confirmei que os dados brutos sejam públicos. Fonte: só o resumo da busca.

---

## 4. Recomendação (no máximo 3 fichas)

1. **MaFaulDa.** É a única base nova com **microfone e tacômetro gravados juntos** cobrindo dezenas de rotações. Serve para **(4)** em regime e para **(ordens)** mecânicas. O download é direto (12,9 GB). A licença não está declarada e precisa ser confirmada.
2. **ToyADMOS2, usado pelo recorte do DCASE 2022 dev** para começar leve (16 kHz, CC BY 4.0). Tem várias unidades físicas com tensão de fonte controlada e conhecida, muito dado normal e ruído real regravado. É a melhor candidata para **(5)**, **(6)** e **(8)**.
3. **IM-VACD.** Tem falhas elétricas reais, incluindo **desequilíbrio de tensão** e estator, nas mesmas 8 unidades do UOEMD, gravadas por celular. Serve para **(2)** e para um teste de troca de microfone em **(6)**. O problema conhecido é que classe e motor se confundem.

A reserva é o UORED-VAFCLS, que tem RPM medido por sensor Hall, mas um motor só.

**Microfone + verdade de rotação + várias unidades ao mesmo tempo:** nenhuma das bases novas tem as três condições com rotação *medida*. As que chegam mais perto:

- **ToyADMOS2/DCASE ToyCar:** 5 unidades e tensão ou nível de velocidade conhecidos, mas sem rpm medido.
- **IM-VACD:** 8 unidades e setpoint de frequência, mas uma falha por unidade.
- **MaFaulDa** e **UORED:** tacômetro ou Hall, mas uma unidade só.

Entre as bases já cobertas por outros agentes, vale conferir se o UOEMD e o KAIST registram a rotação de fato.

## 5. Incertezas

- A licença do MaFaulDa não foi encontrada.
- Não verifiquei a licença do SUBF v2.0 nem o número de casos do ToyADMOS.
- A taxa de áudio do IM-VACD é incerta: 42 kHz declarado, 48 kHz relatado num arquivo.
- Nos atributos `vel` e `volt` do DCASE não sei a unidade nem se são medidos ou só setpoint.
- O conteúdo do Gdańsk e do DREGON não foi verificado na página original.
- SUBF, GUET, FSTF, HB-Bearing e AHU foram descritos a partir de uma lista secundária (github.com/VictorBauler/awesome-bearing-dataset), e só confirmei FSTF e GUET pela API do Mendeley.
