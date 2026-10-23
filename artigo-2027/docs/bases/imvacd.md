# IM-VACD (Universidade de Ottawa, motores de indução gravados por celular)

Escrito em 23/10/2026 a partir dos arquivos baixados. Os números de §2 a §8 saíram de
`scripts/inventario_imvacd.py` (stdout guardado em `data/imvacd/_metadados/saida_inventario.txt`),
rodado sobre `data/imvacd/`. Onde a documentação diverge do medido, as duas versões estão
registradas. A ficha da UOEMD (`docs/bases/uoemd.md`) é a referência cruzada.

## 1. Origem, citação, licença, tamanho

- Base: *Induction Motor Vibration and Acoustic Cellular Device (IM-VACD)*, Mert Sehri
  (Universidade de Ottawa), Mendeley Data, **versão 2** (publicada em 08/09/2026; v1 em
  22/01/2026). DOI da v2: **10.17632/yc8yhg5xjd.2**. Link: https://data.mendeley.com/datasets/yc8yhg5xjd/2
  O nome oficial é "Cellular Device", não "recorded with smartphones".
- Licença: **CC BY 4.0** (campo `data_licence` da API). Sem login: a API pública
  (`/public-api/datasets/yc8yhg5xjd/files?folder_id=<id>&version=2`) deu links e sha256.
- Artigo associado (campo `related_links` da API): Sehri, M. *Physics-informed unit-normalized
  selective embedding with harmonic loss for fault identification using vibration and acoustic
  signals*. Mechanical Systems and Signal Processing, v. 259, 114922, set. 2026.
  DOI 10.1016/j.ymssp.2026.114922 (PII S0888327026010794); acesso aberto CC BY 4.0 segundo
  Crossref/Unpaywall. **Não há artigo de Data in Brief**: não encontrado no Crossref nem no
  Mendeley. O Crossref lista só Mert Sehri como autor.
- Conteúdo publicado: 256 CSV, **3,681 GB**, sem zip. Cada CSV tem áudio e acelerômetro do
  celular, então não dá para baixar só o acústico: baixei tudo.
- Baixado: os 256 CSV em `data/imvacd/` (3,5 GiB). **256/256 conferem com o sha256 da API**; a
  lista está em `data/imvacd/_metadados/mendeley_files_v2.json`. Os 256 sha256 da v1 são
  idênticos aos da v2 (comparado pela API, `mendeley_files_v1.json`), então as duas versões
  têm os mesmos dados. Nenhum par de arquivos tem vetor de áudio idêntico.

## 2. Árvore e nomes

```
data/imvacd/
├── 1_Hand_Held_Mounting_10_seconds/{1_Unloaded,2_Loaded}/{1_iPhone_13,2_Samsung_Galaxy_S6,3_Samsung_Galaxy_A50}/   16 CSV cada
├── 2_Rigid_Mounting_10_seconds/{1_Unloaded,2_Loaded}/1_iPhone_13 (ou 1_Iphone_13)/   16 CSV
│                                                  /{2_Samsung_Galaxy_S6,3_Samsung_Galaxy_A50}/   32 CSV cada
└── _metadados/   (JSON da API, metadados do artigo, saída do inventário)
```

Exemplos reais: `1_Hand_Held_Mounting_10_seconds/1_Unloaded/1_iPhone_13/A-H-H-1-0.csv`,
`2_Rigid_Mounting_10_seconds/2_Loaded/3_Samsung_Galaxy_A50/C-K-A-3-1.csv`. Padrão
`<celular>-<A>-<B>-<v>-<carga>.csv`: celular A = iPhone 13, B = Galaxy S6, C = Galaxy A50;
`<A>-<B>` é a classe, com os mesmos códigos da UOEMD (H-H, R-U, R-M, S-W, V-U, B-R, K-A, F-B);
`<v>` 1..4 = 15/20/25/30 Hz constantes; carga 0/1. A pasta de carga e a pasta do celular
sempre concordam com o nome (verificado por `assert`). O mesmo nome aparece nas duas
montagens (168 nomes únicos para 256 arquivos).

Divergências com a descrição do Mendeley:
- A descrição não menciona as duas montagens (**na mão** e **rígida**), que estão nas pastas.
- "constant and variable speed": só há velocidade constante.
- "the first column is acoustic data": vale nos Galaxy; no iPhone a ordem é `x,y,z,acoustic`.
  O loader lê pelo nome da coluna.
- **Erro de rótulo**: na montagem na mão com carga, os 24 arquivos com código 2 (20 Hz) são de
  30 Hz. O 1x no acelerômetro do celular fica em 29,5–29,8 Hz e o padrão de picos do áudio é o
  dos arquivos de 30 Hz (59,x e 60,x Hz; 3x em ~89 Hz), enquanto os de 20 Hz da montagem rígida
  têm 1x em ~19,5 Hz e 2x em ~39 Hz (o script confere por `assert`). No índice esses arquivos
  estão como 30 Hz, com "ROTULO CORRIGIDO" no `obs`.

