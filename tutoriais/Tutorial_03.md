## Bases de dados relacionais com a gramática do pacote dplyr

Por enquanto trabalhamos apenas com uma base de dados e manipulamos de diversas formas (nomes de variáveis, novas variáveis, seleção de linhas e colunas, agrupamentos e ordenações). Agora veremos como combinar *data\_frames* diferentes usando as funções do tipo *join* do pacote *dplyr*. Já vamos deixar o pacote carregado:

```{r}
library(dplyr)
```

## Comparação dos pagamentos entre janeiro de 2011 e janeiro de 2017 no Programa Bolsa Família

Aqui trabalharemos, diferentemente do tutorial anterior, não com uma amostra de base de dados do Bolsa Família, mas com os dados de pagamentos para um município pequeno do interior de São Paulo, Borá, para janeiro de dois anos distintos (2011 e 2017).

Os dados foram extraídos do Portal da Transparência. Se quiser, pode acessar os dados originais e fazer o processo de filtragem manualmente depois da aula para treinar as habilidades de manipulação vistas no tutorial anterior para gerar os dados que estão no repositório do curso. Baixe os dados manualmente no Portal da Transparência e use a função *fread* do pacote *data.table* (esse *data\_frame* é grande e a função é bem útil nesses casos). Após isso, selecione as linhas de Borá com a função *filter* do pacote *dplyr*.

Para conseguirmos cumprir os tutoriais propostos para essa aula, por enquanto utilizaremos os dados já filtrados e disponíveis nos links do código abaixo:

```{r}
library(readr)
pagamentos11 <- read_delim("https://raw.githubusercontent.com/thiagomeireles/cebrap_programacaoR_2021_ed2/main/data/pagamentos11.csv", delim = ";", col_names = T)
pagamentos17 <- read_delim("https://raw.githubusercontent.com/thiagomeireles/cebrap_programacaoR_2021_ed2/main/data/pagamentos17.csv", delim = ";", col_names = T)
```

Perceba que a estrutura dos dados é igual, sendo as bases bastante parecidas. No entanto, é esperado que haja variação entre os anos e que os beneficiários e valores sejam diferentes, uma vez que as pessoas entram e saem do programa ao longo do tempo, mudam de município, os valores sejam reajustados ou a estrutura das famílias se altere.

Agora buscaremos identificar essas mudanças. Mas como saber quem estava em 2011 e também em 2017? E como calcular a variação dos valores para cada beneficiário?

### Exercício para casa

Examine as bases de dados "pagamentos11" e "pagamentos17" antes de começarmos a trabalhar com elas. Faça também as seguintes alterações:

- Renomeie as variáveis "NIS Favorecido", "Nome Favorecido" e "Valor Parcela" para "nis", "nome" e "valor", repectivamente.
- Transforme a variável valor em numérica.
- Selecione apenas as três variáveis renomeadas.
- Quantas linhas tem cada base de dados?

### Resposta parcial ao exercício anterior (3 primeiros itens)

```{r}
# 2011
pagamentos11 <- pagamentos11 %>% 
  rename(nis   = `NIS Favorecido`, nome = `Nome Favorecido`, valor = `Valor Parcela`) %>%
  mutate(valor = gsub(",", "", valor), valor = as.numeric(valor)) %>%
  select(nis, nome, valor)

# 2017
pagamentos17 <- pagamentos17 %>% 
  rename(nis   = `NIS Favorecido`, nome = `Nome Favorecido`, valor = `Valor Parcela`) %>%
  mutate(valor = gsub(",", "", valor), valor = as.numeric(valor)) %>%
  select(nis, nome, valor)
```

## Left e Right Join

Tendo as bases preparadas e sabendo que os beneficiários variam entre os anos, podemos começar a "juntá-las".

Os *joins* se baseiam em combinações de bases de dados a partir de variáveis que associe as observações em uma base de dados com a outra. Para indivíduos, algumas variáveis naturais para isso são CPF, NIS (Número de Identificação Social no CadÚnico) e título de eleitor. Já para municípios, temos os códigos do município (como do IBGE e TSE), enquanto para os estados normalmente é fácil a utilização da UF ou do código do estado do IBGE. No entanto, nem sempre temos esses dados disponíveis e podemos utilizar outras "chaves" para conectar as bases de dados. No caso da nossa base do PBF, temos o NIS como identificador já disponível.

