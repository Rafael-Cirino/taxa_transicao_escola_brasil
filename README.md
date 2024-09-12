```python
import re

import pandas as pd
import seaborn as sns
import geopandas as gpd
from matplotlib import pyplot as plt

from plotnine import *
from plotnine.data import *
from plotnine.labels import *
from plotnine.themes import *
from mizani.formatters import label_number
```

# Projeto 1 - IA376 - Unicamp

# Análise da taxa de transição das escolas brasileiras 

Esse notebook apresenta uma análise sobre a progressão escolar dos estudantes brasileiros no período de 2008 até 2021 utilizando o indicador educacional **taxa de transição**, que tem como por objetivo trazer informações sobre a trajetória dos estudantes do fundamental até o ensino médio.

## O que é taxa de transição

As taxas de rendimento escolar são informações produzidas anualmente pelo Instituto Nacional de Estudos e Pesquisas Educacionais Anísio Teixeira (Inep), por meio dos dados coletados pelo Censo Escolar da Educação Básica, e são fundamentais para a verificação e o acompanhamento dos dados da escola e do município. Além disso, as taxas de rendimento são variáveis incorporadas ao cálculo do Índice de Desenvolvimento da Educação Básica (Ideb), indicador de qualidade educacional produzido e divulgado a cada dois anos pelo Inep, que congrega as informações de desempenho dos estudantes nos testes padronizados do Sistema de Avaliação da Educação Básica (Saeb) com as informações de rendimento escolar (aprovação).

- **Taxa de promocao:** É a proporção de alunos da matrícula total na série k, no ano t, que são aprovados.
- **Taxa de evasao:**  É a proporção de alunos da matrícula total na série k, no ano t, que abandonaram a escola.
- **Taxa de repetencia:** É a proporção de alunos da matrícula total na série k, no ano t, que são reprovados.


