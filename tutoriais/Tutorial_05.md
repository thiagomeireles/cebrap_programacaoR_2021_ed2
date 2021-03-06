## Visualização de mapas com *sf* e *ggplot2*

Por muito tempo trabalhar com mapas em R possuía diversas barreiras. A manipulação de dados associada aos *shapefiles* (arquivos com os polígonos para a produção dos mapas) era pouco intuitiva. No entanto, há alguns anos, foi apresentado um pacote que permite trabalhar com esses polígonos como trabalhamos com *dataframes* ou *tibbles*: o pacote *sf*. O nome do pacote vem de *simple feature* e dá uma ideia do que era esperado aqui. As coorenadas relacionadas ao polígono (ou ponto) são armazenadas como variável em um dataframe. O principal ganho, aqui, é a integração plena com as ferramentas *tidyverse*, especialmente *dplyr* e *ggplot2*.

Agora que estamos um pouco mais habituados com o R, uma dica importante em termos de preparação de scripts e rmarkdown: as opções e pacotes definidas para tudo que faremos são apresentadas no início. Como agora você já está mais habituado com pacotes e suas funções, vamos seguir essa dica. Primeiro vamos instalar os pacotes utilizados no tutorial de hoje:

```{r, eval = FALSE}
install.packages(c('sf', 'geobr', 'crul', 'basedosdados'))
```

Caso esteja utilizando o Tutorial em *Rmd* verá que o *chunk* de instalação tem uma opção `eval = FALSE`. Basicamente, aqui estamos incluindo o código de instalação mas, ao mesmo tempo, falando para que ele não seja executado se rodarmos todo o código do arquivo. De outra forma, ele não é executado por padrão, mas dá a opção de instalação para quem não tem as bibliotecas em sua máquina. Bacana, não?

Seguindo, carregaremos os pacotes necessários para o tutorial:


```{r}
library(sf)
library(geobr)
library(basedosdados)
library(tidyverse)
```

### Importando um shapefile com o pacote *sf*

A forma tradicional para importar as geometrias específicas para mapas, os *shapefiles*, demanda a presença desse tipo de arquivo. Vamos baixar um arquivo *.zip* com o shapefile para os municípios brasileiros disponível no repositório do curso, e extrair os arquivos: 

```{r, eval=FALSE}
download.file("https://raw.githubusercontent.com/thiagomeireles/cebrap_programacaoR_2021_ed2/main/data/Brazil_s.zip", destfile="Brazil_s.zip")
unzip('Brazil_s.zip')
```

A função para abertura dos dados é bastante simples, basta chamarmos pela *read_sf* e indicar o arquivo *.shp*.

```{r, eval=FALSE}
municipios <- read_sf("Brazil_s.shp")
```

Vamos utilizar a função *glimpse* para entendermos o objeto criado: 

```{r}
glimpse(municipios)
```

Veja que temos um *dataframe* ao chamarmos pelo *sf* e não um objeto identificado como *shapefile*. A geometria está especificada na variável "geometry", poderíamos apenas incorporar os dados que queremos apresentar nos mapas como novas variáveis.

Por enquanto, vamos apenas ver como produzir o mapa com o *ggplot2* utilizando uma nova geometria, a *geom_sf*:

```{r, eval=FALSE}
municipios %>% 
  ggplot() +
  geom_sf()
```

Ao utilizar dados espaciais podemos lidar com um problema relativamente comum: a projeção dos objetos pode ser diferente para cada shapefile. Existem diferentes CRS/projeções (coordinate reference system), ou seja, sistemas de referência de coordenadas. No caso do utilizado aqui, se observarmos a CRS/projeção de 'municipios' - está escrito "+proj=longlat +ellps=GRS80 +no_defs". No entanto, a projeção mais comum é a 'WGS84'. Realizar a conversão a partir da GRS80 é bastante simples. Basta utilizar a função *st\_transform* e aplicar o código 4326, atalho para WGS84. A partir disso, poderíamos incorporar diferentes camadas de mapas que estariam em WGS84:

```{r}
st_crs(municipios)

municipios <- municipios %>% 
  st_transform(4326)

rm(municipios)
```

Esse exemplo é interessante, especialmente para trabalhar com *shapefiles* de outros países ou regiões. Para o Brasil, veremos que existe uma forma muito mais simples. 

### O pacote *geobr*

Na forma tradicional precisamos encontrar o shapefile (em repositórios como do IBGE e CEM), mas, felizmente, para o Brasil existe um pacote para facilitar nossa vida. O *geobr* nos oferece uma alternativa simples e fácil para acessar dados espaciais brasileiros. Com ele é muito simples realizar um download rápido de diversos arquivos de geometrias espaciais para o Brasil. Vamos ver o que está disponível no pacote:

```{r}
mapas_geobr <- list_geobr()

print(mapas_geobr)
```

No dataframe que criamos, vemos as colunas de (1) função; (2) nível geográfico; (3) anos e (4) fontes. Assim temos funções que saem do nível de país até as áreas de ponderação do Censo para diversos anos - ou seja, é possível captar criação de municípios, mudanças em setores censitários, etc. E tudo a partir de uma função bastante simples e intuitiva.

