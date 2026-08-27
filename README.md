
## Legenda: Esse output indica que o arquivo .Rmd será renderizado (convertido) para .md para ir para o Github, através do knitr (pacote do R que serve para executar o código e gerar o documento final).

Assim, conseguimos misturar: texto, código R, resultados, figuras e
tabelas, o que é ótimo para projetos científicos.

<!-- README.md is generated from README.Rmd. Please edit that file -->

## Legenda:

- collapse = true = aparecer o código e o resultado juntos (bom para
  organização).
- warning = false = não mostre avisos quando o este bloco de código for
  executado.
- message = false = não mostrar mensagens dos pacotes carregados que não
  fazem parte da análise de dados.  
- eval = false = não execute o código deste bloco, apenas mostre o
  código no documento.

# INTEGRAÇÃO DE MODELOS DE APRENDIZADO DE MÁQUINA PARA ANÁLISE DE ANOMALIAS DE XCO<sub>2</sub> e XCH<sub>4</sub> NA ZONA COSTEIRA BRASILEIRA

<!-- badges: start -->

<!-- badges: end -->

## 👨‍🔬 Autores

- **Amanda de Oliveira Gravena**  
  Mestranda em Agronomia (Ciência do Solo) - FCAV/Unesp  
  Email: <amanda.gravena@unesp.br>

- **Ms. Witória de Oliveira Araújo**  
  Doutoranda em Agronomia (Ciência do Solo) - FCAV/Unesp  
  Email: <witoria.araujo@unesp.br>

- **Mr. Luís Miguel da Costa**  
  Doutorando em Agronomia (Ciência do Solo) - FCAV/Unesp  
  Email: <lm.costa@unesp.br>

- **Prof. Dr. Alan Rodrigo Panosso**  
  Coorientador — Departamento de Ciências Exatas - FCAV/Unesp  
  Email: <alan.panosso@unesp.br>

## Carregando os pacotes

``` r
library(tidyverse)
library(ggpubr)
library(terra)
library(geobr)
library(vegan)
library(sf)
library(ggplot2)
```

## Legenda:

Ao carregar pacotes, carregamos as funções das diferentes bibliotecas.

- tidyverse: coleção de pacotes para ciências de dados. Inclui: dplyr,
  ggplot2, readr, tibble, string etc.
- geobr: Baixa mapas oficiais do IBGE, não precisando baixar shapefiles
  manualmente.
- sf: Pacote mais importante para os dados espaciais. Permite: ler
  shapefiles, transformar coordenadas, fazer recortes espaciais e unir
  polígonos.
- ggpubr: Facilita gráficos científicos.
- dplyr: Manipulação de tabelas. Exemplos: filter(), mutate(), select(),
  summarise().
- ggplot2: Construção de gráficos.

OBS: não é preciso carregar os que já estão dentro de tidyverse, mas
carregamos.

## Extrair os shapes da costa brasileira

``` r
brasil <- geobr::read_country(showProgress = FALSE, year = 2020)
biomes <- geobr::read_biomes(showProgress = FALSE, year = 2019)
```

## Legenda:

- O geobr baixa automaticamente o limite político do Brasil através do
  read country. Temos que colocar o ano a partir de agora, devido a uma
  atualização do R.

- através do read biomes, baixamos todos os biomas brasileiros.Eles
  possuem:nome; geometria;projeção.

## Resumo do objeto `biomes`

``` r
biomes
```

## Legenda:

Esse comando serve para mostrar o resumo do objeto biomes. Assim,
podemos conferir se os dados foram carregados corretamente antes de
seguir a análise. Nele, podemos ver a classe do objeto (sf), número de
linhas (polígonos), número de colunas, nomes das variáveis e sistema de
referência das coodenadas (CRS), por exemplo.

## Listando os biomas individuais (distintos)

``` r
biomes |> distinct(name_biome)
```

## Legenda:

Esse comando lista apenas os nomes distindos dos biomas, sem repetições.
Ele serve para verificar quais biomas existem na base de dados do geobr.

## Visualização dos mapas, Brasil e Biomas

``` r
# Mapa 1 — Contorno do Brasil
map_country <- ggplot(brasil) +
  geom_sf(fill = "#2d6a4f", color = "#95d5b2", linewidth = 0.8) +
  labs(
    title    = "Brasil",
    subtitle = "Território Nacional",
    caption  = "Fonte: IBGE via geobr"
  ) +
  theme_void(base_family = "serif") +
  theme(
    plot.background  = element_rect(fill = "#0d1b2a", color = NA),
    panel.background = element_rect(fill = "#0d1b2a", color = NA),
    plot.title       = element_text(color = "#95d5b2", size = 22, face = "bold",
                                    hjust = 0.5, margin = margin(t = 16, b = 4)),
    plot.subtitle    = element_text(color = "#74c69d", size = 13,
                                    hjust = 0.5, margin = margin(b = 8)),
    plot.caption     = element_text(color = "#52b788", size = 8,
                                    hjust = 1, margin = margin(b = 10, r = 12)),
    plot.margin      = margin(20, 20, 10, 20)
  )

## Tiramos os itens do Mapa 1, pois não eram úteis para o momento, mas o que ele fazia: Esse bloco possui a finalidade de criar um mapa do contorno do Brasil para visualização, sem alterar nenhum dado. Não analisa nem faz recortes espaciais. Apenas constrói uma figura. 

#Esse bloco possui a finalidade de criar um mapa do contorno do Brasil para visualização, sem alterar nenhum dado. Não analisa nem faz recortes espaciais. Apenas constrói uma figura. 

#map_country <- ggplot(brasil) + aqui, o ggplot() inicia a construção do gráfico utilizando o objeto brasil, que contém o limite territorial do país, armazenando no objeto map_country. 

#A função geom_sf() desenha a geometria do objeto espacial, definindo cores e espessuras de bordas. 

#O theme_void() remove elementos que não são necessários em um mapa, como: eixos; linhas de grade; marcações.

#A função theme() altera o aspecto visual do gráfico.

#Ou seja, esse bloco de código construía uma figura do contorno do Brasil, aplicando uma série de personalizações visuais, como cores, títulos, fonte, fundo e margens.

# Paleta para os biomas
biome_colors <- c(
  "Amazônia"              = "#1b4332",
  "Cerrado"               = "#d4a017",
  "Mata Atlântica"        = "#40916c",
  "Caatinga"              = "#e76f51",
  "Pampa"                 = "#a7c957",
  "Pantanal"              = "#4895ef",
  "Sistema Costeiro"      = "#90e0ef"
  # "Sistema Costeiro-Marinho" = "#90e0ef"
)

## Legenda: definindo as cores dos biomas.

#A função 'c()' significa combine.Neste caso, ela reúne vários pares de informações:nome do bioma;código da cor.
#O símbolo '<-' significa atribuir.
#Ou seja, o R está criando um objeto chamado biome_colors.
#Esse objeto armazenará a relação entre o nome de cada bioma e sua respectiva cor.

# Mapa 2 — Biomas: 

## Legenda: O bloco constrói o mapa dos biomas brasileiros. A lógica é sempre a mesma: inicia um gráfico, desenha os biomas, define as cores, adiciona título e legenda, ajusta a aparência e exibe o mapa. 

map_biomes <- ggplot(biomes) + ## Legenda: A função ggplot() inicia a construção do gráfico com o objeto biomes, que contém os limites espaciais dos biomas brasileiros, armazenando o gráfico em map_biomes. No momento, o gráfico ainda está vazio, pois apenas foi definido o conjunto de dados que será utilizado. 
  
  geom_sf(aes(fill = name_biome), color = "#1a1a2e", linewidth = 0.4) +
  scale_fill_manual(
    values = biome_colors,
    name   = "Bioma",
    # na.value = "#555555"
  ) + ## : A função geom_sf() desenha as geometrias presentes dentro do objeto biomes. Como biomes é um objeto espacial (sf), cada bioma é representado por um polígono no mapa. 
  
#Já a função aes(), de estética, informa as características visuais do gráfico associadas aos dados. Nesse caso, fill=name_biome, ou seja, cada bioma será preenchido por uma cor diferente, de acordo com a variável name_biome. 
  
#Já os itens: scale_fill_manual = permite definir manualmente as cores utilizadas, o values = biomes_colors utiliza a paleta criada anteriormente. Os demais itens são intuitivos. 
  
  labs(
    title    = "Biomas do Brasil",
    subtitle = "Distribuição dos biomas continentais e costeiros",
    caption  = "Fonte: IBGE via geobr"
  ) + ## Legenda: A função labs() adiciona informações descritivas do gráfico, nesse caso título da figura, subtítulo e fonte dos dados. Essas informações ajudam a identificar o conteúdo do mapa. 
  
  theme_void(base_family = "serif") +
  theme(
    # plot.background  = element_rect(fill = "#0d1b2a", color = NA),
    # panel.background = element_rect(fill = "#0d1b2a", color = NA),
    # plot.title       = element_text(color = "#f4d35e", size = 22, face = "bold",
    #                                 hjust = 0.5, margin = margin(t = 16, b = 4)),
    # plot.subtitle    = element_text(color = "#f4a261", size = 11,
                                    # hjust = 0.5, margin = margin(b = 8)),
    plot.caption     = element_text(size = 8,
                                    hjust = 1, margin = margin(b = 10, r = 12)),
    legend.title     = element_text(color = "#ffffff", size = 10, face = "bold"),
    legend.text      = element_text(size = 9),
    legend.position  = "right",
    plot.margin      = margin(20, 20, 10, 20)
  )

## Legenda: O theme_void() remove os elementos gráficos que não são úteis para mapas, como eixos, grades e marcações. Já o base_family = "serif" define que os textos utilizarão uma fonte do tipo serifada.Os demais itens definem a estética do gráfico. 

#As linhas de "plot." alteravam o fundo do gráfico e a aparência do título e do subtítulo, porém, não achamos mais necessário. 

# print(map_country)
print(map_biomes)

## Legenda: Depois de todas as etapas de construção, o comando print() exibe o mapa os biomas na tela, pois já construímos o objeto "map_biomes". 
```

