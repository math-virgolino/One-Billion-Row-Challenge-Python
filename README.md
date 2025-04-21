# Um Bilhão de Linhas: Desafio de Processamento de Dados com Python

## Introdução

O objetivo deste projeto é demonstrar como processar eficientemente um arquivo de dados massivo contendo 1 bilhão de linhas (~14GB), especificamente para calcular estatísticas (Incluindo agregação e ordenação que são operações pesadas) utilizando Python. 

Este desafio foi inspirado no [The One Billion Row Challenge](https://github.com/gunnarmorling/1brc), originalmente proposto para Java.

O arquivo de dados consiste em medições de temperatura de várias estações meteorológicas. Cada registro segue o formato `<string: nome da estação>;<double: medição>`, com a temperatura sendo apresentada com precisão de uma casa decimal.

Aqui estão dez linhas de exemplo do arquivo:

```
Hamburg;12.0
Bulawayo;8.9
Palembang;38.8
St. Johns;15.2
Cracow;12.6
Bridgetown;26.9
Istanbul;6.2
Roseau;34.4
Conakry;31.2
Istanbul;23.0
```

O desafio é desenvolver um programa Python capaz de ler esse arquivo e calcular a temperatura mínima, média (arredondada para uma casa decimal) e máxima para cada estação, exibindo os resultados em uma tabela ordenada por nome da estação.

| station      | min_temperature | mean_temperature | max_temperature |
|--------------|-----------------|------------------|-----------------|
| Abha         | -31.1           | 18.0             | 66.5            |
| Abidjan      | -25.9           | 26.0             | 74.6            |
| Albuquerque  | -35.0           | 14.0             | 61.9            |
| Alexandra    | -40.1           | 11.0             | 67.9            |
| ...          | ...             | ...              | ...             |
| Yangon       | -23.6           | 27.5             | 77.3            |
| Yaoundé      | -26.2           | 23.8             | 73.4            |
| Yellowknife  | -53.4           | -4.3             | 46.7            |
| Zürich       | -42.0           | 9.3              | 63.6            |
| Ürümqi       | -42.1           | 7.4              | 56.7            |
| İzmir        | -34.4           | 17.9             | 67.9            |

Este desafio será em torno das bibliotecas:

* Polars: `0.20.3`
* DuckDB: `0.10.0`
* Dask[complete]: `^2024.2.0`

## Resultados dos Testes

Os testes foram realizados em um laptop equipado com um processador Intel i5 Gen8 e 8Gb RAM. As implementações utilizaram abordagens puramente Python, Pandas, Dask, Polars e DuckDB. Os resultados de tempo de execução para processar o arquivo de 1 bilhão de linhas são apresentados abaixo:

| Implementação | Tempo |
| --- | --- |
| Python | 48 minutos |
| Python + Pandas | 759.77 sec |
| Python + Dask | 155.62 sec  |
| Python + Polars | 77.74 sec |
| Python + Duckdb | 52.31 sec |

## Conclusão

Este desafio destacou claramente a eficácia de diversas bibliotecas Python na manipulação de grandes volumes de dados. Métodos tradicionais Python puro (48 minutos) e até mesmo o Pandas (12 minutos) demandaram uma série de táticas para implementar o processamento em "lotes", enquanto bibliotecas como Dask, Polars e DuckDB provaram ser excepcionalmente eficazes, requerendo menos linhas de código e  sua capacidade inerente de distribuir os dados em "lotes em streaming" de maneira mais eficiente. O DuckDB se sobressaiu, alcançando o menor tempo de execução graças à sua estratégia de execução e processamento de dados.

Esses resultados enfatizam a importância de selecionar a ferramenta adequada para análise de dados em larga escala, demonstrando que Python, com as bibliotecas certas, é uma escolha poderosa para enfrentar desafios de big data.

## Como Executar

Para executar este projeto e reproduzir os resultados:

1. Clone esse repositório
2. Definir a versao do Python usando o `pyenv local 3.12.1`
2. `poetry env use 3.12.1`, `poetry install --no-root` e `poetry lock --no-update`
3. Execute o comando `python src/create_measurements.py` para gerar o arquivo de teste
4. Tenha paciência e vá fazer um café, vai demorar uns 15 minutos para gerar o arquivo
5. Execute os scripts `poetry run python src/using_python.py`, `poetry run python src/using_pandas.py`, `poetry run python src/using_dask.py`, `poetry run python src/using_polars.py` e `poetry run python src/using_duckdb.py` através de um terminal ou ambiente de desenvolvimento que suporte Python.

Este projeto destaca a versatilidade do ecossistema Python para tarefas de processamento de dados, oferecendo valiosas lições sobre escolha de ferramentas para análises em grande escala.

## Características de Cada Biblioteca

### Pandas
Como funciona:
* Pandas usa NumPy por baixo dos panos, armazenando os dados em arrays na RAM
* Cada coluna é um Series, que é basicamente um array tipado
* Todas as operações são monothreaded (rodam em um núcleo apenas)
* Não faz lazy evaluation: tudo roda imediatamente.

Ideal para: 
* Pequenos dados (cabem na RAM)

Motivo:
* Simples, leve e direto. É perfeito para análises exploratórias, protótipos rápidos e pequenos datasets
* Sua API é muito madura e intuitiva, o que facilita a vida no dia a dia

Limite técnico:
* Dependendo de cada máquina, mas se você tem 8Gb RAM e se seu DataFrame tem ~500k a 1M de linhas e você tenta um groupby, ele pode travar ou estourar memória. Observação: Lembrando que também existe a questão de quantidades de colunas do DataFrame

### DuckDB
Como funciona:
* É um motor de banco de dados OLAP embutido, otimizado para consultas analíticas colunarizadas
* Lê e escreve formato Parquet diretamente, com scan incremental (sem carregar tudo na RAM)
* Usa multi-threading com query planner e otimizador de execução como bancos de dados sérios
* Suporta SQL ANSI, inclusive subqueries, CTEs, joins complexos etc

Ideal para:
* OLAP com SQL, e dados grandes que não cabem na memória

Motivo:
* Pode rodar queries em datasets de centenas de milhões de linhas sem carregar tudo na RAM
* Excelente para times acostumados com SQL, e quando o dado está armazenado em arquivos como .parquet, .csv, ou até no Amazon S3

Limite técnico:
* Não tem API pandas-style. É SQL puro (ou .df() para converter o resultado), então pode não ser ideal se você quer manipular os dados linha por linha ou fazer coisas muito customizadas

### Polars
Como funciona:
* Feito em Rust, super rápido e seguro, com bindings para Python
* Usa executores vetorizados (colunas processadas em blocos) e multi-threading nativo
* Pode operar em dois modos:
** Eager (modo imediato) — parecido com Pandas
**Lazy (modo preguiçoso) — compila todo o pipeline de operações e executa só no final, otimizando tudo (como um query planner)

Ideal para
* Grandes dados em memória e processamento super rápido

Motivo:
* É muito mais eficiente que Pandas, especialmente com filtros, joins e agregações complexas
* Lazy evaluation permite otimizações automáticas (ex: elimina colunas não usadas)

Limite técnico:
* Ainda não tão “amigável” quanto Pandas para quem está começando.
* Algumas operações ainda estão sendo desenvolvidas (tipo merge complexos ou funções personalizadas)

### Dask
Como funciona:
* Cria gráficos de tarefas paralelas (DAGs) para operações com dados
* Divide grandes DataFrames em partições menores e processa cada uma em paralelo, podendo escalar para clusters
* API é parecida com Pandas, mas as operações são lazy, ou seja, não executam até o .compute()

Ideal para: 
* Escalar análise de dados para além da RAM, ou para múltiplas máquinas

Motivo:
* Você pode trabalhar com dados maiores que a RAM, já que ele processa em blocos
* Também pode conectar com Hadoop/Spark, usar com AWS/GCP, etc
* Ótimo para pipelines e automações

Limite técnico:
* Nem toda operação é compatível com Pandas 100%
* Performance pode ser menor que Polars ou DuckDB em casos simples
* Curva de aprendizado maior