Fonte: [Indicadores de Rendimento Escolar e Fluxo Escolar](https://professor.ufop.br/sites/default/files/danielmatos/files/indicadores_confeccionados_com_dados_do_inep.pdf)

Fonte: [Taxas de Rendimento Escolar](https://www.gov.br/inep/pt-br/centrais-de-conteudo/acervo-linha-editorial/publicacoes-institucionais/estatisticas-e-indicadores-educacionais/taxas-de-rendimento-escolar#:~:text=As%20taxas%20de%20rendimento%20escolar,da%20escola%20e%20do%20munic%C3%ADpio.)

## Dados

A fonte dos dados é o [INEP](https://www.gov.br/inep/pt-br/acesso-a-informacao/dados-abertos/indicadores-educacionais/taxas-de-rendimento-escolar), e foram obtidos através do [site base de dados](https://basedosdados.org/dataset/63f1218f-c446-4835-b746-f109a338e3a1?table=23c83e1b-ba8e-4d20-b3bb-11ef7bc4ab08).

O dataset contém as seguintes informações entre os anos de **2008 a 2021**:
- As taxas estão apresentadas por estado
- Rede
    - Pública
    - Privada
- Localização: área rural ou urbana
- Taxas: 
    - Promocao
    - Repetencia
    - Evasao
    - Migracao


## Perguntas

A seguir estão alguns questionamentos que serão respondidos durante a analise. Eles apresentam perguntas importantes que visam propor analises sobre a evolução das taxas de trasição de ensino no país.

> Progressão, repetencia e evasão ao longo de todos os anos

> A progressão é maior na área rural ou urbana?

> Qual **estado** tem a maior taxa de progressão e evasao ao longo dos anos?

> Qual **região** tem a maior taxa de progressão e evasao ao longo dos anos?

> Como evouluiu as taxas de promoção, evasão e repetência no primeiro ano de pandemia

> Como os estudantes brasileiros estão progredindo de ano?

## Hipótese

Além das perguntas, ao final da análise será apresentado a resposta para seguinte hipótese:

> As escolas públicas possuem maior taxa de progressão do que as privadas

Essa hipótese surge a partir da chamada [progressão continuada](https://www.bbc.com/portuguese/articles/c72y0zqnvk1o) presente na rede pública em alguns estados do país. Essa regra estabelecida por algumas secretarias de ensino, define que alunos com desempenho abaixo do esperado vão receber aulas de reforço ao invés de serem reprovados, isso a cada ciclo de 3 anos. Será q

Estados que possuem progressão continuada:

![image.png](taxa_transicao_analise_files/image.png)

Fonte: [progressão continuada](https://www.bbc.com/portuguese/articles/c72y0zqnvk1o)

## Ferramentas utilizadas

Para ser possivel analisar esses dados foram utilizadas as seguintes ferramentas:

- Leitura e organização dos dados: 
    - Pandas: Biblioteca utilizada para carregar, organizar e limpar os dados
- Visualização de dados:
    - Plotnine: Foi utilizada para plotar a maioria dos gráficos deste relatório, pois é fácil de utilizar, configurar e coloca os gráficos em escala por padrão.
    - Geopandas: Biblioteca utilizada para carregar dados geográficos do estados do país e assim ser possível plotar um mapa.
    - matplotlib: Utilizada para plotar os mapas.
    - Seaborn: Neste projeto foi utilizado a paleta de cores dessa biblioteca
- Formatação e filtragem de textos e números
    - Regex: Utilizado para filtrar alguns textos, auxiliando na organização dos dados.
    - Mizanni formatter: Utilizado para formatar as escalas dos gráficos, inserindo simbolos como %.

## Importação e limpeza dos dados

O primeiro passo é carregar os dados e analisar através da função info se há informações faltantes e se os tipos das colunas estão corretos


```python
df_raw = pd.read_csv("data/inep_transicao_estado.csv.gz")

df_raw.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 1890 entries, 0 to 1889
    Data columns (total 68 columns):
     #   Column                              Non-Null Count  Dtype  
    ---  ------                              --------------  -----  
     0   ano                                 1890 non-null   int64  
     1   sigla_uf                            1890 non-null   object 
     2   localizacao                         1890 non-null   object 
     3   rede                                1890 non-null   object 
     4   taxa_promocao_ef                    1890 non-null   float64
     5   taxa_promocao_ef_anos_iniciais      1890 non-null   float64
     6   taxa_promocao_ef_anos_finais        1890 non-null   float64
     7   taxa_promocao_ef_1_ano              1890 non-null   float64
     8   taxa_promocao_ef_2_ano              1890 non-null   float64
     9   taxa_promocao_ef_3_ano              1890 non-null   float64
     10  taxa_promocao_ef_4_ano              1890 non-null   float64
     11  taxa_promocao_ef_5_ano              1890 non-null   float64
     12  taxa_promocao_ef_6_ano              1890 non-null   float64
     13  taxa_promocao_ef_7_ano              1890 non-null   float64
     14  taxa_promocao_ef_8_ano              1890 non-null   float64
     15  taxa_promocao_ef_9_ano              1890 non-null   float64
     16  taxa_promocao_em                    1890 non-null   float64
     17  taxa_promocao_em_1_ano              1890 non-null   float64
     18  taxa_promocao_em_2_ano              1890 non-null   float64
     19  taxa_promocao_em_3_ano              1890 non-null   float64
     20  taxa_repetencia_ef                  1890 non-null   float64
     21  taxa_repetencia_ef_anos_iniciais    1890 non-null   float64
     22  taxa_repetencia_ef_anos_finais      1890 non-null   float64
     23  taxa_repetencia_ef_1_ano            1890 non-null   float64
     24  taxa_repetencia_ef_2_ano            1890 non-null   float64
     25  taxa_repetencia_ef_3_ano            1890 non-null   float64
     26  taxa_repetencia_ef_4_ano            1890 non-null   float64
     27  taxa_repetencia_ef_5_ano            1890 non-null   float64
     28  taxa_repetencia_ef_6_ano            1890 non-null   float64
     29  taxa_repetencia_ef_7_ano            1890 non-null   float64
     30  taxa_repetencia_ef_8_ano            1890 non-null   float64
     31  taxa_repetencia_ef_9_ano            1890 non-null   float64
     32  taxa_repetencia_em                  1890 non-null   float64
     33  taxa_repetencia_em_1_ano            1890 non-null   float64
     34  taxa_repetencia_em_2_ano            1890 non-null   float64
     35  taxa_repetencia_em_3_ano            1890 non-null   float64
     36  taxa_evasao_ef                      1890 non-null   float64
     37  taxa_evasao_ef_anos_iniciais        1890 non-null   float64
     38  taxa_evasao_ef_anos_finais          1890 non-null   float64
     39  taxa_evasao_ef_1_ano                1890 non-null   float64
     40  taxa_evasao_ef_2_ano                1890 non-null   float64
     41  taxa_evasao_ef_3_ano                1890 non-null   float64
     42  taxa_evasao_ef_4_ano                1890 non-null   float64
     43  taxa_evasao_ef_5_ano                1890 non-null   float64
     44  taxa_evasao_ef_6_ano                1890 non-null   float64
     45  taxa_evasao_ef_7_ano                1890 non-null   float64
     46  taxa_evasao_ef_8_ano                1890 non-null   float64
     47  taxa_evasao_ef_9_ano                1890 non-null   float64
     48  taxa_evasao_em                      1890 non-null   float64
     49  taxa_evasao_em_1_ano                1890 non-null   float64
     50  taxa_evasao_em_2_ano                1890 non-null   float64
     51  taxa_evasao_em_3_ano                1890 non-null   float64
     52  taxa_migracao_eja_ef                1890 non-null   float64
     53  taxa_migracao_eja_ef_anos_iniciais  1890 non-null   float64
     54  taxa_migracao_eja_ef_anos_finais    1890 non-null   float64
     55  taxa_migracao_eja_ef_1_ano          1890 non-null   float64
     56  taxa_migracao_eja_ef_2_ano          1890 non-null   float64
     57  taxa_migracao_eja_ef_3_ano          1890 non-null   float64
     58  taxa_migracao_eja_ef_4_ano          1890 non-null   float64
     59  taxa_migracao_eja_ef_5_ano          1890 non-null   float64
     60  taxa_migracao_eja_ef_6_ano          1890 non-null   float64
     61  taxa_migracao_eja_ef_7_ano          1890 non-null   float64
     62  taxa_migracao_eja_ef_8_ano          1890 non-null   float64
     63  taxa_migracao_eja_ef_9_ano          1890 non-null   float64
     64  taxa_migracao_eja_em                1890 non-null   float64
     65  taxa_migracao_eja_em_1_ano          1890 non-null   float64
     66  taxa_migracao_eja_em_2_ano          1890 non-null   float64
     67  taxa_migracao_eja_em_3_ano          1890 non-null   float64
    dtypes: float64(64), int64(1), object(3)
    memory usage: 1004.2+ KB
    

Olhando algumas estatisticas dos dados, podemos conferir que a coluna comprova o período de anos esperado para o dataset de 2008 a 2021 


```python
df_raw.describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ano</th>
      <th>taxa_promocao_ef</th>
      <th>taxa_promocao_ef_anos_iniciais</th>
      <th>taxa_promocao_ef_anos_finais</th>
      <th>taxa_promocao_ef_1_ano</th>
      <th>taxa_promocao_ef_2_ano</th>
      <th>taxa_promocao_ef_3_ano</th>
      <th>taxa_promocao_ef_4_ano</th>
      <th>taxa_promocao_ef_5_ano</th>
      <th>taxa_promocao_ef_6_ano</th>
      <th>...</th>
      <th>taxa_migracao_eja_ef_4_ano</th>
      <th>taxa_migracao_eja_ef_5_ano</th>
      <th>taxa_migracao_eja_ef_6_ano</th>
      <th>taxa_migracao_eja_ef_7_ano</th>
      <th>taxa_migracao_eja_ef_8_ano</th>
      <th>taxa_migracao_eja_ef_9_ano</th>
      <th>taxa_migracao_eja_em</th>
      <th>taxa_migracao_eja_em_1_ano</th>
      <th>taxa_migracao_eja_em_2_ano</th>
      <th>taxa_migracao_eja_em_3_ano</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>1890.000000</td>
      <td>1890.000000</td>
      <td>1890.000000</td>
      <td>1890.000000</td>
      <td>1890.000000</td>
      <td>1890.000000</td>
      <td>1890.000000</td>
      <td>1890.000000</td>
      <td>1890.000000</td>
      <td>1890.000000</td>
      <td>...</td>
      <td>1890.000000</td>
      <td>1890.000000</td>
      <td>1890.000000</td>
      <td>1890.000000</td>
      <td>1890.000000</td>
      <td>1890.000000</td>
      <td>1890.000000</td>
      <td>1890.000000</td>
      <td>1890.000000</td>
      <td>1890.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>2014.500000</td>
      <td>86.060582</td>
      <td>89.157354</td>
      <td>82.007672</td>
      <td>94.269101</td>
      <td>90.138730</td>
      <td>86.491058</td>
      <td>88.804815</td>
      <td>88.151640</td>
      <td>80.578201</td>
      <td>...</td>
      <td>0.306508</td>
      <td>0.727460</td>
      <td>1.902116</td>
      <td>2.774974</td>
      <td>2.743757</td>
      <td>2.184286</td>
      <td>2.239630</td>
      <td>2.935397</td>
      <td>2.398677</td>
      <td>0.895608</td>
    </tr>
    <tr>
      <th>std</th>
      <td>4.032196</td>
      <td>7.666824</td>
      <td>7.081946</td>
      <td>9.188975</td>
      <td>3.950990</td>
      <td>9.224082</td>
      <td>8.439545</td>
      <td>7.763740</td>
      <td>7.887825</td>
      <td>11.588508</td>
      <td>...</td>
      <td>0.376444</td>
      <td>0.773436</td>
      <td>1.724481</td>
      <td>2.202182</td>
      <td>1.936240</td>
      <td>1.518951</td>
      <td>1.706327</td>
      <td>2.213831</td>
      <td>1.957525</td>
      <td>0.883348</td>
    </tr>
    <tr>
      <th>min</th>
      <td>2008.000000</td>
      <td>57.400000</td>
      <td>54.300000</td>
      <td>57.300000</td>
      <td>66.500000</td>
      <td>45.700000</td>
      <td>53.900000</td>
      <td>57.400000</td>
      <td>51.700000</td>
      <td>49.600000</td>
      <td>...</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>2011.000000</td>
      <td>80.800000</td>
      <td>85.100000</td>
      <td>75.200000</td>
      <td>92.600000</td>
      <td>87.500000</td>
      <td>80.300000</td>
      <td>84.100000</td>
      <td>82.725000</td>
      <td>72.300000</td>
      <td>...</td>
      <td>0.000000</td>
      <td>0.100000</td>
      <td>0.400000</td>
      <td>0.700000</td>
      <td>0.900000</td>
      <td>0.900000</td>
      <td>0.900000</td>
      <td>1.200000</td>
      <td>0.900000</td>
      <td>0.200000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>2014.500000</td>
      <td>86.400000</td>
      <td>90.600000</td>
      <td>81.200000</td>
      <td>95.400000</td>
      <td>93.500000</td>
      <td>87.300000</td>
      <td>89.900000</td>
      <td>89.900000</td>
      <td>80.200000</td>
      <td>...</td>
      <td>0.200000</td>
      <td>0.400000</td>
      <td>1.300000</td>
      <td>2.200000</td>
      <td>2.400000</td>
      <td>2.200000</td>
      <td>1.800000</td>
      <td>2.600000</td>
      <td>1.700000</td>
      <td>0.600000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>2018.000000</td>
      <td>92.700000</td>
      <td>94.800000</td>
      <td>90.500000</td>
      <td>97.000000</td>
      <td>96.400000</td>
      <td>94.100000</td>
      <td>95.600000</td>
      <td>94.500000</td>
      <td>91.900000</td>
      <td>...</td>
      <td>0.500000</td>
      <td>1.300000</td>
      <td>3.100000</td>
      <td>4.600000</td>
      <td>4.400000</td>
      <td>3.200000</td>
      <td>3.200000</td>
      <td>4.200000</td>
      <td>3.500000</td>
      <td>1.300000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>2021.000000</td>
      <td>98.600000</td>
      <td>98.900000</td>
      <td>98.700000</td>
      <td>99.300000</td>
      <td>99.100000</td>
      <td>99.400000</td>
      <td>99.600000</td>
      <td>99.300000</td>
      <td>99.100000</td>
      <td>...</td>
      <td>1.800000</td>
      <td>3.900000</td>
      <td>7.900000</td>
      <td>9.200000</td>
      <td>7.900000</td>
      <td>20.000000</td>
      <td>12.800000</td>
      <td>22.300000</td>
      <td>9.800000</td>
      <td>5.500000</td>
    </tr>
  </tbody>
</table>
<p>8 rows × 65 columns</p>
</div>



A seguir foi vericado as colunas do tipo object que representam variaveis como string. A coluna sigla_uf apresenta a quantidade de dados esperada, pois temos 27 estados no Brasil.

Para outras duas colunas um sinal de alerta, pois esperavamos 3 valores para cada uma delas.


```python
df_raw.describe(include=object)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>sigla_uf</th>
      <th>localizacao</th>
      <th>rede</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>1890</td>
      <td>1890</td>
      <td>1890</td>
    </tr>
    <tr>
      <th>unique</th>
      <td>27</td>
      <td>6</td>
      <td>6</td>
    </tr>
    <tr>
      <th>top</th>
      <td>AC</td>
      <td>total</td>
      <td>total</td>
    </tr>
    <tr>
      <th>freq</th>
      <td>70</td>
      <td>972</td>
      <td>972</td>
    </tr>
  </tbody>
</table>
</div>



Como obeservado anteriormente a coluna rede e localização possuem valores inconsistentes, será necessário corrigi-los antes de progressir. A solução é simples, converter todos os dados na coluna para minúsculo e substituir Pública por publica.


```python
print("Anteriormente:")
print(df_raw["localizacao"].unique())
print(df_raw["rede"].unique())

df_raw["rede"] = df_raw["rede"].apply(str.lower)
df_raw["rede"] = df_raw["rede"].replace({"pública": "publica"})

df_raw["localizacao"] = df_raw["localizacao"].apply(str.lower)

print("\nApós corrigir:")
print(df_raw["localizacao"].unique())
print(df_raw["rede"].unique())
```

    Anteriormente:
    ['Total' 'total' 'rural' 'Urbana' 'urbana' 'Rural']
    ['Privada' 'privada' 'total' 'Total' 'publica' 'Pública']
    
    Após corrigir:
    ['total' 'rural' 'urbana']
    ['privada' 'total' 'publica']
    

Para facilitar durante geração de gráficos, as seguintes colunas rede e localizacao foram convertidas para o tipo categórico


```python
df_raw["localizacao"] = pd.Categorical(
    df_raw["localizacao"], ["urbana", "rural", "total"]
)
df_raw["rede"] = pd.Categorical(df_raw["rede"], ["publica", "privada", "total"])
```

Não será feita nenhuma analise a respeito da taxa de migracao para o eja, assim, a seguir as colunas referentes a esta informação foram removidas através de uma regex.


```python
drop_cols = df_raw.filter(regex="taxa_migracao_eja").columns

df_raw = df_raw.drop(columns=drop_cols)
df_raw.columns
```




    Index(['ano', 'sigla_uf', 'localizacao', 'rede', 'taxa_promocao_ef',
           'taxa_promocao_ef_anos_iniciais', 'taxa_promocao_ef_anos_finais',
           'taxa_promocao_ef_1_ano', 'taxa_promocao_ef_2_ano',
           'taxa_promocao_ef_3_ano', 'taxa_promocao_ef_4_ano',
           'taxa_promocao_ef_5_ano', 'taxa_promocao_ef_6_ano',
           'taxa_promocao_ef_7_ano', 'taxa_promocao_ef_8_ano',
           'taxa_promocao_ef_9_ano', 'taxa_promocao_em', 'taxa_promocao_em_1_ano',
           'taxa_promocao_em_2_ano', 'taxa_promocao_em_3_ano',
           'taxa_repetencia_ef', 'taxa_repetencia_ef_anos_iniciais',
           'taxa_repetencia_ef_anos_finais', 'taxa_repetencia_ef_1_ano',
           'taxa_repetencia_ef_2_ano', 'taxa_repetencia_ef_3_ano',
           'taxa_repetencia_ef_4_ano', 'taxa_repetencia_ef_5_ano',
           'taxa_repetencia_ef_6_ano', 'taxa_repetencia_ef_7_ano',
           'taxa_repetencia_ef_8_ano', 'taxa_repetencia_ef_9_ano',
           'taxa_repetencia_em', 'taxa_repetencia_em_1_ano',
           'taxa_repetencia_em_2_ano', 'taxa_repetencia_em_3_ano',
           'taxa_evasao_ef', 'taxa_evasao_ef_anos_iniciais',
           'taxa_evasao_ef_anos_finais', 'taxa_evasao_ef_1_ano',
           'taxa_evasao_ef_2_ano', 'taxa_evasao_ef_3_ano', 'taxa_evasao_ef_4_ano',
           'taxa_evasao_ef_5_ano', 'taxa_evasao_ef_6_ano', 'taxa_evasao_ef_7_ano',
           'taxa_evasao_ef_8_ano', 'taxa_evasao_ef_9_ano', 'taxa_evasao_em',
           'taxa_evasao_em_1_ano', 'taxa_evasao_em_2_ano', 'taxa_evasao_em_3_ano'],
          dtype='object')



## Análise dos dados

**Para os dados da rede não há informação de localização**, por exemplo, não é possível filtrar por rede publica e urbana, sendo assim, foi gerado duas tabelas com a função abaixo, uma para cada tipo macro de informação (rede e localização) afim de facilitar a geração de gráficos e filtragem dos dados


```python
def get_macro_table(first_var, second_var):
    taxa_list = ["promocao", "evasao", "repetencia"]

    df_by_ensino = pd.DataFrame()
    for taxa in taxa_list:
        taxa_type = f"taxa_{taxa}"

        df_by_rede = df_raw.query(f"{first_var} != 'total' and {second_var} == 'total'")
        df_by_rede = df_by_rede.filter(
            ["ano", first_var, "sigla_uf", f"{taxa_type}_ef", f"{taxa_type}_em"]
        )

        df_by_rede = df_by_rede.groupby(
            [first_var, "ano", "sigla_uf"], observed=False
        ).mean()
        df_by_rede = df_by_rede.reset_index()
        df_by_rede["ensino"] = None

        df_aux = pd.DataFrame()
        for en in ["em", "ef"]:
            df_ensino = df_by_rede[
                [first_var, "ano", "sigla_uf", "ensino", f"{taxa_type}_{en}"]
            ].rename(columns={f"{taxa_type}_{en}": f"{taxa_type}"})

            df_ensino["ensino"] = en
            df_ensino["valor"] = df_ensino[f"{taxa_type}"]
            df_ensino["taxa"] = taxa
            df_ensino.drop(columns=f"{taxa_type}", inplace=True)

            df_aux = pd.concat([df_aux, df_ensino])

        df_aux.replace(
            {"em": "Ensino médio", "ef": "Ensino fundamental"},
            inplace=True,
        )

        df_by_ensino = pd.concat([df_by_ensino, df_aux])

    df_by_ensino.query(f"{first_var} != 'total'", inplace=True)

    df_by_ensino["taxa"] = pd.Categorical(df_by_ensino["taxa"], taxa_list)
    df_by_ensino["ensino"] = pd.Categorical(
        df_by_ensino["ensino"], ["Ensino fundamental", "Ensino médio"]
    )

    df_by_ensino.sort_values("ano", inplace=True, ignore_index=True)

    return df_by_ensino


df_ensino_rede = get_macro_table("rede", "localizacao")
df_ensino_loc = get_macro_table("localizacao", "rede")

print("Dataframe com dados por rede (publica ou privada)")
display(df_ensino_rede.head())

print("Dataframe com dados por localizao (rural ou urbana)")
display(df_ensino_loc.head())
```

    Dataframe com dados por rede (publica ou privada)
    


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>rede</th>
      <th>ano</th>
      <th>sigla_uf</th>
      <th>ensino</th>
      <th>valor</th>
      <th>taxa</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>publica</td>
      <td>2008</td>
      <td>PE</td>
      <td>Ensino médio</td>
      <td>60.1</td>
      <td>promocao</td>
    </tr>
    <tr>
      <th>1</th>
      <td>publica</td>
      <td>2008</td>
      <td>PB</td>
      <td>Ensino médio</td>
      <td>64.8</td>
      <td>promocao</td>
    </tr>
    <tr>
      <th>2</th>
      <td>publica</td>
      <td>2008</td>
      <td>PA</td>
      <td>Ensino médio</td>
      <td>58.1</td>
      <td>promocao</td>
    </tr>
    <tr>
      <th>3</th>
      <td>publica</td>
      <td>2008</td>
      <td>MT</td>
      <td>Ensino médio</td>
      <td>66.5</td>
      <td>promocao</td>
    </tr>
    <tr>
      <th>4</th>
      <td>publica</td>
      <td>2008</td>
      <td>MS</td>
      <td>Ensino médio</td>
      <td>64.9</td>
      <td>promocao</td>
    </tr>
  </tbody>
</table>
</div>


    Dataframe com dados por localizao (rural ou urbana)
    


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>localizacao</th>
      <th>ano</th>
      <th>sigla_uf</th>
      <th>ensino</th>
      <th>valor</th>
      <th>taxa</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>urbana</td>
      <td>2008</td>
      <td>PE</td>
      <td>Ensino médio</td>
      <td>63.3</td>
      <td>promocao</td>
    </tr>
    <tr>
      <th>1</th>
      <td>urbana</td>
      <td>2008</td>
      <td>PB</td>
      <td>Ensino médio</td>
      <td>67.0</td>
      <td>promocao</td>
    </tr>
    <tr>
      <th>2</th>
      <td>urbana</td>
      <td>2008</td>
      <td>PA</td>
      <td>Ensino médio</td>
      <td>60.2</td>
      <td>promocao</td>
    </tr>
    <tr>
      <th>3</th>
      <td>urbana</td>
      <td>2008</td>
      <td>MT</td>
      <td>Ensino médio</td>
      <td>68.3</td>
      <td>promocao</td>
    </tr>
    <tr>
      <th>4</th>
      <td>urbana</td>
      <td>2008</td>
      <td>MS</td>
      <td>Ensino médio</td>
      <td>68.5</td>
      <td>promocao</td>
    </tr>
  </tbody>
</table>
</div>


Seguindo as regras do padrão [tidy](https://medium.com/datapsico/organizando-banco-de-dados-uma-introducao-ao-conceito-de-tidy-data-1296815aa100), vamos colocar todas as informações necessárias para o gráfico em uma mesma tabela. 

Abaixo está sendo adicionada uma coluna referente as regiões do Brasil, ela foi criada a partir da coluna sigla_uf e um dicionario que discretiza cada região do país conforme a UF.


```python
def add_region(df):
    dict_regions = {
        "Sudeste": ["SP", "RJ", "ES", "MG"],
        "Norte": ["AC", "AM", "AP", "PA", "TO", "RR", "RO"],
        "Nordeste": ["AL", "BA", "CE", "MA", "PB", "PE", "PI", "RN", "SE"],
        "Centro-Oeste": ["DF", "GO", "MT", "MS"],
        "Sul": ["PR", "RS", "SC"],
    }

    df["regiao"] = None
    for key, value in dict_regions.items():
        mask = df["sigla_uf"].isin(dict_regions[key])
        df.loc[mask, "regiao"] = key

    return df


df_ensino_rede = add_region(df_ensino_rede.copy())
df_ensino_rede["rede"] = df_ensino_rede["rede"].cat.remove_unused_categories()

df_ensino_loc = add_region(df_ensino_loc.copy())
df_ensino_loc["localizacao"] = df_ensino_loc[
    "localizacao"
].cat.remove_unused_categories()

print("Dataframe com dados por rede (publica ou privada)")
display(df_ensino_rede.head())

print("Dataframe com dados por localizao (rural ou urbana)")
display(df_ensino_loc.head())
```

    Dataframe com dados por rede (publica ou privada)
    


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>rede</th>
      <th>ano</th>
      <th>sigla_uf</th>
      <th>ensino</th>
      <th>valor</th>
      <th>taxa</th>
      <th>regiao</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>publica</td>
      <td>2008</td>
      <td>PE</td>
      <td>Ensino médio</td>
      <td>60.1</td>
      <td>promocao</td>
      <td>Nordeste</td>
    </tr>
    <tr>
      <th>1</th>
      <td>publica</td>
      <td>2008</td>
      <td>PB</td>
      <td>Ensino médio</td>
      <td>64.8</td>
      <td>promocao</td>
      <td>Nordeste</td>
    </tr>
    <tr>
      <th>2</th>
      <td>publica</td>
      <td>2008</td>
      <td>PA</td>
      <td>Ensino médio</td>
      <td>58.1</td>
      <td>promocao</td>
      <td>Norte</td>
    </tr>
    <tr>
      <th>3</th>
      <td>publica</td>
      <td>2008</td>
      <td>MT</td>
      <td>Ensino médio</td>
      <td>66.5</td>
      <td>promocao</td>
      <td>Centro-Oeste</td>
    </tr>
    <tr>
      <th>4</th>
      <td>publica</td>
      <td>2008</td>
      <td>MS</td>
      <td>Ensino médio</td>
      <td>64.9</td>
      <td>promocao</td>
      <td>Centro-Oeste</td>
    </tr>
  </tbody>
</table>
</div>


    Dataframe com dados por localizao (rural ou urbana)
    


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>localizacao</th>
      <th>ano</th>
      <th>sigla_uf</th>
      <th>ensino</th>
      <th>valor</th>
      <th>taxa</th>
      <th>regiao</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>urbana</td>
      <td>2008</td>
      <td>PE</td>
      <td>Ensino médio</td>
      <td>63.3</td>
      <td>promocao</td>
      <td>Nordeste</td>
    </tr>
    <tr>
      <th>1</th>
      <td>urbana</td>
      <td>2008</td>
      <td>PB</td>
      <td>Ensino médio</td>
      <td>67.0</td>
      <td>promocao</td>
      <td>Nordeste</td>
    </tr>
    <tr>
      <th>2</th>
      <td>urbana</td>
      <td>2008</td>
      <td>PA</td>
      <td>Ensino médio</td>
      <td>60.2</td>
      <td>promocao</td>
      <td>Norte</td>
    </tr>
    <tr>
      <th>3</th>
      <td>urbana</td>
      <td>2008</td>
      <td>MT</td>
      <td>Ensino médio</td>
      <td>68.3</td>
      <td>promocao</td>
      <td>Centro-Oeste</td>
    </tr>
    <tr>
      <th>4</th>
      <td>urbana</td>
      <td>2008</td>
      <td>MS</td>
      <td>Ensino médio</td>
      <td>68.5</td>
      <td>promocao</td>
      <td>Centro-Oeste</td>
    </tr>
  </tbody>
</table>
</div>


### Progressão, repetencia e evasão ao longo de todos os anos

Com a tabela pronta para o plot, através do plotnine foram gerados os gráficos a seguir que apresentam as taxas de promoção, repetencia e evasao em um período de 13 anos.

> A partir da análise dos gráficos, nota-se que as taxas de repetencia e evasão na rede pública são mais altas que na rede privada. Detalhe no ensino médio que apresenta uma taxa de evasão com mediana em 13% ao longo dos 13 anos, cerca de 4x maior do que a mesma taxa para o ensimo fundamental. Com relação a taxa de promoção, ela é mais alta na rede privada, 
>
> Importante notar que o IRQ das escolas privadas são menores, demostrando que há uma uniformidade maior entre a taxa de progressão dos alunos em diferentes estados e ao longo dos anos
>
> **Detalhe importante nesse gráfico são os dois outliers** que aparecem, mais para frente será explicado por que eles existem


```python
df_ensino_rede_gruop = (
    df_ensino_rede.groupby(["rede", "ano", "taxa", "ensino"], observed=True)[["valor"]]
    .mean()
    .reset_index()
)

display(
    ggplot(
        data=df_ensino_rede_gruop.query("taxa != 'promocao'"),
        mapping=aes(x="ensino", y="valor"),
    )
    + geom_boxplot()
    + facet_grid("taxa", "rede")
    + scale_y_continuous(labels=label_number(suffix="%"))
    + theme_bw()
    + theme(figure_size=(7, 6))
    + ylab("Taxa em %")
    + ggtitle("Variação das taxas de evasao e repetencia entre 2008 e 2021")
)


display(
    ggplot(
        data=df_ensino_rede_gruop.query("taxa == 'promocao'"),
        mapping=aes(x="ensino", y="valor"),
    )
    + geom_boxplot()
    + facet_grid("taxa", "rede")
    + scale_y_continuous(labels=label_number(suffix="%"))
    + theme_bw()
    + theme(figure_size=(7, 6))
    + ylab("Taxa de promocao em %")
    + ggtitle("Variação da taxa de promoção entre 2008 e 2021")
)
```


    
![png](taxa_transicao_analise_files/taxa_transicao_analise_23_0.png)
    



    
![png](taxa_transicao_analise_files/taxa_transicao_analise_23_1.png)
    


### A progressão é maior na área rural ou urbana?

O gráfico a seguir apresenta a taxa de promoção tanto para o ensino médio quanto fundamental separadas por rural e urbana.

> A partir da analise, nota-se que a taxa promoção para o ensino fundamental na zona urbana é maior que na zona rural, a evasão é um pouco parecida sendo maior na região rural, e a taxa de repetencia é maior na zona rural, os dois podem estar sendo causados pela dificuldade dos alunos em acompanhar as matérias ou falta de infraestrutura para que eles consigam consigam assistir a aula com qualidade

Portanto, a progressão é maior na zona urbana para o ensino fundamental e na rural e maior para ensino médio


```python
df_ensino_loc_group = (
    df_ensino_loc.groupby(["localizacao", "ensino", "taxa", "ano"], observed=True)[
        ["valor"]
    ]
    .mean()
    .reset_index()
)

(
    ggplot(data=df_ensino_loc_group, mapping=aes(show_legend=True))
    + geom_line(mapping=aes(x="ano", y="valor", color="ensino"), show_legend=True)
    + facet_grid("taxa", "localizacao", scales="free")
    + ylab("Taxas por localização")
    + scale_y_continuous(labels=label_number(suffix="%"))
    + theme_bw()
    + theme(figure_size=(12, 8))
)
```


    
![png](taxa_transicao_analise_files/taxa_transicao_analise_26_0.png)
    


### Qual **estado** tem a maior taxa de progressão e evasao ao longo dos anos?

Nesta seção será apresentado um mapa do Brasil com as taxas por estado. Para gerar esses mapas foi baixado a partir do [site do IBGE](https://www.ibge.gov.br/geociencias/downloads-geociencias.html) um arquqivo que contém lagitude, longitude e o desenho geométrico de cada estado brasileiro.

A tabela gerada anteriormente com os dados por rede de ensino, foram mergeados com a coluna de geometria utilizando as siglas como referência


```python
df_publica_uf = df_ensino_rede.query("rede=='publica'")

df_publica_uf = (
    df_publica_uf.groupby(["sigla_uf", "ensino", "taxa"], observed=True)[["valor"]]
    .mean()
    .reset_index()
)

coord_ibge = gpd.read_file(
    "data/bcim_2016_21_11_2018.gpkg", layer="lim_unidade_federacao_a"
).rename(columns={"sigla": "sigla_uf"})

coord_ibge = coord_ibge[["sigla_uf", "geometry"]].merge(df_publica_uf, on="sigla_uf")

coord_ibge.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>sigla_uf</th>
      <th>geometry</th>
      <th>ensino</th>
      <th>taxa</th>
      <th>valor</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>GO</td>
      <td>MULTIPOLYGON (((-50.15876 -12.41581, -50.15743...</td>
      <td>Ensino fundamental</td>
      <td>promocao</td>
      <td>87.550000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>GO</td>
      <td>MULTIPOLYGON (((-50.15876 -12.41581, -50.15743...</td>
      <td>Ensino fundamental</td>
      <td>evasao</td>
      <td>3.971429</td>
    </tr>
    <tr>
      <th>2</th>
      <td>GO</td>
      <td>MULTIPOLYGON (((-50.15876 -12.41581, -50.15743...</td>
      <td>Ensino fundamental</td>
      <td>repetencia</td>
      <td>7.685714</td>
    </tr>
    <tr>
      <th>3</th>
      <td>GO</td>
      <td>MULTIPOLYGON (((-50.15876 -12.41581, -50.15743...</td>
      <td>Ensino médio</td>
      <td>promocao</td>
      <td>76.885714</td>
    </tr>
    <tr>
      <th>4</th>
      <td>GO</td>
      <td>MULTIPOLYGON (((-50.15876 -12.41581, -50.15743...</td>
      <td>Ensino médio</td>
      <td>evasao</td>
      <td>11.921429</td>
    </tr>
  </tbody>
</table>
</div>



Como será calculado a média, é importante filtrar os outliers para que não atrapalhem a análise.

Para filtrar os outliers após carregar os dados é necessário calcular os quartis, com a função quantile do pandas foi encontrado os 4 quartis. Depois, foi definido o interquartil range 

$$ IRQ = Q3 - Q1 $$

Definido o IRQ, podemos encontrar as borda up e lower, que vai definir a região de outliers.

$$ lower = Q1 - 1,5 * IRQ $$ 
$$ up = Q3 + 1,5 * IRQ $$


```python
q1 = df_publica_uf.groupby(["ensino", "taxa"], observed=True)[["valor"]].quantile(0.25)
q3 = df_publica_uf.groupby(["ensino", "taxa"], observed=True)[["valor"]].quantile(0.75)

irq = (q3 - q1) * 1.5

lower_bound = q1 - irq
upper_bound = q3 + irq

print("Borda inferior")
display(lower_bound)

print("Borda superior")
display(upper_bound)
```

    Borda inferior
    


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>valor</th>
    </tr>
    <tr>
      <th>ensino</th>
      <th>taxa</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="3" valign="top">Ensino fundamental</th>
      <th>promocao</th>
      <td>68.453571</td>
    </tr>
    <tr>
      <th>evasao</th>
      <td>-0.135714</td>
    </tr>
    <tr>
      <th>repetencia</th>
      <td>1.673214</td>
    </tr>
    <tr>
      <th rowspan="3" valign="top">Ensino médio</th>
      <th>promocao</th>
      <td>58.750000</td>
    </tr>
    <tr>
      <th>evasao</th>
      <td>6.023214</td>
    </tr>
    <tr>
      <th>repetencia</th>
      <td>4.725000</td>
    </tr>
  </tbody>
</table>
</div>


    Borda superior
    


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>valor</th>
    </tr>
    <tr>
      <th>ensino</th>
      <th>taxa</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="3" valign="top">Ensino fundamental</th>
      <th>promocao</th>
      <td>97.939286</td>
    </tr>
    <tr>
      <th>evasao</th>
      <td>7.721429</td>
    </tr>
    <tr>
      <th>repetencia</th>
      <td>20.887500</td>
    </tr>
    <tr>
      <th rowspan="3" valign="top">Ensino médio</th>
      <th>promocao</th>
      <td>86.750000</td>
    </tr>
    <tr>
      <th>evasao</th>
      <td>17.866071</td>
    </tr>
    <tr>
      <th>repetencia</th>
      <td>19.696429</td>
    </tr>
  </tbody>
</table>
</div>


A seguir temos os plots por estado para as taxas de promoção e evasão.

> Entre os ensinos é bem diferente o valor das taxas, para o ensino fundamental temos uma evasão mais forte na região norte e sudeste > enquanto no ensino médio temos o Pará e a região centro oeste como piores estados neste quesito
>
> Em relação a taxa de promoção SP lidera nos dois tipos de ensino, seguidos por estados da região sul e Minas Gerais.
>
> Destaque para o **Ceará**, podemos dizer que ele é um outlier na sua região, pois possui altas taxas de promoção tanto ensino médio quanto no fundamental. A alguns anos este estado já vem sendo comentado nas midias, como exemplo de educação no país e neste gráfico podemos comprovar isso.


```python
plt.close("all")
```


```python
colors = ["Blues", "Reds"]
for ensino in coord_ibge["ensino"].unique():
    fig, ax = plt.subplots(1, 2, figsize=(15, 15))
    for i, taxaa in enumerate(["promocao", "evasao"]):

        # Removendo outliers
        lower = lower_bound.loc[ensino, taxaa]["valor"]
        upper = upper_bound.loc[ensino, taxaa]["valor"]

        coord_ibge.query(
            "taxa==@taxaa and ensino == @ensino and valor >= @lower and valor <= @upper"
        ).plot(
            ax=ax[i],
            column="valor",
            cmap=colors[i],
            legend=True,
            legend_kwds={"shrink": 0.4, "format": "{x}%"},
        )

        ax[i].set_title(taxaa)
        ax[i].axis("off")

    fig.suptitle(ensino, y=0.7)
    plt.show()
```


    
![png](taxa_transicao_analise_files/taxa_transicao_analise_34_0.png)
    



    
![png](taxa_transicao_analise_files/taxa_transicao_analise_34_1.png)
    


### Qual **região** tem a maior taxa de progressão e evasao ao longo dos anos?

Como vamos calcular a média, é necessário remover alguns outliers seguindo o mesmo método anterior, mas agora diferente pois precisamos olhar por ano ensino e taxa


```python
gp = df_ensino_rede.groupby(["ano", "ensino", "taxa", "regiao"], observed=True)[
    ["valor"]
]

q1 = gp.quantile(0.25)
q3 = gp.quantile(0.75)

irq = (q3 - q1) * 1.5

lower_bound = q1 - irq
upper_bound = q3 + irq

print("Borda inferior")
display(lower_bound)

print("Borda superior")
display(upper_bound)
```

    Borda inferior
    


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th>valor</th>
    </tr>
    <tr>
      <th>ano</th>
      <th>ensino</th>
      <th>taxa</th>
      <th>regiao</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="5" valign="top">2008</th>
      <th rowspan="5" valign="top">Ensino fundamental</th>
      <th rowspan="5" valign="top">promocao</th>
      <th>Centro-Oeste</th>
      <td>59.0500</td>
    </tr>
    <tr>
      <th>Nordeste</th>
      <td>32.6125</td>
    </tr>
    <tr>
      <th>Norte</th>
      <td>50.1125</td>
    </tr>
    <tr>
      <th>Sudeste</th>
      <td>61.3500</td>
    </tr>
    <tr>
      <th>Sul</th>
      <td>69.5875</td>
    </tr>
    <tr>
      <th>...</th>
      <th>...</th>
      <th>...</th>
      <th>...</th>
      <td>...</td>
    </tr>
    <tr>
      <th rowspan="5" valign="top">2021</th>
      <th rowspan="5" valign="top">Ensino médio</th>
      <th rowspan="5" valign="top">repetencia</th>
      <th>Centro-Oeste</th>
      <td>-3.7875</td>
    </tr>
    <tr>
      <th>Nordeste</th>
      <td>-0.2625</td>
    </tr>
    <tr>
      <th>Norte</th>
      <td>-3.1750</td>
    </tr>
    <tr>
      <th>Sudeste</th>
      <td>-0.3625</td>
    </tr>
    <tr>
      <th>Sul</th>
      <td>-2.6125</td>
    </tr>
  </tbody>
</table>
<p>420 rows × 1 columns</p>
</div>


    Borda superior
    


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th>valor</th>
    </tr>
    <tr>
      <th>ano</th>
      <th>ensino</th>
      <th>taxa</th>
      <th>regiao</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="5" valign="top">2008</th>
      <th rowspan="5" valign="top">Ensino fundamental</th>
      <th rowspan="5" valign="top">promocao</th>
      <th>Centro-Oeste</th>
      <td>115.8500</td>
    </tr>
    <tr>
      <th>Nordeste</th>
      <td>127.3125</td>
    </tr>
    <tr>
      <th>Norte</th>
      <td>119.0125</td>
    </tr>
    <tr>
      <th>Sudeste</th>
      <td>115.5500</td>
    </tr>
    <tr>
      <th>Sul</th>
      <td>110.4875</td>
    </tr>
    <tr>
      <th>...</th>
      <th>...</th>
      <th>...</th>
      <th>...</th>
      <td>...</td>
    </tr>
    <tr>
      <th rowspan="5" valign="top">2021</th>
      <th rowspan="5" valign="top">Ensino médio</th>
      <th rowspan="5" valign="top">repetencia</th>
      <th>Centro-Oeste</th>
      <td>11.5125</td>
    </tr>
    <tr>
      <th>Nordeste</th>
      <td>5.0375</td>
    </tr>
    <tr>
      <th>Norte</th>
      <td>9.0250</td>
    </tr>
    <tr>
      <th>Sudeste</th>
      <td>4.3375</td>
    </tr>
    <tr>
      <th>Sul</th>
      <td>7.2875</td>
    </tr>
  </tbody>
</table>
<p>420 rows × 1 columns</p>
</div>


outliers


```python
df_regiao = pd.DataFrame()
for col, bound in lower_bound.iterrows():
    up_bound = upper_bound.loc[col].values[0]
    lo_bound = bound.values[0]
    col = list(col)
    df_aux = df_ensino_rede.query(
        "ano == @col[0] and ensino == @col[1] and taxa == @col[2] and regiao == @col[3] and valor >= @lo_bound and valor <= @up_bound"
    )

    df_regiao = pd.concat([df_regiao, df_aux])

df_regiao = (
    df_regiao.groupby(["regiao", "ano", "ensino", "taxa"], observed=True)[["valor"]]
    .mean()
    .reset_index()
)

df_regiao
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>regiao</th>
      <th>ano</th>
      <th>ensino</th>
      <th>taxa</th>
      <th>valor</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Centro-Oeste</td>
      <td>2008</td>
      <td>Ensino fundamental</td>
      <td>promocao</td>
      <td>87.087500</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Centro-Oeste</td>
      <td>2008</td>
      <td>Ensino fundamental</td>
      <td>evasao</td>
      <td>3.150000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Centro-Oeste</td>
      <td>2008</td>
      <td>Ensino fundamental</td>
      <td>repetencia</td>
      <td>8.637500</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Centro-Oeste</td>
      <td>2008</td>
      <td>Ensino médio</td>
      <td>promocao</td>
      <td>76.837500</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Centro-Oeste</td>
      <td>2008</td>
      <td>Ensino médio</td>
      <td>evasao</td>
      <td>10.550000</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>415</th>
      <td>Sul</td>
      <td>2021</td>
      <td>Ensino fundamental</td>
      <td>evasao</td>
      <td>1.433333</td>
    </tr>
    <tr>
      <th>416</th>
      <td>Sul</td>
      <td>2021</td>
      <td>Ensino fundamental</td>
      <td>repetencia</td>
      <td>1.566667</td>
    </tr>
    <tr>
      <th>417</th>
      <td>Sul</td>
      <td>2021</td>
      <td>Ensino médio</td>
      <td>promocao</td>
      <td>89.933333</td>
    </tr>
    <tr>
      <th>418</th>
      <td>Sul</td>
      <td>2021</td>
      <td>Ensino médio</td>
      <td>evasao</td>
      <td>6.433333</td>
    </tr>
    <tr>
      <th>419</th>
      <td>Sul</td>
      <td>2021</td>
      <td>Ensino médio</td>
      <td>repetencia</td>
      <td>1.920000</td>
    </tr>
  </tbody>
</table>
<p>420 rows × 5 columns</p>
</div>



Com a tabela acima foi gerado o gráfico a seguir

> Para o ensino fundamental a menor taxa de promoção é no nordeste que também possui uma maior taxa de evasão, demonstrando que os alunos neste estado possuem mai dificuldade de seguir as aulas
>
> Para o ensino médio tanto promoção quanto evasão possuem taxas bem semelhantes para todas as regiões 


```python
for ensino in df_regiao["ensino"].unique():
    display(
        ggplot(
            data=df_regiao.query("taxa=='promocao' and ensino == @ensino"),
            mapping=aes("ano", "regiao", fill="valor"),
        )
        + geom_tile(color="black")
        + scale_color_brewer(type="seq", palette=9)
        + scale_fill_gradientn(
            colors=sns.color_palette("Blues", n_colors=20),
            trans="sqrt",
            labels=label_number(suffix="%"),
        )
        + scale_x_continuous(trans="sqrt", breaks=(2010, 2013, 2016, 2019))
        + theme(
            panel_grid=element_blank(), legend_key="bottom", text=element_text(size=8)
        )
        + labs(title=f"Taxa de promoção - {ensino}")
        + theme_bw()
    )
```

    c:\Users\rafac\anaconda3\envs\ia376\Lib\site-packages\plotnine\guides\guides.py:207: PlotnineWarning: Cannot generate legend for the 'color' aesthetic. Make sure you have mapped a variable to it
    


    
![png](taxa_transicao_analise_files/taxa_transicao_analise_41_1.png)
    


    c:\Users\rafac\anaconda3\envs\ia376\Lib\site-packages\plotnine\guides\guides.py:207: PlotnineWarning: Cannot generate legend for the 'color' aesthetic. Make sure you have mapped a variable to it
    


    
![png](taxa_transicao_analise_files/taxa_transicao_analise_41_3.png)
    



```python
for ensino in df_regiao["ensino"].unique():
    display(
        ggplot(
            data=df_regiao.query("taxa=='evasao' and ensino == @ensino"),
            mapping=aes("ano", "regiao", fill="valor"),
        )
        + geom_tile(color="black")
        + scale_color_brewer(type="seq", palette=9)
        + scale_fill_gradientn(
            colors=sns.color_palette("Reds", n_colors=20),
            trans="sqrt",
            labels=label_number(suffix="%"),
        )
        + scale_x_continuous(trans="sqrt", breaks=(2010, 2013, 2016, 2019))
        + theme(
            panel_grid=element_blank(), legend_key="bottom", text=element_text(size=8)
        )
        + labs(title=f"Taxa de evasão - {ensino}")
        + theme_bw()
    )
```

    c:\Users\rafac\anaconda3\envs\ia376\Lib\site-packages\plotnine\guides\guides.py:207: PlotnineWarning: Cannot generate legend for the 'color' aesthetic. Make sure you have mapped a variable to it
    


    
![png](taxa_transicao_analise_files/taxa_transicao_analise_42_1.png)
    


    c:\Users\rafac\anaconda3\envs\ia376\Lib\site-packages\plotnine\guides\guides.py:207: PlotnineWarning: Cannot generate legend for the 'color' aesthetic. Make sure you have mapped a variable to it
    


    
![png](taxa_transicao_analise_files/taxa_transicao_analise_42_3.png)
    


## Como evoluiram as taxas de promoção, evasão e repetencia no primeiro ano de pandemia

Para ter uma noção inicial das taxas, primeiro foi plotado todas elas para veriguar se houve alguma grande diferença. Observando o gráfico abaixo podemos encontrar a explicação para aqueles 2 outliers presentes no box plot da promoção, eles aconteceram por conta da pandemia. Nesse ano as taxas de promoção foram mais altas no geral tanto para a rede pública quanto privada.

No entanto, no ensino fundamental da rede privada essa taxa caiu, sendo muito diferente do que é visto no restante dos gráficos, ela caiu devido a repetencia ter sido mais alta, nos próximos gráficos vamos averiguar isso com mais detalhe.


```python
(
    ggplot(data=df_ensino_rede_gruop, mapping=aes(show_legend=True))
    + geom_line(mapping=aes(x="ano", y="valor", color="rede"))
    + geom_line(mapping=aes(x="ano", y="valor", color="rede"))
    + scale_y_continuous(labels=label_number(suffix="%"))
    + facet_grid("taxa", "ensino", scales="free")
    + geom_vline(xintercept=2020, color="black")
    + ylab(f"Taxas")
    + theme_bw()
    + theme(figure_size=(10, 8))
    + labs(title="Taxas ao longos dos anos para cada tipo de ensino")
)
```


    
![png](taxa_transicao_analise_files/taxa_transicao_analise_45_0.png)
    


Antes de plotar os gráficos, é necessário montar uma tabela com os dados para cada ano do ensino fundamental. Essa tabela será criada a partir do dado raw


```python
taxa = "repetencia"
ensino = "ef"
ano_graph = [2019, 2020, 2021]
taxa_type = f"taxa_{taxa}"


df_ensino_ano = df_raw.query("rede!='total' & localizacao=='total' & ano in @ano_graph")
df_ensino_ano = df_ensino_ano.filter(
    regex=(rf"(^{taxa_type}_{ensino}_\d_.*)|(^rede)|(^ano)")
)

df_plot = pd.DataFrame()

cols = df_ensino_ano.filter(regex=(rf"(^{taxa_type}_{ensino}_\d_.*)")).columns

for col in cols:
    df_aux = df_ensino_ano[["rede", "ano"]].copy()
    df_aux.loc[:, "taxa"] = df_ensino_ano[col]
    df_aux.loc[:, "ensino_ano"] = re.search(r"\d", col)[0]

    df_plot = pd.concat([df_plot, df_aux])

df_plot = (
    df_plot.groupby(["rede", "ensino_ano", "ano"], observed=True)[["taxa"]]
    .mean()
    .reset_index()
)

df_plot.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>rede</th>
      <th>ensino_ano</th>
      <th>ano</th>
      <th>taxa</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>publica</td>
      <td>1</td>
      <td>2019</td>
      <td>2.870370</td>
    </tr>
    <tr>
      <th>1</th>
      <td>publica</td>
      <td>1</td>
      <td>2020</td>
      <td>2.451852</td>
    </tr>
    <tr>
      <th>2</th>
      <td>publica</td>
      <td>1</td>
      <td>2021</td>
      <td>1.966667</td>
    </tr>
    <tr>
      <th>3</th>
      <td>publica</td>
      <td>2</td>
      <td>2019</td>
      <td>4.474074</td>
    </tr>
    <tr>
      <th>4</th>
      <td>publica</td>
      <td>2</td>
      <td>2020</td>
      <td>4.174074</td>
    </tr>
  </tbody>
</table>
</div>



No gráficos a seguir pode se obsevar que para a rede publica em 2019 e 2020 as taxas de repetencia para todos os anos são maiores, mas em 2020 todas caíram, demonstrando assim que a politica adotada naquele ano foi de aprovar os alunos apesar do desempenho escolar.

> Ainda para a rede pública, interessante notar como a cada 3 anos a taxa de repetência é elevada, devido ao fim do ciclo a aprovação continuada.
>
> Para a rede privada aconteceu o contrário, houve um aumento na taxa de repetencia para os primeiros anos, podemos inferir a partir disso que os alunos desses anos tiveram uma maior dificuldade para acompanhar as aulas, muito provavelmente em virtude de não terem habilidades ainda com computadores e celulares ou por não ter a mesma capacidade de concentração de um adulto para acompanhar as aulas de casa.


```python
(
    ggplot(
        data=df_plot,
        mapping=aes("ensino_ano", f"taxa", fill="taxa"),
    )
    + geom_bar(stat="identity")
    + scale_fill_gradientn(
        colors=sns.color_palette("rocket_r"), labels=label_number(suffix="%")
    )
    + scale_y_continuous(labels=label_number(suffix="%"))
    + facet_grid("rede", "ano")
    + ggtitle(ano_graph)
    + theme_bw()
    + theme(figure_size=(12, 8), axis_text_y=element_text(size=6))
    + labs(title="Taxa de repetencia para os ultimos 3 anos por rede de ensino")
)
```


    
![png](taxa_transicao_analise_files/taxa_transicao_analise_49_0.png)
    


## Hipótese: **As escolas públicas possuem maior taxa de progressão do que as privadas**

Analisando o gráfico plotado anteriormente, a reposta é **não**, durante todo o período analisado a taxa de aprovação foi maior para as escola da rede privada, mostrando que apesar da aprovação continuada, na rede publica os alunos não são aprovados automaticamente, contudo ao longo dos anos se percebe um aumento na taxa de promoção, que pode indicar que o ensino melhorou ou que o ensino está "afrouxando" a analise de desempenho dos alunos para aumentar essa taxa. 


```python
(
    ggplot(
        data=df_ensino_rede_gruop.query("taxa == 'promocao'"),
        mapping=aes(show_legend=True),
    )
    + geom_line(mapping=aes(x="ano", y="valor", color="rede"))
    + geom_line(mapping=aes(x="ano", y="valor", color="rede"))
    + scale_y_continuous(labels=label_number(suffix="%"))
    + facet_grid("taxa", "ensino", scales="free")
    + theme_bw()
    + theme(figure_size=(10, 4))
    + labs(title="Taxa de promoção ao longo dos anos", y="Promocao")
)
```


    
![png](taxa_transicao_analise_files/taxa_transicao_analise_52_0.png)
    


### Como funciona a progressão continuada

Aproveitando para ir mais a fundo na nossa analise podemos verificar a questão da chamada progressão continuada entre dois estados.

Foi escolhido Parana e São Paulo, por possuirem politicas diferentes e economias parecidas.

Fonte: [BBC - progressão continuada](https://www.bbc.com/portuguese/articles/c72y0zqnvk1o)

![image.png](taxa_transicao_analise_files/image.png)

Primeiramente analisando a taxa ao longo dos anos, podemos ver que em SP devido a politica ela sobe de forma mais **acentuada**, principalmente para o ensino médio, enquanto no Paraná a taxa fica abaixo de 80% até 2019 e subiu provavelmente por conta da pandemia


```python
df_SP = (
    df_ensino_rede.query("sigla_uf in ['SP', 'PR'] and taxa == 'promocao'")
    .groupby(["rede", "ensino", "sigla_uf", "ano"], observed=True)[["valor"]]
    .mean()
    .reset_index()
)

(
    ggplot(data=df_SP, mapping=aes(show_legend=True))
    + geom_line(mapping=aes(x="ano", y="valor", color="rede"))
    + geom_line(mapping=aes(x="ano", y="valor", color="rede"))
    + scale_y_continuous(labels=label_number(suffix="%"))
    + facet_grid("sigla_uf", "ensino")
    + ylab(f"Taxas")
    + theme_bw()
    + theme(figure_size=(9, 6))
    + labs(
        title="Taxa de promoção ao longo dos anos para os estados PR e SP", y="Promocao"
    )
)
```


    
![png](taxa_transicao_analise_files/taxa_transicao_analise_57_0.png)
    


Antes de plotar os gráficos, é necessário montar uma tabela com os dados para cada ano do ensino fundamental e estado Parana e São Paulo. Essa tabela será criada a partir do dado raw


```python
taxa = "repetencia"
ensino = "ef"
ano_graph = [2017, 2018, 2019]
taxa_type = f"taxa_{taxa}"


df_ensino_ano = df_raw.query(
    "rede!='total' & sigla_uf in ['SP', 'PR'] & localizacao=='total' & ano in @ano_graph"
)
df_ensino_ano = df_ensino_ano.filter(
    regex=(rf"(^{taxa_type}_{ensino}_\d_.*)|(^sigla_uf)|(^ano)")
)

df_plot = pd.DataFrame()

cols = df_ensino_ano.filter(regex=(rf"(^{taxa_type}_{ensino}_\d_.*)")).columns

for col in cols:
    df_aux = df_ensino_ano[["sigla_uf", "ano"]].copy()
    df_aux.loc[:, "taxa"] = df_ensino_ano[col]
    df_aux.loc[:, "ensino_ano"] = re.search(r"\d", col)[0]

    df_plot = pd.concat([df_plot, df_aux])

df_plot = (
    df_plot.groupby(["sigla_uf", "ensino_ano", "ano"], observed=True)[["taxa"]]
    .mean()
    .reset_index()
)

df_plot.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>sigla_uf</th>
      <th>ensino_ano</th>
      <th>ano</th>
      <th>taxa</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>PR</td>
      <td>1</td>
      <td>2017</td>
      <td>2.40</td>
    </tr>
    <tr>
      <th>1</th>
      <td>PR</td>
      <td>1</td>
      <td>2018</td>
      <td>2.05</td>
    </tr>
    <tr>
      <th>2</th>
      <td>PR</td>
      <td>1</td>
      <td>2019</td>
      <td>2.30</td>
    </tr>
    <tr>
      <th>3</th>
      <td>PR</td>
      <td>2</td>
      <td>2017</td>
      <td>5.35</td>
    </tr>
    <tr>
      <th>4</th>
      <td>PR</td>
      <td>2</td>
      <td>2018</td>
      <td>5.15</td>
    </tr>
  </tbody>
</table>
</div>



O gráfico a seguir apresenta muito bem como a progressão continuada funciona, em SP a cada ciclo de 3 anos há um significativo aumento na repetencia, enquanto no Parana a taxa ao longo dos anos é bem definida e costuma ser mais alta nos últimos anos do ensino médio.


```python
(
    ggplot(
        data=df_plot,
        mapping=aes("ensino_ano", f"taxa", fill="taxa"),
    )
    + geom_bar(stat="identity")
    + scale_fill_gradientn(
        colors=sns.color_palette("rocket_r"), labels=label_number(suffix="%")
    )
    + scale_y_continuous(labels=label_number(suffix="%"))
    + facet_grid("sigla_uf", "ano")
    + ggtitle(ano_graph)
    + theme_bw()
    + theme(figure_size=(12, 8), axis_text_y=element_text(size=6))
    + labs(title="Taxa de repetencia para os ultimos 3 anos para PR e SP")
)
```


    
![png](taxa_transicao_analise_files/taxa_transicao_analise_61_0.png)
    


# Conclusão

A partir das análises acima podemos tirar as seguintes conslusões:
- A rede privada possui um IRQ menor em todas as suas taxas, portanto, seus alunos possuem um desepenho mais homegeneo ao longo do anos e entre todos os estados
- A região nordeste possui uma taxa de promoção bem menor e alta evasão quando comparada com outras regiões, que pode acontecer devido a falta de estrutura e recursos para educação no estado
- O Ceará é um outlier da região nordeste, pois possui indices melhores que seus vizinhos.
- Apesar do que todo mundo pensa a rede pública não possui uma régua baixa para aprovar os alunos.
- Ao longo de 2020, no primeiro ano de pandemia, houve um aumento significtivo da taxa de promoção na rede pública
- A progressão continuada existe e faz com que os estados que a possuem tenham uma taxa de aprovação mais alta do que outros.
- A hipótese estava errada e a rede privadas possuem uma taxa de promoção maior
