# Rastreabilidade dos resultados do artigo

Auditoria somente dos artefatos existentes; nenhum treinamento, leitura de áudio ou teste físico
foi repetido.

## Teste oficial IDMT

O congelamento está em
`configs/experiments/congelado_teste_idmt_20261025-150739.json`. Ele registra o commit
`ac0f113f35e9d531653268d1177a81d20b80ca10`, árvore limpa, configurações das quatro variantes,
semente do bootstrap e hashes SHA-256 dos arquivos de código. A execução está registrada em
`results/csv/teste_idmt_20261025-150749_meta.json`, no commit
`6f8caee993881a1f97d8d185532e56f4ea322ca6`, também com árvore limpa.

Os hashes do congelamento coincidem com os blobs versionados nos commits de congelamento e
execução para os arquivos comuns de pipeline e avaliação: `pipelines/b00_nivel.py`,
`pipelines/p04_classicas.py`, `pipelines/p11_decifrae_harmsum.py`, `pipelines/_decifrae.py`,
`motorsound/runner.py`, `motorsound/io.py`, `motorsound/config.py`,
`motorsound/classificador.py`, `motorsound/rotacao.py` e
`embarcados/src/python/diagnostico.py`. `embarcados/src/python/teste_idmt.py` ainda não existia no
commit de congelamento, mas seu hash registrado coincide com o blob no commit da execução. A
diferença entre os dois commits não alterou os arquivos comuns cobertos pelos hashes. Os arquivos
que entraram entre os commits incluem o script do teste, o congelamento e artefatos de build e
resultados embarcados.

As predições por janela estão em
`results/predictions/teste_idmt_janelas_20261025-150749.csv`; os resultados agregados, inclusive
os ICs individuais de BA e ROC-AUC, em `results/csv/teste_idmt_20261025-150749.csv`. O script
`src/python/teste_idmt.py` gerou esses ICs com `diagnostico.bootstrap`, reamostrando `rec_id`
estratificado pela classe; os metadados confirmam 2.000 réplicas e seed `20261025`. As 2.021
janelas cobrem 27 gravações, nove por classe. Os resultados da tabela usam scores por janela e
não aplicam HMM; o HMM é medido em experimentos sequenciais separados.

Foi recalculada a diferença `BA(p04) - BA(p11)` por bootstrap pareado e estratificado por classe,
reamostrando `rec_id` com reposição e aplicando a mesma amostra de gravações aos dois métodos em
cada réplica. Com 2.000 réplicas e a seed `20261025`, a diferença observada é `0.10895`, com
IC95% `[0.02889, 0.19555]`. Essa incerteza é calculada entre as gravações disponíveis e não deve
ser interpretada como generalização para outros motores. O cálculo e a emissão das macros estão
em `src/python/gerar_artigo_teste.py::bootstrap_diferenca_ba`; a origem é o CSV de predições acima
e a seed/número de réplicas vêm do metadata da execução.

## Validação A2212/IDMT e limites de rastreabilidade

Os metadados `results/predictions/diagnostico_janelas_20261023_222831_meta.json` registram o
commit `9081500`, `sujo: true`, seeds de split/bootstrap `20261014` e `20261023`, 2.000
reamostragens e as variantes de nível, p04 e p11. O CSV de predições contém fold, identificador
de janela, rótulo, classe prevista e scores; o CSV agregado está em
`results/csv/diagnostico_20261023_222831.csv`. Entretanto, não há hashes do código usado nessa
execução e a árvore estava suja: o snapshot exato que gerou essas métricas não pode ser
reconstruído com segurança apenas pelos metadados disponíveis. Isso é uma limitação de
rastreabilidade, não evidência suficiente para descartar os resultados.

Uma reexecução com as mesmas seeds em checkout limpo não foi realizada porque `data/idmt/`,
`data/a2212/` e `.venv/` não estavam disponíveis no ambiente de validação. Não foram gerados
novos resultados nem hashes de código para essa validação. Os valores da Tabela IV permanecem
arquivados como resultados exploratórios da execução acima; não há evidência para afirmar que uma
reexecução limpa os reproduziria. O manuscrito registra expressamente essa limitação.

## Escopo das comparações e manuscrito

O artigo embarcado avalia p04, p11 e dois baselines de nível nos protocolos indicados na tabela
`tables/latex/desempenho_compacto.tex`. Não inclui p21 nem a tabela exploratória de 20 pipelines
do ramo anterior; esses valores não são usados como comparação no manuscrito embarcado. A
estrutura de seis seções recomendada está em `paper/main.tex`: Introduction, Related Work,
Methodology (com Experimental setup como subseção), Results, Discussion e Conclusion. A proposta
de aplicação futura é triagem acústica periódica em ambiente não crítico; aquisição ao vivo,
operação em tempo real, energia e generalização industrial continuam não demonstradas.
