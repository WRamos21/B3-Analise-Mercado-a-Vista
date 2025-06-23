# Analise mercado à vista B3

Análise exploratória de dados históricos do mercado à vista da B3 (2023-2025). O objetivo é investigar a volatilidade dos ativos, seu volume negociado e a correlação entre essas variáveis ao longo do tempo, utilizando SQL para extrair insights relevantes sobre comportamento de mercado.

## Problema

Investidores e analistas enfrentam o desafio de identificar ativos com bom equilíbrio entre risco e liquidez. Volatilidade excessiva pode indicar instabilidade nos preços, enquanto alto volume pode refletir interesse do mercado e maior facilidade de compra/venda. Compreender a relação entre essas variáveis permite decisões mais estratégicas, principalmente no mercado de FIIs.

## Como a análise foi feita

 Os dados utilizados neste projeto são públicos e foram obtidos diretamente do site da B3:

- [Histórico de Cotações do Mercado à Vista – B3](https://www.b3.com.br/pt_br/market-data-e-indices/servicos-de-dados/market-data/historico/)

As cotações históricas foram coletadas para os anos de **2023**, **2024** e **2025**, abrangendo todos os pregões disponíveis para ações e fundos imobiliários listados no mercado à vista da B3.

Cada arquivo contém informações detalhadas como:  
**nome da empresa**, **preço máximo/mínimo/último**, **volume negociado**, **quantidade de negócios**, entre outros dados essenciais para análise de comportamento de ativos.

### Consulta SQL

```sql
WITH todos_anos AS (
  SELECT  nome_empresa, preco_maximo, preco_minimo, volume_negociado
  FROM b3_cotacoes_2023 
  UNION ALL
  SELECT  nome_empresa, preco_maximo, preco_minimo, volume_negociado
  FROM b3_cotacoes_2024 
  UNION ALL
  SELECT  nome_empresa, preco_maximo, preco_minimo, volume_negociado
  FROM b3_cotacoes_2025), 
analise AS(
  SELECT 
    nome_empresa,
    COUNT(*) AS dias_negociados,
    AVG(preco_maximo - preco_minimo) AS volatilidade,
    SUM(volume_negociado) AS volume_total,
    CORR(preco_maximo - preco_minimo, volume_negociado) AS correlacao_volume_volatilidade
  FROM todos_anos
  GROUP BY nome_empresa
)
SELECT * FROM analise
WHERE dias_negociados >=200    
ORDER BY volatilidade DESC
LIMIT 5;
```


### Justificativas técnicas

#### Uso de `WITH` (CTE - Common Table Expression)

Utilizei `WITH` para organizar a consulta em **etapas lógicas e reutilizáveis**, facilitando a leitura e a manutenção do código. As CTEs permitem separar o carregamento e união dos dados (`todos_anos`) da parte analítica propriamente dita (`analise`), mantendo o SQL limpo e modular.

#### CTE `todos_anos`

Essa CTE junta os dados de 2023, 2024 e 2025 com `UNION ALL`, consolidando os registros de cotações históricas em um único conjunto. Essa união preserva **todas as linhas** dos três anos (sem eliminação de duplicatas), o que é essencial para não distorcer a contagem de dias negociados e os cálculos de médias.

#### CTE `analise`

A CTE `analise` foi criada para **agrupar os dados por fundo** e calcular os seguintes indicadores:
- **`dias_negociados`**: mede a consistência de presença do fundo no mercado.
- **`volatilidade`**: média da variação diária de preço (preço máximo - preço mínimo).
- **`volume_total`**: total movimentado financeiramente no período.
- **`correlacao_volume_volatilidade`**: mede o grau de associação entre liquidez e volatilidade.

Essa separação facilita a reutilização da lógica agregada, filtragens e ordenações na consulta principal.

#### Filtro `dias_negociados >= 200`

Esse filtro garante que os fundos considerados tenham **presença significativa nos dados** (aproximadamente 200 pregões = quase 1 ano útil completo). Isso evita distorções estatísticas causadas por fundos recém-lançados ou com baixa liquidez, cujos indicadores seriam pouco representativos.

## Resultados

| Fundo        | Dias Negociados | Volatilidade Média | Volume Total       | Correlação Volume × Volatilidade |
|--------------|-----------------|--------------------|--------------------|----------------------------------|
| FII D PEDRO  | 615             | 40.87              | R$ 11.78 bilhões   | 0.15                             |
| FII ALMIRANT | 589             | 34.49              | R$ 5.31 bilhões    | 0.51                             |
| FII BRIO II  | 310             | 30.64              | R$ 659 milhões     | 0.31                             |
| FII BB PROGR | 615             | 30.08              | R$ 13.92 bilhões   | 0.62                             |
| FII FLORIPA  | 501             | 28.16              | R$ 1.55 bilhões    | 0.28                             |

Liquidez e volatilidade caminham juntas em boa parte dos FIIs analisados, revelando que fundos com maior atividade de mercado tendem a apresentar maior instabilidade nos preços — um dado crucial para estratégias de curto prazo.

## Tecnologias utilizadas

- SQL (consultas avançadas e CTEs)
- Data.world para carregamento e consulta de dados
- Dados públicos da B3 (Mercado à Vista)

## Próximos passos

- Análise por categoria de FII (papel, tijolo, híbrido)
- Inclusão de indicadores como dividend yield e retorno acumulado
- Visualizações com gráficos interativos
- Comparação entre FIIs e ações comuns

## Considerações finais

Acesse: data.world/wramos/mapa-inteligente-dos-fundos-brasileiros-fiis-e-fidc  
Se ficou interessado e quiser explorar ou modificar a análise. 


## Fonte dos dados

- [Histórico de Cotações do Mercado à Vista – B3](https://www.b3.com.br/pt_br/market-data-e-indices/servicos-de-dados/market-data/historico/)
---

Este projeto tem como objetivo demonstrar habilidades práticas em análise de dados financeiros com foco no mercado brasileiro, usando apenas SQL e dados públicos para gerar insights reais e aplicáveis.
