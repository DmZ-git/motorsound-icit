# Ideias de artigo a partir da fase exploratória

Escrito em 14/10/2026. Cada tema traz a evidência que os experimentos já dão (com a tabela de
`RESULTADOS_tabelas.md`), o que falta medir, a novidade em relação a
`docs/literatura/ESTADO_DA_ARTE.md` e um veredito. "Forte" quer dizer que os dados atuais já
sustentam a tese central e o que falta cabe até 26/10; "possível" quer dizer que há sinal, mas
falta uma medição que pode virar o resultado; "fraco" quer dizer que a evidência não sustenta ou
que o esforço não cabe no prazo.

## 1. Transferência de técnicas de transcrição musical para monitoramento de máquinas

A transferência funcionou em parte. O rastreador de rotação montado com a emissão do baixo
(soma harmônica sobre 48 harmônicos, estado de silêncio, custo de oitava) mede uma rotação
plausível no IDMT (25,7, 28,0 e 24,5 Hz nas três classes) e, sozinho, é a feature mais robusta
ao ruído da comparação (T2). O termo pYIN da emissão ajuda a 5 dB (+0,10, confirmado; T4), e o
decodificador de estados do violão reduz trocas espúrias quando as probabilidades são calibradas
(T5). Por outro lado, o Viterbi de rotação, que era a peça central herdada do baixo, não melhora
nada na validação e atrapalha mudanças rápidas na bancada (T7, T4), a pipeline completa perde
para CNNs e features clássicas no A2212 (T1) e a faixa de busca não se transferiu entre bases.
Viterbi sobre representação tempo-frequência para seguir rotação já existe (Xi et al. 2015 e a
revisão de Peeters et al. 2019), então a novidade seria o conjunto, não a peça. Falta: testar a
faixa de busca escolhida por um critério fixado antes (por exemplo, o pico dominante da
saliência no treino de cada fold), calibrar as probabilidades e rodar o teste oficial.
**Veredito: possível como enquadramento, fraco como tese "a nossa pipeline vence".**

## 2. Auditoria de vazamento: quanto da acurácia publicada nestas bases vem do split

É o tema com a evidência mais forte e o menor esforço restante. Todo trabalho com 96–100 % nas
duas bases embaralha recortes da mesma gravação (ESTADO_DA_ARTE §1). Nos nossos dados, o split
aleatório infla a acurácia no A2212 em 2 a 18 pontos, conforme a pipeline (T1); a reimplementação
do ETNeXt, que publica 99,95 % no A2212, fica em 0,905 com a seleção de atributos dentro do fold
e o split por arquivo; e deixar um motor de fora derruba todas as pipelines para 0,33–0,78. No
IDMT a auditoria mostra uma segunda camada: a validação limpa satura porque classe é igual a
motor físico e a uma sessão de gravação, a ponto de dois números (RMS e pico) darem 0,991. Nenhum
trabalho encontrado faz essa auditoria nem declara a confusão classe × unidade. Falta: rodar o
teste oficial do IDMT uma vez, por condição, para pôr nossos números ao lado dos de Grollmisch et
al.; reproduzir o protocolo da literatura no IDMT (treino e teste oficiais misturados e
embaralhados) para medir a inflação ali; e uma tabela pareada final com IC por bootstrap.
**Veredito: forte.**

## 3. Generalização entre condições de ruído e entre bases (features físicas × CNN)

Com treino limpo e ruído só na validação, as features ligadas à estrutura harmônica e à rotação
seguram a acurácia até 0 dB, onde as CNNs pequenas e a forma espectral já estão no acaso (T2,
Figura `snr_idmt.png`); a diferença p20b × p11 a 5 dB é confirmada. A ressalva estrutural do IDMT
(rotação diferente = motor diferente) limita a interpretação, e entre bases não há como
generalizar, porque as classes são outras. Falta: o teste oficial do IDMT, que tem ruído real
(fala, ambiente fabril, ruído branco, *stress test*) e é exatamente o cenário "treina em pure,
testa nas condições"; a pergunta é se o ranking sintético se repete no ruído real. **Veredito:
possível, e combina naturalmente com o tema 2.**

