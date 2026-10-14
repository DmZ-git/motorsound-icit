# Metodologia (fase exploratória)

Rascunho em estilo de artigo, escrito em 14/10/2026. Descreve o que foi feito na
validação; o conjunto de teste do IDMT não foi lido. Os números estão em `RESULTADOS.md`,
que é gerado a partir dos CSVs de `out/experiments/` por `experimentos/analise.py`.

## Dados

Usamos duas bases públicas. A IDMT-ISA-ELECTRIC-ENGINE (Grollmisch et al., 2019) contém
gravações de três unidades de um motor BLDC 2ACT 42BLF01 a 44,1 kHz, nos estados *good*,
*broken* e *heavyload*, provocados por tensão de alimentação e carga. O inventário que
fizemos dos arquivos (`docs/DATASETS.md`) mostra três propriedades que condicionam toda a
avaliação. A primeira é que o treino oficial tem uma única gravação por classe, de 316 a
386 s, toda na condição sem ruído (`pure`), e que os recortes de 3 s distribuídos pelos
autores (`train_cut`) são trechos contíguos dessas gravações; verificamos isso amostra a
amostra. A segunda é que cada classe foi gravada em uma unidade física diferente
(`engine1_good`, `engine2_broken`, `engine3_heavyload`), de modo que classe e identidade do
motor estão confundidas em todos os splits. A terceira é que o nível de sinal difere entre as
classes (RMS de 0,057 a 0,260 no treino), o que oferece um atalho trivial. A segunda base é a
BLDC A2212 (Estacio et al., 2023), com 43 gravações estéreo de cerca de 10 s em quatro
motores, nos estados saudável, hélice danificada e rolamento danificado, este último presente
apenas no motor 4. Reamostramos tudo para 16 kHz e convertemos para mono pela média dos canais.

## Protocolo de validação

A unidade independente é a gravação de origem, nunca o segmento. No A2212 a validação é
cruzada em cinco dobras estratificadas e agrupadas por arquivo, com janelas de 1 s; em
paralelo, deixamos um motor de fora para expor a confusão entre rolamento e motor 4. No IDMT,
como o treino tem uma gravação por classe, agrupar por gravação removeria uma classe inteira.
Cortamos então cada gravação de treino em segmentos de 3 s, os mesmos dos recortes oficiais,
e os agrupamos em cinco blocos contíguos alinhados por posição relativa entre as classes; a
validação cruzada deixa um bloco de fora e descarta o primeiro segmento de cada bloco como
guarda, para que nenhum segmento de treino seja vizinho imediato de um de validação. Essa
validação é otimista por construção (mesma sessão, sem ruído, classe igual a unidade física)
e foi assumida como tal. O conjunto de teste oficial fica trancado por código
(`motorsound/io.py`) e só será lido uma vez, depois do congelamento das configurações.

Como a validação limpa do IDMT satura (seção de resultados), a comparação entre pipelines usa
quatro critérios adicionais definidos antes de olhar os resultados dessas comparações: a
validação com ruído sintético aplicado só às janelas de validação, com o treino sempre limpo,
em ruído branco e rosa de 30 a −5 dB e em ruído grave, impulsos, reverberação e degraus de
ganho, que imitam as condições do teste oficial sem lê-lo; o A2212 por arquivo e por motor; o
custo estimado para o ESP32; e as métricas de sequência.

Duas auditorias acompanham cada pipeline. A auditoria de vazamento roda a mesma pipeline com
split aleatório por segmento, que é o protocolo da maior parte da literatura destas bases, e
compara com o split agrupado; a diferença é tratada como resultado. A auditoria de nível roda
com e sem normalização do RMS de cada janela. Um controle com rótulos permutados por grupo
verifica que o código não deixa a validação vazar para o treino.

## Pipelines

Todas as pipelines expõem a mesma interface (extração cacheável, treino, previsão de
probabilidades por janela e estimativa de custo) e são avaliadas pelo mesmo runner, com o
mesmo split, as mesmas janelas e as mesmas métricas. Distinguimos reproduções, que seguem o
artigo sem suposições relevantes, de reimplementações inspiradas; as suposições estão no
cabeçalho de cada arquivo e em `docs/PIPELINES.md`.

A família Decifrae transfere a pipeline de transcrição musical do projeto. A camada contínua é
o decodificador de Viterbi de altura do transcritor de baixo, com estado de silêncio e custo de
oitava, adaptado para seguir a frequência de rotação numa grade logarítmica de 15 a 60 Hz em
passos de 10 cents; a emissão combina, como no baixo, a proximidade a uma estimativa pYIN
ponderada pela probabilidade de voz e a energia espectral normalizada, que aqui é uma soma
harmônica sobre 48 harmônicos, porque a energia do motor se concentra em harmônicos altos da
rotação. As transições ficam restritas a saltos vizinhos, o que reduz o custo de O(T·S²) para
O(T·S·k). Da trilha de rotação saem 13 atributos físicos por janela (rotação mediana e sua
dispersão, saliência e razão harmônica, energia por grupo de ordem, pico de modulação do
envelope e forma espectral), classificados por cosseno contra protótipos de classe, como no
classificador de acordes do violão. A camada discreta é o decodificador de estados do violão,
com penalidade de troca fixa, aplicado sobre as probabilidades por janela, em versão offline
(Viterbi) e causal (filtro *forward*). As variantes isolam cada peça: p10 usa pYIN, p11 só a
soma harmônica (embarcável), p12 troca o Viterbi de rotação pelo argmax por quadro, e a
decodificação argmax das sequências isola o HMM de estados.

## Métricas e estatística

Por janela, acurácia e F1 macro, por condição. Por sequência, concatenamos trechos contíguos
de janelas de validação de estados diferentes, com permanências sorteadas de 5 a 20 janelas
(15 a 60 s no IDMT), e medimos o atraso de detecção de cada troca, as trocas perdidas e as
trocas espúrias por minuto, varrendo a penalidade de troca do HMM. As comparações entre
pipelines usam bootstrap pareado com 10 000 reamostragens de grupos (gravação no A2212,
gravação × bloco no IDMT), Wilcoxon pareado como critério secundário e correção de Holm, com a
regra confirmado/empate/frágil herdada do violão. O custo para o ESP32 é estimado
analiticamente (parâmetros, RAM, MFLOP por segundo de áudio) e ainda não foi medido no hardware.
