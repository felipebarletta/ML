---
output:
  html_document: default
  pdf_document: default
  word_document: default
---
# Extração de dados  - mongoBD via _python_


![ ](python_jupyter_mongoDB.png)

## Pacotes necessário do python

Para instalar os pacotes(bibliotecas) no python, basta usar o comando `!pip3 install nome_pacote` ou `!pip3 install requirements.txt` em que *requirements.txt* é um  o arquivo texto com os nomes dos pacotes a serem instalados.




```python
#!pip3 install -r requirements.txt # Basta rodar uma vez apenas
```


```python
import numpy as np #tarefas matemáticas e machine learning
import pandas as pd #manipulação de data frames e machine learning
from pandas import Series, DataFrame 
from datetime import datetime #manipulação de variáveis em formato de data
from pymongo import DESCENDING #util para acessar o mongoDB
from collections import OrderedDict
pd.set_option('display.max_columns', None)# Não limita o número de colunas para visualização

```

## Abaixo um exemplo de como acessar o mongoDB


```python
from pymongo import MongoClient
def db_connect(ambient):

    if ambient == "prod":
        db_uri = "mongodb://credenciais_de_acesso_ao_mongoDB"
        db_name = "nome_da_base_de_dados"

    client = MongoClient(db_uri)
    db = client[db_name]

    if not db:
        print("ERROR: Couldn't connect to database")
        return None

    return db

db = db_connect("prod")
```


```python
# Define o período da coleta dos dados

period_start = datetime(2020, 8, 1, 0, 0, 0) 
period_end =   datetime(2020, 8, 2, 0, 0, 0)
entity_id = 12

```

## Documentos na base de dados


```python
# Distinct nos 'document_types' do banco
document_types = db.history_people.distinct('document_type', {'entidade_id': entity_id})
```


```python
document_types
```




    ['medical_record',
     'vital_signs',
     'exams',
     'medicines',
     'pharmacy',
     'movements',
     'alert']



## Consulta para extrair as informações de sinais vitais 


```python
query = {
   'entidade_id': entity_id,
   'document_type': 'vital_signs',
   'date': {'$gte': period_start,
            '$lte': period_end}
}

limiters = {
    '_id': 0,
    'date':1,
    'document.data_coleta':1,
    'document.data_liberacao':1,
    'document.freq_cardiaca':1,
    'document.freq_respiratoria':1,
    'document.glicemia_capilar':1,
    'document.nivel_consciencia':1,
    'document.pa_diastolica':1,
    'document.pa_sistolica':1,
    'document.sat_o2':1,
    'document.temperatura':1,
    'document.paciente_id':1
}

vs = list(db.history_people.find(query, limiters))
```

### Tamano da amostra


```python
len(vs)
```




    7469



## Formatando os dados para data frame

### Dados de Sinais vitais


```python
from pandas.io.json import json_normalize 
```


```python
### Transformando de formato JSON para data frame
df_vs = json_normalize(vs)
### Ordenando por data
df_vs.sort_values(by=['date'], inplace=True)
### Visualizando as  primiras linhas do data frame
df_vs.head(5)
```

    <ipython-input-22-51dfdbf79ed6>:2: FutureWarning: pandas.io.json.json_normalize is deprecated, use pandas.json_normalize instead
      df_vs = json_normalize(vs)





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
      <th>date</th>
      <th>document.data_coleta</th>
      <th>document.data_liberacao</th>
      <th>document.freq_cardiaca</th>
      <th>document.freq_respiratoria</th>
      <th>document.glicemia_capilar</th>
      <th>document.nivel_consciencia</th>
      <th>document.pa_diastolica</th>
      <th>document.pa_sistolica</th>
      <th>document.paciente_id</th>
      <th>document.sat_o2</th>
      <th>document.temperatura</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2020-08-01</td>
      <td>2020-08-01</td>
      <td>2020-08-01 00:03:45</td>
      <td>131.0</td>
      <td>22.0</td>
      <td>NaN</td>
      <td>None</td>
      <td>62.0</td>
      <td>135.0</td>
      <td>731032</td>
      <td>95.0</td>
      <td>38.1</td>
    </tr>
    <tr>
      <th>75</th>
      <td>2020-08-01</td>
      <td>2020-08-01</td>
      <td>2020-08-01 03:02:58</td>
      <td>86.0</td>
      <td>16.0</td>
      <td>NaN</td>
      <td>None</td>
      <td>54.0</td>
      <td>93.0</td>
      <td>2782882</td>
      <td>100.0</td>
      <td>35.9</td>
    </tr>
    <tr>
      <th>74</th>
      <td>2020-08-01</td>
      <td>2020-08-01</td>
      <td>2020-08-01 03:02:05</td>
      <td>117.0</td>
      <td>16.0</td>
      <td>87.0</td>
      <td>None</td>
      <td>66.0</td>
      <td>122.0</td>
      <td>2802042</td>
      <td>100.0</td>
      <td>36.2</td>
    </tr>
    <tr>
      <th>73</th>
      <td>2020-08-01</td>
      <td>2020-08-01</td>
      <td>2020-08-01 02:54:08</td>
      <td>100.0</td>
      <td>24.0</td>
      <td>NaN</td>
      <td>None</td>
      <td>62.0</td>
      <td>103.0</td>
      <td>750156</td>
      <td>97.0</td>
      <td>38.2</td>
    </tr>
    <tr>
      <th>72</th>
      <td>2020-08-01</td>
      <td>2020-08-01</td>
      <td>2020-08-01 02:52:44</td>
      <td>82.0</td>
      <td>28.0</td>
      <td>NaN</td>
      <td>None</td>
      <td>67.0</td>
      <td>128.0</td>
      <td>1958456</td>
      <td>92.0</td>
      <td>36.7</td>
    </tr>
  </tbody>
</table>
</div>




```python
len(df_vs)
```




    7469



### Nome das variáveis


```python
df_vs.columns
```




    Index(['entidade_id', 'atendimento_id', 'date', 'document.data_coleta',
           'document.data_liberacao', 'document.freq_cardiaca',
           'document.freq_respiratoria', 'document.glicemia_capilar',
           'document.nivel_consciencia', 'document.pa_diastolica',
           'document.pa_sistolica', 'document.paciente_id', 'document.sat_o2',
           'document.temperatura'],
          dtype='object')



### Exportando os dados localmente em formato `csv`


```python
df_vs.to_csv('nome.csv', sep = ',', index = False)
```
