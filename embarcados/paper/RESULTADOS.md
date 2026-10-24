# Resultados (artigo ICIT 2027, `embarcados/`)

> Gerado por `embarcados/src/python/gerar_artigo_textos.py` a partir de `paper/numeros.json`
> (commit 8c97c00). Não editar à mão. Validação apenas; o teste oficial do IDMT não foi lido.
> Valores **[pendente]** dependem da bancada do ESP32 físico.

## Equivalência Python × C (medição no PC)

Em double, as 26 verificações de etapa passam em todas as 164 janelas e nas
240 configurações de sequência (26 de 26); a maior razão
erro/tolerância é 0,55. Caminho de rotação, decisão e todas as saídas do HMM
coincidem exatamente. Em float32, que é o build do microcontrolador, as 150 de
150 janelas de áudio real (limpas e ruidosas) passam em todas as etapas. As falhas
restantes estão em 2 janelas, ambas o caso de borda sintético de nível constante
com ruído (uma por modelo: borda_dc_ruido3s_ma2212, borda_dc_ruido3s_midmt). Ali o envelope é quase constante e o espectro de
modulação fica no piso de arredondamento do float32 nas duas implementações, de modo que a posição
do pico é indeterminada nessa precisão: o pico da referência vale 4,0 vezes a
potência da referência no bin escolhido pelo C, então não é empate. A frequência de modulação muda
37,1 Hz. Sob a regra congelada o comparador marcou PASS_EMPATE em `env_mod` e
`features` e FAIL em `z`; adotamos a leitura conservadora e contamos as três etapas
(env_mod, features, z) como FAIL nesses casos, o que deixa 23 de
26 etapas passando em todas as janelas (erro máximo de z 1,61). As
tolerâncias e o comparador não foram alterados. Cossenos, probabilidades e decisão passam em
todas as janelas (164 de 164). Com a variante nomeada `p11_c_sosd`, as
26 verificações passam; ela reduz o erro do envelope dizimado (máximo relativo ao pico, mediana entre
janelas) de 1,2 × 10^-5 para 1,2 × 10^-7, porque os polos do passa-baixa ficam perto
do círculo unitário e o arredondamento em float32 se acumula no estado do filtro. A afirmação que os
dados sustentam é concordância etapa a etapa dentro de tolerâncias congeladas antes da comparação
formal, com a exceção acima; não é equivalência incondicional. A auditoria independente (agente F)
reproduziu a comparação a partir de um build limpo, e as correções dela estão aplicadas aqui. A
mesma comparação no chip está pendente (**[pendente]** de **[pendente]** etapas; decisão
**[pendente]**) e não foi coberta pela auditoria.

## Custo

Medido no PC, o C float32 leva 13,29 ms por janela de 3 s (p95
15,00 ms, máximo 15,80 ms) e 3,18 ms por janela de
1 s, contra 95,3 ms e 22,1 ms da referência Python; a razão
(7,2 vezes) inclui o custo do interpretador e não é uma comparação de algoritmos. No PC,
STFT por quadro com soma harmônica fica com 41 % da mediana, o Viterbi de rotação
com 40 % e o ramo do envelope com 17 %; features e cosseno são
desprezíveis. O fator de tempo real no PC (0,0044) não diz nada sobre o microcontrolador.

A estimativa analítica da arena em float32 é 1,86 MB para 3 s e
0,96 MB para 1 s (1,52 e 0,61 MB
sem a tabela de interpolação harmônica, que ocupa 0,35 MB); os buffers do Hilbert
dominam (fase do envelope = 64 % do total em 3 s). Em double
seriam 3,60 e 1,79 MB. Numa unidade só, a arena
(1820 e 936 KiB) é pelo menos 5,5 e
2,8 vezes a região DIRAM interna inteira que o build reporta para o ESP32-S3
(334 KiB, dos quais só 280 KiB sobram depois da alocação
estática; a IRAM do S3 está 100 % usada), e 10,3 e
5,3 vezes a DRAM do ESP32 (176 KiB): a referência exige PSRAM, e qualquer redução (Hilbert por blocos,
tabelas menores) muda a aritmética e teria de ser verificada como variante nomeada. No build do
firmware, a imagem ocupa 209 KiB (ESP32, 20 % da partição) e
233 KiB (ESP32-S3, 23 %); o uso estático de RAM interna é
15 KiB de DRAM mais 58 KiB de IRAM no ESP32 e 54 KiB
de DIRAM no ESP32-S3. A arena é alocada em tempo de execução e não entra nesses números.

Medido no ESP32: modelo **[pendente]**, **[pendente]** MHz, PSRAM **[pendente]** MB; latência mediana
**[pendente]** ms em 3 s (p95 **[pendente]**, máximo **[pendente]**)
e **[pendente]** ms em 1 s; fator de tempo real **[pendente]** e **[pendente]**;
heap interno mínimo livre **[pendente]** KiB; folga de pilha **[pendente]** bytes. Nenhuma
estimativa foi posta no lugar desses valores.

