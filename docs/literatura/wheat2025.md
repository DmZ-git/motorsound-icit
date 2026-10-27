# wheat2025 — Correção de mudança de domínio em vibração de motor elétrico para condições de operação não vistas (Harmonic Feature Space + regressão)

## 1. Referência

Lesley Wheat, Martin v. Mohrenschildt, Saeid R. Habibi, Dhafar Al-Ani.
"Correcting Domain Shifts in Electric Motor Vibration Data for Unseen Operating Conditions".
In *Proc. 2025 5th International Conference on Electrical, Computer and Energy Technologies
(ICECET)*, pp. 1–6, 2025. DOI: 10.1109/ICECET63943.2025.11472330.

- Texto completo consultado: preprint arXiv:2504.10661v1 [eess.SP], 14 abr. 2025,
  https://arxiv.org/pdf/2504.10661. Os dados da publicação ICECET foram acrescentados da
  referência bibliográfica fornecida pelo autor.
- HFS original citado como [6]: E. Hofer, M. v. Mohrenschildt, "Model-Free Data Mining of Families of Rotating
  Machinery", *Applied Sciences* 12(6):3178, 2022, DOI 10.3390/app12063178 (DOI conferido no Crossref; **texto não
  lido**, o site da MDPI retornou 403).

## 2. Leitura

**Texto completo** do preprint (6 páginas, extraído do PDF). Figuras não vistas (só legendas). A definição original
do HFS (Hofer & Mohrenschildt 2022) **não foi lida**; o que segue é o HFS conforme descrito e modificado neste artigo.

## 3. Método

