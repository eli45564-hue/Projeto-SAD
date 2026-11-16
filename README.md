## Dashboard de análise no Power BI

Depois que o Data Mart é carregado pelo ETL, o dashboard no **Power BI Desktop** lê o banco
`dm_anp_pernambuco` e apresenta a **análise dos preços dos combustíveis no estado de Pernambuco**
(períodos 2020.1 a 2026.1).

```text
ANP (CSV) → Pentaho (ETL) → PostgreSQL (Data Mart) → Power BI (dashboard)
```

Arquivos na pasta [`powerbi/`](powerbi/):

- `consulta OLAP.pbix`: relatório completo (modelo, medidas e visuais)
- `consulta OLAP.pbit`: modelo do relatório, sem os dados
- imagens e PDF do dashboard

### Conexão e modelo

- Fonte: **PostgreSQL**, servidor `localhost`, banco `dm_anp_pernambuco` (container do `docker-compose.yml`).
- Modo de importação das tabelas do Data Mart.
- Relacionamentos (esquema estrela, um-para-muitos, direção única):

| Fato                          | Dimensão          | Chave            |
|-------------------------------|-------------------|------------------|
| `fato_preco_combustivel`      | `dim_tempo`       | `id_tempo`       |
| `fato_preco_combustivel`      | `dim_local`       | `id_local`       |
| `fato_preco_combustivel`      | `dim_combustivel` | `id_combustivel` |
| `fato_preco_combustivel`      | `dim_bandeira`    | `id_bandeira`    |

### Medidas (DAX)

```dax
Preco Medio Venda = AVERAGE('public fato_preco_combustivel'[valor_venda])
Maior Preco       = MAX('public fato_preco_combustivel'[valor_venda])
Menor Preco       = MIN('public fato_preco_combustivel'[valor_venda])

Variacao Preco % =
VAR AnoAtual = MAX('public dim_tempo'[ano])
VAR PrecoAtual = [Preco Medio Venda]
VAR PrecoAnterior =
    CALCULATE(
        [Preco Medio Venda],
        REMOVEFILTERS('public dim_tempo'),
        'public dim_tempo'[ano] = AnoAtual - 1
    )
RETURN
    IF(ISBLANK(PrecoAnterior), BLANK(),
       DIVIDE(PrecoAtual - PrecoAnterior, PrecoAnterior))
```

A variação percentual compara o preço médio do ano selecionado com o do ano anterior.

### Visuais e filtros

- **Segmentações:** ano, município e combustível/produto.
- **Cartões:** maior preço, menor preço, preço médio e variação percentual.
- **Gráficos:** preço médio ao longo do tempo, preço médio por combustível,
  preço mínimo e máximo por bandeira e preço médio por município.

### Como abrir o dashboard

1. Suba o banco e execute o ETL conforme o passo a passo acima.
2. Abra `powerbi/consulta OLAP.pbix` no Power BI Desktop.
3. Se pedir, informe o servidor (`localhost`), o banco (`dm_anp_pernambuco`) e o usuário e senha definidos no `docker-compose.yml`.
4. Clique em **Atualizar** na Página Inicial.

### Autoria

| Parte                                          | Responsável                     |
|------------------------------------------------|---------------------------------|
| ETL no Pentaho e Data Mart no PostgreSQL       | Rodrigo Santos                  |
| Modelagem no Power BI, medidas DAX e dashboard | Samara Lima   |