Os dois tipos mais comuns são o *left_join* e o *right_join*. Com essa duas combinações, a primeira base de dados é mantida integralmente, ou seja, mesmo as observações sem correspondência na outra tabela ficam; a inclusão dos dados da segunda base se dá **apenas** quando encontra correspondência na primeira.

Vamos operar os dois "joins" e ver o que ocorre. Primeiro, *left_join*:

```{r}
comb_left <- left_join(pagamentos11, pagamentos17, by = "nis")
```

Veja que informamos ao R que a variável que conecta as tabelas é "nis" informando o parâmetro "by".

Quantas linhas resultaram da combinação? Exatamente o mesmo número de linhas de "pagamentos11". Use o *View* para ver o resultado.

Veja que as variáveis exceto NIS são incluídas. Assim,  há duas variáveis de valor, uma com final ".x" e outra com o final ".y". Essa diferenciação acontece porque as duas bases de dados tem os mesmos nomes de variáveis. O mesmo processo acontece com a variável "nome". Vamos renomeá-las:

```{r}
comb_left <- comb_left %>% rename(valor11 = valor.x,
                                  nome11  = nome.x, 
                                  valor17 = valor.y,
                                  nome17  = nome.y)
```

Em diversas observações, o valor e o nome para 2017 contém "NA", o símbolo do R para "missing values". Do ponto de vista prático, isso significa que aquela observação existia em 2011, mas não em 2017. Como não encontrou correspondência, o R manteve a observação sem incluir nenhum valor para 2017.

E as outras observações para 2017? Perceba que o número de linhas da combinação é o mesmo, logo elas não foram incluídas. Ao utilizar o *left_join* garantimos a manutenção de todas as observações de 2011, mesmo sem encontrar correspondência. Só que esse processo leva à exclusão de todas as observações de 2017 que não encontraram par em 2011.

Vamos deixar de lado rapidamente esta primeira combinação e realizar o *right_join* das mesmas tabelas e observar o processo inverso:

```{r}
comb_right <- right_join(pagamentos11, pagamentos17, by = "nis")
```

Novamente, vamos renomear as variáveis:

```{r}
comb_right <- comb_right %>% rename(valor11 = valor.x,
                                    nome11  = nome.x, 
                                    valor17 = valor.y, 
                                    nome17  = nome.y)
```

Veja que agora há "missing values" nos valores e nomes de 2011. O *right_join* preserva todas as observações de 2017, mesmo sem correspondência em 2011, e não inclui nenhum de 2011 que não encontre correspondência em 2017.

Note que "left" e "right" são exatamente a mesma operação, mas com as tabelas em posição (x e y, 1a e 2a, etc) invertidas.

## Inner e Full Join

No entanto, é possível manter apenas as observações que estão em ambos os anos utiizando o *innner_join*. Vamos observar o resultado já renomeando as variáveis:

```{r}
comb_inner <- inner_join(pagamentos11, pagamentos17, by = "nis")
comb_inner <- comb_inner %>% rename(valor11 = valor.x, 
                                    nome11  = nome.x, 
                                    valor17 = valor.y, 
                                    nome17  = nome.y)
```

Não existem "missing values", uma vez que permaneceram apenas os casos que estão em ambos os *data\_frames*.

E se quisermos manter todas as observações, mesmo as que não possuem correspondência? Aí temos o *full_join*. Ele inclui todas as observações, independente da correspondência, inserindo "missing values" quando não há.

```{r}
comb_full <- full_join(pagamentos11, pagamentos17, by = "nis")
comb_full <- comb_full %>% rename(valor11 = valor.x, 
                                  nome11  = nome.x, 
                                  valor17 = valor.y, 
                                  nome17  = nome.y)
```

## Semi e anti joins

Os quatro tipos de "join" apresentados anteriormente cobrem a totalidade de situações de combinação entre tabelas a partir de um "chave", ou seja, de um índice ou variável que permita estabelecer a relação entre elas.

Há, porém, dois outros tipos de "joins" disponíveis no R bastante úteis.

Se quisermos trabalhar apenas em uma única base de dados, por exemplo, pagamentos11, mas queremos saber quais das observações de 2011 também estão na tabela de 2017, então utilizamos a função *semi\_join*. O resultado será semelhante ao da aplicação de *inner\_join*, mas sem que novas colunas com os dados de 2017 tenham sido criadas:

```{r}
comb_semi <- semi_join(pagamentos11, pagamentos17, by = "nis")
```

Por fim, *anti\_join*, tem comportamento semelhante a *semi\_join*, mas, em vez de retornar as observações de 2011 que têm correspondência em 2017, retorna as que nâo têm par em 2017:

```{r}
comb_anti <- anti_join(pagamentos11, pagamentos17, by = "nis")
```

É perfeitamente possível usar o operador %>% (pipe, como é chamado), para os "joins". Basta colocar a base na posição "x" (primeira a ser inserida) antes do operador. Veja um exemplo:

```{r}
comb_left <- pagamentos11 %>% 
  left_join(pagamentos17, by = "nis")
```

## Combinação de tabela e agregações cumulativas

Vamos supor que queremos calcular os valores total, médio, máximo, etc, por município e, a seguir, apresentar esses valores como colunas para cada observação. Uma maneira eficiente de fazer isso é a usando a combinação de tabelas. Vamos ver como voltando ao exemplo da amostra de saques do Programa Bolsa Família em 2017.

### Exercício

Abra a base de dados e faça as transformações necessárias (renomear variáveis e transformar a variável valor em numérica) antes de prosseguir. Tente fazê-lo sem olhar a resposta abaixo.

## Resposta ao exercício anterior

```{r}
saques_amostra_201701 <- read_delim("https://raw.githubusercontent.com/thiagomeireles/cebrap_programacaoR_2021_ed2/main/data/saques_amostra_201701.csv",
                                    delim = ";", escape_double = FALSE, 
                                    col_types = cols(`Valor Parcela` = col_character())) %>% 
  rename(uf         = UF, 
         munic      = `Nome Município`,
         cod_munic  = `Código SIAFI Município`, 
         nome       = `Nome Favorecido`,
         valor      = `Valor Parcela`, 
         mes        = `Mês Competência`, 
         data_saque = `Data do Saque`) %>% 
  select(uf, munic, cod_munic, nome, valor, mes, data_saque) %>% 
  mutate(valor_num  = as.numeric(gsub(",", "", valor)))
```

## Pasos para agregações cumulativas

Em primeiro lugar, vamos construir uma tabela agrupada por município usando *group\_by* e *summarise*:

```{r}
valores_munic <- saques_amostra_201701 %>% 
  group_by(cod_munic) %>% 
  summarise(contagem = n(),
            soma     = sum(valor_num),
            media    = mean(valor_num),
            mediana  = median(valor_num),
            desvio   = sd(valor_num),
            minimo   = min(valor_num),
            maximo   = max(valor_num)) %>% 
  ungroup()
```

Agora temos dois *data\_frames*, o original e a tabela "valores_munic". Podemos combiná-los a partir da variável "cod_munic". Assim, com o *left\_join* podemos levar as colunas da nova tabela à base de dados original:

```{r}
saques_amostra_201701 <- saques_amostra_201701 %>% 
  left_join(valores_munic, by = "cod_munic")
```

Use _View_ para observar que a base de dados original tem agora 7 novas colunas com informações agregadas por município (e que, portanto, se repetem para observações de um mesmo município).

Se quisermos fazer de forma direta, basta substituir o *summarise* por *mutate* quando criamos a tabela "valores_munic".

```{r}
saques_amostra_201701 <- saques_amostra_201701 %>% 
  group_by(cod_munic) %>% 
  mutate(contagem = n(),
         soma    = sum(valor_num),
         media   = mean(valor_num),
         mediana = median(valor_num),
         desvio  = sd(valor_num),
         minimo  = min(valor_num),
         maximo  = max(valor_num)) %>% 
  ungroup()
```

### Exercício para casa.

Respire fundo e gaste um tempo refletindo sobre os "joins". Você acabou de aprender como operar bancos de dados relacionais e pode parecer bastante difícil num primeiro momento. Em seguida:

- Calcule o total de valores por UF em um novo _data frame_.
- Combine o novo _data frame_ com o original para levar a coluna de total de valores ao último.
- A seguir, calcule quanto cada indivíduo na amostra representa, em termor percentuais (dica: crie uma nova variável utilizando _mutate_).