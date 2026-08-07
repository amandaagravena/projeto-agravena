
## Legenda: Esse output indica que o arquivo .Rmd será renderizado (convertido) para .md para ir para o Github, através do knitr (pacote do R que serve para executar o código e gerar o documento final).

Assim, conseguimos misturar: texto, código R, resultados, figuras e
tabelas, o que é ótimo para projetos científicos.

<!-- README.md is generated from README.Rmd. Please edit that file -->

\##Legenda:

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
library(geobr)
library(sf)
```

\##Legenda: Ao carregar pacotes, carregamos as funções das diferentes
bibliotecas.

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

\##Legenda:

- O geobr baixa automaticamente o limite político do Brasil através do
  read country. Temos que colocar o ano a partir de agora, devido a uma
  atualização do R.

- através do read biomes, baixamos todos os biomas brasileiros.Eles
  possuem:nome; geometria;projeção.

## Resumo do objeto `biomes`

``` r
biomes
```

\##Legenda:

Esse comando serve para mostrar o resumo do objeto biomes. Assim,
podemos conferir se os dados foram carregados corretamente antes de
seguir a análise. Nele, podemos ver a classe do objeto (sf), número de
linhas (polígonos), número de colunas, nomes das variáveis e sistema de
referência das coodenadas (CRS), por exemplo.

## Listando os biomas individuais (distintos)

``` r
biomes |> distinct(name_biome)
```

\##Legenda:

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

##Legenda: O bloco constrói o mapa dos biomas brasileiros. A lógica é sempre a mesma: inicia um gráfico, desenha os biomas, define as cores, adiciona título e legenda, ajusta a aparência e exibe o mapa. 

map_biomes <- ggplot(biomes) + ##Legenda: A função ggplot() inicia a construção do gráfico com o objeto biomes, que contém os limites espaciais dos biomas brasileiros, armazenando o gráfico em map_biomes. No momento, o gráfico ainda está vazio, pois apenas foi definido o conjunto de dados que será utilizado. 
  
  geom_sf(aes(fill = name_biome), color = "#1a1a2e", linewidth = 0.4) +
  scale_fill_manual(
    values = biome_colors,
    name   = "Bioma",
    # na.value = "#555555"
  ) + ##Legenda: A função geom_sf() desenha as geometrias presentes dentro do objeto biomes. Como biomes é um objeto espacial (sf), cada bioma é representado por um polígono no mapa. 
  
#Já a função aes(), de estética, informa as características visuais do gráfico associadas aos dados. Nesse caso, fill=name_biome, ou seja, cada bioma será preenchido por uma cor diferente, de acordo com a variável name_biome. 
  
#Já os itens: scale_fill_manual = permite definir manualmente as cores utilizadas, o values = biomes_colors utiliza a paleta criada anteriormente. Os demais itens são intuitivos. 
  
  labs(
    title    = "Biomas do Brasil",
    subtitle = "Distribuição dos biomas continentais e costeiros",
    caption  = "Fonte: IBGE via geobr"
  ) + ##Legenda: A função labs() adiciona informações descritivas do gráfico, nesse caso título da figura, subtítulo e fonte dos dados. Essas informações ajudam a identificar o conteúdo do mapa. 
  
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

##Legenda: O theme_void() remove os elementos gráficos que não são úteis para mapas, como eixos, grades e marcações. Já o base_family = "serif" define que os textos utilizarão uma fonte do tipo serifada.Os demais itens definem a estética do gráfico. 

#As linhas de "plot." alteravam o fundo do gráfico e a aparência do título e do subtítulo, porém, não achamos mais necessário. 

# print(map_country)
print(map_biomes)

##Legenda: Depois de todas as etapas de construção, o comando print() exibe o mapa os biomas na tela, pois já construímos o objeto "map_biomes". 
```

## Ler e interpretar os dados em “data-raw”

