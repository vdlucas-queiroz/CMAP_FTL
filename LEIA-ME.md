# CMAP com TerraClass — Pará

Versão redesenhada do projeto: um modelo de cada classificador (MaxVer, RF, RF com contexto, CNN) treinado com o TerraClass 2008 e 2010 do Pará, aplicado às seis imagens da área de estudo (Belterra, 227/062, 2005–2010) e avaliado contra o TerraClass 2008 e 2010 da área de estudo.

Plano completo: doc "Plano prático — CMAP com TerraClass no Pará" (https://claude.ai/artifact/USqbxe7ACWZxB96KR236jD).

## Regras da pasta

- Tudo o que os notebooks daqui gravam fica em `resultados\<nome do notebook>\`.
- `dados\` guarda só entradas baixadas (TerraClass e CSVs do GEE). Nenhum notebook grava em `dados\`.
- Do projeto atual (`C:\cmap_ftl\`), os notebooks daqui só **leem** `dados_preparados\imagens\` (as seis imagens e máscaras da área de estudo). Nada é gravado fora de `terraclass\`.
- `relatorio\` guarda o relatório técnico; ele não é saída de notebook.
- **Antes de eu atualizar um notebook, feche a aba dele no VS Code sem salvar.** Uma aba aberta com a versão antiga, se salva, sobrescreve a nova (foi o que aconteceu com a v1 do 0a).
- Notebooks com o mesmo nome dos atuais (`1_baselines` etc.) serão versões adaptadas; os originais em `C:\cmap_ftl\` não são alterados.

## Estrutura

```
terraclass\
  LEIA-ME.md
  0a_aquisicao.ipynb               versão 2 (sorteio em qualquer posição + rótulo da vizinhança)
  0b_conjunto_treino.ipynb         CSVs -> patches_9x9.npy + metadados (formato do 0_sample_design)
  0c_referencia_terraclass.ipynb   TerraClass 2008 e 2010 na grade da área de estudo (referência da avaliação)
  1_baselines.ipynb                MaxVer, RF e RF com contexto (5x5, 7x7, 9x9) nas divisões por órbita-ponto
  2_treino_cnn.ipynb               CNN nas mesmas divisões; um experimento por tamanho de patch (5, 7, 9)
  3_inferencia.ipynb               probabilidades dos 4 classificadores nas seis imagens da área de estudo
  4a_matrizes_transicao.ipynb      matriz de validade 4x4 (só a volta para F é proibida), uma por par de datas
  4_cmap.ipynb                     CMAP e data a data; avaliação contra o TerraClass 2008 e 2010
  dados\
    terraclass\                    entrada: TerraClass Pará 2008 e 2010 (GeoTIFF + .qml)
    patches_csv\                   entrada: CSVs da v2, baixados da pasta cmap_terraclass_patches_v2 do Drive
    patches_csv_v1_interior\       arquivo: 113 CSVs da v1 (interior 5x5), fora do treino principal
  resultados\
    0a_aquisicao\                  saídas do 0a (legenda, zona, cenas, conferência; tarefas e resumo da v2)
      v1_interior\                 arquivo: notebook executado, tarefas, consumo, resumo e mapa da v1
    0b_conjunto_treino\6bandas\    saídas do 0b (uma subpasta por NOME_CONJUNTO)
    0b_conjunto_treino\6bandas_v1_270m\   arquivo: 1.ª execução do 0b (270 m também na mesma cena), com o notebook executado
    0c_referencia_terraclass\      referência 2008/2010, pureza, bordas, transições, concordância com a dissertação
    1_baselines\6bandas_experimento_inicial\   saídas do 1_baselines (uma subpasta por NOME_EXPERIMENTO)
    2_treino_cnn\patch5\ (patch7\, patch9\)   saídas do 2_treino_cnn (uma subpasta por NOME_EXPERIMENTO)
    3_inferencia\6bandas_patch5\   saídas do 3_inferencia (uma subpasta por NOME_INFERENCIA)
    4a_matrizes_transicao\         matrizes de transição e tabela de trajetórias
    4_cmap\6bandas_patch5\         CMAP, data a data, avaliação e figuras (uma subpasta por NOME_INFERENCIA)
  relatorio\
    relatorio_tecnico_cmap_terraclass.docx   relatório técnico (base para o artigo), atualizado a cada etapa
    figuras\                       figuras do relatório
    fonte\                         scripts que geram o relatório e as figuras
```

## Ordem de execução

| # | Notebook | Situação | O que faz | Grava em |
|---|---|---|---|---|
| 1 | `0a_aquisicao` | v2 executada (v1 arquivada) | Patches de treino pelo GEE (CSV) | `resultados\0a_aquisicao\` |
| 2 | `0b_conjunto_treino` | executado (6bandas: 100.643 patches; 1.ª execução arquivada) | CSVs → patches 9×9, escala fixa, folds por órbita-ponto | `resultados\0b_conjunto_treino\<NOME_CONJUNTO>\` |
| 3 | `0c_referencia_terraclass` | executado (comparação com a dissertação desligada de propósito: `COMPARAR_DISSERTACAO = False`) | TerraClass 2008 e 2010 na grade da área de estudo | `resultados\0c_referencia_terraclass\` |
| 4 | `1_baselines` | executado (6bandas_experimento_inicial, ~9 min) | MaxVer, RF e RF com contexto | `resultados\1_baselines\<NOME_EXPERIMENTO>\` |
| 5 | `2_treino_cnn` | patch5 executado; patch7 e patch9 a rodar (só a repetição 0) | CNN | `resultados\2_treino_cnn\<NOME_EXPERIMENTO>\` |
| 6 | `3_inferencia` | executado (6bandas_patch5, ~13 min) | Probabilidades nas seis imagens | `resultados\3_inferencia\<NOME_INFERENCIA>\` |
| 7 | `4a_matrizes_transicao` | pronto (segundos) | Matriz 4×4 (só a volta para F é proibida) | `resultados\4a_matrizes_transicao\` |
| 8 | `4_cmap` | pronto (rodar depois do 4a; ~2 min) | CMAP e avaliação contra o TerraClass | `resultados\4_cmap\<NOME_INFERENCIA>\` |

Os notebooks pendentes entram nesta pasta à medida que forem escritos.

## Antes de rodar o 0a_aquisicao

1. Baixar o TerraClass do Pará, 2008 e 2010, formato Matricial (Raster), em terraclass.gov.br → Downloads → Amazônia → Unidade da Federação: Pará. Extrair em `dados\terraclass\`; o nome de cada arquivo precisa conter o ano, e o `.qml` fica junto do GeoTIFF.
2. Registrar um projeto do Google Cloud no Earth Engine para uso não comercial e instalar `earthengine-api` no ambiente `cmap` (`conda install -c conda-forge earthengine-api`). Preencher `PROJETO_GEE` no notebook.
3. Rodar a seção 1 (mostra a legenda e o no-data) e só então subir os dois GeoTIFF ao GEE como assets `terraclass_pa_2008` e `terraclass_pa_2010`, com Pyramiding policy = MODE e o no-data informado.