Cada CSV traz `x,y,z` (acelerômetro do celular, m/s², só nas primeiras ~1 000–5 000 linhas),
`acoustic` (todas as linhas) e, só na primeira linha de dados, taxas pedida e "real", duração,
unidades, modelo do aparelho e **carimbo de tempo** (ISO, microssegundos).

## 3. Canal acústico: taxa, canais, resolução (medidos)

| celular | `device` | taxa pedida (`_set`) | taxa "real" do app | amostras/arquivo |
|---|---|---|---|---|
| iPhone 13 | iPhone14,5 | **48 000 Hz** | 46 510–47 671 (med. 47 569) | 466 217–476 903 |
| Galaxy S6 | SM-G925W8 | **44 100 Hz** | 42 934–44 686 (med. 44 335) | 430 096–450 971 |
| Galaxy A50 | SM-A505W | **44 100 Hz** | 44 192–45 003 (med. 44 977) | 443 533–450 340 |

- **Não é 42 kHz** (o que a descrição diz para todos os celulares) nem 420 000 amostras por
  arquivo. O relato de 48 kHz vale para o iPhone.
- A taxa "real" do app é amostras / duração de relógio e varia por arquivo. Teste: o pico 2x de
  rotação do mesmo evento, medido em celulares diferentes com a taxa **pedida**, bate.
  A razão mediana entre celulares fica entre 0,9989 e 1,0000 (montagem rígida: 0,9999–1,0000).
  Com a taxa "real" ela seria 0,971–0,986. Logo a taxa verdadeira é a pedida, e o loader usa
  `acoustic_sampling_frequency_set`.
- 1 canal (mono). Unidade `pcm16_normalized`: todos os valores são inteiros de 16 bit / 32 768
  (resíduo 0). Níveis distintos usados por arquivo: iPhone 8 563–44 587, S6 9 159–53 422,
  A50 21 654–54 594.
- **Saturação nos Galaxy**: 216 amostras em ±1 em 35 arquivos do S6 e 166 em 20 arquivos do A50
  (poucas por arquivo); iPhone nenhuma (pico máx. 0,94). Não há calibração em Pa nem informação
  de AGC/filtro do app.
- Acelerômetro do celular (não é o foco): 100 Hz pedido (iPhone, real 96,7–99,1); S6 208/210/220
  pedido (real 207–232); A50 490 pedido (real 498–500). A descrição diz 200 Hz para o S6.

## 4. Duração

Por arquivo, 9,713 a 10,226 s (amostras / taxa pedida); a descrição diz 10 s. Cada classe tem
32 arquivos, ~322 s (321,4–322,9 s). Total **2 577,7 s (43,0 min)**: 8 classes × 32.

## 5. Matriz classe × unidade física (motor)

| classe (nome do `uoemd.csv`) | código | unidade | arquivos |
|---|---|---|---|
| saudavel | H-H | motor_H_H | 32 |
| desequilibrio_rotor | R-U | motor_R_U | 32 |
| desalinhamento_rotor | R-M | motor_R_M | 32 |
| falha_enrolamento_estator | S-W | motor_S_W | 32 |
| desequilibrio_tensao_e_falta_fase | V-U | motor_V_U | 32 |
| rotor_empenado | B-R | motor_B_R | 32 |
| barras_rotor_quebradas | K-A | motor_K_A | 32 |
| rolamento_defeituoso | F-B | motor_F_B | 32 |

A tabela cruzada 8 × 8 do script é diagonal. **Classe = motor**, como na UOEMD. A descrição diz
"8 different D396 Marathon Electric 3-phase motors. Each motor has different types of faults
artificially created by SpectraQuest" (o mesmo texto e os mesmos códigos da UOEMD). Os
arquivos não trazem número de série. Que sejam **os mesmos 8 motores físicos da UOEMD** é
**provável mas não verificado**: mesmo laboratório, mesmo autor, mesmo modelo e códigos, mas
nem o artigo (só o resumo foi lido) nem a descrição afirmam isso. As unidades usam os mesmos
nomes do `uoemd.csv` para permitir o cruzamento, com essa ressalva.

Sobre o nome de V-U: a descrição só diz "V voltage / U unbalance" (desequilíbrio de tensão).
Na UOEMD o artigo junta desequilíbrio de tensão e falta de fase na mesma classe. Mantive o nome
da UOEMD para cruzar, mas não há como saber se o IM-VACD tem falta de fase.

## 6. Matriz classe × condição

