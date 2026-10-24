# UOEMD-VAFCVS (Universidade de Ottawa, motores de indução)

Escrito em 22/10/2026 a partir dos arquivos baixados. Todos os números de §3 a §7 saíram de
`scripts/inventario_uoemd.py` (saída em stdout, rodado sobre `data/uoemd/`); as frequências
do inversor saíram do texto do artigo, lido pelo mesmo script a partir do XML do Europe PMC
guardado em `data/uoemd/_metadados/`. Onde a documentação diverge do medido, as duas versões
estão registradas.

## 1. Origem, citação, licença, tamanho

- Base: *University of Ottawa Electric Motor Dataset – Vibration and Acoustic Faults under
  Constant and Variable Speed Conditions (UOEMD-VAFCVS)*, Mert Sehri e Patrick Dumond,
  Mendeley Data, versão 2 (publicada em 09/02/2026; v1 em 15/12/2023).
  DOI da versão 2: 10.17632/msxs4vj48g.2 (o artigo cita a v1, 10.17632/msxs4vj48g.1).
  Link: https://data.mendeley.com/datasets/msxs4vj48g/2
- Artigo: Sehri, M.; Dumond, P. *University of Ottawa constant and variable speed electric
  motor vibration and acoustic fault signature dataset*. Data in Brief, v. 53, 110144, 2024.
  DOI 10.1016/j.dib.2024.110144 (PMC11636780). Lido inteiro (texto completo do Europe PMC).
- Licença: **CC BY 4.0** (campo `data_licence` da API e do artigo). Sem login: a API pública
  (`/public-api/datasets/msxs4vj48g/files?folder_id=<id>&version=2`) deu os links e o sha256 de
  cada arquivo.
- Conteúdo publicado (API): `1_Time/` (1 CSV, 5,0 MB), `2_CSV_Data_Files/{1_Unloaded_Condition,
  2_Loaded_Condition}/` (2 × 64 CSV, 2,64 GB) e `3_MatLab_Data_Files/{...}/` (2 × 64 `.mat`,
  0,60 GB); total 3,25 GB. **Não há zip** e não há arquivo só de áudio: o microfone é a coluna 2
  de cada arquivo de 5 colunas, então não dá para baixar só o canal acústico.
- Baixado: os 128 `.mat` (602,8 MB) e o vetor de tempo, em `data/uoemd/`; mais 2 CSV
  (`H_H_1_0.csv`, `K_A_6_1.csv`) em `data/uoemd/_conferencia_csv/`, só para conferência.
  Os CSV têm os mesmos valores do `.mat` (diferença máxima 0 nas 5 colunas, nos dois pares).
  Os 129 arquivos baixados conferem com o sha256 da API (129/129); a lista de hashes está em
  `data/uoemd/_metadados/mendeley_files_*_v2.json`. Os 128 `.mat` da v1 têm sha256 idêntico
  aos da v2 (comparado pela API), então as duas versões têm os mesmos dados.

## 2. Árvore e nomes

```
data/uoemd/
├── 1_Time/Time_10_seconds_420000_samples.csv
├── 3_MatLab_Data_Files/
│   ├── 1_Unloaded_Condition/<A>_<B>_<v>_0.mat     64 arquivos
│   └── 2_Loaded_Condition/<A>_<B>_<v>_1.mat       64 arquivos
├── _conferencia_csv/2_CSV_Data_Files/...           2 CSV
└── _metadados/   (JSON da API, XML do artigo)
```

Exemplos reais: `H_H_1_0.mat`, `K_A_6_1.mat`, `V_U_8_1.mat`. O artigo e a descrição escrevem
`H-H-1-0` com hífen; os arquivos usam sublinhado. `<A>_<B>` é a classe (H_H saudável, R_U
desequilíbrio do rotor, R_M desalinhamento, S_W enrolamento do estator, V_U desequilíbrio de
tensão **e** falta de fase, B_R rotor empenado, K_A barras quebradas, F_B rolamento);
`<v>` ∈ 1..8 é a velocidade (1–4 = 15/30/45/60 Hz constantes; 5 = 15→45, 6 = 30→60,
7 = 45→15, 8 = 60→30 Hz); o último dígito é a carga (0 sem, 1 com; a carga são dez parafusos
num disco no eixo). A pasta de carga sempre concorda com o último dígito (verificado por
`assert`).

