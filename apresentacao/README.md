# Do pixel à trajetória — página de apresentação

Projeto final do Space Tech Bootcamp 2026 (Future Tech Leaders Brasil).

Página estática para o GitHub Pages: `index.html` + pastas `dados/` e `img/`.

## Como publicar

1. Copie `index.html`, `dados/` e `img/` para a raiz do repositório (ou para a pasta `docs/`).
2. No GitHub: *Settings → Pages → Build and deployment → Deploy from a branch*, escolha a branch e a pasta (`/root` ou `/docs`).
3. A página fica em `https://<usuário>.github.io/<repositório>/`.

## Ver localmente

O explorador de pixels lê os arquivos de `dados/` com `fetch`, que não funciona ao abrir o arquivo direto (`file://`). Use um servidor local:

```
python -m http.server 8000
```

e abra `http://localhost:8000`.

## Conteúdo

| Arquivo | Conteúdo |
|---|---|
| `dados/<R>_base.txt` | TerraClass 2008 e 2010 no recorte R (A ou B), 360 × 360 pixels |
| `dados/<R>_<clf>.txt` | data a data (6 datas), CMAP (6 datas), confiança da trajetória (0–200; 255 = sem valor) e probabilidades de VS, PA e AG por data (%, F = 100 − soma); uint8, gzip, base64 |
| `img/<R>_<ano>.jpg` | Landsat 5 em falsa-cor (B5, B4, B3), mesmo realce nas seis datas |
| `img/invalidas_<clf>.png` | trajetórias inválidas na classificação data a data, cena inteira |
| `img/cena_2010.jpg` | cena inteira em 2010 |
| `img/para_patches.png` | patches de treino no Pará |

Recorte A: linhas 60–419, colunas 390–749 da grade de 750 × 1.200 pixels. Recorte B: linhas 600–959, colunas 260–619.
Classificadores: MaxVer, RF, RFc (RF com contexto 5 × 5) e CNN (patch 5 × 5).