## Ler e interpretar os dados em “data-raw”

``` r
df <- read_rds("data-raw/data-set-xco2-br-002.rds") |> 
  mutate(year = year - 20)

## Legenda: A função read_rds() lê o arquivo que contém os dados de XCO₂. O conteúdo do arquivo será armazenado em df (dataframe), que será utilizado nas etapas seguintes. 

## Como estamos apenas lendo e corrigindo os dados raw, por enquanto, chamaremos de df. 

## Já a função mutate() é utilizada para modificar colunas de um conjunto de dados. Nesse, ela altera a coluna year, retirando 20 anos, pois notamos pelo nome dos arquivos que esse dado está com esse erro. 
```

## Resumo do data-set

``` r
# glimpse(df |> 
#           filter(xco2_quality_flag == 0))
```

\##Legenda: Aqui ocorre a inspeção da estrutura do conjunto ed dados
antes de continuar a análise (glimpse: olhadela).

\#O operador \|\> envia o objeto df para a próxima função, tipo “pegue a
tabela df e aplique a função seguinte”.

O df é a tabela de dados que foi lida anteriormente. Pegamos ela e
filtramos xco2_quality_flag = 0 apenas. 0 → dado considerado válido ou
recomendado para uso.

## Listando os anos individuais (distintos)

``` r
# df |> distinct(year)
```

## “Utilize a tabela df na função seguinte.” → distinct() seleciona apenas os valores únicos de uma variável, removendo as repetições. Ou seja, cada ano aparece apenas uma vez. Esse comando é uma forma rápida de verificar quais anos estão presentes na base de dados depois da correção anterior.

## Plotar o shape do brasil e as coordenadas de pontos amostrados dentro do objeto df

``` r
biomes |> 
  ggplot() +
  geom_sf(aes(fill = name_biome), color = "#1a1a2e", linewidth = 0.4) +
  scale_fill_manual(
    values = biome_colors,
    name   = "Bioma",
    # na.value = "#555555"
  ) + theme_minimal() +
  geom_point( data = df |> 
          filter(xco2_quality_flag == 0) |> 
            sample_n(1000)
              , aes(longitude, latitude), color="gray")
```

## Legenda:

Esse trecho responde à pergunta: Os pontos amostrados de XCO₂ realmente
estão localizados sobre o território brasileiro e nos biomas esperados?

------------------------------------------------------------------------

Como? O código cria um mapa dos biomas brasileiros e, sobre esse mapa,
desenha uma amostra de pontos do conjunto de dados df.

- O objeto biomes contém os limites espaciais dos biomas brasileiros.
  Ele será utilizado como base para construir o mapa.

- A função ggplot() inicia a construção do gráfico. Como o objeto biomes
  já foi enviado pelo operador \|\>, ele será a base do mapa.

- A função geom_sf() desenha os polígonos dos biomas. fill = name_biome
  faz com que cada bioma seja preenchido com uma cor diferente, de
  acordo com a variável name_biome. Aqui o ggplot2 utiliza a paleta de
  cores criada anteriormente, definida em biome_colors.

- Agora, utilizamos o conjunto de dados df novamente, mantemos o filtro
  de quality flag e sorteamos uma amostra de 1000 dados.

- Depois, a função geom_point() adiciona os pontos no mapa.

- Esse tipo de inspeção é uma etapa de controle de qualidade espacial
  antes de realizar recortes, integrações ou modelagens.O objetivo
  principal é verificar visualmente se as coordenadas geográficas dos
  dados estão corretamente posicionadas sobre a área de estudo.

## Transformar o df de XCO2 em objeto espacial (sf)

``` r
# df_sf <- df |> 
#   filter(xco2_quality_flag == 0) |> 
#   st_as_sf(coords = c("longitude", "latitude"), crs = 4326, remove = FALSE)
```

## Legenda:

Até aqui, estávamos trabalhando com uma tabela comum.A partir desse
ponto, passamos a trabalhar com um objeto espacial, que permite realizar
operações geográficas, como recortar pela costa brasileira.

## Por que transformar em um objeto espacial?

- Antes desse código, o df é apenas uma tabela.
- Para o leitor, imaginamos esses números como locais no mapa (latitude
  e longitude). Já para o R, são apenas colunas numéricas. É justamente
  isso que st_as_sf() faz.A função st_as_sf() converte uma tabela comum
  em um objeto espacial. É ela que permite utilizar todas as funções do
  pacote sf.
- Depois dele, cada linha da tabela passa a possuir uma geometria. Para
  isso, a coluna geometry é criada automaticamente.
- Aqui, será criado um novo objeto, chamado df_sf. O final sf significa
  simple features. Filtramos de novo os dados de boa qualidade para não
  repetir dados já descartados.
- crs = 4326 corresponde ao sistema geográfico WGS 84, que é o padrão
  utilizado por GPS e pela maioria dos satélites.
- Quando a geometria é criada, o sf poderia remover as colunas longitude
  e latitude, pois elas já estariam representadas na coluna geometry.,
  por isso, remove = false. Elas permanecem ali para análises
  posteriores.

## Join espacial: associa cada ponto ao bioma em que ele cai (pontos fora do Brasil recebem NA em name_biome)

``` r
# Transforma os biomas para o mesmo sistema de referência (CRS) do objeto df_sf (WGS84)
biomes <- st_transform(biomes, crs = 4326)
brasil <- st_transform(brasil, crs = 4326)

# Realiza a junção espacial entre os pontos de XCO2 e os biomas
# df_biomas <- st_join(
#   df_sf,
#   biomes |>
#     select(name_biome, geometry)
# )
```

## Legenda:

Aqui, fazemos a junção espacial. Até aqui, sabemos a localização de cada
ponto df_sf (lat e long), e o objeto biomes são os polígonos. O
st_join() juntará as duas informações. O objetivo desse código é
associar cada observação de XCO₂ ao bioma em que ela está localizada.

Como?

- Primeiro, o st_transform() altera o sistema crs do bioma e do brasil.
- Depois, fazemos a junção. O objeto biomes possui várias colunas, mas
  queremos apenas name_biome e geometry.
- Já o df_sf são os pontos.
- Com o join, mesclamos, dando para cada ponto um name_biome e uma
  localização no polígono.

## Filtrar apenas pontos do Brasil

``` r
# df_brasil <- df_biomas |> 
#   filter(!is.na(name_biome))
# 
# glimpse(df_brasil)
```

## Legenda:

- df_brasil \<- cria um novo objeto chamado **df_brasil**, onde será
  armazenado o resultado do filtro.