Cada `.mat` (MATLAB 5.0, criado em 22/11/2023) tem uma variável `data` de 420 000 × 5
float64, sem rótulos de coluna. Os CSV têm cabeçalho: `Accelerometer 1 (m/s^2), Microphone
(V), Accelerometer 2 (m/s^2), Accelerometer 3 (m/s^2), Temperature (Celsius)`.

## 3. Canal acústico: taxa, canais, resolução (medidos)

- Microfone PCB 130F20 (45 mV/Pa nominal), a ~2 cm do mancal esquerdo do eixo, em suporte
  isolado; condicionador PCB 482C; aquisição NI USB-6212 (16 bit) via LabVIEW. Valor em volts.
- 128 arquivos, todos com 420 000 amostras × 5 colunas; 1 canal acústico (mono).
- Taxa: **42 kHz declarados** (artigo e descrição). Os arquivos não guardam taxa. O vetor
  `1_Time` tem 420 000 valores numéricos (mais 6 linhas vazias), passo constante de 23,8 µs
  e último valor 9,995976 s; o ajuste linear dá 42 016,8 Hz, não 42 000. É exatamente
  80 MHz / 1904, o que sugere (hipótese, não confirmada) que o USB-6212 arredondou a taxa
  pedida; a diferença é de 0,04 %. O loader usa 42 000.
- Resolução: passo de quantização de 0,329 mV em todos os arquivos (compatível com 16 bit numa
  faixa de ~±10,8 V). Como o sinal ocupa só ±2 V, o número de níveis usados é pequeno:
  mínimo 381 níveis (`H_H_1_0`, ~8,6 bit efetivos), mediana 5 216, máximo 11 565 (`V_U_5_1`).
- Faixa absoluta em toda a base: −2,0065 a 1,9715 V; offset DC por arquivo de −0,068 a
  +0,064 V. **Sem saturação**: o máximo de amostras no valor extremo de um arquivo é 4, e o
  pico fica a ~20 % do fundo de escala do ADC.

## 4. Duração

Todos os arquivos têm 10,000 s (420 000 / 42 000). Cada classe tem 16 arquivos = 160 s; total
1 280 s (21,3 min) de áudio. A documentação bate (128 conjuntos, 10 s, 420 000 amostras).

## 5. Matriz classe × unidade física

| classe | unidade (motor) | arquivos |
|---|---|---|
| saudavel | motor_H_H | 16 |
| desequilibrio_rotor | motor_R_U | 16 |
| desalinhamento_rotor | motor_R_M | 16 |
| falha_enrolamento_estator | motor_S_W | 16 |
| desequilibrio_tensao_e_falta_fase | motor_V_U | 16 |
| rotor_empenado | motor_B_R | 16 |
| barras_rotor_quebradas | motor_K_A | 16 |
| rolamento_defeituoso | motor_F_B | 16 |

(A tabela cruzada completa 8 × 8 do script é diagonal: todas as células fora da diagonal são 0.)

**A classe está totalmente confundida com a unidade física.** O artigo diz: "A total of 8
electric motors (same model) are used to collect the data (1 healthy and 7 faulty motors)",
"Every electric motor tested has its individual artificially created fault", e nas
limitações: "only one motor having each type of fault was available". Logo existe um motor
saudável de referência separado, cada falha está num motor diferente (Marathon D396, mesmo
modelo) e nenhum motor aparece saudável e com falha. Os arquivos não trazem número de série
nem identificador de motor; a unidade do índice (`motor_<classe>`) é inferida desse texto.
Não há como separar "estado" de "unidade": qualquer classificador pode estar reconhecendo o
motor (montagem, folgas, ressonâncias) e não a falha. Deixa-um-motor-de-fora é impossível,
pois tira a classe inteira.

Agravante: `V_U` junta desequilíbrio de tensão e falta de fase numa só classe e os arquivos
não dizem qual condição é qual. Desalinhamento, desequilíbrio e rotor empenado são falhas
mecânicas implantadas pela SpectraQuest no próprio motor (segundo o artigo).

## 6. Matriz classe × condição

Cada classe tem exatamente 1 arquivo em cada uma das 16 condições (4 velocidades constantes
+ 4 rampas) × (sem carga, com carga). O desenho é balanceado e completo; a matriz do script tem
1 em todas as 128 células. Não há repetição: **uma única gravação de 10 s por classe e por
condição**. Não há informação de dia nem de ordem de gravação.

Indícios de sessão, medidos:

- Temperatura média do sensor (coluna 5), °C, sem carga / com carga: saudável 28,16 / 27,19;
  R_U 29,81 / 27,97; R_M 32,25 / 28,62; S_W **40,01** / 27,32; V_U 29,01 / 27,60;
  B_R 31,65 / 27,34; K_A 32,26 / 28,12; F_B 29,12 / 28,45. O motor com falha de estator sem
  carga estava ~10 °C mais quente; os
  arquivos sem carga são sempre mais quentes que os com carga. As cargas parecem ter sido
  gravadas em sessões diferentes, e a temperatura do canal 5 identifica o motor.
- Nível acústico (abaixo): o motor saudável é muito mais silencioso que todos os outros.

### Nível acústico (RMS sem DC, mV; média das duas cargas)

| classe | 15 | 30 | 45 | 60 | 45→15 | 60→30 | 15→45 | 30→60 |
|---|---|---|---|---|---|---|---|---|
| barras_rotor_quebradas | 157.0 | 183.8 | 245.6 | 320.3 | 524.9 | 384.2 | 527.1 | 371.2 |
| desalinhamento_rotor | 156.7 | 370.6 | 232.0 | 262.7 | 559.0 | 434.9 | 561.6 | 456.6 |
| desequilibrio_rotor | 158.1 | 291.0 | 222.9 | 265.9 | 472.6 | 421.1 | 490.9 | 415.7 |
| desequilibrio_tensao_e_falta_fase | 170.8 | 457.5 | 234.8 | 310.6 | 579.5 | 494.4 | 570.2 | 448.2 |
| falha_enrolamento_estator | 140.2 | 383.5 | 221.0 | 266.7 | 483.8 | 471.7 | 486.1 | 386.8 |
| rolamento_defeituoso | 160.4 | 294.3 | 179.1 | 234.1 | 506.8 | 374.1 | 497.4 | 391.4 |
| rotor_empenado | 150.5 | 363.3 | 247.2 | 340.3 | 446.7 | 413.5 | 426.9 | 414.2 |
| saudavel | **24.5** | **46.7** | **84.8** | **100.4** | **78.3** | **152.3** | **93.4** | **170.2** |

Pico |AC| máximo: saudável 124–924 mV; as falhas 348–2 025 mV. A razão entre o RMS da classe
mais alta e da mais baixa, por velocidade e carga, vai de 2,8 a 17,6 (maior a 15 e 30 Hz sem
carga). **O motor saudável é o mais silencioso nas 16 de 16 condições**: o RMS da falha mais
silenciosa dividido pelo do saudável vai de 1,63 (45 Hz com carga) a 8,43 (15 Hz sem carga),
e é maior sem carga em todas as velocidades. "Saudável × falha" sai em boa parte só do nível. Entre as sete falhas o nível é parecido
(na tabela acima, até ~2,5× entre a mais alta e a mais baixa, a 30 Hz). Como o próprio motor saudável não aparece com falha, não se sabe se é o
estado ou a unidade (ou a montagem do microfone naquela sessão). Também por isso o saudável a
15 Hz usa só 381 níveis do ADC.

## 7. Verdade de rotação

- **Não há tacômetro nem encoder nos arquivos.** As cinco colunas são três acelerômetros, o
  microfone e a temperatura. `carregar_tacometro()` devolve `None`.
