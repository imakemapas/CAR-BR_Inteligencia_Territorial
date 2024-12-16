``` {r}
# 🌍 ────────────────────────────────────────────────────────────────────────────────────────────────────── 🌍 #
# 🌍                                     THERESA ROCCO PEREIRA BARBOSA                                      🌍 #
# 🌍                               Brazilian | Geologist and Soil Scientist                                 🌍 #
# 🌍                imakemapas@outlook.com.br | theresa.rocco@ufrrj.br |  +55 24 998417085                  🌍 #
# 🌍 ────────────────────────────────────────────────────────────────────────────────────────────────────── 🌍 #
```

# **Tutorial - PARTE 1**

Este código é **parte 1** de uma série que realiza o processamento e
integração dos dados do Cadastro Ambiental Rural (CAR), bem como de
outras bases de dados geoespaciais, disponíveis para o território
brasileiro.

## Dados utilizados:

-   **CAR**

A obtenção do dado oficial dos limites dos imóveis rurals cadastrados no
CAR pode ser acessado diretamente no [site oficial do
SICAR](https://consultapublica.car.gov.br/publico/estados/downloads).

-   **Biomas**

A obtenção do dado oficial dos limites dos biomas brasileiros (2019,
escala 1:250 mil) pode ser acessado diretamente no [site oficial do
IBGE, seção informações
ambientais](https://www.ibge.gov.br/geociencias/informacoes-ambientais/vegetacao/15842-biomas.html?=&t=downloads).

-   **Amazônia Legal**

A obtenção do dado oficial do limite da Amazônia Legal pode ser acessado
diretamente no [site oficial do IBGE, seção mapas
regionais](https://www.ibge.gov.br/geociencias/cartas-e-mapas/mapas-regionais/15819-amazonia-legal.html?=&t=o-que-e).

-   **Municípios**

A obtenção do dado oficial dos limites dos municípios brasileiros pode
ser acessado diretamente no [site oficial do IBGE, seção organização do
território](https://www.ibge.gov.br/geociencias/organizacao-do-territorio/malhas-territoriais/15774-malhas.html?=&t=downloads).

-   **Mapa Vegetação**

A obtenção do dado oficial das cartas sobre a vegetação brasileira
(2023, escala 1:250 mil) pode ser acessado diretamente no [site oficial
do IBGE, seção informações
ambientais](https://geoftp.ibge.gov.br/informacoes_ambientais/vegetacao/vetores/escala_250_mil/).

## R Packages

``` {r}
library(purrr)
library(dplyr)
library(sf)
#library(terra) # pacote terra, famoso para usuários R, mostrou-se muito lento para grandes volumes de dados, quando comparado ao pacote sf. Além disso, pacote terra exigiu mais RAM que o pacote sf ao longo do processamento. Devido a isto, foi optado utilizar apenas o pacote sf nesta parte do tutorial.
```

Use `install.packages("nome do pacote)` se for a primeira vez utilizando
os pacotes necessários.

## CAR-BR

**1. Acesso aos dados do CAR**

A construção de uma base consolidada do CAR para todo o território
brasileiro começa com o acesso aos dados dos perímetros dos imóveis
rurais cadastrados. Há dois caminhos principais para obter esses dados:
o [serviço geoespacial WFS do
CAR](https://geoserver.car.gov.br/geoserver/wfs) ou o [site oficial do
SICAR](https://consultapublica.car.gov.br/publico/estados/downloads),
onde os arquivos estão organizados por unidade federativa.

Neste tutorial, optaremos pela segunda alternativa, que resulta no
download de 27 pastas compactadas, cada uma contendo os limites do CAR
para uma unidade federativa.

*Acesso: 10 de dezembro, 2024*

**2. Descompactar e Renomear**

Após baixar os arquivos compactados dos 27 Estados, o próximo passo é
descompactá-los e organizá-los com nomes padronizados, preparando-os
para a integração posterior.

Os dados baixados estão armazenados na pasta indicada por
`diretorio_entrada`, enquanto os arquivos descompactados e renomeados
serão salvos na pasta especificada por `diretorio_entrada`.

``` {r}
diretorio_entrada  <- "../dados_brutos/CAR_zipped"
diretorio_saida    <- "../dados_limpos/CAR_unzipped"

# Criar o diretório de saída (outp) caso não exista
if (!dir.exists(diretorio_saida)) {
  dir.create(diretorio_saida, recursive = TRUE)
}
```

Por fim, realiza-se uma interação para descompactar e renomear os
arquivos nas pastas listadas:

``` {r}
# Lista todos os arquivos .zip no diretório de entrada com o caminho completo
pastas_zip <- list.files(diretorio_entrada, pattern = "\\.zip$", full.names = TRUE)

# Itera sobre cada arquivo .zip encontrado
for (pasta_zip in pastas_zip) {
  
  nome_original <- basename(pasta_zip)
  nome_base     <- sub("\\.zip$", "", nome_original)

  # Cria uma pasta temporária para extrair o conteúdo do arquivo .zip
  pasta_temporaria <- tempfile(pattern = nome_base, 
                               tmpdir = diretorio_saida)
  
  # Extrai o conteúdo do arquivo .zip para a pasta temporária
  unzip(pasta_zip, exdir = pasta_temporaria)

  # Lista todos os arquivos extraídos na pasta temporária
  itens_subpasta <- list.files(pasta_temporaria, full.names = TRUE)
  
  # Itera sobre cada arquivo extraído
  for (item in itens_subpasta) {
    

    extensao_arquivo <- tools::file_ext(item)
    base_item        <- tools::file_path_sans_ext(basename(item))
    novo_nome        <- paste0(base_item, "_", nome_base, ".", extensao_arquivo)
    file.rename(item, file.path(diretorio_saida, novo_nome))
  }

  # Remove a pasta temporária
  unlink(pasta_temporaria, recursive = TRUE)
}
```

**3. Verificar e Corrigir Geometrias Inválidas**

``` {r}
# Lista todos os arquivos .shp no diretório de saída
arquivos_shp <- list.files(diretorio_saida,
                           pattern = "\\.shp$",
                           full.names = TRUE)

# Função para Verificar e Corrigir Geometrias Inválidas em Arquivos shapefile
verificar_corrigir_sf_shp <- function(arquivo_shp) {
  
  nome_original           <- tools::file_path_sans_ext(basename(arquivo_shp))
  novo_nome               <- file.path(dirname(arquivo_shp),
                                       paste0(nome_original, "_valid.shp"))
  
  cat("Processando arquivo:", arquivo_shp, "\n")
  
  # Lê o arquivo shapefile como um objeto sf
  dados_sf                <- sf::st_read(arquivo_shp, quiet = TRUE)
  
  dados_validos           <- sf::st_make_valid(dados_sf)
  dados_validos$is_valid  <- sf::st_is_valid(dados_validos)
  geometrias_invalidas    <- dados_validos |> dplyr::filter(!is_valid)
  
  # Verifica se há geometrias inválidas
  if (nrow(geometrias_invalidas) > 0) {
    cat("Geometrias inválidas encontradas:", nrow(geometrias_invalidas), "\n")
    
    # Se houuver geometrias inválidas, exibe os IDs ("cod_imovel") correspondentes
    if ("cod_imovel" %in% names(geometrias_invalidas)) {
      cat("IDs dos inválidos (cod_imovel):\n")
      print(geometrias_invalidas$cod_imovel)
    }
    
    # Remove as geometrias inválidas do conjunto de dados
    dados_validos <- dados_validos |> dplyr::filter(is_valid)
  } else {
    cat("Nenhuma geometria inválida encontrada no arquivo.\n")
  }
  
  # Salva o arquivo corrigido com o 'novo nome' definido
  sf::st_write(dados_validos, 
               novo_nome, 
               quiet = FALSE, 
               delete_layer = TRUE)
  
  cat("Arquivo salvo como:", novo_nome, "\n")
  return(novo_nome)
}

# Aplica a função verificar_corrigir_sf_shp a cada arquivo .shp encontrado
arquivos_corrigidos <- purrr::map(arquivos_shp, verificar_corrigir_sf_shp)
```

CARs com geometria inválida que foram removidos:

"GO-5208004-D741A39C89064521BAB78D6DBF4D5BDE"

"GO-5214606-E0CA0960660B4259B70846FCD876797A"

"MA-2111300-045ECDCB05174B03A89C8081BA13A1BA"

"MG-3137007-7C2DA1C7F33A49B6AE13D8B9F02232F2"

"MG-3100906-EE5933BD8D674441B0D81E8DC4A6BFE6"

"MG-3171006-F7979C057B3D444FA8CE921650BE83FF"

"MG-3152808-3A91B1B697564A719D8E4052F83F1B78"

"MG-3134004-6F76C0D994B04AC393864A313AACA275"

"MG-3138302-B4F022F921A0496FABC6C105D7C46A7D"

"PA-1502764-A09298D7DE304A30818CE445176FBCEE"

"RS-4313409-0089D72927044FF48146AE9FFAAEB8B2"

"RS-4306601-540BA89BCBAD45D5A249E90666E5D77B"

"SP-3539301-46831C5BE3D04C8A88133C1E453C15CE"

**4. Mesclar - Merge**

``` {r} 
sf_mesclado <- purrr::map_df(list.files(diretorio_saida, pattern = "_valid\\.shp$", full.names = TRUE), sf::st_read)
```

``` {r}
# Verificar a validade das geometrias no objeto mesclado
verif_geom_valida <- sf::st_is_valid(sf_mesclado)
table(verif_geom_valida)
```

**5. Remover colunas indesejadas**

``` {r}
names(sf_mesclado)
str(sf_mesclado)

sf_mesclado <- sf_mesclado[, !(names(sf_mesclado) %in% c("nom_tema", 
                                                         "cod_tema", 
                                                         "num_area",
                                                         "municipio",
                                                         "des_condic",
                                                         "cod_estado",
                                                         "is_valid"))
                           ]

names(sf_mesclado)
```

**6. Reprojetar**

-   **Projeção recomendada pelo IBGE**

A projeção Cônica Equivalente de Albers com o datum horizontal
SIRGAS2000 é recomendada pelo Instituto Brasileiro de Geografia e
Estatística (IBGE) para preservação e cálculo de áreas no território
nacional.

Referência: [Informações técnicas e legais para a utilização dos dados
publicados (IBGE,
2023).](https://biblioteca.ibge.gov.br/index.php/biblioteca-catalogo?view=detalhes&id=2101998).
Acesso em novembro de 2024.

``` {r}
crs_final <- 'PROJCS["Conica_Equivalente_de_Albers_Brasil",
    GEOGCS["GCS_SIRGAS2000",
    DATUM["D_SIRGAS2000",
    SPHEROID["Geodetic_Reference_System_of_1980",6378137,298.2572221009113]],
    PRIMEM["Greenwich",0],
    UNIT["Degree",0.017453292519943295]],
    PROJECTION["Albers"],
    PARAMETER["standard_parallel_1",-2],
    PARAMETER["standard_parallel_2",-22],
    PARAMETER["latitude_of_origin",-12],
    PARAMETER["central_meridian",-54],
    PARAMETER["false_easting",5000000],
    PARAMETER["false_northing",10000000],
    UNIT["Meter",1]]'
```

``` {r}
#crs atual
sf::st_crs(sf_mesclado)
```

``` {r}
sf_mesclado_prj <- sf::st_transform(sf_mesclado,
                                  crs = crs_final)
remove(sf_mesclado)
```

**7. Exportar**

O objeto final é muito grande para ser exportado como shapefile, devido
à limitação de tamanho desse formato (máximo de 2 GB). Como
alternativas, é possível utilizar formatos como GeoJSON ou GPKG, por
exemplo. Apesar de o GeoJSON ser excelente para visualização web e
interoperabilidade entre diferentes plataformas, o **formato GPKG** foi
optado, pois oferece maior compactação e eficiência de armazenamento em
comparação ao GeoJSON e este tutorial se propõe a realizar o
processamento diretamente em máquina local.

``` {r}
sf::st_write(sf_mesclado_prj,
             file.path(diretorio_saida, "CAR_merged.gpkg"),
             delete_layer = TRUE,
             quiet = FALSE)

# sf::st_write(sf_mesclado_prj,
#              file.path(diretorio_saida, "CAR_merged.geojson"),
#              delete_layer = TRUE,
#              quiet = FALSE)

# sf::st_write(sf_mesclado_prj,
#              file.path(diretorio_saida, "CAR_merged.shp"),
#              delete_layer = TRUE,
#              quiet = FALSE)
```

------------------------------------------------------------------------

## OUTRAS CAMADAS - IBGE

Para cada imóvel rural na base CAR-BR, serão atribuídos: o bioma
correspondente, a indicação de presença ou não na Amazônia Legal e
informações de sua organização no território.

*Acesso: 10 de dezembro, 2024*

**1. Acesso**

Utilizamos os dados dos limites de biomas, Amazônia Legal e municípios,
obtidos conforme descrito a seguir:

-   **Biomas**

A obtenção do dado oficial dos limites dos biomas brasileiros pode ser
acessado diretamente no [site oficial do IBGE, seção informações
ambientais](https://www.ibge.gov.br/geociencias/informacoes-ambientais/vegetacao/15842-biomas.html?=&t=downloads).
Ao acessar o site, certifique-se de baixar o dado mais recente,
referente ao ano de **2019**, com escala de **1:250 mil**, para garantir
a precisão dos dados na análise.

-   **Amazônia Legal**

A obtenção do dado oficial do limite da Amazônia Legal pode ser acessado
diretamente no [site oficial do IBGE, seção mapas
regionais](https://www.ibge.gov.br/geociencias/cartas-e-mapas/mapas-regionais/15819-amazonia-legal.html?=&t=o-que-e).

A Amazônia Legal corresponde à área de atuação da Superintendência de
Desenvolvimento da Amazônia -- SUDAM delimitada em consonância ao Art.
2o da Lei Complementar n. 124, de 03.01.2007.

-   **Municípios**

A obtenção do dado oficial dos limites dos municípios brasileiros pode
ser acessado diretamente no [site oficial do IBGE, seção organização do
território](https://www.ibge.gov.br/geociencias/organizacao-do-territorio/malhas-territoriais/15774-malhas.html?=&t=downloads).

**2. Descompactar e Reprojetar**

Após o download, esses dados serão descompactados e reprojetados.

**Definir os diretórios de entrada e saída**

Os dados baixados estão armazenados na pasta indicada por
`diretorio_entrada`, enquanto os arquivos descompactados e renomeados
serão salvos na pasta especificada por `diretorio_saida`.

``` {r}
diretorio_entrada  <- "../dados_brutos"
diretorio_saida    <- "../dados_limpos"
```

``` {r}
crs_final <- 'PROJCS["Conica_Equivalente_de_Albers_Brasil",
    GEOGCS["GCS_SIRGAS2000",
    DATUM["D_SIRGAS2000",
    SPHEROID["Geodetic_Reference_System_of_1980",6378137,298.2572221009113]],
    PRIMEM["Greenwich",0],
    UNIT["Degree",0.017453292519943295]],
    PROJECTION["Albers"],
    PARAMETER["standard_parallel_1",-2],
    PARAMETER["standard_parallel_2",-22],
    PARAMETER["latitude_of_origin",-12],
    PARAMETER["central_meridian",-54],
    PARAMETER["false_easting",5000000],
    PARAMETER["false_northing",10000000],
    UNIT["Meter",1]]'
```

**Função para descompactar e reprojetar dados**

``` {r}
descompactar_reprojetar_sf_shp <- function(nome_arquivo_zip,
                                           pasta_saida,
                                           nome_shp_original,
                                           nome_shp_projetado) {
  
  diretorio_saida   <- file.path(diretorio_saida, pasta_saida)
  
  if (!dir.exists(diretorio_saida)) {
    dir.create(diretorio_saida, recursive = TRUE)
  }

  # Descompactar o arquivo ZIP
  arquivo_zip       <- file.path(diretorio_entrada, nome_arquivo_zip)
  unzip(arquivo_zip, exdir = diretorio_saida)

  # Carregar e reprojetar o arquivo vetorial
  caminho_shp       <- file.path(diretorio_saida, nome_shp_original)
  
  vetor             <- sf::st_read(caminho_shp, quiet = FALSE)
  vetor_prj         <- sf::st_transform(vetor, crs = crs_final)

  # Salvar o arquivo reprojetado
  caminho_saida_prj <- file.path(diretorio_saida, nome_shp_projetado)
  
  sf::st_write(vetor_prj,
               caminho_saida_prj,
               delete_layer = TRUE,
               quiet = FALSE)
}
```

**Processar os diferentes conjuntos de dados**

``` {r}
descompactar_reprojetar_sf_shp(
  nome_arquivo_zip   = "Biomas_250mil.zip",
  pasta_saida        = "biomas_unzipped",
  nome_shp_original  = "lm_bioma_250.shp",
  nome_shp_projetado = "bioma_prj.shp"
)

descompactar_reprojetar_sf_shp(
  nome_arquivo_zip   = "Limites_Amazonia_Legal_2022_shp.zip",
  pasta_saida        = "amazonia_legal_unzipped",
  nome_shp_original  = "Limites_Amazonia_Legal_2022.shp",
  nome_shp_projetado = "amazonia_legal_prj.shp"
)

descompactar_reprojetar_sf_shp(
  nome_arquivo_zip   = "BR_Municipios_2022.zip",
  pasta_saida        = "municipios_unzipped",
  nome_shp_original  = "BR_Municipios_2022.shp",
  nome_shp_projetado = "municipios_prj.shp"
)
```

## FITOFISIONOMIAS, AMAZÔNIA LEGAL E ART.12 DO CODIGO FLORESTA

Os percentuais de Reserva Legal (RL) na Amazônia Legal não dependem dos
limites do bioma Amazônia ou do bioma Cerrado, mas sim das
fitofisionomias presentes no imóvel.

O Art. 12 do Código Florestal determina que os percentuais mínimos de RL
são:

-   80% para áreas de florestas.
-   35% para áreas de cerrado.
-   20% para áreas de campos gerais.

Para imóveis com mais de uma fitofisionomia, os percentuais devem ser
aplicados *separadamente* para cada formação.

**1. Acesso**

Utilizamos os dados dos limites de biomas, Amazônia Legal e municípios,
obtidos conforme descrito a seguir.

-   **Mapa Vegetação**

A obtenção do dado oficial das cartas sobre a vegetação brasileira pode
ser acessado diretamente no [site oficial do IBGE, seção informações
ambientais](https://geoftp.ibge.gov.br/informacoes_ambientais/vegetacao/vetores/escala_250_mil/).
Ao acessar o site, certifique-se de baixar o dado mais recente,
referente ao ano de **2023**, com escala de **1:250 mil**, para garantir
a precisão dos dados na análise.

*Acesso: 10 de dezembro, 2024*

**2. Descompactar e Reprojetar**

``` {r}
descompactar_reprojetar_sf_shp(
  nome_arquivo_zip   = "vege_area.zip",
  pasta_saida        = "fitofisionomia_unzipped",
  nome_shp_original  = "vege_area.shp",
  nome_shp_projetado = "fitofisionomia_prj.shp"
)
```

**3. Simplificar Mapa Vegetação - Fitofisionomias (legenda_1)**

-   **Leitura**

``` {r}
mapa_veg <- sf::st_read(file.path(diretorio_saida, "fitofisionomia_unzipped/fitofisionomia_prj.shp"))
```

-   **Verificar as colunas**

``` {r}
print(names(mapa_veg))
```

-   **Calcular a frequência da coluna "legenda_1"**

``` {r}
freq_leg1 <- table(mapa_veg$legenda_1)
print(freq_leg1)
```

-   **Simplificação**

``` {r}
mapa_veg$legenda_1 <- ifelse(grepl("Floresta",
                                        mapa_veg$legenda_1, 
                                        ignore.case = TRUE), "floresta",
                                  
                                  ifelse(grepl("água",
                                               mapa_veg$legenda_1,
                                               ignore.case = TRUE), "agua_cont",
                                        
                                        ifelse(grepl("Savana",
                                                     mapa_veg$legenda_1,
                                                     ignore.case = TRUE), "savana",
                                               
                                               ifelse(grepl("Contato",
                                                            mapa_veg$legenda_1,
                                                            ignore.case = TRUE), "ecotono",
                                                      
                                                      ifelse(grepl("Pioneira",
                                                                   mapa_veg$legenda_1,
                                                                   ignore.case = TRUE), "pioneira",
                                                             
                                                             ifelse(grepl("Estepe",
                                                                   mapa_veg$legenda_1,
                                                                   ignore.case = TRUE), "estepe",
                                                                   
                                                                   ifelse(grepl("Capinara",
                                                                                mapa_veg$legenda_1,
                                                                                ignore.case = TRUE), "capinara",
                                                             
                                                             mapa_veg$legenda_1)))))))
```

``` {r}
freq_leg1 <- table(mapa_veg$legenda_1)
print(freq_leg1)
```

-   **Dissolver**

Dissolver combina todas as geometrias que têm o mesmo valor para a(s)
variável(eis) especificada(s) com o argumento `group_by`

Geometrias inválidas ou problemas com valores nulos em colunas do
SpatVector podem gerar problemas

*Verificar e corrigir polígonos*

``` {r}
any(is.na(mapa_veg$legenda_1))

validity <- sf::st_is_valid(mapa_veg)
table(validity)
```

``` {r}
mapa_veg <- sf::st_make_valid(mapa_veg)
```

``` {r}
any(is.na(mapa_veg$legenda_1))

validity <- sf::st_is_valid(mapa_veg)
table(validity)
```

*Mapa Vegetação - Fitofisionomias final*

``` {r}
# Dissolver geometrias com base na coluna "legenda_1"
mapa_veg_dissolved <- mapa_veg |> 
  dplyr::group_by(legenda_1) |> 
  dplyr::summarize(geometry = sf::st_union(geometry), .groups = "drop")
```

**4. Exportar**

``` {r}
sf::st_write(mapa_veg, 
             file.path(diretorio_saida, "fitofisionomia_unzipped/fitofisionomia_prj_final.shp"), 
             delete_layer = TRUE)
```

**`<mark>`{=html}Continua na PARTE 2 deste tutorial`</mark>`{=html}**

``` {r}
rmarkdown::render("Parte_1_Integracao_Processamento_CAR_BR.Rmd",
                  output_format = "github_document")
```