Não é um rastreador: **a frequência de operação `f_o` é suposta conhecida** (setpoint da bancada; "speed and load are
known at test time"). O artigo é sobre normalizar features pela rotação e pela carga para classificar rolamento em
condições de operação não vistas no treino.

### 3.1 Harmonic Feature Space (Algoritmo 1, "Creating Harmonic Hilbert Space")

Entrada: sinal `x`, frequência de amostragem `f_s`, frequência de operação `f_o`, "harmonic bin" `d ∈ ℕ`.

1. `N = round(f_s · d / f_o)` (Eq. 1). Com isso, o bin k da FFT de tamanho N fica em `k·f_s/N ≈ k·f_o/d`, ou seja,
   **o bin k corresponde à ordem k/d** da rotação para qualquer `f_o`. Com d = 4 há 4 bins por harmônico (resolução
   de ¼ de ordem) e os harmônicos caem sempre nos bins 4, 8, 12, … (Fig. 1 usa f_s = 100 Hz, d = 4).
2. `w = Blackman(N)`; `c = N / Σ w²` (normalização de potência da janela).
3. Para cada segmento i = 1 … ⌊len(x)/N⌋ (segmentos **sem sobreposição**, cada um vira uma amostra/linha):
   - `z = w ⊙ segmento_i` (o pseudocódigo escreve `GET SEGMENT(z, i, N)`, erro de digitação para `x`);
   - `z = z − mean(z)` (remoção de DC, feita **depois** da janela, como escrito);
   - `h = envelope_Hilbert(z)` → variante **HARH**; a variante **HAR** pula este passo;
   - `s = c · PowerSpectrum(h)`;
   - `H[i,:] = s / (N/2)` (como escrito; se "/(N/2)" é escala ou "primeira metade" não está claro).
4. Pós-processamento (seção III, passo 2): com vários canais, magnitude Euclidiana entre canais em cada frequência;
   dado o `f_o` máximo, `N_min` é a menor janela e **mantêm-se `N_min/2` features** para todas as velocidades (o
   resto é descartado, para todas terem o mesmo número de features; isso faz rotações baixas "perderem energia", o
   que é discutido); remove DC e converte para **dB**.
5. No experimento: d = 4 e "passa-baixa em 60 harmônicos", o que dá 60·4 = 240 features por canal (a conta é nossa;
   o número não aparece explícito).

O artigo diz que, diferente do HFS original, **as amostras não podem ser normalizadas por energia**, porque a energia
relativa é indicador de falha. Portanto o HFS original normaliza por energia; aqui não.

### 3.2 Ajuste de condição de operação (Algoritmo 2)

Treinado **só com dados saudáveis**, em estado estacionário. Para cada linha, monômios de grau 2 em `a = f_o` e
`b = carga (Nm)`: `X = [1, a, b, a², ab, b²]`. Para cada feature m (cada bin do HFS, em dB), ajusta-se regressão
linear por mínimos quadrados `y = H[:, m] ~ X`, e **zera-se o coeficiente do intercepto**. O ajuste é
`Ĥ[:, m] = H[:, m] − X·A[m, :]ᵀ` (o pseudocódigo escreve `A[i,:]·X[:,i]`, inconsistente nas dimensões; a forma
correta é a predição da regressão sem o intercepto). Ou seja: remove-se de cada bin a tendência de energia em função
de velocidade e carga aprendida em motores sãos, preservando o nível médio.

Suposições declaradas: estado estacionário; condições de teste dentro do intervalo de velocidade/carga do treino
(só interpolação); nenhuma condição perto de ressonância; estado (são/falho) do treino conhecido.

### 3.3 Classificação

Padronização (média zero, variância um) → **PCA com 2 componentes** → **kNN** com k* escolhido por leave-one-out no
treino (erro reponderado por desbalanceio de classe; k de 1 até o número de amostras do rolamento com menos amostras).
Classificador simples de propósito, para não mascarar o efeito do pré-processamento.

Causalidade: o HFS por segmento é causal (janela de N amostras, sem sobreposição) desde que `f_o` seja conhecido; o
filtro passa-baixa Butterworth de **fase zero** aplicado antes (frequência de corte não encontrada) é não causal.
Custo: uma FFT de tamanho N variável por segmento, uma transformada de Hilbert (outra FFT direta + inversa) e uma
regressão linear (6 coeficientes por feature). Não reportado em números; o artigo só diz "computacionalmente
eficiente".

## 4. Protocolo de avaliação

- Dados: CMHT, gerador-partida por correia (BSG, modelo BM1048564F) na bancada HT-250 (D&V Electronics); dois
  acelerômetros triaxiais PCB 356A16 (A1 no chassi, A2 sob a bancada) a 48 kHz; rolamento SKF 6305-2RS1/C3.
  Rampa de 0 a 6000 RPM com patamares de ~1 s; os patamares estacionários foram extraídos em arquivos de ~1 s.
  15 condições de operação em 0–6000 RPM e 0–20 Nm. Rolamentos: sãos AM-01 (6 corridas), AM-02 (2), AM-03 (2);
  falhos F3-01, F5-01, F7-01 (4 corridas cada; combinações de defeito em pista externa, interna e esfera).
- Split: **cada rolamento é um domínio** — está só no treino ou só no teste; as condições de teste (5 Nm a 2000,
  3000 e 4000 RPM) **não aparecem no treino**, com no mínimo 1000 RPM e 5 Nm de distância. O teste é único em
  (rolamento, velocidade, carga).
- Comparação: FFT (janela 8196 [sic], remove DC, passa-baixa 6000 Hz), HFFT (análise de envelope), HAR e HARH.
- Métricas: (a) acurácia de classificação são/falho do kNN(k*) no teste, com cada rolamento pesando igual;
  (b) **"operating condition ID error"**: junta dados da mesma classe do treino e do teste, rotula "treino"/"teste",
  e mede o erro LOO do kNN; erro alto = distribuições indistinguíveis = pouca mudança de domínio.
- Riscos de vazamento / fragilidade:
  - o LOO para escolher k* e o LOO da métrica (b) são por **segmento**; segmentos do mesmo arquivo de 1 s são vizinhos
    quase certos, o que favorece k pequeno e infla a separabilidade "treino × teste" (afeta a métrica (b));
  - onde padronização e PCA são ajustados (só treino ou tudo) não é dito;
  - amostra minúscula: 6 rolamentos × 3 condições = 18 células de teste; um único erro (AM-01 a 3000 RPM, 0 %)
    move a média em vários pontos;
  - como os rolamentos falhos combinam tipos de defeito, "elementos" das falhas de teste aparecem no treino (os
    autores reconhecem).

## 5. Números reportados (CMHT BSG, Tabela II e III)

Acurácia de classificação (são/falho, teste):

| Canal | FFT | HFFT | HAR | HARH |
|---|---|---|---|---|
| A1 | 59 % | 34 % | 88 % | 89 % |
| A2 | 31 % | 30 % | 50 % | 63 % |
| A1 + A2 | 58 % | 34 % | 80 % | **94 %** |

Erro de identificação de condição (maior = menos mudança de domínio): A1+A2 = 30 % (FFT), 41 % (HFFT), 33 % (HAR),
68 % (HARH). Com HARH A1+A2, todas as 18 células são 100 % exceto AM-01 a 3000 RPM (0 %). O resumo cita "38 % de
melhoria" na mudança de domínio em relação à análise de frequência (68 % − 30 %).

## 6. Reimplementação

O Algoritmo 1 e o 2 são completos o bastante para implementar. Teria de ser suposto: frequência de corte do
passa-baixa de fase zero; se o envelope de Hilbert é `|sinal analítico|` ou seu quadrado; o significado de
`s/(N/2)`; o ponto exato onde o corte "60 harmônicos" é aplicado em relação ao `N_min/2`; onde se ajustam
padronização e PCA. A base CMHT não é pública ("Unpublished" tech. report). **Veredito: só inspiração** quanto aos
números (sem dados); **reproduzível** quanto ao algoritmo HFS + ajuste.

## 7. Relevância para o MotorSound

(a) **Rastreador offline.** O artigo depende de `f_o` conhecido. No MotorSound essa é exatamente a entrada que o
nosso rastreador produz; o HFS vira o **consumidor natural do rastreador** e um teste indireto dele: se o HFS
calculado com o `f_o` estimado classifica tão bem quanto com o `f_o` "verdadeiro" (quando houver), o rastreador é
bom o bastante para diagnóstico. Também dá uma medida de sensibilidade: erro de oitava no `f_o` desloca todos os
bins por um fator 2, então o HFS amplifica erro de oitava e serve de teste duro.

(b) **ESP32.** O HFS precisa de FFT de tamanho N variável (não potência de 2). Duas saídas embarcáveis (nossas, não do
artigo): FFT de tamanho fixo potência de 2 com zero-padding e **interpolação linear do espectro nas ordens k/d**
(k = 1 … K); ou janela de N variável com DFT apenas nos K bins necessários (Goertzel; K = 60·d = 240 → custo O(K·N)).
A remoção de tendência é uma multiplicação 240 × 6 por quadro. A transformada de Hilbert do HARH dobra o custo.

(c) **Features normalizadas pela rotação — especificação para `pipelines/p20_hfs.py`.**

- `NOME = "p20_hfs"`, `ORIGEM = "wheat2025"`, `TIPO = "inspirada"` (o `f_o` vem do nosso rastreador, não de setpoint,
  e a carga não pode ser regressor; ver abaixo).
- `extrair(y, sr, pre)`:
  1. obter `f_o` por janela: do rastreador de rotação (caminho Viterbi) ou, na versão mínima, um `f_o` por arquivo
     por soma harmônica; registrar a origem em `pre`;
  2. para cada segmento sem sobreposição: `N = round(sr·d/f_o)` com `d = pre.d` (padrão 4); Blackman; `c = N/Σw²`;
     remover média; variante `pre.hilbert ∈ {False (HAR), True (HARH)}` com `h = |scipy.signal.hilbert(z)|`;
     `s = c·|rfft(h)|²/(N/2)`;
  3. manter os bins `1 … d·H_max` (`H_max = pre.h_max`, padrão 60), descartando DC; se `H_max·f_o > sr/2` (só HAR; o
     HARH é envelope e não tem esse limite prático), preencher com NaN e registrar;
  4. converter para dB (`10·log10(s + eps)`);
  5. guardar também `f_o` do segmento como coluna, para os controles abaixo.
  Cada segmento é uma amostra; o rótulo e o **id da gravação** acompanham cada linha (regra 1 do CLAUDE.md).
- `treinar(features, rotulos, post)`:
  1. ajuste de condição só com a classe "saudável" (`good` no IDMT, motor são no A2212), regressor
     `X = [1, f_o, f_o²]`, intercepto zerado, subtração da predição em todos os segmentos. **A carga não entra**: no
     IDMT "heavyload" é uma das classes, e usar carga como regressor vaza o rótulo;
  2. padronização + PCA (2 componentes como no artigo; testar também sem PCA) ajustados só no treino;
  3. kNN com k* por validação **agrupada por gravação** (não LOO por segmento, que o artigo usa e que aqui seria
     vazamento).
- `prever(...)`: aplicar ajuste, padronização, PCA e kNN; probabilidades por janela = fração de vizinhos.
- Variantes e controles necessários para interpretar o resultado no IDMT, onde as classes são produzidas por **tensão
  e carga**, ou seja, provavelmente por **rotação**: (i) HFS sem ajuste; (ii) HFS com ajuste em `f_o`; (iii) HFS +
  `f_o` como feature explícita; (iv) só `f_o` como feature (classificador de uma dimensão). Se (iv) já separa as
  classes, o IDMT mede rotação, e o HFS (que remove rotação por construção) deve **piorar**; isso é um resultado a
  registrar, não um defeito. No A2212 a comparação relevante é a de generalização entre motores (agrupada por
  arquivo, com a ressalva do rolamento só no motor 4).
- `custo_mcu()`: RAM ≈ N floats do segmento + K = d·H_max floats de saída + 3·K coeficientes de ajuste com
  regressor `[1, f_o, f_o²]` (para d = 4, H_max = 60: 720; seriam 6·K = 1440 com a carga, como no artigo); custo por segmento ≈ uma FFT de N pontos (ou K Goertzel) mais o HARH
  opcional.
