# CMAP_FTL — Do pixel à trajetória

Contexto espacial e CMAP na classificação de trajetórias de cobertura na Amazônia. Projeto final do Space Tech Bootcamp 2026 (Future Tech Leaders Brasil).

**Apresentação:** <https://vdlucas-queiroz.github.io/CMAP_FTL/apresentacao/>

O projeto classifica as trajetórias de cobertura da terra de Belterra (PA), órbita-ponto 227/062, numa série anual Landsat 5 TM de 2005 a 2010, com o classificador de Máximo a Posteriori Composto (CMAP; Reis et al., 2020). As probabilidades de cada data vêm de quatro classificadores: máxima verossimilhança gaussiana (MaxVer), Random Forest (RF), RF com contexto 5 × 5 e uma rede neural convolucional (CNN) rasa com a mesma janela. Os classificadores são treinados com o TerraClass 2008 e 2010 de todo o Pará, sem nenhuma amostra da área de estudo, e avaliados contra o TerraClass 2008 e 2010 da área de estudo.

**Pergunta de pesquisa:** o contexto espacial da CNN melhora, no CMAP, a acurácia das trajetórias e a detecção do desmatamento?

**Classes:** Floresta (F), Vegetação Secundária (VS), Pastagem (PA) e Agricultura (AG). Só a volta para a Floresta (VS → F, PA → F, AG → F) é proibida; das 4.096 trajetórias possíveis em seis datas, 1.093 são válidas.

## Conteúdo do repositório

| Arquivo ou pasta | Conteúdo |
|---|---|
| `apresentacao/` | Página da apresentação final (HTML estático publicado pelo GitHub Pages) |
| `CMAP_teoria_e_aplicacao.ipynb` | Teoria do CMAP, com exemplo calculado à mão e aplicação numa cena fictícia |
| `0a_aquisicao.ipynb` … `4_cmap.ipynb` | Pipeline do projeto (ordem de execução abaixo) |
| `functions/` | Pacote Python com modelagem estatística, MaxVer, matrizes de transição e CMAP; usado pelo `4a_matrizes_transicao` e pelo `4_cmap` |
| `Proposta_FTL_Vinicius_Queiroz.pdf` | Proposta do projeto |
| `requirements.txt` | Versões dos pacotes usados nos notebooks e no pacote `functions/` |
| `LICENSE` | Licença MIT |

## Dados

Os dados e os resultados não fazem parte do repositório. Os notebooks leem e gravam numa pasta de projeto externa, definida pelo parâmetro `PASTA_PROJETO` (no ambiente original, `C:\cmap_ftl\terraclass`):

```
<PASTA_PROJETO>\
  dados\
    terraclass\        entrada: TerraClass Pará 2008 e 2010 (GeoTIFF + .qml)
    patches_csv\       entrada: CSVs dos patches exportados pelo Google Earth Engine (etapa 0a)
  resultados\
    <nome do notebook>\<nome da variação>\   saídas de cada etapa
```

Além disso, o pipeline usa as seis imagens Landsat 5 da área de estudo já preparadas (`landsat_AAAA.tif` e `mascara_AAAA.tif`; Collection 2, nível 2, seis bandas, grade EPSG:31981), que também não estão no repositório. O `0a` e o `0c` usam a imagem de 2010 para delimitar a área de estudo e definir a grade; o `3_inferencia` lê as seis datas.

Regras seguidas por todos os notebooks:

- cada notebook grava só em `resultados\<nome do notebook>\`;
- nenhum notebook grava em `dados\`;
- as imagens da área de estudo são apenas lidas.

Para rodar em outro computador, ajuste nos parâmetros de cada notebook:

- `PASTA_PROJETO`: pasta de dados e resultados;
- `ARQ_IMAGEM_AREA`, `ARQ_MASCARA_AREA` (`0a`), `ARQ_GRADE` (`0c`) e `PASTA_IMAGENS` (`3_inferencia`): caminho das imagens da área de estudo;
- `PASTA_REPO` (`4a_matrizes_transicao` e `4_cmap`): caminho deste repositório, de onde é importado o pacote `functions`.

## Ordem de execução

| # | Notebook | O que faz | Grava em `resultados\` |
|---|---|---|---|
| 1 | `0a_aquisicao` | Sorteia e exporta os patches de treino pelo Google Earth Engine (CSV) | `0a_aquisicao\` |
| 2 | `0b_conjunto_treino` | CSVs → patches 9 × 9, escala fixa e divisões por órbita-ponto | `0b_conjunto_treino\<NOME_CONJUNTO>\` |
| 3 | `0c_referencia_terraclass` | TerraClass 2008 e 2010 na grade da área de estudo (referência da avaliação) | `0c_referencia_terraclass\` |
| 4 | `1_baselines` | MaxVer, RF e RF com contexto (5 × 5, 7 × 7 e 9 × 9), validação cruzada por órbita-ponto | `1_baselines\<NOME_EXPERIMENTO>\` |
| 5 | `2_treino_cnn` | CNN nas mesmas divisões; um experimento por tamanho de patch (`TAMANHO_PATCH` = 5, 7 ou 9) | `2_treino_cnn\<NOME_EXPERIMENTO>\` |
| 6 | `3_inferencia` | Probabilidades dos quatro classificadores nas seis imagens da área de estudo | `3_inferencia\<NOME_INFERENCIA>\` |
| 7 | `4a_matrizes_transicao` | Matriz de validade 4 × 4, uma por par de datas | `4a_matrizes_transicao\` |
| 8 | `4_cmap` | CMAP e classificação data a data, avaliação contra o TerraClass 2008 e 2010 e confiança das trajetórias | `4_cmap\<NOME_INFERENCIA>\` |

Cada variação (conjunto de bandas, experimento, tamanho de patch) ganha um nome próprio e fica numa subpasta separada. O `3_inferencia` combina um experimento do `1_baselines` com um do `2_treino_cnn`, e o `4_cmap` lê o resultado pelo `NOME_INFERENCIA`. Os resultados apresentados usam `NOME_CONJUNTO = "6bandas"`, `NOME_EXPERIMENTO = "patch5"` e `NOME_INFERENCIA = "6bandas_patch5"`.

No ambiente original (Intel i7-11700, 16 GB de RAM, sem GPU), o `1_baselines` levou cerca de 9 min, o `3_inferencia` cerca de 13 min, o `4a_matrizes_transicao` alguns segundos e o `4_cmap` cerca de 2 min.

## Antes de rodar o 0a_aquisicao

1. Baixe o TerraClass do Pará, 2008 e 2010, em formato matricial (raster), em terraclass.gov.br → Downloads → Amazônia → Unidade da Federação: Pará. Extraia em `dados\terraclass\`; o nome de cada arquivo precisa conter o ano, e o `.qml` fica junto do GeoTIFF.
2. Registre um projeto do Google Cloud no Earth Engine para uso não comercial e instale o `earthengine-api` no ambiente (`conda install -c conda-forge earthengine-api`). Preencha `PROJETO_GEE` no notebook.
3. Rode a seção 1 (mostra a legenda e o no-data) e só então suba os dois GeoTIFF ao GEE como assets `terraclass_pa_2008` e `terraclass_pa_2010`, com Pyramiding policy = MODE e o no-data informado.

## Dependências

Ambiente conda com Python 3.12. As versões usadas estão em `requirements.txt` (`pip install -r requirements.txt`): `numpy`, `pandas`, `matplotlib`, `scipy`, `rasterio`, `geopandas` (importado pelo pacote `functions`), `scikit-learn`, `torch` (PyTorch, executado em CPU) e, só no `0a`, `earthengine-api`.

## Apresentação

A pasta `apresentacao/` contém a página final (`index.html`, `dados/` e `img/`), publicada pelo GitHub Pages em <https://vdlucas-queiroz.github.io/CMAP_FTL/apresentacao/>. Para abrir localmente, rode `python -m http.server 8000` dentro de `apresentacao/` e acesse `http://localhost:8000`; aberta direto do disco, a página não carrega o explorador de pixels.

## Referências

- REIS, M. S.; DUTRA, L. V.; ESCADA, M. I. S.; SANT'ANNA, S. J. S. Avoiding invalid transitions in land cover trajectory classification with a compound maximum a posteriori approach. *IEEE Access*, v. 8, p. 98787–98799, 2020. doi:10.1109/ACCESS.2020.2997019.
- INPE; EMBRAPA. *TerraClass Amazônia*: mapas de uso e cobertura da terra de 2008 e 2010. terraclass.gov.br.
- U.S. GEOLOGICAL SURVEY. *Landsat 4–5 TM Collection 2 Level-2 Science Products*.

## Autoria e licença

Autor: Vinícius D'Lucas Bezerra e Queiroz.

A página de apresentação e a documentação dos códigos foram confeccionadas com apoio de um assistente de IA (Claude - Opus 5.5, da Anthropic); métodos, resultados e conclusões são de responsabilidade do autor.

Licença MIT. Veja `LICENSE`.
