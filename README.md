
<!-- README.md is generated from README.Rmd. Please edit that file -->

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

- **Prof. Dr. Alan Rodrigo Panosso**  
  Coorientador — Departamento de Ciências Exatas - FCAV/Unesp  
  Email: <alan.panosso@unesp.br>

## Carregando os pacotes

``` r
library(tidyverse)
library(geobr)
library(sf)
```

## Extrair os shapes da costa brasileira

``` r
brasil <- geobr::read_country(showProgress = FALSE, year = 2020)
biomes <- geobr::read_biomes(showProgress = FALSE, year = 2019)
```

## Resumo do objeto `biomes`

``` r
biomes
```

## Listando os biomas individuais (distintos)

``` r
biomes |> distinct(name_biome)
```

## Visualização dos mapas, Brasil e Biomas

``` r
# Mapa 1 — Contorno do Brasil
# map_country <- ggplot(brasil) +
#   geom_sf(fill = "#2d6a4f", color = "#95d5b2", linewidth = 0.8) +
#   labs(
#     title    = "Brasil",
#     subtitle = "Território Nacional",
#     caption  = "Fonte: IBGE via geobr"
#   ) +
#   theme_void(base_family = "serif") +
#   theme(
#     plot.background  = element_rect(fill = "#0d1b2a", color = NA),
#     panel.background = element_rect(fill = "#0d1b2a", color = NA),
#     plot.title       = element_text(color = "#95d5b2", size = 22, face = "bold",
#                                     hjust = 0.5, margin = margin(t = 16, b = 4)),
#     plot.subtitle    = element_text(color = "#74c69d", size = 13,
#                                     hjust = 0.5, margin = margin(b = 8)),
#     plot.caption     = element_text(color = "#52b788", size = 8,
#                                     hjust = 1, margin = margin(b = 10, r = 12)),
#     plot.margin      = margin(20, 20, 10, 20)
#   )

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

# Mapa 2 — Biomas
map_biomes <- ggplot(biomes) +
  geom_sf(aes(fill = name_biome), color = "#1a1a2e", linewidth = 0.4) +
  scale_fill_manual(
    values = biome_colors,
    name   = "Bioma",
    # na.value = "#555555"
  ) +
  labs(
    title    = "Biomas do Brasil",
    subtitle = "Distribuição dos biomas continentais e costeiros",
    caption  = "Fonte: IBGE via geobr"
  ) +
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

# print(map_country)
print(map_biomes)
```

## Ler e interpretar os dados em “data-raw”

``` r
df <- read_rds("data-raw/data-set-xco2-br-002.rds") |> 
  mutate(year = year - 20)
```

## Resumo do data-set

``` r
glimpse(df |> 
          filter(xco2_quality_flag == 0))
```

## Listando os anos individuais (distintos)

``` r
df |> distinct(year)
```

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

## Transformar o df de XCO2 em objeto espacial (sf)

``` r
df_sf <- df |> 
  filter(xco2_quality_flag == 0) |> 
  st_as_sf(coords = c("longitude", "latitude"), crs = 4326, remove = FALSE)
```

## Join espacial: associa cada ponto ao bioma em que ele cai (pontos fora do Brasil recebem NA em name_biome)

``` r
# Transforma biomes para o mesmo CRS do df_sf (WGS84)
biomes <- st_transform(biomes, crs = 4326)
brasil <- st_transform(brasil, crs = 4326)
df_biomas <- st_join(df_sf, biomes  |>  
                       select(name_biome, geometry))
```

\##Filtrar apenas pontos do Brasil

``` r
df_brasil <- df_biomas |> 
  filter(!is.na(name_biome))
glimpse(df_brasil)
```

## Salvar os dados na pasta data

``` r
write_rds(df_brasil, "data/xco2-brasil-biomas.rds")
```

## Lendo o arquivo salvo até aqui

``` r
df_brasil <- read_rds("data/xco2-brasil-biomas.rds")
```

``` r
##Conferir quantos pontos por bioma
df_brasil |> 
  st_drop_geometry() |> 
  count(name_biome, sort = TRUE)
```

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

## Calcular a interseção geométrica: só a parte do Sistema Costeiro que também é terra (contorno do Brasil) -\> remove a faixa marítima

``` r
costeiro_terrestre <- st_intersection(costeiro, brasil)

st_crs(costeiro)
st_crs(brasil)
```

## Conferir se a geometria resultante faz sentido (plot rápido)

``` r
plot(st_geometry(costeiro_terrestre))
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

## Filtrar apenas os pontos que caem dentro da faixa costeira terrestre

``` r
costeiro_terrestre <- st_set_crs(costeiro_terrestre, 4326)

pontos_costeiro_terra <- st_join(pontos_brasil, costeiro_terrestre  |> 
                                   mutate(name_biome = "Costeiro Terrestre") |> 
                       select(name_biome, geometry))
```

## Voltar para data.frame puro, se precisar

``` r
df_costeiro_terra <- pontos_costeiro_terra |> filter(name_biome.y ==  "Costeiro Terrestre")  |>  st_drop_geometry()
```

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

## De qualquer forma, quando você chegar na etapa final de recorte pelos manguezais propriamente ditos, esse problema desaparece sozinho — como manguezal é vegetação terrestre/intertidal, a máscara de manguezal (MapBiomas/ICMBio) naturalmente exclui os pontos oceânicos.
