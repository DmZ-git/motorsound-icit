# CLAUDE.md — Artigo completo 2027 (linha MotorSound)

Fase atual: auditoria das bases públicas de som de motores e máquinas rotativas. Não é para
rodar as pipelines; é para descobrir qual base sustenta um artigo completo sem os vieses
encontrados no IDMT (classe = motor) e no A2212 (rolamento = motor 4).

## Regras

- `../Artigo3(MotorSound)` é **somente leitura**. Código reaproveitado é copiado para
  `scripts/lib/` com cabeçalho de procedência; nada é importado por caminho relativo.
- A unidade independente é a gravação ou o motor, nunca o segmento.
- `data/` nunca entra no git. Áudio, zips e `.mat` também não.
- Um CSV por experimento em `out/experiments/`, com `_meta.json`, nunca sobrescrito.
- Nenhum número em `analise/` é digitado à mão: sai de script lendo CSV, com a origem.
- Seeds fixas e registradas.
- Não inventar referência nem detalhe de base: "não encontrado" quando for o caso.
- Windows: caminhos com parênteses sempre entre aspas e com `pathlib`.

## Pare e pergunte

Antes de download acima de 20 GB, de base que exija login, de licença restritiva, ou se for
preciso tocar em `../Artigo3(MotorSound)`.

## Estrutura

```
data/<id>/               áudio baixado (nunca commitado)
docs/bases/<id>.md       ficha por base, escrita a partir dos arquivos
docs/OUTRAS_BASES.md     candidatas encontradas e descartadas
scripts/                 inventário e experimentos; scripts/lib/ = utilitários copiados
out/experiments/         CSV + _meta.json
out/figuras/
analise/                 METODOLOGIA, RESULTADOS, COMPARATIVO_BASES, RECOMENDACAO
```

Ambiente: `.venv` (Python 3.14, uv). Português nos comentários e na documentação; textos de
`analise/` em parágrafos.