## 4. Tacômetro acústico de baixo custo

Na bancada sintética o rastreador com grade de 2 cents erra 0,02–0,04 % nos casos nominal, sag,
swell e interrupção a 10 dB (T7). Em dado real, a rotação do IDMT é plausível mas não tem
referência, e no A2212 a frequência rastreada acompanha a velocidade comandada só em parte dos
motores (T8). Flicker rápido e o chaveamento do *broken* não são seguidos. A novidade em relação a
Peeters 2019 e Yoo 2023 seria o microfone e o MCU, mas a tese só se sustenta com um tacômetro de
referência, o que exige bancada própria. **Veredito: possível como trabalho futuro; fraco para
26/10.**

## 5. Compromisso atraso de detecção × falso alarme com HMM online

O HMM causal reduz as trocas espúrias da p11 a 5 dB de 3,7 para 1,6 por minuto (0,9 com
penalidade 4), à custa de mais trocas perdidas (T5), e não faz nada sobre saídas não calibradas.
Strantzalis et al. são os únicos a medir resposta a trocas (3,8–4,6 s). Falta refazer com janelas
de 1 s (com 3 s o atraso fica quase sempre em zero), calibrar as probabilidades de todas as
pipelines por temperatura ajustada no treino e traçar a curva atraso × trocas espúrias variando a
penalidade. **Veredito: possível; é uma boa seção de um artigo, talvez não o artigo inteiro.**

## 6. Fronteira de Pareto custo no ESP32 × acurácia

Há estimativas analíticas para todas as pipelines (T9) e uma fronteira visível
(`pareto_custo.png`): ETNeXt e envelope ficam bem posicionados, a CNN de Strantzalis custa pouco e
cai no ruído, o rastreador com 2 cents é robusto mas custa cerca de 16 MFLOP/s. Sem medição em
hardware, um revisor do ICIT vai tratar isso como especulação. **Veredito: fraco até medir pelo
menos três pipelines no ESP32; com medição, vira uma tabela forte dentro do tema 2 ou 3.**

## 7. Detecção de anomalia treinada só no estado normal

A Mahalanobis de 4 kB dá AUC 0,944 no único teste não confundido do A2212 (saudável × hélice nos
motores 1 e 2; T6), mas são 28 arquivos de dois motores, o IDMT não serve (troca de motor) e com
ruído o limiar quebra (falso alarme de 100 %). **Veredito: fraco nestas bases.**

## 8. Distúrbios de qualidade de energia como bancada acústica (Lee & Shen)

A bancada cumpriu o papel de validação: encontrou o travamento em 4/3·f, que mudou o padrão do
rastreador, e mostrou o limite do Viterbi de rotação em mudanças rápidas (T7). Como artigo
próprio falta realismo: o caso *broken* não é seguido por nenhuma variante e os 15 %↔75 % a cada
18 ms não foram confirmados em fonte primária. **Veredito: fraco como artigo; bom como seção de
validação e como linha de continuidade entre os três artigos.**

## 9. Calibração automática dos hiperparâmetros (BO/PSO/GA), ligando com o BIMO

Não há evidência ainda. A validação limpa do IDMT satura e não serve de objetivo; a validação
ruidosa e o A2212 por arquivo serviriam, mas com 15 e 43 grupos o risco de sobreajustar a
validação é alto. **Veredito: fraco para 26/10.**

## 10. Outros temas que os dados sugerem

O atalho de nível e de fator de crista no IDMT (T1) e a confusão classe × unidade física merecem
uma figura própria dentro do tema 2. A falha do Glowacz (p04b) no A2212, causada por linhas que
dependem da velocidade, é um exemplo concreto de seleção de bins que aprende a rotação e não o
defeito, também útil no tema 2.
