# Automação de Coleta de Dados com API de Países

Este projeto visa criar uma automação para a coleta e armazenamento de dados de países usando a API REST Countries. Os dados são transformados e salvos em um banco de dados SQLite para análises futuras.

## 🚀 Começando

Essas instruções permitirão que você obtenha uma cópia do projeto em operação na sua máquina local para fins de desenvolvimento e teste.

Consulte a seção de Implantação para saber como implantar o projeto.

## 📋 Pré-requisitos

De que coisas você precisa para instalar o software e como instalá-lo?

- Python

### Instalando virtualenv

Para criar ambientes isolados para seus projetos Python, você precisará instalar o virtualenv:

%pip install virtualenv


## 🔧 Instalação

Uma série de exemplos passo-a-passo que informam o que você deve executar para ter um ambiente de desenvolvimento em execução.

### 1. Clone o repositório

Primeiro, clone o repositório do projeto para sua máquina local usando git:

sh
git clone https://github.com/seu-usuario/seu-repositorio.git
cd seu-repositorio


### 2. Crie um ambiente virtual

Crie um ambiente virtual para isolar as dependências do projeto:

sh
virtualenv venv
source venv/bin/activate  # Linux/Mac
venv\Scripts\activate  # Windows


### 3. Instale as dependências

Instale todas as dependências necessárias listadas no arquivo requirements.txt:

sh
pip install -r requirements.txt


### 4. Execute o Jupyter Notebook

Inicie o Jupyter Notebook para executar o código do projeto:

sh
jupyter notebook


Abra o notebook seu_notebook.ipynb e execute as células para coletar, transformar e salvar os dados.


### 🔩 Analise os testes de ponta a ponta

Os testes de ponta a ponta verificam a integração completa do sistema, desde a coleta de dados até o armazenamento no banco de dados. Eles garantem que todas as partes do sistema funcionem corretamente juntas.

Para executar os testes de ponta a ponta, use:

sh
pytest tests/test_end_to_end.py


### ⌨️ Testes de estilo de codificação

Os testes de estilo de codificação garantem que o código siga os padrões estabelecidos, melhorando a legibilidade e a manutenção do código.

Para executar os testes de estilo de codificação, use:

sh
flake8 src/


## 📦 Implantação

Adicione notas adicionais sobre como implantar isso em um sistema ativo.

### Passos para Implantação

1. *Preparação do ambiente*: Certifique-se de que todas as dependências estão instaladas em seu ambiente de produção.
2. *Configuração do banco de dados*: Configure um banco de dados SQLite ou qualquer outro banco de dados de sua preferência.
3. *Agendamento de tarefas*: Configure uma tarefa agendada (usando cron jobs no Linux ou o Agendador de Tarefas no Windows) para executar o script de coleta de dados periodicamente.

## 🛠️ Construído com

Mencione as ferramentas que você usou para criar seu projeto

- *Pandas* - Manipulação e análise de dados
- *Requests* - Requisições HTTP para APIs
- *Plyer* - Notificações do sistema
- *SQLite* - Banco de dados leve
- *TQDM* - Barra de progresso


---

## Código Fonte do Projeto

python
# Instalação das dependências necessárias
%pip install plyer
%pip install tqdm

# Importação das bibliotecas
import pandas as pd
import numpy as np
import sqlite3
from datetime import datetime
from plyer import notification 
import requests
from tqdm import tqdm

# URL da API REST Countries
url = "https://restcountries.com/v3.1/all"

# Requisição à API
resposta = requests.get(url)
if resposta.status_code == 200:
    data_json = resposta.json()
else:
    print('erro ao acessar a API')

# Transformação dos dados em DataFrame
paises = pd.DataFrame(data_json)

# Seleção e transformação de colunas específicas
subregion = [sub for sub in paises['subregion']]
semana = [week for week in paises['startOfWeek']]
nome = [pais['common'] for pais in paises['name']]

dados_filtrados = {
    'Nome': nome,
    'Semana': semana,
    'Subregiao': subregion
}

paises_novo = pd.DataFrame(dados_filtrados)

# Criação de outro DataFrame com diferentes colunas
area_countries = [area for area in paises['area']]
capital_countries = [cap for cap in paises['capital']]
status = [st for st in paises['status']]

dados_filtrados_countries = {
    'Area': area_countries,
    'Capital': capital_countries,
    'Status': status
}

paises_novo_2 = pd.DataFrame(dados_filtrados_countries)

# Transformação e seleção de colunas adicionais
colunas = ['name', 'currencies', 'region', 'latlng', 'borders', 'area']
novo_dataframe = paises[colunas].copy()

def trocar_moeda(currencies):
    if isinstance(currencies, dict):
        for moeda in currencies:
            name = currencies[moeda].get('name', '')
            symbol = currencies[moeda].get('symbol', '')
            return f"{name} ({symbol})"
    return None

novo_dataframe['currency'] = novo_dataframe['currencies'].apply(trocar_moeda)
nome = [pais['common'] for pais in novo_dataframe['name']]
novo_dataframe['name'] = nome

columns_to_select = ['name', 'currency', 'region', 'latlng', 'borders', 'area']
paises_novo2 = novo_dataframe[columns_to_select]

# Função para ajustar tipos de dados complexos
def ajustar_tipos_de_dados(df):
    def processar_valor(valor):
        if isinstance(valor, list):
            return ', '.join(str(v) for v in valor)
        elif isinstance(valor, dict):
            return str(valor)
        else:
            return valor

    def processar_coluna(coluna):
        return coluna.apply(processar_valor)

    return df.apply(processar_coluna)

# Função para salvar no banco de dados SQLite
def salva_bd(df, nome_tabela):
    conn = sqlite3.connect('projetofinal.db')
    df = ajustar_tipos_de_dados(df)
    df.to_sql(nome_tabela, conn, if_exists='replace', index=False)
    conn.close()
    return True

# Salvando os DataFrames no banco de dados
salva_bd(paises_novo, 'paises_informacoes_basicas')
salva_bd(paises_novo_2, 'paises_informacoes_adicionais')
salva_bd(paises_novo2, 'paises_moedas')

# Função para listar as tabelas no banco de dados SQLite
def tabelas_bd():
    conn = sqlite3.connect('projetofinal.db')
    cursor = conn.cursor()
    cursor.execute("SELECT name FROM sqlite_master WHERE type='table';")
    tables = cursor.fetchall()
    conn.close()
    return [table[0] for table in tables]

# Função para carregar dados de uma tabela do banco de dados SQLite
def carrega_bd(nome_tabela):
    conn = sqlite3.connect('projetofinal.db')
    query = f"SELECT * FROM {nome_tabela};"
    df = pd.read_sql_query(query, conn)
    conn.close()
    return df

# Carregando uma tabela do banco de dados para verificar os dados salvos
df_carregado_1 = carrega_bd('paises_informacoes_basicas')
df_carregado_2 = carrega_bd('paises_informacoes_adicionais')
df_carregado_3 = carrega_bd('paises_moedas')

# Usando df.head() para imprimir as primeiras linhas de cada DataFrame
df_carregado_1.head()
df_carregado_2.head()
df_carregado_3.head()


---

Com esta documentação detalhada, você deve ser capaz de configurar, executar e testar o projeto de automação de coleta de dados com a API de países. Se precisar de mais informações ou tiver dúvidas, consulte a documentação adicional fornecida ou entre em contato com os autores do projeto.
