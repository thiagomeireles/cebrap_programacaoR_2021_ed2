# [Cebrap.lab] Curso de Programação em R: Manipulação de dados

## Objetivos gerais

Os tutoriais de hoje tratam de um tema que vai nos acompanhar por todo o curso: a manipulação de data frames. 

Nós vamos aprender a fazer isso de duas formas: usando a biblioteca *base* do R ("jeito velho") e a pacote *dplyr* ("o moderno").

## Roteiro para a aula

1- Apresentação do curso e dos participantes;

2- Ver muito rapidamente o ambiente de programação de R e o software RStudio e como utilizar o Git.

3- O [Tutorial 1](https://github.com/thiagomeireles/cebrap_programacaoR_2021_ed2/blob/main/tutoriais/Tutorial_01.md) apresenta os aspectos mais importantes para manipulação de bases de dados em R. No entanto, ele apresenta uma forma "velha" de fazer isso. No tutorial seguinte veremos formas mais eficientes de realizar as mesmas tarefas. Aqui o [Tutorial 1 versão Rmd](https://github.com/thiagomeireles/cebrap_programacaoR_2021_ed2/blob/main/tutoriais/Tutorial_01.Rmd)

4- Já o [Tutorial 2](https://github.com/thiagomeireles/cebrap_programacaoR_2021_ed2/blob/main/tutoriais/Tutorial_02.md) é o principal da aula. Ele apresentar a gramática do *dplyr* para manipulação de dados em R. Será nosso primeiro passo no curso dentro do universo *tidyverse*. Aqui o [Tutorial 2 versão Rmd](https://github.com/thiagomeireles/cebrap_programacaoR_2021_ed2/blob/main/tutoriais/Tutorial_02.Rmd)

5- O [Tutorial 3](https://github.com/thiagomeireles/cebrap_programacaoR_2021_ed2/blob/main/tutoriais/Tutorial_03.md) apresenta as bases de dados relacionais. Em outros termos, como fazemos para trabalhar com diferentes bases de dados que possuam algum identificador em comum. Ele é um pouco mais curto e opcional, só o faça se terminar os dois primeiros ou deixe como tarefa para casa. Aqui o [Tutorial 03 versão Rmd](https://github.com/thiagomeireles/cebrap_programacaoR_2021_ed2/blob/main/tutoriais/Tutorial_03.Rmd)

### RMarkdown

Os nossos tutoriais são feitos em Markdown e você pode "copiar" sua estrutura para um documento *RMarkdown* dentro do RStudio ou baixar (salvar como) o próprio arquivo *Rmd*. Ele permite a separação do que é código e do que é texto dentro dos *chunks*, as estruturas destacadas em cinza nos tutoriais que fizeram em casa.

Caso queiram explorar um pouco mais as opções para formatação de textos e código em *RMarkdown* podem acessar os tutoriais da aula do curso de [Programação em R para Ciências Sociais](http://htmlpreview.github.io/?https://github.com/leobarone/FLS6397_2018/blob/master/tutorials/tutorial08.html). 

O RStudio oferece dois guias com as principais funções do *RMarkdown*, um mais [curto](https://rstudio.com/wp-content/uploads/2015/02/rmarkdown-cheatsheet.pdf) e um pouco mais [longo](https://rstudio.com/wp-content/uploads/2015/03/rmarkdown-reference.pdf).

## Link para aula

É possível acessar o encontro pelo Zoom por [este link](https://zoom.us/j/92711183574?pwd=RENGbHUrZHpVbE9hMVRFckc2dzdtUT09).

A lista de presença está disponível [aqui](https://docs.google.com/spreadsheets/d/1MooKHN4icP2XBbXaIInmyEg-rxsdKgGV3ZtvY3NRor0/edit#gid=764662017).

## Slides

Os slides utilizados na aula podem ser baixados por [este link](https://github.com/thiagomeireles/cebrap_programacaoR_2021_ed2/blob/main/slides/cebraplab___Programa__o_em_R__Dia_1_.pdf).

## Dica de Leitura

O livro *R for Data Science* possui algumas seções que tratam dos tópicos de hoje. Ele é gratuito e já está entre as referências indicadas para o curso. Sua estrutrura também é voltada para uma abordagem mais prática, então a maioria dos capítulos são curtos e de leitura rápida e fácil. Se puderem, dêem uma olhada na [Introdução](http://r4ds.had.co.nz/introduction.html)

A Parte I do livro (Explore) trata dos temas de hoje. É possível encontrar uma introdução ao [*tidyverse*] (http://r4ds.had.co.nz/explore-intro.html). Por lá encontrará uma seção sobre manipulação de dados com o *dplyr* e do tema do próximo encontro, a produção de gráficos com a biblioteca *ggplot2*.