Vamos observar o mapa para os estados no ano de 2020 já gerando o gráfico:

```{r}
geom_estados <- read_state(year = 2020)

geom_estados %>% 
  ggplot() +
  geom_sf(color = 'yellow', fill = 'black', alpha =0.5) +
  labs(x = NULL,
       y = NULL,
       title = "Mapa dos estados brasileiros (2020)") +
  theme_bw()
```

Perceba que podemos aplicar os parâmetros utilizados em outras geometrias do *ggplot2*. Não ficou muito bacana, vamos incorporar alguns dados ao nosso shapefile e produzir mapas mais bonitos.

Dica: muito cuidado na construção dos mapas. **SEMPRE** mantenha as coordenadas geográficas, retirar as referências é conceitualmente errado para a construção de mapas.

### A principal vantagem do *sf*: incorporar dados aos nossos mapas

Como dito, a principal vantagem que o *sf* traz para a produção de mapas em R é permitir a incorporação da dados a partir de identificadores, como já vimos com o dplyr. Vamos utilizar o pacote *basedosdados* para buscar dados no nível municipal e estadual. Primeiros vamos definir o ID do projeto de cobrança de download dos dados:

```{r}
set_billing_id('exemplary-tide-310619') # insira seu ID
```

### Nível municipal: dados de mortalidade

Primeiro vamos começar com um mapa para o Sistema de Informações sobre Mortalidade para o ano de 2019. O primeiro passo é baixar os dados do SIM no nível municipal:

```{r}
query_sim <- 'SELECT * FROM `basedosdados.br_ms_sim.municipio` WHERE ano = 2019'

df_sim    <- read_sql(query_sim)
```

Em sequência, vamos buscar os dados para a população estimada em 2019 no nível municipal:

```{r}
query_pop <- 'SELECT id_municipio, populacao FROM `basedosdados.br_ibge_populacao.municipio` WHERE ano = 2019'

df_pop <- read_sql(query_pop) 
```

Podemos, assim, cruzar os dados entre as duas bases e calcular a taxa de mortalidade em cada município:

```{r}
df_sim <- df_sim %>% 
  mutate(id_municipio = as.character(id_municipio)) %>% 
  left_join(df_pop) %>% 
  mutate(taxa_mortalidade = (numero_obitos/populacao)*100,
         id_municipio     = as.double(id_municipio))
```

Agora geramos um mapa com os dados já integrados a partir do download do mapa no nível municipal para 2019 com o *geobr*:

```{r}
mapa_mun <- read_municipality(year = 2019) %>% 
  rename(id_municipio = code_muni) %>% 
  left_join(df_sim)
```

Enfim, a primeira produçaõ de um mapa utilizando o *geom\_sf* e dados incorporados via *dplyr*. Note que os parâmetros para a geometria são comuns aos que vimos no Tutorial 4 sobre o *ggplot2*:

```{r}
mapa_mun %>% 
  ggplot() + 
  geom_sf(aes(fill = taxa_mortalidade))
```

Sejamos sinceros, é um mapa muito feio. Vamos tirar as linhas que marcam a divisão entre os municípios. Como é um parâmetro não relacionado aos dados, ele não fica dentro do *aes*, lembram?

```{r}
mapa_mun %>% 
  ggplot() + 
  geom_sf(aes(fill = taxa_mortalidade), color = NA)
```

Melhorou bastante, não? Vamos inverter a ordem das cores (mais fortes com maior taxa), corrigir legenda, tema e título do mapa pensando nos daltônicos:

```{r}
mapa_mun %>% 
  ggplot() + 
  geom_sf(aes(fill = taxa_mortalidade), color = NA) + 
  scale_fill_viridis_c(option = "A", direction = -1) +
  labs(x = NULL,
       y = NULL,
       title = "Taxa de mortalidade por município Brasileiro em 2019",
       fill  = "Taxa de Mortalidade (%)") +
  theme_bw()
```

O mapa ficou bonito, não? Mas ele é de difícil interpretação justamente por ter muita informação e o número de municípios ser gigantesco. Vamos fazer uma divisão em categorias e ver a diferença? Olhemos para os quartis, por exemplo. Para isso, utilizamos a função *quantile* e atribuimos as faixas de cada quantil que queremos. Com a função seq, indicamos que queremos as probabilidades de 0 a 1 em intervalos de 0.25. Ou seja, os quartis. Incluímos o argumento *na.rm* como verdadeiro para remover os dois missing values pela diferença entre o tamanho de observações do mapa (5572) e do nosso dataframe de mortalidade (5570). 

```{r}
quantile(mapa_mun$taxa_mortalidade, probs = seq(from = 0, to = 1, by = 0.25), na.rm = T)
```