``` r
df <- read_rds("data-raw/data-set-xco2-br-002.rds") |> 
  mutate(year = year - 20)

##Legenda: A função read_rds() lê o arquivo que contém os dados de XCO₂. O conteúdo do arquivo será armazenado em df (dataframe), que será utilizado nas etapas seguintes. 

# Como estamos apenas lendo e corrigindo os dados raw, por enquanto, chamaremos de df. 

#Já a função mutate() é utilizada para modificar colunas de um conjunto de dados. Nesse, ela altera a coluna year, retirando 20 anos, pois notamos pelo nome dos arquivos que esse dado está com esse erro. 
```

## Resumo do data-set

``` r
glimpse(df |> 
          filter(xco2_quality_flag == 0))
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
df |> distinct(year)
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
df_sf <- df |> 
  filter(xco2_quality_flag == 0) |> 
  st_as_sf(coords = c("longitude", "latitude"), crs = 4326, remove = FALSE)
```

\##Legenda: Até aqui, estávamos trabalhando com uma tabela comum.A
partir desse ponto, passamos a trabalhar com um objeto espacial, que
permite realizar operações geográficas, como recortar pela costa
brasileira.

\#Por que transformar em um objeto espacial? - Antes desse código, o df
é apenas uma tabela. - Para o leitor, imaginamos esses números como
locais no mapa (latitude e longitude). Já para o R, são apenas colunas
numéricas. É justamente isso que st_as_sf() faz.A função st_as_sf()
converte uma tabela comum em um objeto espacial. É ela que permite
utilizar todas as funções do pacote sf. - Depois dele, cada linha da
tabela passa a possuir uma geometria. Para isso, a coluna geometry é
criada automaticamente. - Aqui, será criado um novo objeto, chamado
df_sf. O final sf significa simple features. Filtramos de novo os dados
de boa qualidade para não repetir dados já descartados. - crs = 4326
corresponde ao sistema geográfico WGS 84, que é o padrão utilizado por
GPS e pela maioria dos satélites. - Quando a geometria é criada, o sf
poderia remover as colunas longitude e latitude, pois elas já estariam
representadas na coluna geometry., por isso, remove = false. Elas
permanecem ali para análises posteriores.

## Join espacial: associa cada ponto ao bioma em que ele cai (pontos fora do Brasil recebem NA em name_biome)

``` r
# Transforma os biomas para o mesmo sistema de referência (CRS) do objeto df_sf (WGS84)
biomes <- st_transform(biomes, crs = 4326)
brasil <- st_transform(brasil, crs = 4326)

# Realiza a junção espacial entre os pontos de XCO2 e os biomas
df_biomas <- st_join(
  df_sf,
  biomes |>
    select(name_biome, geometry)
)
```

\##Legenda: Aqui, fazemos a junção espacial. Até aqui, sabemos a
localização de cada ponto df_sf (lat e long), e o objeto biomes são os
polígonos. O st_join() juntará as duas informações. O objetivo desse
código é associar cada observação de XCO₂ ao bioma em que ela está
localizada.

Como?

- Primeiro, o st_transform() altera o sistema crs do bioma e do brasil.
- Depois, fazemos a junção. O objeto biomes possui várias colunas, mas
  queremos apenas name_biome e geometry.
- Já o df_sf são os pontos.
- Com o join, mesclamos, dando para cada ponto um name_biome e uma
  localização no polígono.

## Filtrar apenas pontos do Brasil

``` r
df_brasil <- df_biomas |> 
  filter(!is.na(name_biome))

glimpse(df_brasil)
```

\##Legenda: - df_brasil \<- cria um novo objeto chamado **df_brasil**,
onde será armazenado o resultado do filtro. - df_biomas é o conjunto de
dados que contém os pontos de XCO₂ e o bioma associado a cada ponto. -
filter(!is.na(name_biome)) mantém apenas as linhas em que a coluna
name_biome possui um valor, ou seja, apenas os pontos que pertencem a
algum bioma brasileiro. Os pontos fora do Brasil possuem NA nessa coluna
e são removidos. - glimpse(df_brasil) mostra um resumo do novo conjunto
de dados, permitindo conferir as colunas, seus tipos e algumas
informações gerais.

## Salvar os dados na pasta data

``` r
write_rds(
  df_brasil,
  "data/xco2-brasil-biomas.rds"
)
```

