# CLAUDE.md — Artigo completo 2027 (linha MotorSound)

Fase atual: auditoria das bases públicas de som de motores e máquinas rotativas. Não é para
rodar as pipelines; é para descobrir qual base sustenta um artigo completo sem os vieses
encontrados no IDMT (classe = motor) e no A2212 (rolamento = motor 4).

## Regras

- Esta pasta (`artigo-2027/`) vive dentro do repositório `Artigo3(MotorSound)` desde
  23/10/2026 (antes era um repositório próprio em `Decifrae-Artigo/Artigo(2027)`; o histórico
  foi importado com `git subtree`). **Fora desta pasta, o Artigo3 é somente leitura**: o
  trabalho do ICIT (`../motorsound/`, `../artigo-ICIT/`, `../docs/` etc.) não é editado
  daqui. Código reaproveitado é copiado para `scripts/lib/` com cabeçalho de procedência;
  nada é importado de `../motorsound/`.
- Scripts rodam com o `.venv` **desta pasta** (`artigo-2027/.venv`), não o do Artigo3.
- A unidade independente é a gravação ou o motor, nunca o segmento.
- `data/` nunca entra no git. Áudio, zips e `.mat` também não.
- Um CSV por experimento em `out/experiments/`, com `_meta.json`, nunca sobrescrito.
- Nenhum número em `analise/` é digitado à mão: sai de script lendo CSV, com a origem.
- Seeds fixas e registradas.
- Não inventar referência nem detalhe de base: "não encontrado" quando for o caso.
- Windows: caminhos com parênteses sempre entre aspas e com `pathlib`.

## Pare e pergunte

Antes de download acima de 20 GB, de base que exija login, de licença restritiva, ou se for
preciso tocar em qualquer arquivo do Artigo3 fora de `artigo-2027/`.

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

Ambiente: `.venv` (Python 3.12, uv). Português nos comentários e na documentação; textos de
`analise/` em parágrafos.