- df_biomas é o conjunto de dados que contém os pontos de XCO₂ e o bioma
  associado a cada ponto.
- filter(!is.na(name_biome)) mantém apenas as linhas em que a coluna
  name_biome possui um valor, ou seja, apenas os pontos que pertencem a
  algum bioma brasileiro. Os pontos fora do Brasil possuem NA nessa
  coluna e são removidos.
- glimpse(df_brasil) mostra um resumo do novo conjunto de dados,
  permitindo conferir as colunas, seus tipos e algumas informações
  gerais.

## Salvar os dados na pasta data

``` r
# write_rds(
#   df_brasil,
#   "data/xco2-brasil-biomas.rds"
# )
```

\##Lendo o arquivo salvo até aqui

``` r
#df_brasil <- read_rds("data/xco2-brasil-biomas.rds")
```

``` r
##Conferir quantos pontos por bioma
# df_brasil |> 
#   st_drop_geometry() |> 
#   count(name_biome, sort = TRUE)
```

## Legenda: st_drop_geometry() temporariamente remove a coluna geometry, transformando o objeto espacial (sf) em uma tabela comum, pois queremos contar observações. Para contar, usamos a função count(), que contabilizará quantos linhas possuem para cada valor da coluna name_biome.O sort = TRUE faz com que o resultado seja ordenado do maior para o menor.

## A partir daqui, a ideia surgiu da conversa: como vamos ver apenas os locais de possíveis manguezais, sem entrarmos nas 12 milhas de área marítima do Sistema Costeiro?

## Aqui, surgiram duas ideias: Cruzamento com o shape do Brasil (Terra firme) com o shape do Sistema Costeiro ou pegamos os dados do MapBiomas focado em Manguezais e cruzamos com os dados de Sistema costeiro.

## Cruzamento com o shape de terra firme do Brasil

``` r
costeiro <- biomes |> 
  filter(name_biome == "Sistema Costeiro")
ggplot() +
  geom_sf(data = brasil, fill = "grey95", color = "grey50", linewidth = 0.3) +
  geom_sf(data = costeiro, fill = "#2c7fb8", color = NA, alpha = 0.6) +
  labs(
    title = "Brasil e Sistema Costeiro-Marinho (IBGE/geobr)",
    subtitle = "Faixa costeira usada como recorte para análise de XCO2"
  ) +
  theme_minimal() +
  theme(
    axis.text = element_blank(),
    axis.ticks = element_blank(),
    panel.grid = element_blank()
  )
```

## Legenda:

Primeiro, criou-se o objeto “costeiro”, que é o biomes, filtrando apenas
o Sistema Costeiro.

Em ggplot(), começamos a construção do mapa.

Depois disso, fazemos o desenho do Brasil. A função geom_sf() desenha o
shapefile do Brasil.

Depois, na segunda camada, preenchemos o sistema costeiro com uma cor,
também colocamos NA cor em contorno dos polígonos e aplicamos
transparência com o alpha = 0.6, podendo visualizar o mapa do Brasil por
baixo.

OBS: - axis.text = element_blank() → remove os números dos eixos. -
axis.ticks = element_blank() → remove as marcas dos eixos. - panel.grid
= element_blank() → remove a grade de fundo. \*Como se trata de um mapa
ilustrativo, esses elementos não são necessários.

## Calcular a interseção geométrica: só a parte do Sistema Costeiro que também é terra (contorno do Brasil) -\> remove a faixa marítima

``` r
costeiro_terrestre <- st_intersection(costeiro, brasil)

st_crs(costeiro)
st_crs(brasil)
```

## Legenda: A função st_crs() mostra o CRS (Coordinate Reference System) do objeto.

## Conferir se a geometria resultante faz sentido (plot rápido)

``` r
plot(st_geometry(costeiro_terrestre))
costeiro_terrestre_buffer <- sf::st_buffer(costeiro_terrestre,0.04)
plot(st_geometry(costeiro_terrestre_buffer))
```

## Converter df_brasil em objeto sf de pontos (ajuste nomes de colunas)

``` r
# pontos_brasil <- st_as_sf(
#   df_brasil,
#   coords = c("longitude", "latitude"),
#   crs = 4326,
#   remove = FALSE
# )
```

\##Esse código cria um novo objeto espacial (sf), denominado
pontos_brasil, a partir do conjunto de dados df_brasil, utilizando as
colunas de longitude e latitude para representar cada observação como um
ponto geográfico.

## Filtrar apenas os pontos que caem dentro da faixa costeira terrestre

``` r
# costeiro_terrestre <- st_set_crs(costeiro_terrestre, 4326)
# 
# pontos_costeiro_terra <- st_join(pontos_brasil, costeiro_terrestre  |>
#                                    mutate(name_biome = "Costeiro Terrestre") |>
#                        select(name_biome, geometry))
# 
# pontos_costeiro_terra_buffer <- st_join(pontos_brasil, 
#                                         costeiro_terrestre_buffer  |> 
#                                    mutate(name_biome = "Costeiro Terrestre") |> 
#                        select(name_biome, geometry))
```

## Legenda: Essa etapa: Quais pontos de XCO₂ estão dentro da faixa costeira terrestre?

- Criou-se um novo objeto, pontos_costeiro_terra, através de uma junção
  st_join(), que relaciona os pontos_brasil com o polígono
  costeiro_terrestre, que vem da intersecção st_intersection(costeiro,
  brasil).

Depois, passou a função mutate() para adicionar o name_biome “Costeiro
Terrestre” e selecionou com select apenas a coluna name_biome e
geometry.

## Voltar para data.frame puro, se precisar

``` r
# df_costeiro_terra_buffer <- pontos_costeiro_terra_buffer |> filter(name_biome.y ==  "Costeiro Terrestre")  |>  st_drop_geometry()
# 
# df_costeiro_terra <- pontos_costeiro_terra |> filter(name_biome.y ==  "Costeiro Terrestre")  |>  st_drop_geometry()
```

## Legenda: Essa etapa serve para obter apenas os pontos que pertencem à faixa costeira terrestre e transformá-los novamente em uma tabela comum.

## Verificar se eles formam a linha literânea

``` r
# df_costeiro_terra |> 
#   ggplot(aes(longitude, latitude)) +
#   geom_point()
# 
# df_costeiro_terra_buffer |> 
#   ggplot(aes(longitude, latitude)) +
#   geom_point()
```

## Salvando na pasta data

``` r
# write_rds(df_costeiro_terra_buffer,"data/xco2-costeiro-terrestre-buffer.rds")
```

\##A partir daqui, se inicia a parte de análise de dados que iniciamos.

A sequência é bem lógica:

Quantos dados ficaram após o recorte? Como esses dados estão
distribuídos no tempo? Como estão distribuídos no espaço? Como se
comporta o XCO₂? Existem diferenças entre biomas? Existe um padrão
temporal?

Lembrando, ainda não afunilamos para apenas manguezais, e sim para zona
costeira terrestre.

## Volume de dados retido

``` r
df_costeiro_terra_buffer <- read_rds("data/xco2-costeiro-terrestre-buffer.rds")
nrow(df_costeiro_terra_buffer)
```

## Legenda: Aqui, leu o arquivo e contou o número de fileiras após os recortes. Aqui, respondemos: “Depois de todos os filtros, ainda tenho uma quantidade suficiente de dados?” SIM!

# Cobertura temporal — quantos pontos por ano/mês

``` r
df_costeiro_terra_buffer |> 
  st_drop_geometry() |> 
  count(year) |>   # ajuste nome da coluna de data
  arrange(year)
```

## Legenda: Verificar quantas observações existem em cada ano e organiza em ordem crescente, para verificar se algum ano possui poucos dados ou se há anos ausentes.

``` r
# Estatística descritiva do XCO2 nessa faixa
summary(df_costeiro_terra_buffer$xco2)  # ajuste nome da coluna
```

## Legenda: O summary() calcula automaticamente: mínimo; primeiro quartil; mediana; média; terceiro quartil; máximo.Assim, verificamos: valores muito altos;

valores muito baixos;possíveis erros.

Nesse ponto, de início, apareceu o bioma Pantanal. O que é estranho,
pois não há Pantanal na zona costeira.