- O nome do arquivo dá só a velocidade nominal (15/30/45/60 Hz ou rampa). O artigo lista, por
  arquivo, a frequência lida do inversor ("operating frequency of 15.10 Hz", "increased from
  14.99 Hz to 44.86 Hz" etc.), 128 valores; o script extrai esses valores do XML e os põe em
  `f_alim_hz` (velocidade constante) ou no `obs` (rampas: início→fim). O artigo diz que o
  inversor "is used to collect the rotational frequency of the motor"; não diz se é a
  frequência de saída ou uma estimativa de rotação, nem o instante da leitura.
- Motor (Tabela 3 do artigo): Marathon Electric D396, 3 HP, 60 Hz, 3 600 rpm máx., trifásico,
  indução, rolamentos 6205. 3 600 rpm a 60 Hz ⇒ 2 polos (1 par): rotação síncrona = f_alim e
  rotação mecânica = f_alim·(1−s). O escorregamento nominal não está no artigo.
- Conferência independente (estimativa, não verdade): pico de 1x no acelerômetro 2 na faixa
  [0,85; 1,02]·f_alim, FFT de 10 s (resolução 0,1 Hz). Razão 1x/f_alim: mediana 0,9894
  (s ≈ 1 %), mínimo 0,9770, máximo 1,0173. Isso é coerente com f_alim ser a frequência de
  saída do inversor e a rotação ficar ~1 % abaixo. Os arquivos do motor saudável dão razão
  ≈ 1,00 (0,995–1,017), o que indica que ali o pico escolhido não é o 1x mecânico (vibração
  mecânica fraca) — mais uma diferença do motor saudável.
- Rampas: o 1x no primeiro e no último segundo acompanha os extremos do artigo (ex.
  `H_H_5_0`: 15,25 → 44,9 Hz para 14,99 → 44,86), mas nas rampas 30→60 o fim medido fica em
  56–58 Hz, abaixo dos ~59,9 Hz do artigo; e em `S_W_8_0` o início medido é 51,0 Hz contra
  59,91. A rampa não está sincronizada com a janela de 10 s; os extremos do artigo não servem
  como verdade instantânea.
- Conclusão: verdade de rotação **indireta**. `rot_esperada_hz` fica vazio no índice.

## 8. Gravação de origem

Cada arquivo é uma aquisição separada de 10 s; não há arquivos recortados de uma gravação
maior, e nenhum par de arquivos tem canal acústico idêntico (hash). `gravacao_id =
uoemd/<nome sem extensão>`, por exemplo `uoemd/H_H_1_0`. Como classe = motor, o agrupamento
por gravação não resolve a confusão de §5; agrupar por motor equivale a agrupar por classe.

## 9. Trabalhos que usam a base

- Artigo da própria base (Sehri & Dumond, 2024, lido inteiro): não propõe split nem reporta
  acurácia. Declara as limitações "only one motor having each type of fault", "no filter was
  applied", laboratório, e interferência do PWM no acelerômetro 1.
- Kilickaya, S.; Eren, L. *Padé Approximant Neural Networks for Enhanced Electric Motor Fault
  Diagnosis Using Vibration and Acoustic Data*. J. Vib. Eng. Technol., 2025,
  DOI 10.1007/s42417-025-02129-5 (arXiv 2507.02599v2). Lidos do texto completo do arXiv os
  trechos de dados, split e resultados. Usa só velocidade constante, as duas cargas, janelas de
  1 000 amostras sem sobreposição, min-max por janela; split temporal dentro de cada arquivo
  (80 % iniciais treino, 10 % validação, 10 % teste, sem embaralhar). Acurácia com o
  microfone 98,33 % (acelerômetros 1/2/3: 99,96/98,26/97,61 %). Risco de vazamento: treino e
  teste saem do mesmo arquivo de 10 s e do mesmo motor; com classe = motor, o número mede
  reconhecimento de motor/gravação. A normalização min-max por janela reduz, mas não elimina,
  o atalho de nível (forma de onda e SNR mudam).
- Zhuang, Y. et al. *A multi-level constrained vibration–acoustic multimodal contrastive
  learning for cross-machine motor...*, Meas. Sci. Technol., 2026, DOI 10.1088/1361-6501/ae8bca:
  só o resumo foi lido (texto pago); cita a base, mas não foi possível confirmar como a usa.
  Reporta >99,57 % com 10 amostras rotuladas por classe em tarefas entre duas máquinas.
- O OpenAlex lista 55 trabalhos que citam o artigo (consulta de 22/10/2026), entre eles
  Liao et al. 2025 (Electronics 14, 2005, fusão por GNN; o PDF devolveu 403) e Ishaq et al.
  2025 (IEEE Access, DOI 10.1109/access.2025.3574017). Não foram lidos; split e números:
  não encontrado.

## 10. Consequências para o protocolo

1. Classe = motor em todas as 8 classes, com um arquivo por classe e condição. Nenhum
   protocolo com estes dados separa falha de unidade; toda tabela da UOEMD deve declarar isso.
2. O motor saudável é o mais silencioso em todas as condições (1,6–8,4×): "saudável × falha"
   tem atalho de nível. Registrar desempenho com e sem normalização RMS.
3. A única generalização honesta disponível é entre condições do mesmo motor (deixar uma
   velocidade ou uma carga de fora), sempre com classe = motor.
4. A rotação só é conhecida pela frequência do inversor citada no artigo (≈ síncrona; 1x
   mecânico ~1 % abaixo pela vibração). Para rastreamento de rotação, as rampas não têm
   verdade instantânea.
5. Canal de temperatura e nível também identificam o motor/sessão; não usar como atributo.
