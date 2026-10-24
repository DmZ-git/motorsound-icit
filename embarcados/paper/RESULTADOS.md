# Resultados (artigo ICIT 2027, `embarcados/`)

> Gerado por `embarcados/src/python/gerar_artigo_textos.py` a partir de `paper/numeros.json`
> (commit 9df3b2c). Não editar à mão. Validação apenas; o teste oficial do IDMT não foi lido.
> Valores **[pendente]** dependem da bancada do ESP32 físico.

## Equivalência Python × C (medição no PC)

Em double, as 26 verificações de etapa passam em todas as 164 janelas e nas
240 configurações de sequência (26 de 26); a maior razão
erro/tolerância é 0,55. Caminho de rotação, decisão e todas as saídas do HMM
coincidem exatamente. Em float32, que é o build do microcontrolador, passam 25 de
26 verificações. A exceção é o z-score, que falha em 2 de 164
janelas, ambas o caso de borda sintético de nível constante com ruído (uma por modelo:
borda_dc_ruido3s_ma2212, borda_dc_ruido3s_midmt). Ali o envelope é quase constante e o espectro de Welch tem dois picos cujas
potências na referência diferem de 9,4 × 10^-16 (tolerância de empate 6,3 × 10^-11); o C
escolhe o outro pico, a feature de frequência de modulação muda 37,1 Hz e o z-score,
cuja tolerância é propagada das features e não herda a exceção de empate, falha (erro máximo
1,61). Cossenos, probabilidades e decisão passam em todas as janelas
(164 de 164). Com a variante nomeada `p11_c_sosd`, as 26
verificações passam; ela reduz o erro do envelope dizimado (máximo relativo ao pico, mediana entre
janelas) de 1,2 × 10^-5 para 1,2 × 10^-7, porque os polos do passa-baixa ficam perto
do círculo unitário e o arredondamento em float32 se acumula no estado do filtro. A afirmação que os
dados sustentam é concordância etapa a etapa dentro das tolerâncias pré-registradas, com a exceção
acima; não é equivalência incondicional. A mesma comparação no chip está pendente
(**[pendente]** de **[pendente]** etapas; decisão **[pendente]**), e a auditoria
independente (agente F) também.

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
seriam 3,60 e 1,79 MB. Isso é 5,5 e
2,8 vezes a região DIRAM interna que o build reporta para o ESP32-S3
(334 KiB), e 10,3 e 5,3 vezes a DRAM do ESP32
(176 KiB): a referência exige PSRAM, e qualquer redução (Hilbert por blocos,
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
0,685. A p11 não supera as features clássicas em nenhum protocolo e, com motor
fora, fica abaixo também do baseline de nível. ROC-AUC macro da p11 por arquivo:
0,935; das clássicas: 0,996.

## Rotação estimada (interpretabilidade)

No IDMT a rotação mediana rastreada é 25,76, 27,99 e 24,65
Hz para good, broken e heavyload, com IQR de 0,18, 0,03 e
0,20 Hz; 8,6 % das janelas good ficam acima de 1,5 vezes a
mediana da classe, um erro do tipo oitava. Como classe = motor no IDMT, essas diferenças não podem
ser atribuídas ao estado. No A2212 a rotação rastreada não acompanha a velocidade que dá nome aos
arquivos: a correlação de postos entre a mediana por arquivo e esse rótulo é 0,02 nos
43 arquivos (por motor: M1 0,30, M2 0,30, M3
0,54, M4 1,00; 14, 16,
6 e 7 arquivos), e 3,7 % das janelas ficam a
menos de 1 % do limite superior da faixa. A faixa foi escolhida no IDMT; no A2212 a
saída do rastreador não deve ser lida como a velocidade do eixo. Arquivos do mesmo motor não são
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
(0,017 a 0,033), enquanto tirar o motor custou 0,368 às
clássicas e 0,360 à p11.