O desenho é balanceado: cada classe tem exatamente 1 arquivo em cada uma das 32 células
(montagem, celular, velocidade, carga) que existem:

| montagem | celulares | velocidades | cargas | datas (carimbos) |
|---|---|---|---|---|
| mão | iPhone, S6, A50 | 15, 30* | sem carga (14/06/2024 11:47–13:03) e com carga (12/06/2024 17:43–19:03) | 1 sessão por carga |
| rígida | iPhone, S6, A50 | 15, 30 | com (19/07/2024 13:49–15:00) e sem (22/07/2024 10:25–12:24) | |
| rígida | **só S6 e A50** | 20, 25 | com (29/10/2024 18:51–19:47) e sem (29/10 19:53 – 30/10 09:00) | |

\* código 2 corrigido para 30 Hz (§2).

- O iPhone não tem 20 nem 25 Hz. Os pares 20/25 Hz foram gravados **três meses depois** e só
  nos Galaxy, então velocidade 20/25 se confunde com sessão e com celular.
- Em cada sessão os motores foram trocados um a um: cada classe ocupa um bloco contíguo de alguns minutos
  (ex.: mão sem carga, R-U 11:47, H-H 11:58, R-M 12:24, …). A ordem das classes muda entre
  sessões. Não há repetição: uma gravação por classe, montagem, celular, velocidade e carga.
- **Outra sessão em relação à UOEMD**: os `.mat` da UOEMD foram criados em 22/11/2023, e o
  IM-VACD é de junho a outubro de 2024. As velocidades também diferem (UOEMD 15/30/45/60 Hz e
  rampas; IM-VACD 15/20/25/30). Só 15 e 30 Hz existem nas duas. **Não foi gravado ao mesmo
  tempo que a UOEMD**: não existe par celular × microfone PCB do mesmo evento.

### Nível acústico (RMS sem DC × 10³, pcm normalizado; mediana por classe)

| classe | mão A50 | mão S6 | mão iPhone | rígida A50 | rígida S6 | rígida iPhone |
|---|---|---|---|---|---|---|
| barras_rotor_quebradas | 284.3 | 316.1 | 205.6 | 189.7 | 240.7 | 141.9 |
| desalinhamento_rotor | 298.9 | 337.7 | 211.6 | 200.0 | 260.9 | 149.7 |
| desequilibrio_rotor | 286.6 | 299.5 | 222.4 | 193.0 | 238.7 | 133.1 |
| desequilibrio_tensao_e_falta_fase | 292.3 | 316.4 | 190.9 | 190.4 | 225.9 | 141.8 |
| falha_enrolamento_estator | 297.6 | 328.4 | 204.5 | 190.6 | 245.3 | 144.6 |
| rolamento_defeituoso | 307.2 | 323.9 | 217.0 | 207.8 | 249.8 | 129.0 |
| rotor_empenado | 306.4 | 306.3 | 225.3 | 196.6 | 252.3 | 136.2 |
| saudavel | **179.0** | **162.4** | **126.2** | 181.7 | 140.2 | 144.9 |

O motor saudável é o mais silencioso em 23 das 32 condições. A razão entre a falha mais
silenciosa e o saudável vai de 0,74 a 2,35 (mediana 1,24). O atalho de nível é bem menor que
na UOEMD (lá 1,6–8,4× e 16/16 condições). Ele é forte na montagem na mão (~1,6–2×) e quase
some na rígida com iPhone. O nível depende muito do celular e da montagem (o S6 grava até
~1,7× mais alto que o iPhone nas falhas). Pico |y| máximo: 1,0 (saturado) em todas as classes nos Galaxy;
0,80–0,94 no iPhone.

## 7. Verdade de rotação

- **Não há tacômetro nem leitura do inversor nos arquivos.** A rotação só é conhecida pelo
  setpoint no nome (15/20/25/30 Hz). Não se sabe se esse setpoint é o valor exato da saída do
  inversor; na UOEMD, o "15 Hz" do inversor era 15,10–15,22 Hz. No índice, `f_alim_hz` é o
  setpoint nominal (corrigido para 30 nos 24 arquivos de §2) e `rot_esperada_hz` fica vazio.
- Motor Marathon D396, 2 polos (3 600 rpm a 60 Hz, ficha da UOEMD). Logo síncrona = f_alim e
  rotação = f_alim·(1−s).
- Estimativa pelo pico 2x no áudio (faixa 1,9–2,1·f, resolução 0,1 Hz), como razão 2x/(2f),
  mediana por celular: 15 Hz 0,976–0,978; 20 Hz 0,981; 25 Hz 0,987; 30 Hz 0,988–0,990. Isso
  dá escorregamento de ~2,3 % a 15 Hz e ~1,1 % a 30 Hz. O 1x no acelerômetro do A50 concorda
  (0,978/0,979/0,987/0,989). Os extremos (0,96–1,04) vêm de picos mal escolhidos, sobretudo no
  motor saudável, que tem pouca vibração mecânica (como na UOEMD).
