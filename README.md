# Inteligência Territorial: Guia de Análise do Cadastro Ambiental Rural (CAR) e Conformidade Ambiental

por Theresa Rocco Pereira Barbosa* (Brasil, 2025)

**imakemapas@outlook.com.br; theresa.rocco@ufrrj.br; theresa.rocco@gmail.com*

## **Objetivo**

Integração e Processamento de Dados do CAR-BR e Consolidação de uma a Base Territorial Brasileira.

## **Estrutura do Tutorial**

### **Parte 1 - Manipulação de Dados em linguagem R**

Inicia-se com o acesso aos dados dos limites dos imóveis rurais cadastrados, extraídos a partir de arquivos compactados fornecidos pelo SICAR. O processo inclui a descompactação e renomeação dos arquivos, a correção de geometrias inválidas nos shapefiles e a posterior realização de um merge de todos os dados, com a remoção de colunas desnecessárias. A base consolidada é então reprojetada para o sistema de coordenadas recomendado pelo IBGE (SIRGAS2000, Cônica Equivalente de Albers).

Além disso, outros dados geoespaciais, como limites de biomas, Amazônia Legal, mapa de vegetação e informações sobre os municípios, passam por processos de extração, reprojeção e/ou simplificação. Em etapas posteriores, esses dados serão integrados ao CAR-BR, enriquecendo a análise territorial.

Dados utilizados:

* **CAR**- [limites dos imóveis rurals cadastrados no CAR](https://consultapublica.car.gov.br/publico/estados/downloads).

* **Biomas** - [limites dos biomas brasileiros](https://www.ibge.gov.br/geociencias/informacoes-ambientais/vegetacao/15842-biomas.html?=&t=downloads).

* **Amazônia Legal** - [limite da Amazônia Legal](https://www.ibge.gov.br/geociencias/cartas-e-mapas/mapas-regionais/15819-amazonia-legal.html?=&t=o-que-e).

* **Municípios** - [limites dos municípios brasileiros](https://www.ibge.gov.br/geociencias/organizacao-do-territorio/malhas-territoriais/15774-malhas.html?=&t=downloads).

* **Mapa Vegetação** - [cartas sobre a vegetação brasileira](https://geoftp.ibge.gov.br/informacoes_ambientais/vegetacao/vetores/escala_250_mil/).

### **Parte 2 - Manipulação de Dados em linguagem R (continuação)**

[Em breve]