``` r
# costeiro <- biomes |> 
#   filter(name_biome == "Sistema Costeiro")
# ggplot() +
#   geom_sf(data = brasil, fill = "grey95", color = "grey50", linewidth = 0.3) +
#   geom_sf(data = costeiro, fill = "#2c7fb8", color = NA, alpha = 0.6) +
#   geom_point(data = df_costeiro_terra |> filter(name_biome.x == "Pantanal") |> 
#   st_drop_geometry(), aes(longitude,latitude), colour = "red") +
#   labs(
#     title = "Brasil e Sistema Costeiro-Marinho (IBGE/geobr)",
#     subtitle = "Faixa costeira usada como recorte para análise de XCO2"
#   ) +
#   theme_minimal() +
#   theme(
#     axis.text = element_blank(),
#     axis.ticks = element_blank(),
#     panel.grid = element_blank()
#   ) 
```

## Legenda: Nesse ponto, queríamos comparar a distribuição do XCO₂ entre os biomas. Nele, evidenciamos o erro (Pantanal) e o Sistema Costeiro, pois não deve aparecer pontos de Pantanal no Sistema costeiro.

## Durante a segunda junção espacial (st_join), o objeto pontos_brasil já possuía

uma coluna chamada “name_biome”, indicando o bioma em que cada ponto
estava (Amazônia, Cerrado, Mata Atlântica etc.).

Como o objeto costeiro_terrestre também recebeu uma coluna com o mesmo
nome(“name_biome”), o R renomeou automaticamente as duas colunas para
evitar conflito: a coluna original passou a se chamar “name_biome.x”,
enquanto a coluna proveniente da junção passou a se chamar
“name_biome.y”(Costeiro Terrestre).

``` r
df_costeiro_terra_buffer |> 
  st_drop_geometry() |> 
  filter(year == 2023,
         name_biome.x != "Sistema Costeiro") |> 
  # group_by(year) |> 
  ggplot(aes(x=name_biome.x, y=xco2, fill=name_biome.x)) +
  geom_boxplot() 
```

## Legenda: Boxplot é usado para comparar a distribuição do XCO₂ entre os biomas. Podemos visualizar mediana; quartis; dispersão; outliers. Excluímos Sistema Costeiro, pois não é um bioma de fato.

``` r
df_costeiro_terra_buffer |> 
  st_drop_geometry() |> 
  filter(year == 2015,
         name_biome.x != "Sistema Costeiro") |> 
  group_by(name_biome.x, month) |> 
  summarise(
    xco2 = mean(xco2)
  ) |> 
  ggplot(aes(x=month, y=xco2, color=name_biome.x) ) +
  geom_point() + geom_line()
```

## Legenda: Média mensal por bioma, agrupando por bioma e por mês, ou seja, “Qual foi o XCO₂ médio daquele bioma naquele mês?” - Resulta em um gráfico temporal. Buscamos ver se os biomas apresentam comportamento sazonal diferente.

``` r
df_costeiro_terra_buffer |> 
  st_drop_geometry() |> 
  filter(year == 2015,
         name_biome.x != "Sistema Costeiro") |> 
  mutate( class_latitude = cut(latitude, 20)) |> 
  group_by(class_latitude, month) |> 
  summarise(
    xco2 = mean(xco2)
  ) |> 
  ggplot(aes(x=month, y=xco2, color=class_latitude) ) +
  geom_point() + geom_line()
```

## Legenda: Média mensal por faixa de latitude (20 faixas). Então, calcula a média do XCO₂ para cada faixa de latitude em cada mês. O objetivo é verificar se existe um gradiente latitudinal, ou seja, “O comportamento do XCO₂ muda conforme se avança do sul para o norte do Brasil?”

## Existe uma tendência regional nos dados, e ela deve ser retirada para esse trabalho

## Análise de regressão linear simples para caracterização da tendência.

``` r
mod_trend_xco2 <- lm(xco2 ~ date, 
          data = df_costeiro_terra_buffer |> 
            mutate(
              date = make_date(year, month, day),
              date = as.numeric(date - min(date))
              ))
# mod_trend_xco2
sm <- summary.lm(mod_trend_xco2)
```

## Legenda:

A função `lm()` realiza uma regressão linear simples entre o XCO₂ e a
data, permitindo identificar a tendência temporal dos dados. A data é
convertida para uma escala numérica em dias, facilitando o ajuste da
regressão. O resultado é armazenado no objeto `mod_trend_xco2`.

A função `summary.lm()` gera o resumo estatístico do modelo de
regressão.

## Mostrando o gráfico da regressão

``` r
df_costeiro_terra_buffer |>
  # sample_n(1000) |>
  drop_na() |>
  mutate( date= make_date(year, month, day),
    year = year - min(year)
    ) |>
  ggplot(aes(x=date, y=xco2)) +
  geom_point(alpha = 0.25, size = 1) +
  # geom_point(shape=21,color="black",fill="gray") +
  geom_smooth(method = "lm",
              color = "red", linetype = "dashed",
              size=1) +
  stat_regline_equation(aes(
  label =  paste(..eq.label.., ..rr.label.., sep = "*plain(\",\")~~"))) +
  theme_minimal() +
  labs(x="Data",y=expression(paste(X[CO2]," (ppm)"))) +
  scale_x_date(date_breaks = "1 year", date_labels = "%Y")
```

## Legenda:

Esse bloco mostra graficamente a tendência temporal do XCO₂.
`geom_point()` apresenta os valores observados, enquanto
`geom_smooth(method = "lm")` adiciona a reta da regressão linear.
`stat_regline_equation()` mostra a equação da reta e o R² no gráfico.
Assim, é possível visualizar a tendência regional dos dados ao longo do
período analisado.

## retirada de tendência Propriamente dita

``` r
a_co2 <- mod_trend_xco2$coefficients[[1]]
b_co2 <- mod_trend_xco2$coefficients[[2]]

df_costeiro_terra_buffer <- df_costeiro_terra_buffer |>
  mutate(
    date= make_date(year, month, day),
    date_modif = as.numeric(date - min(date)),
    xco2_est = a_co2+b_co2*date_modif, ## estima o xco2 pela reta de regressão no dia específico
    delta = xco2_est-xco2, ## estimado menos o observado real
    xco2_detrend = (a_co2-delta) - (mean(xco2) - a_co2)
  ) |> select(-xco2_quality_flag, -path, -name_biome.y) |> 
  rename(name_biome = name_biome.x)
```

## Legenda:

Esse bloco realiza a retirada da tendência temporal do XCO₂. Primeiro,
são obtidos os coeficientes da regressão (`a_co2` e `b_co2`). Em
seguida, calcula-se o XCO₂ estimado pela reta de regressão para cada
data (`xco2_est`) e a diferença entre o valor estimado e o observado
(`delta`). Por fim, é calculado o `xco2_detrend`, que representa o XCO₂
após a remoção da tendência regional.

### PARTE AMANDA

``` r
df_costeiro_terra_buffer |>
  mutate(
    class_latitude = cut(
      latitude,10
    ),
    latitude_media = (
      as.numeric(sub("\\(([-0-9.]+),.*", "\\1", class_latitude)) +
      as.numeric(sub(".*[,]([-0-9.]+)\\]", "\\1", class_latitude))
    ) / 2
  ) |>
  group_by(year, latitude_media) |>
  summarise(
    xco2 = mean(xco2_detrend, na.rm = TRUE),
    .groups = "drop"
  ) |> 
  # filter(year > 2022) |> 
  ggplot(aes(x=latitude_media, y = xco2, fill=as_factor(year))) +
  geom_col(color="black", position = "dodge") +
  coord_flip(ylim = c(380, 386)) +
  labs(
    x = "Latitude (°)",
    y = "Concentração média de XCO2 (ppm)",
    color = "Ano"
  ) +
  theme_minimal()
```

## Legenda:

O código divide as latitudes em 10 faixas e calcula a latitude média de
cada faixa. Em seguida, agrupa os dados por ano e faixa de latitude e
calcula a média do XCO₂ sem tendência (`xco2_detrend`). O gráfico de
barras permite comparar a concentração média de XCO₂ entre as diferentes
latitudes e anos.

