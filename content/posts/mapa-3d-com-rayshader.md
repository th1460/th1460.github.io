---
title: "Mapa 3D com Rayshader"
author:
  name: Thiago Pires
date: '2022-08-07'
series:
- Packages
- R
tags:
- Rayshader
- Map
- R
linktitle: Mapa 3D com Rayshader
type:
- post
- posts
weight: 10
---

Aqui um exemplo de como construir um mapa 3D de uma parte do município do Rio de Janeiro com a linguagem R e a biblioteca [`rayshader`](https://www.rayshader.com/).

## Dados de elevação

Primeiro é necessário obter as imagens de elevação. Nesta página [opentopography.org](https://opentopography.org) é possível conseguir estes dados. Selecione uma área e escolha uma base com as imagens disponíveis aqui no [portal.opentopography.org/datasets](https://portal.opentopography.org/datasets) e salve em um formato `.tif`. 

Salve os valores utilizados de latitude e longitude na seleção dos limites da região do mapa na variável `bbox`.

```
bbox <- list(
    p2 = list(long = -43.2328217452992, lat = -22.99560928307949),
    p1 = list(long = -43.133008448808454, lat = -22.930329210944166)
)
```

Carregue a imagem com as informações de elevação em `.tif` utilizando a função
`raster::raster`.

```
# Carregar o arquivo tif
elev_file <- "data/rj.tif"
elev_img <- raster::raster(elev_file)
elev_matrix <- 
    matrix(raster::extract(elev_img, 
                           raster::extent(elev_img), 
                           buffer = 1000), 
           nrow = ncol(elev_img), 
           ncol = nrow(elev_img)
    )
    
    
# Detectar camanda de água
watermap <- rayshader::detect_water(elev_matrix)
```

## Adicionando a textura

A textura do mapa foi obtida através desta rotina. As funções `define_image_size` e `get_arcgis_map_image` são deste [repositório](https://github.com/wcmbishop/rayshader-demo/tree/master/R). 

```
source("src/define_image_size.R")
source("src/get_arcgis_map_image.R")

image_size <- define_image_size(bbox, major_dim = 600)

overlay_file <- "../data/rj_overlay.png"

get_arcgis_map_image(bbox, map_type = "World_Imagery", file = overlay_file,
                     width = image_size$width, height = image_size$height, 
                     sr_bbox = 4326)

overlay_img <- png::readPNG(overlay_file)
```

## Renderizando o mapa em 3D

A última etapa é a renderização do mapa

```
# Renderizando
zscale <- 30
rgl::clear3d()
elev_matrix |> 
    rayshader::sphere_shade(texture = "imhof4") |> 
    rayshader::add_water(watermap, color = "imhof4") |> 
    rayshader::add_overlay(overlay_img, alphalayer = .8) |> 
    plot_3d(elev_matrix, zscale = zscale, windowsize = c(1500, 1200),
            water = TRUE, soliddepth = -max(elev_matrix)/zscale, wateralpha = 1,
            theta = 25, phi = 30, zoom = 0.65, fov = 60)

# Salvando o resultado final
rayshader::render_snapshot("data/rj-3d.png")
```

Após a renderização foram adicionados os rótulos no [GIMP](https://www.gimp.org).

![](https://raw.githubusercontent.com/th1460/linguagem-r/main/data/rj-3d-editado.png)

## Referências

- A Rayshader base tutorial + Bonus : Hawaii https://www.davidsolito.com/post/a-rayshader-base-tutortial-bonus-hawaii/
- I can rayshade, and so can you https://wcmbishop.github.io/rayshader-demo/