## Classificação nos três protocolos

No IDMT todos os métodos ficam perto do teto nos dois protocolos. O baseline de nível com RMS
normalizado, que só vê o fator de crista, chega a 0,991 de acurácia balanceada nos
blocos (0,982–0,997), e a p11 e as clássicas chegam a
1,000 e 1,000. Como cada classe é uma gravação de um motor, esses
números medem a identidade da gravação e não ordenam os métodos.

No A2212 com split por arquivo, as clássicas chegam a 0,981
(0,950–0,998), a p11 a 0,816
(0,747–0,881) e o nível bruto a 0,608. A
p11 erra sobretudo em Healthy (sensibilidade 0,577). O split aleatório
infla a acurácia balanceada em 0,033 na p11 e 0,017 nas clássicas. Com o
motor fora, todos caem: as clássicas para 0,613 e a p11 para
0,456, com sensibilidade de Bearing 0,000 por construção.
Sem o fold do M4 (Healthy × Propeller em motor não visto), as clássicas ficam em
0,920, o nível com RMS normalizado em 0,814 e a p11 em
0,685. Comparando igual por igual (Healthy e Propeller dos motores M1–M3 no
split por arquivo: clássicas 0,972, p11 0,753, nível com RMS
normalizado 0,843), tirar o motor custa 0,052 às clássicas e
0,068 à p11; a queda grande do protocolo de motor fora completo vem sobretudo da
classe Bearing, que perde o seu único motor de treino. A p11 não supera as features clássicas em
nenhum protocolo (empata no teto do IDMT) e, com motor fora, a estimativa pontual dela fica abaixo
também do baseline de nível, com intervalos sobrepostos. ROC-AUC macro da p11 por arquivo:
0,935; das clássicas: 0,996. No protocolo de motor fora a AUC junta
scores de modelos de folds diferentes e, no fold do M4, p_Bearing = 0 é imposto; por isso não a
usamos para ordenar métodos. Em toda linha com Bearing, a classe está confundida com o motor M4.

## Rotação estimada (interpretabilidade)

No IDMT a rotação mediana rastreada é 25,76, 27,99 e 24,65
Hz para good, broken e heavyload, com IQR de 0,18, 0,03 e
0,20 Hz; 8,6 % das janelas good ficam acima de 1,5 vezes a
mediana da classe (todas entre 2,00 e 2,02 vezes), um erro de oitava.
Como classe = motor no IDMT, essas diferenças não podem ser atribuídas ao estado. No A2212, juntando
os 43 arquivos, a correlação de postos entre a mediana por arquivo e a velocidade
que dá nome aos arquivos é 0,02, mas dentro de cada motor ela é positiva (M1
0,30, M2 0,30, M3 0,54, M4
1,00; 14, 16, 6 e
7 arquivos): o valor global perto de zero vem de diferenças de nível entre
motores (efeito de Simpson). O nível absoluto não é comparável entre motores e 3,7 %
das janelas ficam a menos de 1 % do limite superior da faixa. A faixa e a grade foram
escolhidas no IDMT; no A2212 a saída do rastreador não deve ser lida como a velocidade do eixo. Arquivos do mesmo motor não são
independentes, então a correlação é descritiva.

## Decisão em fluxo

Numa sequência de 24 janelas de validação do IDMT (8 por classe, good →
broken → heavyload), o HMM online com penalidade 2 detectou as 2 trocas com
atraso de 0 janelas no C e 0 no Python, com decisões iguais (diferença
máxima de probabilidade 3,2 × 10^-6), medido no PC. Dada a saturação do IDMT, isso confere a
implementação, não o detector. No chip: atraso máximo **[pendente]** janelas.

## O que as bases permitem concluir

O IDMT sem o seu teste não sustenta nenhuma afirmação sobre discriminar estado de operação: uma
gravação por classe torna classe, motor e sessão a mesma variável, e o baseline de dois números é
tão bom quanto qualquer modelo. O A2212 com split por arquivo sustenta afirmações sobre o estado de
motores já vistos no treino, e ali as features clássicas são melhores que a p11. Generalizar para
um motor novo é a pergunta relevante para um monitor implantado, e com 4 motores,
um defeito de rolamento num só deles e faixas de velocidade diferentes por classe, os dados só
mostram que todo método piora. Nenhum experimento trata de predição de falha futura e nenhum usa
motores industriais. Nos nossos dados o split aleatório inflou pouco a acurácia do A2212
(0,017 a 0,033), e tirar o motor, igual por igual em Healthy × Propeller,
custou um valor parecido (0,052 às clássicas, 0,068 à p11); a queda
de 0,368 e 0,360 do protocolo de motor fora completo é dominada pela
classe Bearing, que só existe no M4.