\##Lendo o arquivo salvo até aqui

``` r
df_brasil <- read_rds("data/xco2-brasil-biomas.rds")
```

``` r
##Conferir quantos pontos por bioma
df_brasil |> 
  st_drop_geometry() |> 
  count(name_biome, sort = TRUE)
```

## Legenda: st_drop_geometry() temporariamente remove a coluna geometry, transformando o objeto espacial (sf) em uma tabela comum, pois queremos contar observações. Para contar, usamos a função count(), que contabilizará quantos linhas possuem para cada valor da coluna name_biome.O sort = TRUE faz com que o resultado seja ordenado do maior para o menor.

## A partir daqui, a ideia surgiu da conversa: como vamos ver apenas os locais de possíveis manguezais, sem entrarmos nas 12 milhas de área marítima do Sistema Costeiro?

\#Aqui, surgiram duas ideias: Cruzamento com o shape do Brasil (Terra
firme) com o shape do Sistema Costeiro ou pegamos os dados do MapBiomas
focado em Manguezais e cruzamos com os dados de Sistema costeiro.

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

\##Legenda: A função st_crs() mostra o CRS (Coordinate Reference System)
do objeto.

## Conferir se a geometria resultante faz sentido (plot rápido)

``` r
plot(st_geometry(costeiro_terrestre))
costeiro_terrestre_buffer <- sf::st_buffer(costeiro_terrestre,0.045)
plot(st_geometry(costeiro_terrestre_buffer))
```

## Converter df_brasil em objeto sf de pontos (ajuste nomes de colunas)

``` r
pontos_brasil <- st_as_sf(
  df_brasil,
  coords = c("longitude", "latitude"),
  crs = 4326,
  remove = FALSE
)
```

\##Esse código cria um novo objeto espacial (sf), denominado
pontos_brasil, a partir do conjunto de dados df_brasil, utilizando as
colunas de longitude e latitude para representar cada observação como um
ponto geográfico.

## Filtrar apenas os pontos que caem dentro da faixa costeira terrestre

``` r
costeiro_terrestre <- st_set_crs(costeiro_terrestre, 4326)

pontos_costeiro_terra <- st_join(pontos_brasil, costeiro_terrestre  |> 
                                   mutate(name_biome = "Costeiro Terrestre") |> 
                       select(name_biome, geometry))
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
df_costeiro_terra <- pontos_costeiro_terra |> filter(name_biome.y ==  "Costeiro Terrestre")  |>  st_drop_geometry()
```

\##Legenda: Essa etapa serve para obter apenas os pontos que pertencem à
faixa costeira terrestre e transformá-los novamente em uma tabela comum.

## Verificar se eles formam a linha literânea

``` r
df_costeiro_terra |> 
  ggplot(aes(longitude, latitude)) +
  geom_point()
```

## Salvando na pasta data

``` r
write_rds(df_costeiro_terra,"data/xco2-costeiro-terrestre.rds")
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
df_costeiro_terra <- read_rds("data/xco2-costeiro-terrestre.rds")
nrow(df_costeiro_terra)
```

## Legenda: Aqui, leu o arquivo e contou o número de fileiras após os recortes. Aqui, respondemos: “Depois de todos os filtros, ainda tenho uma quantidade suficiente de dados?” SIM!

# Cobertura temporal — quantos pontos por ano/mês

``` r
df_costeiro_terra |> 
  st_drop_geometry() |> 
  count(year) |>   # ajuste nome da coluna de data
  arrange(year)
```

## Legenda: Verificar quantas observações existem em cada ano e organiza em ordem crescente, para verificar se algum ano possui poucos dados ou se há anos ausentes.

``` r
# Estatística descritiva do XCO2 nessa faixa
summary(df_costeiro_terra$xco2)  # ajuste nome da coluna
```

## Legenda: O summary() calcula automaticamente: mínimo; primeiro quartil; mediana; média; terceiro quartil; máximo.Assim, verificamos: valores muito altos;

valores muito baixos;possíveis erros.

