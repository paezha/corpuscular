
<!-- README.md is generated from README.Rmd. Please edit that file -->

# corpuscular

<!-- badges: start -->
<!-- badges: end -->

In this notebook I experiment with floating corpuscules (spheres or
cubes) using rayrender.

These packages are needed:

``` r
library(glue)
library(MetBrewer)
library(MexBrewer)
#> Registered S3 method overwritten by 'MexBrewer':
#>   method        from     
#>   print.palette MetBrewer
library(rayrender)
```

Sample a number to use as a random seed:

``` r
seed <- sample.int(10000000, 1)
```

Select the number of spheres that will populate the scene:

``` r
set.seed(seed)

n_sph <- runif(1, min = 10, max = 30) |>
  round()
```

Randomly choose a color palette from {MetBrewer} or {MexBrewer}:

``` r
set.seed(seed)

# Select collection of color palettes
edition <- sample(c("MexBrewer", "MetBrewer"), 1)

if(edition=="MexBrewer"){
  # Randomly select a color palette (MexBrewer Edition)
  palette_name <- sample(c("Alacena", "Atentado", 
                           "Aurora", "Concha", "Frida",
                           "Maiz", "Naturaleza", "Revolucion",
                           "Ofrenda", "Ronda", "Tierra", "Vendedora"), 1)
  # col_palette <- mex.brewer(palette_name, n = 25)
  col_palette <- mex.brewer(palette_name, n = 15)
  
}else{
  # Randomly select a color palette (MetBrewer Edition)
  palette_name <- sample(c("Archambault", "Austria", "Benedictus", "Cassatt1", "Cassatt2", "Cross", "Degas", "Demuth", "Derain", "Egypt", "Gauguin", "Greek", "Hiroshige", "Hokusai1", "Hokusai2", "Hokusai3", "Homer1", "Homer2", "Ingres", "Isfahan1", "Isfahan2", "Java", "Johnson", "Juarez", "Kandinsky", "Klimt", "Lakota", "Manet", "Monet", "Moreau", "Morgenstern", "Nattier", "Navajo", "NewKingdom", "Nizami", "OKeeffe1", "OKeeffe2", "Paquin", "Peru1", "Peru2", "Pillement", "Pissaro", "Redon", "Renoir", "Signac", "Tam", "Tara", "Thomas", "Tiepolo", "Troy", "Tsimshian", "VanGogh1", "VanGogh2", 'VanGogh3', "Veronese", "Wissing"), 1)
  col_palette <- met.brewer(palette_name, n = 15)
}
```

Create a data frame with the parameters for the spheres. The position in
x is on the plane (left-right), y is the vertical axis, and z is on the
plane (front-back). The radius must be positive, so I use the absolute
of the normal distribution to generate random numbers. I’d like some
*big* spheres, but not too many. Use the color palette chosen:

``` r
set.seed(seed)

df <- data.frame(x = runif(n_sph, 
                           min = -2, 
                           max = 2),
                 y = runif(n_sph, 
                           min = 0, 
                           max = 4),
                 z = runif(n_sph, 
                           min = -2, 
                           max = 2),
                 r = abs(rnorm(n_sph, 
                               0, 0.25)),
                 color = sample(col_palette, 
                                n_sph, 
                                replace = TRUE))
```

Initialize a scene with a white ground:

``` r
scene <- generate_ground(material = diffuse(color = "white"))
```

To create the spheres for the scene, initialize an empty data frame and
then use `rayrender::sphere()` to create the spheres. Experiment with
different materials (`diffuse()`, `glossy()`, etc.). Populate the
parameters to create the sphere:

``` r
set.seed(seed)

# Initialize data frame for corpuscules 
obj <- data.frame()

# Select shape
edition <- sample(c("sphere", "cube"), 1)

# Select cube rotation
if(sample(c(TRUE, FALSE), 1)){
  angle = c(0, 0, 0)
}else{
  angle = c(45, 45, 45)
}

if(edition == "sphere"){
  for(i in 1:nrow(df)){
    obj <- rbind(obj,
                 sphere(x = df$x[i],
                        y = df$y[i],
                        z = df$z[i],
                        radius = df$r[i] * runif(1, min = 1, max = 1.5),
                        material = glossy(color=df$color[i])))
  }}else{
    for(i in 1:nrow(df)){
      obj <- rbind(obj,
                   cube(x = df$x[i],
                        y = df$y[i],
                        z = df$z[i],
                        angle = angle,
                        width = df$r[i] * runif(1, min = 1, max = 2), # cubes tend to be a bit smaller compared to spheres: increase dimensions
                        material = glossy(color=df$color[i])))
    }
  }
```

Add the objects to the scene, including a source of light with random
intensity:

``` r
set.seed(seed)
scene <- scene |> 
  add_object(objects = obj) |>
  add_object(sphere(x = mean(df$x),
                    y = 40,
                    z = mean(df$z), 
                    material=light(intensity = rlnorm(1, meanlog = 3),
                                   invisible = TRUE)))
```

Render the scene using randomly drawn colors from the chosen palette for
the background.

``` r
set.seed(seed)

# Select colors for background
bkg_c <- sample.int(11, 1) + 4

render_scene(
  # Square
  #file = glue::glue("outputs/corpuscular-{seed}.png"),
  #width = 1500, 
  #height = 1500,
  
  # Mastodon Header
  #file = glue::glue("m_outputs/corpuscular-{seed}.png"),
  #width = 1500, 
  #height = 500,#1500,
  
  # iPhone 11
  file = glue::glue("outputs/ip11_corpuscular-{seed}.png"),
  width = 828, 
  height = 1792, 
  
  # windows wallpaper
  #file = glue::glue("w_outputs/corpuscular-{seed}.png"),
  #width = 2560, 
  #height = 1440, 
  scene, parallel = TRUE,
  ambient_light = TRUE,
  samples = 75 + abs(rnorm(1, 0, 100)), 
  backgroundhigh = col_palette[bkg_c],
  backgroundlow = col_palette[bkg_c - sample.int(4, 1)],
  lookfrom = c(sample(c(-1, 1), 1) * 13,
                          3 + runif(1, 
                                    min = -3, 
                                    max = 6), 
                          sample(c(-1, 1), 1) * 16), 
             lookat = c(0, 2, 0))
```

<img src="outputs/ip11_corpuscular-5554714.png" width="500px" />