- Entre os celulares do mesmo evento, o 2x bate em 0,01 % (§3). Para o mesmo motor e a mesma
  velocidade em sessões diferentes, ele varia ~1,2 % (mediana max/min−1). A rotação muda de
  sessão para sessão, então o setpoint não é verdade de rotação melhor que ~1–2 %.
- Conclusão: verdade de rotação **indireta** (setpoint + escorregamento estimado), sem rampas.

## 8. Gravação de origem e simultaneidade

Os carimbos de tempo (primeira linha de cada CSV) resolvem a questão:
- **Montagem na mão: celulares em sequência, não simultâneos.** Em todos os 32 eventos a ordem
  é iPhone → S6 → A50, com 13,1–36,2 s entre celulares consecutivos (mediana 17,4 s). Não há
  sobreposição em nenhum evento. Mas é o mesmo motor ligado na mesma condição, sem parar
  (o 2x bate entre os celulares a 0,1 % em 44–88 % dos pares).
- **Montagem rígida: celulares simultâneos.** 64/64 eventos têm arquivos sobrepostos, com
  0,1–7,5 s entre carimbos (mediana 1,0 s). O pico 2x coincide entre os celulares a 0,01 %
  (razão mediana 0,9999–1,0000). Numa conferência exploratória (fora do script), a
  correlação dos envelopes RMS entre celulares deu fraca, e o atraso de melhor correlação não
  bateu com a diferença de carimbos. Os carimbos não servem para alinhar amostra a amostra.
- `gravacao_id = imvacd/<montagem>_<classe>_<f>_<carga>`, ex.
  `imvacd/rigida_saudavel_15_0`: **96 gravações de origem** (64 com 3 celulares, 32 com 2).
  Os arquivos dos celulares de um mesmo evento contam como a mesma gravação, também na mão
  (mesma corrida do motor, segundos de diferença). A mesma classe na mão e na rígida são
  sessões diferentes (dias diferentes).
- Para split, ainda vale que classe = motor (§5). Agrupar por `gravacao_id` evita só o
  vazamento entre celulares do mesmo evento.

## 9. Trabalhos que usam a base

- Sehri, M., MSSP 259, 114922 (2026), o artigo ligado à base. **Só o resumo foi lido**
  (OpenAlex). O texto completo deu 403 no ScienceDirect, e a API da Elsevier devolveu só os
  metadados. O resumo diz que usa "induction motor datasets collected under multiple operating
  conditions with both structurally mounted sensors and cell phones" (provavelmente a UOEMD +
  IM-VACD, não confirmado) e propõe normalização para velocidade equivalente (UNSE-HL). Também
  diz que harmônicos do rotor são observáveis nos celulares e que a acurácia melhora "on the
  datasets and devices evaluated". **Split, números e protocolo: não encontrados.** Como classe
  = motor, um split por segmento ou por arquivo mede reconhecimento de motor e sessão.
- Citações do artigo (OpenAlex, 23/10/2026): 1 trabalho (*Selective embedding for data-efficient
  deep learning in medical image classification*, doi:10.1007/s41060-026-01314-3, imagens
  médicas). Pelo título não usa a base; não foi lido. Outros trabalhos que usem o IM-VACD:
  não encontrados.

## 10. Consequências para o protocolo

1. Classe = motor (8 × 8 diagonal). A mesma limitação da UOEMD vale para qualquer
   classificador de falha.
2. O que a base acrescenta de fato é a **troca de microfone sobre o mesmo evento**: na montagem
   rígida, 3 celulares (15/30 Hz) ou 2 (20/25 Hz) gravando ao mesmo tempo. Ex.: treinar no
   iPhone e testar no A50, com o mesmo motor e o mesmo evento. Também dá para trocar a montagem
   (mão × rígida, sessões de dias diferentes).
3. Agrupar sempre por `gravacao_id` (evento). Em tabelas, declarar que 20/25 Hz só existem nos
   Galaxy e em outra sessão.
4. O atalho de nível é menor que na UOEMD, mas depende do celular. Reportar com e sem
   normalização RMS. Há saturação esporádica nos Galaxy.
5. Usar a taxa pedida (48 000 / 44 100), não a "real" do app nem os 42 kHz da descrição.
   Corrigir o código 2 da montagem na mão com carga para 30 Hz.
6. Cruzar com a UOEMD (microfone PCB) só nas velocidades 15 e 30 Hz, sabendo que são outras
   sessões (2023 × 2024) e que a identidade dos motores não foi verificada.
