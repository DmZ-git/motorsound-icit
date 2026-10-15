# Decisão ao fim da fase exploratória

Escrito em 14/10/2026, para a reunião com o orientador em 15/10. Base: validação apenas; o
teste oficial do IDMT não foi lido. Números em `RESULTADOS.md` e `RESULTADOS_tabelas.md`.

## A ideia funciona?

Em parte, e não do jeito que o plano previa. O rastreador de rotação construído com as peças do
transcritor de baixo funciona: mede uma rotação fisicamente plausível nas três classes do IDMT,
acerta a rotação sintética com erro abaixo de 0,1 % depois da correção da grade, e é a feature
mais robusta ao ruído de toda a comparação, a única acima de 0,9 com ruído branco a 0 dB. O
decodificador de estados do violão também funciona como esperado, reduzindo as trocas espúrias
quando as probabilidades por janela são calibradas. O que não funcionou foi a tese de que a
pipeline Decifrae completa supera a literatura. O Viterbi de rotação não ajuda e atrapalha
mudanças rápidas, a pipeline perde por cerca de 20 pontos para CNNs e features clássicas no
A2212, e a faixa de busca fixada a partir do IDMT não serve para os motores do A2212. A ideia é
boa como fonte de uma feature robusta e barata, e fraca como "nossa pipeline vence".

A fase também produziu um resultado que não estava no plano e que é mais forte do que qualquer
pipeline: as duas bases usadas pela literatura não medem o que se pensa que medem. No IDMT cada
classe é um motor físico gravado numa sessão só, e dois números (RMS e pico) bastam para 99 % na
validação. No A2212 o split aleatório infla a acurácia em até 18 pontos, deixar um motor de fora
derruba tudo para 0,33–0,78, e o rolamento é indistinguível do motor 4. Todos os trabalhos com
96–100 % nessas bases usam o split que infla.

## Qual tema tem a melhor relação evidência × esforço para o ICIT até 26/10?

O tema 2, auditoria de vazamento, combinado com o tema 3, robustez de features físicas ao ruído.
Título de trabalho: *"How much of acoustic motor-state accuracy is the split? A leakage audit of
two public datasets, with a noise-robust rotation feature for microcontrollers"*. A narrativa tem
três partes, todas com dados já medidos ou mensuráveis em dias. Primeiro, a literatura reporta
96–100 %, mas com o split que embaralha recortes; nós refazemos as principais pipelines com split
por gravação e medimos a inflação, além de mostrar as confusões estruturais (classe = motor,
rolamento = motor 4, atalho de nível). Segundo, sob split correto, o ruído é o que separa os
métodos, e uma feature de rotação vinda de transcrição musical, barata e embarcável, é a mais
robusta. Terceiro, o custo estimado no ESP32 ao lado da acurácia. Os temas 5 (HMM online) e 8
(bancada Lee & Shen) entram como seções curtas.

O que falta, em ordem de prioridade, para os 12 dias até 26/10:

1. Congelar as configurações (`congelar.py` no molde do violão, ainda não escrito) e, com a sua
   aprovação, rodar o teste oficial do IDMT uma única vez, por condição, para comparar com
   Grollmisch et al. (82–100 %) e verificar se o ranking sintético se repete no ruído real.
2. Reproduzir no IDMT o protocolo da literatura (treino e teste oficiais misturados e
   embaralhados) para medir a inflação também ali; isso exige o teste e entra no mesmo
   congelamento.
3. Definir a faixa de busca do rastreador por um critério fixado antes de olhar a validação e
   rodar o A2212 de novo; calibrar as probabilidades por temperatura no treino.
4. Refazer as sequências com janelas de 1 s.
5. Opcional: medir duas ou três pipelines no ESP32 (a CNN de Strantzalis, o envelope e o
   rastreador) para trocar estimativa por medição.
6. Escrever as 6 páginas.

Se o teste oficial mostrar que todas as pipelines ficam perto de 100 % também nas condições
ruidosas reais, a parte de robustez enfraquece e o artigo fica só com a auditoria, que ainda se
sustenta. Se a auditoria parecer pouco para o ICIT, o plano B é submeter a auditoria como artigo
curto ou workshop e deixar o tacômetro acústico com bancada própria para depois.

## O que levar ao orientador

Levar quatro coisas. A primeira é o achado das bases: classe = motor físico no IDMT (com o
inventário verificado e o resultado de 99 % com RMS e pico) e a inflação de 2 a 18 pontos pelo
split no A2212, com a tabela da literatura mostrando que ninguém declarou isso. A segunda é a
figura de SNR, que mostra a rotação rastreada como a feature mais robusta, e a ressalva de que no
IDMT rotação diferente também é motor diferente. A terceira é o resultado negativo honesto: a
pipeline Decifrae completa não vence no A2212, e o Viterbi de rotação não ajuda. A quarta é a
proposta de artigo acima, com três perguntas para ele: se a auditoria de vazamento é um artigo
aceitável para o ICIT (ou se ele prefere outro veículo), se vale abrir o teste oficial agora, e se
há como conseguir um tacômetro e um ESP32 para medição real.

Pendências de procedimento que ele pode questionar: a validação do IDMT por blocos temporais foi
uma decisão sua (registrada em `docs/DATASETS.md` §3), porque o treino tem uma gravação por
classe; a grade do rastreador foi trocada por causa da bancada sintética, não da validação; os
notebooks do Kaggle não foram lidos porque exigem login; nenhuma pipeline é reprodução fiel,
porque todos os artigos omitem parâmetros.
