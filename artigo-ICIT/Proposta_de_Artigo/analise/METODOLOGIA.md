# Metodologia da auditoria das bases

Este documento descreve como as bases públicas de som de motores e máquinas rotativas foram
auditadas antes da escolha da base do artigo completo de 2027. A auditoria não compara métodos
de diagnóstico. Ela procura, em cada base, os cinco problemas estruturais que comprometeram as
duas bases usadas no trabalho do ICIT (`../artigo-ICIT/DECISAO.md`): classe confundida com a
unidade física, classe confundida com a sessão de gravação, atalho de nível, poucas unidades
para deixar um motor de fora, e falta de verdade de rotação. Os números citados nos demais
documentos de `analise/` vêm de `analise/tabelas_auditoria.md`, gerado por
`scripts/consolidar.py` a partir dos CSVs de `out/experiments/`.

## Seleção das bases

Três bases foram indicadas de início: o conjunto de motores da Universidade de Ottawa sob
velocidade constante e variável (`uoemd`), a bancada rotativa de Jung et al. (`kaist`) e o
MIMII com o MIMII DUE (`mimii`). Uma busca dirigida, registrada em `docs/OUTRAS_BASES.md`,
listou cerca de 25 outras candidatas e recomendou fichas completas para três delas: o ToyCar
do DCASE 2022, que é um subconjunto público do ToyADMOS2 (`toycar`), as gravações por celular
dos mesmos motores de Ottawa (`imvacd`) e a Machinery Fault Database da UFRJ, que grava
microfone e tacômetro juntos (`mafaulda`). O IDMT-ISA-ELECTRIC-ENGINE (`idmt`) e o BLDC A2212
(`a2212`) entram só como referência, com os números já medidos no trabalho do ICIT e lidos dos
CSVs daquele trabalho por `scripts/referencia_artigo3.py`, sem reanálise.

Antes de cada download, o tamanho, a licença, a exigência de login e o formato foram
verificados nas APIs dos repositórios (Mendeley Data e Zenodo). Ficou combinado parar e
consultar o autor antes de qualquer download acima de 20 GB, de qualquer base com login e de
qualquer licença restritiva. A MaFaulDa, cuja licença não foi encontrada, só foi baixada
depois da autorização explícita do autor. Do ToyADMOS2, cuja versão completa tem licença não
comercial, só se usou o subconjunto do DCASE, distribuído sob CC BY 4.0. Da MIMII, só se baixou
um subconjunto.

## Inventário a partir dos arquivos

Cada base tem uma ficha em `docs/bases/<id>.md`, escrita a partir da leitura dos arquivos e
não da documentação. Onde as duas divergem, a ficha registra as duas versões. A ficha traz a
árvore de pastas, a taxa de amostragem e a resolução medidas no canal acústico, a duração por
classe, a matriz classe × unidade física, a matriz classe × condição, a origem da verdade de
rotação, a definição da gravação de origem e a leitura da literatura que usa a base. Ela
também diz, para cada referência, se o texto completo foi lido ou só o resumo.

O inventário de cada base gera um índice, `out/indices/<id>.csv`, com uma linha por arquivo de
áudio e colunas comuns a todas as bases: classe, unidade física, condição, velocidade, carga,
identificador da gravação de origem, taxa, canais, duração, frequência de alimentação conhecida
e rotação medida. Cada base tem também um carregador, `scripts/lib/load_<id>.py`, que devolve
o canal acústico mono. A unidade física é o objeto que se tiraria do treino num teste de
"motor fora". Ela é o motor nas bases de Ottawa, o carrinho no ToyCar, o exemplar de máquina
(model ID) no MIMII e, na KAIST, o rolamento montado na bancada única.

## Pré-processamento comum

Todas as bases passam pelo mesmo caminho, implementado em `scripts/lib/comum.py`. O canal
acústico é reamostrado para 16 kHz (`scipy.signal.resample_poly`), a taxa prevista para o
microcontrolador, e cortado em segmentos de 1 s sem sobreposição, descartando a sobra final.
Em bases com arquivos longos, o número de segmentos por arquivo pode ser limitado a valores
igualmente espaçados ao longo da gravação. O limite usado em cada experimento fica registrado
no `_meta.json` do CSV: todos os segmentos na KAIST, na UOEMD, na IM-VACD e na MaFaulDa, 5 por
arquivo no ToyCar e 3 por arquivo na MIMII. Filtros aplicados depois da extração ficam também
no `_meta.json`. Na MIMII, entram só as máquinas do MIMII original (ventilador e bomba a 6 dB,
CC BY-SA 4.0); as seções do MIMII DUE, sob licença não comercial, ficaram de fora até decisão
do autor. No ToyCar, saem os arquivos de teste do conjunto de avaliação, cuja unidade é
desconhecida. Na MaFaulDa, os experimentos 1, 2 e 5 foram rodados com todos os arquivos e,
de novo, sem os 166 arquivos em que o canal do microfone está defeituoso; as tabelas usam a
rodada sem eles. Os espectros médios do experimento 3 são agregados por célula e não podem ser
filtrados por arquivo, por isso incluem os arquivos defeituosos. As features por segmento foram copiadas, com cabeçalho de procedência, dos baselines
do trabalho do ICIT. O controle de nível usa o RMS e o pico em dB (`b00_nivel`). A forma
espectral usa a energia relativa em 32 bandas logarítmicas entre 50 Hz e 8 kHz, mais
centróide, espalhamento, rolloff de 85 %, planura e taxa de cruzamentos por zero
(`b01_forma_espectral`). A forma espectral não carrega nível absoluto.