``` r
df_costeiro_terra_buffer |>
  mutate(
    class_latitude = cut(
      latitude, 10
    ),
    latitude_media = (
      as.numeric(sub("\\(([-0-9.]+),.*", "\\1", class_latitude)) +
      as.numeric(sub(".*[,]([-0-9.]+)\\]", "\\1", class_latitude))
    ) / 2
  ) |>
  group_by(year, latitude_media) |>
  summarise(
    xco2 = mean(xco2_detrend, na.rm = TRUE),
    .groups = "drop"
  ) |> 
  # filter(year > 2022) |> 
  ggplot(aes(x=latitude_media, y = xco2, color=as_factor(year))) +
  # geom_col(color="black", position = "dodge") +
  geom_line() +
  # geom_point() +
  coord_flip(ylim = c(380, 384)) +
  labs(
    x = "Latitude (°)",
    y = "Concentração média de XCO2 (ppm)",
    color = "Ano"
  ) +
  theme_minimal()
```

## Legenda:

O código utiliza as mesmas 10 faixas de latitude e calcula a média do
XCO₂ sem tendência para cada ano e faixa. O gráfico de linhas permite
visualizar e comparar o comportamento espacial do XCO₂ ao longo das
diferentes latitudes em cada ano.

``` r
df_costeiro_terra_buffer |>
  mutate(
    class_latitude = cut(
      latitude, 115
    ),
    latitude_media = (
      as.numeric(sub("\\(([-0-9.]+),.*", "\\1", class_latitude)) +
      as.numeric(sub(".*[,]([-0-9.]+)\\]", "\\1", class_latitude))
    ) / 2
  ) |>
  group_by(year, latitude_media) |>
  summarise(
    xco2 = mean(xco2_detrend, na.rm = TRUE),
    .groups = "drop"
  ) |> 
  # filter(year > 2022) |> 
  ggplot(aes(x=factor(latitude_media), y = xco2, fill = factor(latitude_media))) +
  geom_boxplot() + 
  coord_flip(ylim = c(380, 386)) +
  labs(
    x = "Latitude (°)",
    y = "Concentração média de XCO2 (ppm)",
    color = "Ano"
  ) +
  theme_minimal()+
  theme(
    legend.position = "none"
  ) +
  scale_fill_viridis_d()
```

## Legenda: O código divide os dados em $35$ faixas de latitude e calcula a média do XCO₂ sem tendência para cada faixa. O boxplot permite visualizar a distribuição dos valores de XCO₂ ao longo das latitudes, mostrando a variação dos dados entre as diferentes regiões latitudinais.

## Análise de cluster da série temporal

``` r
df_cluster <- df_costeiro_terra_buffer |>
  filter(year >2014) |> 
  mutate(
    class_latitude = cut(
      latitude, 115 ),
   year_month = paste(as.character(year), as.character(month), sep="_"),
    latitude_media = (
      as.numeric(sub("\\(([-0-9.]+),.*", "\\1", class_latitude)) +
      as.numeric(sub(".*[,]([-0-9.]+)\\]", "\\1", class_latitude))
    ) / 2
  ) |>
  group_by(year_month, latitude_media) |>
  summarise(
    xco2 = mean(xco2_detrend, na.rm = TRUE),
    .groups = "drop"
  ) |> 
  pivot_wider(names_from = year_month, values_from = xco2) |> 
  mutate(across(
    where(is.numeric), .fns = ~replace_na(.x, mean(.x,na.rm=TRUE))
  ))
df_cluster
```

## Legenda: Precisávamos definir grupos de latitude com comportamento temporal semelhante de XCO₂ para, posteriormente, calcular as anomalias dentro desses grupos. Para isso, os dados foram organizados em classes de latitude e agregados mensalmente, considerando o período posterior a 2014. Para cada faixa de latitude e mês, foi calculada a média de XCO₂ com a tendência regional previamente removida (XCO₂ detrend). Em seguida, os dados foram reorganizados em uma matriz, na qual cada linha representava uma faixa de latitude e cada coluna correspondia a um mês da série temporal. Os valores ausentes foram preenchidos pela média da respectiva série para possibilitar a aplicação do agrupamento.

## Matriz de correlação entre as latitudes

``` r
mc <- cor(df_cluster |> select(-latitude_media))
corrplot::corrplot(mc)
```

## Legenda: Para avaliar a similaridade temporal entre as diferentes faixas de latitude, foi calculada a matriz de correlação entre as séries mensais de XCO₂. Essa etapa permitiu verificar o grau de associação entre o comportamento temporal das diferentes regiões antes da realização do agrupamento.

``` r
da_pad<-decostand(df_cluster |> select(-latitude_media), 
                  method = "standardize",
                  na.rm=TRUE)

da_pad_euc<-vegdist(da_pad,"euclidean") 
da_pad_euc_ward<-hclust(da_pad_euc, method="ward.D")
da_pad_euc_ward$labels <- df_cluster$latitude_media
plot(da_pad_euc_ward, 
     ylab="Distância Euclidiana",
     xlab="Acessos", hang=-1,
     col="blue", las=1,
     cex=.6,lwd=1.5);box()
grupo<-cutree(da_pad_euc_ward,4)
colunas <- df_cluster |> add_column(grupo) |> 
  select(latitude_media,grupo)
```

## Legenda: Após a avaliação da correlação entre as séries, os dados foram padronizados para tornar as diferentes séries temporais comparáveis. Em seguida, foi calculada a distância euclidiana entre as séries e aplicado o agrupamento hierárquico pelo método de Ward. A partir do dendrograma obtido, foram definidos quatro grupos de faixas de latitude com comportamento temporal semelhante.

``` r
df_costeiro_terra_buffer |>
  filter(year >2014) |> 
  mutate(
    class_latitude = cut(
      latitude, 115 ),
   year_month = paste(as.character(year), as.character(month), sep="_"),
    latitude_media = (
      as.numeric(sub("\\(([-0-9.]+),.*", "\\1", class_latitude)) +
      as.numeric(sub(".*[,]([-0-9.]+)\\]", "\\1", class_latitude))
    ) / 2
  ) |> 
  left_join(colunas, by = "latitude_media") |> 
  ggplot(aes(longitude, latitude, color = as_factor(grupo))) +
  geom_point()
```

## Legenda: Após a definição dos grupos pelo agrupamento hierárquico, a classificação obtida para cada faixa de latitude foi associada às observações originais. A distribuição espacial dos grupos foi então visualizada para verificar como as regiões com comportamento temporal semelhante estavam distribuídas ao longo da faixa costeira.

``` r
# Basemap do Brasil (uma vez só, fora do pipe principal)
brasil <- read_country(year = 2020, showProgress = FALSE)

df_costeiro_terra_buffer |>
  filter(year > 2014) |> 
  mutate(
    class_latitude = cut(latitude, 115),
    year_month = paste(as.character(year), as.character(month), sep = "_"),
    latitude_media = (
      as.numeric(sub("\\(([-0-9.]+),.*", "\\1", class_latitude)) +
      as.numeric(sub(".*[,]([-0-9.]+)\\]", "\\1", class_latitude))
    ) / 2
  ) |> 
  left_join(colunas, by = "latitude_media") |> 
  ggplot() +
  geom_sf(data = brasil, fill = "grey96", color = "grey70", linewidth = 0.3) +
  geom_point(
    aes(x = longitude, y = latitude, color = as_factor(grupo)),
    size = 1.4, alpha = 0.75
  ) +
  coord_sf(
    xlim = range(df_costeiro_terra_buffer$longitude, na.rm = TRUE),
    ylim = range(df_costeiro_terra_buffer$latitude, na.rm = TRUE)
  ) +
  scale_color_brewer(palette = "Set1", name = "Grupo") +
  labs(
    title = "Agrupamento espacial de XCO2 na faixa costeira",
    subtitle = "Grupos formados a partir da análise de séries temporais (2015\u20132023)",
    x = "Longitude", y = "Latitude"
  ) +
  theme_minimal(base_size = 12) +
  theme(
    plot.title = element_text(face = "bold", size = 14),
    plot.subtitle = element_text(color = "grey40", size = 10),
    legend.position = "right",
    panel.grid.minor = element_blank(),
    panel.grid.major = element_line(color = "grey92"),
    axis.text = element_text(color = "grey30")
  ) +
  guides(color = guide_legend(override.aes = list(size = 3, alpha = 1)))
```

## Legenda: Para facilitar a interpretação espacial dos agrupamentos, os grupos foram representados sobre o mapa do Brasil, permitindo visualizar sua distribuição ao longo da região costeira e verificar a coerência espacial dos agrupamentos definidos a partir das séries temporais.