Vamos pensar em categorias aqui. Podemos criar 3: uma baixa próxima ao primeiro quartil, uma média próxima ao terceiro quartil e uma alta acima dele. Faremos isso utilizando a função *cut*, utilizada para estalecer pontos de corte em variáveis contínuas. Aqui estabelecemos o início do intervalo, os cortes para cada faixa e o limite superior. Em sequência, converteremos em uma *factor* ordenada atribuindo os labels. Vejamos tudo isso em poucas linhas de código: 

```{r}
mapa_mun <- mapa_mun %>% 
  mutate(taxa_cat = cut(taxa_mortalidade, breaks = c(0,
                                                     .55,
                                                     .80,
                                                     2.5)),
         taxa_cat = factor(taxa_cat,
                           labels = c("Baixa", "Média", "Alta"),
                           ordered = T))
```

Vamos agora produzir nosso mapa com categorias:

```{r}
mapa_mun %>% 
  filter(!is.na(taxa_cat)) %>% 
  ggplot() + 
  geom_sf(aes(fill = taxa_cat), color = NA) + 
  scale_fill_viridis_d(option = "B", direction =  -1) +
  labs(x = NULL,
       y = NULL,
       title = "Taxa de mortalidade por município Brasileiro em 2019",
       fill  = "Taxa de Mortalidade") +
  theme_bw()
```

Curioso o resultado, não?

### Nível estadual

Vamos, primeiro, fazer o download dos microdados da vacinação no Brasil agrupando o número total de aplicações para uma ou duas doses:

```{r}
query_vacinacao<- "SELECT sigla_uf, dose, SUM(1) AS vacinas_aplicadas FROM `basedosdados.br_ms_vacinacao_covid19.microdados_vacinacao` GROUP BY sigla_uf, dose"

df_vacinacao <- read_sql(query_vacinacao) %>% 
  filter(!is.na(dose))
```

Ok, temos os dados para primeira e segunda dose por estado. Msa comparar o número absoluto não parece uma boa ideia. Vamos mais uma vez ver a porcentagem de vacinados para cada dose em cada um dos estados. Para isso precisamos da população por UF. Podemos obter essa informação em um dataframe que utilizamos, o *df_sim* agrupando a população por unidade federativa. Vamos lá:

```{r}
df_pop_est <- df_sim %>% 
  group_by(sigla_uf) %>% 
  summarise(pop = sum(populacao)) %>% 
  ungroup()
```

Fácil, não? Vamos adicionar a coluna da população e ver a cobertura vacinal total para cada estado e número da dose

```{r}
df_vacinacao <- df_vacinacao %>% 
  left_join(df_pop_est) %>% 
  mutate(cobertura = (vacinas_aplicadas/pop)*100)
```

Agora uniremos os dados de vacinação ao mapa dos estados obtido com o pacote *geobr*:

```{r}
mapa_est <- read_state(year = 2020) %>% 
  rename(sigla_uf = abbrev_state) %>% 
  left_join(df_vacinacao)
```

Vamos produzir o mapa de forma direta, sem muitas etapas aqui, com parâmetros utilizados no mapa municipal:

```{r}
mapa_est %>% 
  ggplot() + 
  geom_sf(aes(fill = cobertura), color = NA) + 
  scale_fill_viridis_c(option = "A", direction = -1) +
  labs(x = NULL,
       y = NULL,
       title = "Cobertura vacinal para Covid-19 por Unidade Federativa",
       fill  = "Cobertura Vacinal (%)") +
  theme_bw() +
  facet_wrap(~as.factor(dose))
```

Vamos aplicar faixas percentuais? A visualização deve melhorar e tornar mais comprável o que fizemos. Tudo isso dentro de um mesmo "pipe", você já deve ser capaz de entender bem a sequência. Além disso, mudaremos a cor de fundo do título do quadrante para branco, substituindo o cinza.

```{r}
mapa_est %>% 
  mutate(taxa_cobertura = cut(cobertura, breaks = c(0,
                                                    10,
                                                    20,
                                                    30,
                                                    40,
                                                    Inf)),
         taxa_cobertura = factor(taxa_cobertura,
                                 labels = c("0 a 10",
                                            "Acima de 10 a 20",
                                            "Acima de 20 a 30",
                                            "Acima de 30 a 40",
                                            "Acima de 40"),
                                 ordered = T)) %>% 
  ggplot() + 
  geom_sf(aes(fill = taxa_cobertura), color = "white") + 
  scale_fill_viridis_d(option = "A", direction = -1) +
  labs(x = NULL,
       y = NULL,
       title = "Cobertura vacinal para Covid-19 por Unidade Federativa",
       fill  = "Cobertura Vacinal (%)") +
  theme_bw() +
  facet_wrap(~as.factor(dose)) +
  theme(strip.background = element_rect(fill = 'white'))
```

Muito mais fácil interpretar, não?

Aqui cobrimos uma boa parte do que fazemos com gráficos e mapas utilizando o *ggplot2*, mas existem diversas opções de personalização que vão além do que fizemos. Tente explorar essas opções, especialmente ligadas a *theme* quando tiver um pouco de tempo ou quando trabalhar nos seus próprios gráficos e mapas.