O classificador de todos os experimentos é uma regressão logística sobre features
padronizadas, com pesos de classe balanceados. O balanceamento é um desvio em relação aos
baselines do ICIT, adotado porque algumas bases têm muito mais normal que anomalia. A métrica
é a acurácia balanceada sobre as predições concatenadas de todas as dobras, comparada com o
acaso, 1 sobre o número de classes presentes no teste. A semente é fixa (20261022) e fica
registrada em cada `_meta.json`, junto com o commit e a configuração.

## Experimentos de auditoria

O primeiro experimento é o controle de nível. O classificador recebe só o RMS e o pico do
segmento e é avaliado com três divisões. A primeira é aleatória por segmento, em 5 dobras
estratificadas, e reproduz o protocolo que infla os números da literatura. A segunda agrupa
por gravação de origem, também em 5 dobras estratificadas, de modo que recortes da mesma
gravação nunca ficam dos dois lados. A terceira deixa uma unidade física de fora por vez. O
experimento roda na base inteira e, separadamente, em cada valor de condição, quando há entre
2 e 12 valores.

O segundo experimento é um controle de sanidade com rótulos permutados, em cinco permutações
com sementes registradas. Na permutação por segmento, todo split tem de dar o acaso; se não
der, há erro no código. Na permutação por gravação, cada gravação recebe o rótulo de outra
gravação e todos os seus segmentos herdam esse rótulo. Os splits agrupados têm de dar o acaso.
Se o split aleatório ficar acima do acaso, as features reconhecem a gravação, e qualquer número
obtido com split aleatório naquela base mede em parte essa identificação.

O terceiro experimento calcula o espectro médio, por Welch com 2048 pontos a 16 kHz, de cada
combinação de classe e unidade. Os espectros são plotados em dois arranjos: um painel por
unidade com uma curva por classe, e um painel por classe com uma curva por unidade. As figuras
estão em `out/figuras/`. Depois de remover o nível médio de cada espectro, a diferença entre
dois espectros é resumida pelo RMS da diferença em dB, entre 50 e 7900 Hz. Comparar a mediana
dessa distância entre classes dentro da mesma unidade com a mediana entre unidades dentro da
mesma classe mostra se a diferença visível no espectro é de classe ou de unidade. Numa matriz
diagonal, em que cada classe vive numa unidade só, nenhum dos dois tipos de par existe, e a
comparação é impossível por construção.

O quarto experimento testa o rastreador de rotação do trabalho do ICIT, copiado sem alteração
para `scripts/lib/rotacao.py` e usado com os parâmetros daquele trabalho: grade de 2 cents,
soma harmônica dos harmônicos 1 a 48, FFT de 8192 pontos, salto de 1024 e Viterbi. Só a faixa
de busca muda, e ela foi definida por duas regras escritas e commitadas antes de qualquer
execução com dado real. Nas bases com frequência de alimentação conhecida, a faixa vai de
0,40 a 1,05 vez essa frequência, o que cobre motores de 2 e de 4 polos com escorregamento de
até 20 %. Nas bases com rotação medida por tacômetro e sem frequência de alimentação, a faixa
é única para a base inteira, de 0,9 vez a menor a 1,1 vez a maior rotação medida, para que o
rastreador não receba nenhuma informação do arquivo que rastreia. A referência para o erro é a
rotação medida, quando existe. Na falta dela, usa-se a rotação síncrona dada pela frequência de
alimentação e pelos pares de polos da placa; nesse caso, o erro inclui o escorregamento, que é
reportado à parte. As métricas são o erro relativo mediano por arquivo e a fração de arquivos e
de segmentos de 1 s dentro de 2 % da referência. Um diagnóstico a posteriori, sinalizado como
tal nas tabelas, conta também como acerto os valores a meia e a duas vezes a referência, para
separar erro de oitava de erro de rastreamento.

O quinto experimento mede a separabilidade por unidade. A mesma regressão logística, agora com
a forma espectral, é treinada para prever a unidade física em vez da classe, com split
aleatório e com split por gravação, na base inteira e dentro de cada classe. Uma acurácia alta
com split por gravação significa que a base permite reconhecer o motor. Nesse caso, qualquer
resultado de classe numa base em que classe e unidade se confundem pode estar medindo a
identidade do motor.

## Limites do protocolo

Nenhum desses experimentos é resultado de artigo. Os classificadores são deliberadamente
simples, e uma acurácia baixa do controle de nível não prova que não exista outro atalho. Na
UOEMD e na IM-VACD, a verdade de rotação é o setpoint do inversor, não uma medida no eixo. No
ToyCar, os arquivos de teste do conjunto de avaliação do DCASE têm unidade desconhecida e foram
excluídos por filtro, registrado no `_meta.json`. A referência do IDMT e do A2212 segue o
protocolo do trabalho do ICIT (janelas de 3 s com RMS normalizado e validação por blocos no
IDMT; janelas de 1 s no A2212), e não o desta auditoria.
