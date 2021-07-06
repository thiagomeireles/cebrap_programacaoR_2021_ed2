# Tutorial Extra 1: Acessando o Base dos Dados com SQL e queries

Assista o vídeo disponível no [YouTube](https://www.youtube.com/watch?v=5_Ir8neyFf4&ab_channel=BasedosDados) para ver os passos para acessar os dados via requisição de BigQuery.

## Criando usuário e projeto no BigQuery

1. Caso nunca tenha utilizado o [BigQuery do Google](https://cloud.google.com/bigquery), acesse e crie um novo usuário.

2. Após fazer o cadastro/login, crie um novo projeto (atribua qualquer nome que seja conveniente) acessando o "Console". 

3. Em seguida, selecione o projeto e copie o ID.

4. Copie esse link substituindo o <project_id>: https://console.cloud.google.com/apis/credentials/serviceaccountkey?project=<project_id>. Selecione uma nova conta de serviço para o seu projeto.

a. Atribua o nome da conta de serviço

b. Selecione o papel de "Proprietário"

c. Não conceda nenhum acesso adicional à conta de serviço

5. Voltando para a tela das contas de serviço do seu projeto, clique nos três pontinhos de "Ações" e em "Gerenciar Chaves".

a. Clique em "Adicionar Chave"

b. Mantenha a opção "JSON"

c. Clique em "criar"

## Abrindo pacotes e estabelecendo a conexão com BigQuery

### Pacotes necessários

Precisamos de três pacotes para seguir o Tutorial, então vamos instalá-los caso não estejam disponíveis em nossas máquinas:

```{r, eval=FALSE}
install.packages('DBI')
install.packages('bigrquery')
install.packages('ggplot2')
```

Em seguida, vamos carregálos:

```{r}
library(DBI)
library(bigrquery)
library(ggplot2)
```

### Realizando a autenticação para conectar com BigQuery

Para acessar os dados disponíveis na BigQuery, precisamos estabelecer uma conexão entre nossa máquina e nosso projeto criado no início do Tutorial.

Em um primeiro momento, realizamos a atuenticação indicando onde está o arquivo "json" que criamos e baixamos em nosso projeto do Google Cloud

```{r}
bq_auth(path = "Caminho no seu computador/arquivojson.json")

## Exemplo do caminho no meu computador
#bq_auth(path = "C:/Users/meire/Downloads/testecebrap-f01cc3184f09.json")
```

Em seguida, criamos nossa conexão com o BigQuery especificando o project_id no arqumento "billing".

```{r}
con <- dbConnect(
  bigrquery::bigquery(),
  billing = "", #aqui entra o projectID mais 
  project = "basedosdados"
)
```

## Exemplo 1: Dados do Sistema de Informações sobre Mortalidade (SIM)

### A nível de município

```{r}
query  <- "SELECT * FROM `basedosdados.br_ms_sim.municipio`"

df.sim <- dbGetQuery(con, query)
```

### Microdados para as 100 primeiras observações para 2010

```{r}
query <- "SELECT * FROM `basedosdados.br_ms_sim.microdados` WHERE ano = 2010 LIMIT 100"

df.sim.microdados <- dbGetQuery(con, query)
```

## Exemplo 2: Cruzando Dados

### Obtendo dados do PIB per capita

```{r}
query.pib_pc <- "SELECT 
    pib.id_municipio ,
    pop.ano, 
    pib.PIB / pop.populacao as pib_pc
FROM `basedosdados.br_ibge_pib.municipios` as pib
INNER JOIN `basedosdados.br_ibge_populacao.municipios` as pop
ON pib.id_municipio = pop.id_municipio AND pib.ano = pop.ano"

df.pib_pc = dbGetQuery(con, query.pib_pc)
```

### Obtendo dados de Desmatamento do Prodes do INPE

```{r}
query = "SELECT * FROM `basedosdados.br_inpe_prodes.desmatamento_municipios`"

df.prodes = dbGetQuery(con, query)
```

### Cruzando os dados com o base

```{r}
df.analise = merge(df.pib_pc, df.prodes, on=c("id_municipio","ano"))

# Convertendo o PIB per capita para log
df.analise$ln_pib_pc = log(df.analise$pib_pc)
```



## Brincando com requisições por SQL