Nesse ponto, de início, apareceu o bioma Pantanal. O que é estranho,
pois não há Pantanal na zona costeira.

``` r
costeiro <- biomes |> 
  filter(name_biome == "Sistema Costeiro")
ggplot() +
  geom_sf(data = brasil, fill = "grey95", color = "grey50", linewidth = 0.3) +
  geom_sf(data = costeiro, fill = "#2c7fb8", color = NA, alpha = 0.6) +
  geom_point(data = df_costeiro_terra |> filter(name_biome.x == "Pantanal") |> 
  st_drop_geometry(), aes(longitude,latitude), colour = "red") +
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

## Legenda: Nesse ponto, queríamos comparar a distribuição do XCO₂ entre os biomas. Nele, evidenciamos o erro (Pantanal) e o Sistema Costeiro, pois não deve aparecer pontos de Pantanal no Sistema costeiro.

\##Durante a segunda junção espacial (st_join), o objeto pontos_brasil
já possuía uma coluna chamada “name_biome”, indicando o bioma em que
cada ponto estava (Amazônia, Cerrado, Mata Atlântica etc.).

Como o objeto costeiro_terrestre também recebeu uma coluna com o mesmo
nome(“name_biome”), o R renomeou automaticamente as duas colunas para
evitar conflito: a coluna original passou a se chamar “name_biome.x”,
enquanto a coluna proveniente da junção passou a se chamar
“name_biome.y”(Costeiro Terrestre).

``` r
df_costeiro_terra |> 
  st_drop_geometry() |> 
  filter(year == 2023,
         name_biome.x != "Sistema Costeiro") |> 
  # group_by(year) |> 
  ggplot(aes(x=name_biome.x, y=xco2, fill=name_biome.x)) +
  geom_boxplot() 
```

\##Legenda: Boxplot é usado para comparar a distribuição do XCO₂ entre
os biomas. Podemos visualizar mediana; quartis; dispersão; outliers.
Excluímos Sistema Costeiro, pois não é um bioma de fato.

``` r
df_costeiro_terra |> 
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

\##Legenda: Média mensal por bioma, agrupando por bioma e por mês, ou
seja, “Qual foi o XCO₂ médio daquele bioma naquele mês?” - Resulta em um
gráfico temporal. Buscamos ver se os biomas apresentam comportamento
sazonal diferente.

``` r
df_costeiro_terra |> 
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

\##Legenda: Média mensal por faixa de latitude (20 faixas). Então,
calcula a média do XCO₂ para cada faixa de latitude em cada mês. O
objetivo é verificar se existe um gradiente latitudinal, ou seja, “O
comportamento do XCO₂ muda conforme se avança do sul para o norte do
Brasil?”

\##A partir daqui, peguei os dados de Manguezais do MapBiomas.

## Carregar os rasters de manguezais do MapBiomas

``` r
library(terra)
```

``` r
arquivos <- list.files(
  "data/EarthEngine",
  pattern = "\\.tif$",
  full.names = TRUE
)
```

\##Ver se ele encontra os artigos

``` r
arquivos
```

\##Ver quantos rasters ele encontra:

``` r
length(arquivos)
```

\##Criar o objeto mangue

``` r
rasters <- lapply(arquivos, terra::rast)

mangue <- do.call(merge, rasters)
terra::writeRaster(mangue, "data/mangue-brasil.tif")
```

\##Legenda: lapply() → aplica uma função em cada item da lista. arquivos
→ são os arquivos .tif dos manguezais. terra::rast → lê cada arquivo
.tif e transforma em um objeto raster. rasters → guarda todos os rasters
carregados.

do.call() → executa uma função usando os elementos de uma lista como
argumentos. merge → junta os rasters. rasters → são os vários pedaços do
mapa. mangue → recebe o raster final unido.

\##Visualização rápida

``` r
plot(
  mangue,
  main = "Manguezais do Brasil (MapBiomas - 2018)",
  col = c("white", "darkgreen"),
  legend = TRUE
)

plot(
  st_geometry(brasil),
  add = TRUE,
  border = "black",
  lwd = 0.6
)
```