``` r
df_costeiro_terra_buffer |>
  filter(year > 2014) |> 
  mutate(
    class_latitude = cut(latitude, 115),
    year_month = paste(as.character(year), as.character(month), sep = "_"),
    latitude_media = (
      as.numeric(sub("\\(([-0-9.]+),.*", "\\1", class_latitude)) +
      as.numeric(sub(".*[,]([-0-9.]+)\\]", "\\1", class_latitude))
    ) / 2
  ) |> 
  left_join(colunas, by = "latitude_media") |> 
  arrange(latitude_media) |> 
  mutate(
    grupo = case_when(
      latitude_media > 0.37 ~ 5,
      latitude_media <= -28.6 ~2,
      latitude_media < -20 & latitude_media > -28.6 ~1,
      latitude_media > -5.5 & latitude_media <0.37 ~3,
      .default=4
    )
  ) |> 
  #filter(latitude_media < -5 & latitude_media > -10) |> 
  # group_by(grupo) |> 
  # summarise(
  #   lat_max = max(latitude_media),
  #   lat_min = min(latitude_media)
  # ) |> 
  # 
  ggplot() +
  geom_sf(data = brasil, fill = "grey96", color = "grey70", linewidth = 0.3) +
  geom_point(
    aes(x = longitude, y = latitude, color = as_factor(grupo)),
    size = 1.4, alpha = 0.75
  ) +
  coord_sf(
    xlim = range(df_costeiro_terra_buffer$longitude, na.rm = TRUE),
    ylim = range(df_costeiro_terra_buffer$latitude, na.rm = TRUE)
  ) +
  scale_color_brewer(palette = "Set1", name = "Grupo") +
  labs(
    title = "Agrupamento espacial de XCO2 na faixa costeira",
    subtitle = "Grupos formados a partir da análise de séries temporais (2015\u20132023)",
    x = "Longitude", y = "Latitude"
  ) +
  theme_minimal(base_size = 12) +
  theme(
    plot.title = element_text(face = "bold", size = 14),
    plot.subtitle = element_text(color = "grey40", size = 10),
    legend.position = "right",
    panel.grid.minor = element_blank(),
    panel.grid.major = element_line(color = "grey92"),
    axis.text = element_text(color = "grey30")
  ) +
  guides(color = guide_legend(override.aes = list(size = 3, alpha = 1)))
```

## Legenda: A partir da distribuição espacial observada no agrupamento, foram estabelecidos limites latitudinais para definir as regiões utilizadas na análise subsequente. Os limites foram ajustados de acordo com a organização espacial dos agrupamentos, resultando em cinco grupos latitudinais. Essa classificação foi utilizada como a divisão regional definitiva para o cálculo das anomalias.

``` r
df_grupos <- df_costeiro_terra_buffer |>
  filter(year > 2014) |> 
  mutate(
    class_latitude = cut(latitude, 115),
    year_month = paste(as.character(year), as.character(month), sep = "_"),
    latitude_media = (
      as.numeric(sub("\\(([-0-9.]+),.*", "\\1", class_latitude)) +
      as.numeric(sub(".*[,]([-0-9.]+)\\]", "\\1", class_latitude))
    ) / 2
  ) |> 
  left_join(colunas, by = "latitude_media") |> 
  arrange(latitude_media) |> 
  mutate(
    grupo = case_when(
      latitude_media > 0.37 ~ 5,
      latitude_media <= -28.6 ~2,
      latitude_media < -20 & latitude_media > -28.6 ~1,
      latitude_media > -5.5 & latitude_media <0.37 ~3,
      .default=4
    )
  ) 
```

## Legenda: Com os limites dos grupos definidos, a classificação foi incorporada à base de dados original, mantendo as informações espaciais e temporais de cada observação. Dessa forma, foi criada a base que seria utilizada no cálculo das anomalias de XCO₂ dentro de cada grupo.

``` r
df_grupos |> 
  group_by(year_month,grupo) |> 
  nest() |> 
  mutate(
    anomalia = map(data, 
                      \(df) df$xco2 - median(df$xco2, na.rm = TRUE)
                   )
  ) |> 
  unnest() |> 
  group_by(year,
           latitude_media) |>
  summarise(
    anom = mean(anomalia, na.rm = TRUE),
    .groups = "drop"
  ) |> 
  # filter(year > 2022) |> 
  ggplot(aes(x=latitude_media, y = anom, 
             color=as_factor(year)
             )) +
  # geom_col(color="black", position = "dodge") +
  geom_line() +
  # geom_point() +
  coord_flip(#ylim = c(380, 384)
             ) +
  labs(
    x = "Latitude (°)",
    y = expression(Delta~"XCO2 (ppm)"),
    color = "Ano"
  ) +
  theme_minimal()
```

## Legenda: Com os grupos latitudinais definidos, foi possível calcular a anomalia de XCO₂ considerando como referência o comportamento típico de cada grupo. Para cada combinação de mês e grupo, foi calculada a mediana de XCO₂ e, posteriormente, esse valor foi subtraído de cada observação correspondente. As anomalias resultantes foram então agregadas por ano e latitude média, permitindo representar a variação de XCO₂ ao longo da costa em relação ao comportamento de referência de cada região.

## A partir daqui, peguei os dados de Manguezais do MapBiomas.

## Carregar os rasters de manguezais do MapBiomas

``` r
arquivos <- list.files(
  "data/EarthEngine",
  pattern = "\\.tif$",
  full.names = TRUE
)
```

## Ver se ele encontra os artigos

``` r
arquivos
```

## Ver quantos rasters ele encontra:

``` r
length(arquivos)
```

## Criar o objeto mangue

``` r
# rasters <- lapply(arquivos, terra::rast)
# 
# mangue <- do.call(merge, rasters)
# terra::writeRaster(mangue, "data/mangue-brasil.tif")
```

## Legenda:

lapply() → aplica uma função em cada item da lista. arquivos → são os
arquivos .tif dos manguezais. terra::rast → lê cada arquivo .tif e
transforma em um objeto raster. rasters → guarda todos os rasters
carregados.

do.call() → executa uma função usando os elementos de uma lista como
argumentos. merge → junta os rasters. rasters → são os vários pedaços do
mapa. mangue → recebe o raster final unido.

## Visualização rápida

Carregando os dados processados pelo terra

``` r
#mangue <- terra::rast("data/mangue-brasil.tif")
#mangue
```

demais informações do arquivo

``` r
dim(mangue)      # nrow, ncol, nlyr
res(mangue)      # resolução (tamanho do pixel)
ext(mangue)      # extensão (bounding box)
crs(mangue, describe = TRUE)  # sistema de coordenadas
nlyr(mangue)     # número de bandas/camadas
summary(mangue)  # por padrão já faz um SAMPLE, não lê tudo — geralmente ok
```

- converter df_grupos para vetor espacial (terra), já no mesmo CRS do
  mangue

``` r
xco2_vect <- vect(df_grupos, geom = c("longitude", "latitude"), crs = "EPSG:4326")
```

criar buffer em torno de cada ponto (ajuste width conforme a escala que
faz sentido pra você – ex: 5000 = 5 km. Pegada do OCO-2/3 é ~1.3x2.25
km, então algo entre 1000-5000 m costuma ser razoável)

``` r
xco2_buffer <- buffer(xco2_vect, width = 5000)
```

extrair a fração média de mangue dentro de cada buffer

``` r
#    (evita o bad_alloc porque terra processa por blocos)
# frac_mangue <- extract(mangue, xco2_buffer, fun = mean, na.rm = TRUE)


# 5. juntar de volta no data.frame original
df_grupos$frac_mangue <- frac_mangue[, 2]

write_rds(frac_mangue, "data/df_frac_mangue.rds")

# 6. checar rapidamente
summary(df_grupos$frac_mangue)
sum(df_grupos$frac_mangue > 0, na.rm = TRUE)  # quantos pontos têm mangue por perto
```

# Iniciando a parte do SIF

``` r
df_sif <- read_rds("data-raw/data-set-sif-br.rds")
glimpse(df_sif)
```

## Legenda: O que encontramos: 46.098.810 observações

17 colunas: coordenadas: longitude e latitude; data/hora original: time;
variáveis de geometria solar/visada: sza, vza, saz, vaz; SIF: sif740;
sif740_uncertainty; daily_sif740; daily_sif757; daily_sif771;
quality_flag; path; year; month; day.

- O SIF já possui year, month e day.

# Entendendo qual SIF vamos usar e como o quality_flag está distribuído:

# Arrange por ano:

``` r
df_sif |>
  distinct(year) |>
  arrange(year)
```

# Ver quantas quality flags temos:

``` r
df_sif |>
  count(quality_flag)
```

# FIltrar quality:

``` r
df_sif_q0 <- df_sif |>
  filter(quality_flag == 0)
glimpse(df_sif_q0)
```

# Distribuição básica do sif:

``` r
summary(df_sif_q0$daily_sif757)
```

``` r

summary(df_sif_q0$daily_sif771)
```

## Legenda:

Min. e Max. mostram a amplitude total dos valores observados; 1st Qu.,
Median e 3rd Qu. descrevem a distribuição central dos dados; Mean
representa a média dos valores; e NA’s indica o número de observações
sem informação para SIF740.

# Filtrar a quality flag para pega só zero e conferir:

``` r
df_sif_q0 <- df_sif |>
  filter(quality_flag == 0)
table(df_sif_q0$quality_flag)
```

# Fazer o summary dos itens desejados:

``` r
summary(df_sif_q0$daily_sif757)
summary(df_sif_q0$daily_sif771)
```

## Investigar os valores extremos que sobreviveram ao quality flag.Primeiro, os 10 maiores e os 10 menores.

``` r
df_sif_q0 |>
  arrange(desc(daily_sif757)) |>
  slice_head(n = 10)
```

``` r
df_sif_q0 |>
  arrange(daily_sif757) |>
  slice_head(n = 10)
```

``` r
df_sif_q0 |>
  arrange(desc(daily_sif771)) |>
  slice_head(n = 10)
```

``` r
df_sif_q0 |>
  arrange(daily_sif771) |>
  slice_head(n = 10)
```

# Fazendo os Histogramas

``` r
ggplot(df_sif_q0, aes(x = daily_sif757)) +
  geom_histogram() +
  labs(
    x = "SIF 757 nm",
    y = "Frequência"
  ) +
  theme_minimal()
```

``` r
ggplot(df_sif_q0, aes(x = daily_sif771)) +
  geom_histogram() +
  labs(
    x = "SIF 771 nm",
    y = "Frequência"
  ) +
  theme_minimal()
```

# Entender comportamento dos dados sif757

``` r
df_sif_q0 |> 
  summarise(
    n = n(),
    min = min(daily_sif757, na.rm = TRUE),
    q01 = quantile(daily_sif757, 0.01, na.rm = TRUE),
    q05 = quantile(daily_sif757, 0.05, na.rm = TRUE),
    q25 = quantile(daily_sif757, 0.25, na.rm = TRUE),
    mediana = median(daily_sif757, na.rm = TRUE),
    media = mean(daily_sif757, na.rm = TRUE),
    q75 = quantile(daily_sif757, 0.75, na.rm = TRUE),
    q95 = quantile(daily_sif757, 0.95, na.rm = TRUE),
    q99 = quantile(daily_sif757, 0.99, na.rm = TRUE),
    max = max(daily_sif757, na.rm = TRUE)
  )
```

## Legenda:

A maior parte dos dados está concentrada em uma faixa relativamente
pequena.

Os percentis mostram que:

- 25% dos valores estão abaixo de 0,095;
- 50% dos valores estão abaixo de 0,232;
- 75% dos valores estão abaixo de 0,374;
- 95% dos valores estão abaixo de 0,579;
- 99% dos valores estão abaixo de 0,740.

O valor mínimo observado foi de aproximadamente -1,39, enquanto o máximo
foi de 5,67. Assim, embora a maior parte das observações esteja
concentrada próxima de zero, existem valores extremos positivos e
negativos.

Essa distribuição explica o aspecto do histograma, com grande
concentração das observações próximas de zero e uma cauda formada por
valores extremos.

``` r
df_sif_q0 |> 
  summarise(
    negativos = sum(daily_sif757 < 0, na.rm = TRUE),
    positivos = sum(daily_sif757 >= 0, na.rm = TRUE)
  )
```

``` r
df_sif_q0 |> 
  summarise(
    percentual_negativos = mean(daily_sif757 < 0, na.rm = TRUE) * 100
  )
```

``` r
df_sif_q0 |> 
  summarise(
    acima_1 = sum(daily_sif757 > 1, na.rm = TRUE),
    acima_2 = sum(daily_sif757 > 2, na.rm = TRUE),
    acima_3 = sum(daily_sif757 > 3, na.rm = TRUE),
    acima_4 = sum(daily_sif757 > 4, na.rm = TRUE),
    acima_5 = sum(daily_sif757 > 5, na.rm = TRUE)
  )
```

# Localizando os extremos

## Menores valores

``` r
df_sif_q0 |> 
  arrange(daily_sif757) |> 
  slice_head(n = 20)
```

## Maiores valores

``` r
df_sif_q0 |> 
  arrange(desc(daily_sif757)) |> 
  slice_head(n = 20)
```

## Para sif771

``` r
df_sif_q0 |> 
  summarise(
    n = n(),
    min = min(daily_sif771, na.rm = TRUE),
    q01 = quantile(daily_sif771, 0.01, na.rm = TRUE),
    q05 = quantile(daily_sif771, 0.05, na.rm = TRUE),
    q25 = quantile(daily_sif771, 0.25, na.rm = TRUE),
    mediana = median(daily_sif771, na.rm = TRUE),
    media = mean(daily_sif771, na.rm = TRUE),
    q75 = quantile(daily_sif771, 0.75, na.rm = TRUE),
    q95 = quantile(daily_sif771, 0.95, na.rm = TRUE),
    q99 = quantile(daily_sif771, 0.99, na.rm = TRUE),
    max = max(daily_sif771, na.rm = TRUE)
  )
```

## Quantidade de negativos

``` r
df_sif_q0 |> 
  summarise(
    negativos = sum(daily_sif771 < 0, na.rm = TRUE),
    positivos = sum(daily_sif771 >= 0, na.rm = TRUE),
    percentual_negativos = mean(daily_sif771 < 0, na.rm = TRUE) * 100
  )
```

## Extremos positivos

``` r
df_sif_q0 |> 
  summarise(
    acima_1 = sum(daily_sif771 > 1, na.rm = TRUE),
    acima_2 = sum(daily_sif771 > 2, na.rm = TRUE),
    acima_3 = sum(daily_sif771 > 3, na.rm = TRUE),
    acima_4 = sum(daily_sif771 > 4, na.rm = TRUE),
    acima_5 = sum(daily_sif771 > 5, na.rm = TRUE)
  )
```

## Proporção de negativos:

``` r
df_sif_q0 |> 
  summarise(
    negativos = sum(daily_sif771 < 0, na.rm = TRUE),
    positivos = sum(daily_sif771 >= 0, na.rm = TRUE),
    percentual_negativos = mean(daily_sif771 < 0, na.rm = TRUE) * 100
  )
```

## Legenda: O que podemos concluir neste momento?

Os dois SIFs apresentam uma característica muito semelhante:

a maior parte dos valores está concentrada em valores baixos, com uma
pequena quantidade de valores extremos positivos.

E os valores negativos não são raríssimos:

757 → 11,6% 771 → 15,1%

Portanto, não devemos simplesmente excluir todos os negativos.

Também não devemos cortar, por exemplo, SIF \> 1, porque embora sejam
poucos, precisamos primeiro saber se existe uma justificativa para
considerá-los inválidos.

# Os valores extremos observados nos SIF 757 e 771 representam observações que devemos manter, ou há evidências de que sejam valores problemáticos mesmo após o quality_flag == 0?

## Verificar se os extremos são isolados ou se aparecem também no outro SIF

### Maiores

``` r
df_sif_q0 |> 
  arrange(desc(daily_sif757)) |> 
  slice_head(n = 20) |> 
  select(
    time,
    longitude,
    latitude,
    daily_sif757,
    daily_sif771
  )
```

### Menores

``` r
df_sif_q0 |> 
  arrange(daily_sif757) |> 
  slice_head(n = 20) |> 
  select(
    time,
    longitude,
    latitude,
    daily_sif757,
    daily_sif771
  )
```

## Legenda: Os valores extremos do SIF 757 não são acompanhados, de forma consistente, por valores igualmente extremos do SIF 771.

## Verificar se os extremos estão espacialmente concentrados

### Maiores

``` r
df_sif_q0 |> 
  arrange(desc(daily_sif757)) |> 
  slice_head(n = 20) |> 
  select(
    time,
    longitude,
    latitude,
    daily_sif757,
    daily_sif771
  )
```

### Menores

``` r
df_sif_q0 |> 
  arrange(daily_sif757) |> 
  slice_head(n = 20) |> 
  select(
    time,
    longitude,
    latitude,
    daily_sif757,
    daily_sif771
  )
```

## Legenda: A análise dos 20 maiores e 20 menores valores de SIF757 mostra que os valores extremos estão espacialmente distribuídos, sem evidência visual de concentração em uma única região. Os extremos ocorrem em diferentes posições geográficas e anos, indicando que não há, neste momento, um padrão espacial simples associado aos valores extremos.

# Existe alguma característica das observações que diferencia os valores extremos dos valores comuns?

``` r
glimpse(df_sif_q0)
```

### Definindo o que é um extremo, apenas para avaliação.

``` r
df_extremos <- df_sif_q0 |> 
  mutate(
    q01 = quantile(daily_sif757, 0.01, na.rm = TRUE),
    q99 = quantile(daily_sif757, 0.99, na.rm = TRUE),
    grupo = case_when(
      daily_sif757 < q01 ~ "extremo_baixo",
      daily_sif757 > q99 ~ "extremo_alto",
      TRUE ~ "comum"
    )
  )
```

### Investigando por tempo e espaço

``` r
df_extremos |> 
  group_by(grupo) |> 
  summarise(
    n = n(),
    latitude_media = mean(latitude, na.rm = TRUE),
    latitude_sd = sd(latitude, na.rm = TRUE),
    longitude_media = mean(longitude, na.rm = TRUE),
    longitude_sd = sd(longitude, na.rm = TRUE)
  )
```

``` r
df_extremos |> 
  group_by(grupo, year) |> 
  summarise(
    n = n(),
    .groups = "drop"
  )
```

``` r
df_extremos |> 
  group_by(grupo, month) |> 
  summarise(
    n = n(),
    .groups = "drop"
  )
```

## Legenda: O que concluímos sobre os valores extremos de sif757?

Após a aplicação do quality_flag == 0, foram investigadas as
características espaciais e temporais das observações classificadas, de
forma exploratória, como extremas (1% inferior e superior da
distribuição de SIF 757).

Os extremos baixos apresentaram médias de latitude e longitude
diferentes das observações comuns, enquanto os extremos altos
apresentaram médias espaciais semelhantes às do grupo comum.

As frequências absolutas por ano e mês não foram utilizadas para inferir
associações temporais, pois a quantidade de observações varia entre os
períodos.

Portanto, há indicação de uma diferença espacial para os valores
extremos baixos, mas essa análise, isoladamente, não fornece evidência
suficiente para considerar essas observações inválidas ou para
justificar sua remoção.

## Extremos exploratórios de 771

``` r
df_extremos_771 <- df_sif_q0 |> 
  mutate(
    q01 = quantile(daily_sif771, 0.01, na.rm = TRUE),
    q99 = quantile(daily_sif771, 0.99, na.rm = TRUE),
    grupo = case_when(
      daily_sif771 < q01 ~ "extremo_baixo",
      daily_sif771 > q99 ~ "extremo_alto",
      TRUE ~ "comum"
    )
  )
```

### Distribuição espacial:

``` r
df_extremos_771 |> 
  group_by(grupo) |> 
  summarise(
    n = n(),
    latitude_media = mean(latitude, na.rm = TRUE),
    latitude_sd = sd(latitude, na.rm = TRUE),
    longitude_media = mean(longitude, na.rm = TRUE),
    longitude_sd = sd(longitude, na.rm = TRUE)
  )
```

### Distr Temporal por ano

``` r
df_extremos_771 |> 
  group_by(grupo, year) |> 
  summarise(
    n = n(),
    .groups = "drop"
  )
```

### Distr Temporal por mês

``` r
df_extremos_771 |> 
  group_by(grupo, month) |> 
  summarise(
    n = n(),
    .groups = "drop"
  )
```

### Maiores valores

``` r
df_sif_q0 |> 
  arrange(desc(daily_sif771)) |> 
  slice_head(n = 20) |> 
  select(
    time,
    longitude,
    latitude,
    daily_sif757,
    daily_sif771
  )
```

### Menores valores

``` r
df_sif_q0 |> 
  arrange(daily_sif771) |> 
  slice_head(n = 20) |> 
  select(
    time,
    longitude,
    latitude,
    daily_sif757,
    daily_sif771
  )
```

### Baixar resultados para excel:

``` r

# Resultados espaciais
espacial_757 <- df_extremos |> 
  group_by(grupo) |> 
  summarise(
    n = n(),
    latitude_media = mean(latitude, na.rm = TRUE),
    latitude_sd = sd(latitude, na.rm = TRUE),
    longitude_media = mean(longitude, na.rm = TRUE),
    longitude_sd = sd(longitude, na.rm = TRUE)
  )

espacial_771 <- df_extremos_771 |> 
  group_by(grupo) |> 
  summarise(
    n = n(),
    latitude_media = mean(latitude, na.rm = TRUE),
    latitude_sd = sd(latitude, na.rm = TRUE),
    longitude_media = mean(longitude, na.rm = TRUE),
    longitude_sd = sd(longitude, na.rm = TRUE)
  )


# Resultados anuais
ano_757 <- df_extremos |> 
  group_by(grupo, year) |> 
  summarise(n = n(), .groups = "drop")

ano_771 <- df_extremos_771 |> 
  group_by(grupo, year) |> 
  summarise(n = n(), .groups = "drop")


# Resultados mensais
mes_757 <- df_extremos |> 
  group_by(grupo, month) |> 
  summarise(n = n(), .groups = "drop")

mes_771 <- df_extremos_771 |> 
  group_by(grupo, month) |> 
  summarise(n = n(), .groups = "drop")


# 20 maiores
maiores_757 <- df_sif_q0 |> 
  arrange(desc(daily_sif757)) |> 
  slice_head(n = 20) |> 
  select(time, longitude, latitude, daily_sif757, daily_sif771)

maiores_771 <- df_sif_q0 |> 
  arrange(desc(daily_sif771)) |> 
  slice_head(n = 20) |> 
  select(time, longitude, latitude, daily_sif757, daily_sif771)


# 20 menores
menores_757 <- df_sif_q0 |> 
  arrange(daily_sif757) |> 
  slice_head(n = 20) |> 
  select(time, longitude, latitude, daily_sif757, daily_sif771)

menores_771 <- df_sif_q0 |> 
  arrange(daily_sif771) |> 
  slice_head(n = 20) |> 
  select(time, longitude, latitude, daily_sif757, daily_sif771)


# Criar Excel
write.xlsx(
  list(
    espacial_757 = espacial_757,
    ano_757 = ano_757,
    mes_757 = mes_757,
    maiores_757 = maiores_757,
    menores_757 = menores_757,
    
    espacial_771 = espacial_771,
    ano_771 = ano_771,
    mes_771 = mes_771,
    maiores_771 = maiores_771,
    menores_771 = menores_771
  ),
  file = "resultados_extremos_SIF.xlsx"
)
```

# Iniciar o recorte de área antes de analisar mais dados extremos

# Precisamos transformar os pontos em sf:

``` r
df_sif_q0_sf <- df_sif_q0 |>
  st_as_sf(
    coords = c("longitude", "latitude"),
    crs = 4326,
    remove = FALSE
  )
```

\#Conferir CRS

``` r
st_crs(df_sif_q0_sf)
```

``` r
st_crs(costeiro_terrestre)
```

## Legenda: vi que o CRS está igual para os dois! Uhu!

# Fazendo o corte

``` r
df_sif_costeiro <- df_sif_q0_sf |>
  st_filter(costeiro_terrestre)
```

# Testando:

``` r
df_sif_costeiro
```

``` r
nrow(df_sif_q0)
nrow(df_sif_costeiro)
```

# Salvando:

``` r
write_rds(df_sif_costeiro, "data/sif-costeiro-terrestre.rds")
```
