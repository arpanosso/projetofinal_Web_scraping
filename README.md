
<!-- README.md is generated from README.Rmd. Please edit that file -->

# Projeto Final - Web scraping

### Aluno: Alan R. Panosso

### Data: 26/07/2021

<!-- badges: start -->
<!-- badges: end -->

Projeto final apresentado aos instrutores **Julio Trecenti** e **Caio
Lente** da Curso-R, como parte das exigências para a finalização do
curso de **Web scraping**.

Objetivo do projeto é criar um produto de dados a partir do portal de
transparência da UNESP.

Além a aquisição e parseamento da informação, serão apresentados os
tratamentos iniciais e a construção de alguns gráficos e apresentação de
resultados.

# Aquisição de dados

**1)** Acesse o endereço <https://www2.unesp.br/>

<img src="https://raw.githubusercontent.com/arpanosso/projetofinal_Web_scraping/master/inst/ws_01.png" width="800px" style="display: block; margin: auto;" />

<br />

**2)** Clique em *Transparência* e em seguida selecione *Transparência
Unesp*.

<img src="https://raw.githubusercontent.com/arpanosso/projetofinal_Web_scraping/master/inst/ws_02.png" width="800px" style="display: block; margin: auto;" />

<br />

**3)** Na próxima página, no setor *GESTÃO*, selecione **Despesas
pagas**, para acessar os dados da execussão orçamentária da universidade
a partir de 2016.

<img src="https://raw.githubusercontent.com/arpanosso/projetofinal_Web_scraping/master/inst/ws_03.png" width="800px" style="display: block; margin: auto;" />

<br />

**4)** Será apresentado um campo para selecionar uma das unidades da
universidade, no exemplo, selecione **10061 - UNESP CONSOLIDADA**.

<img src="https://raw.githubusercontent.com/arpanosso/projetofinal_Web_scraping/master/inst/ws_04.png" width="800px" style="display: block; margin: auto;" />

<br />

**5)** Novos campos serão apresentados, então, fizemos o preenchimento
dos mesmos, como apresentado abaixo, para verificar os filtros
disponíveis. Será gerado um relatório com *infobox*’s e gráficos,
contudo, sem a tabela de dados de interesse.

<img src="https://raw.githubusercontent.com/arpanosso/projetofinal_Web_scraping/master/inst/ws_05.png" width="800px" style="display: block; margin: auto;" />

<br />

**6)** Rolando essa página até o final, pode-se observar a opção para
acesso da **Tabela Completa**, como apresentado abaixo.

<img src="https://raw.githubusercontent.com/arpanosso/projetofinal_Web_scraping/master/inst/ws_06.png" width="800px" style="display: block; margin: auto;" />

<br />

**7)** Será apresentado a tabela com os dados, ou seja, encontramos a
nossa **url base**:
“<https://ape.unesp.br/orcamento_anual/ddp_tabela.php>”.

<img src="https://raw.githubusercontent.com/arpanosso/projetofinal_Web_scraping/master/inst/ws_07.png" width="800px" style="display: block; margin: auto;" />

<br />

**8)** Para entender as requisições realizadas, utilizamos a opção
Inspecionar do navegador, clicando **Network**, atualizamos a página com
o **F5** e ordenamos os processos por tipo (Type). Observando o tipo
“document”, clicamos em **ddp\_tabela.php**.

<img src="https://raw.githubusercontent.com/arpanosso/projetofinal_Web_scraping/master/inst/ws_08.png" width="800px" style="display: block; margin: auto;" />

<br />

**9)** A requisição realizada foi do tipo **POST**.

<img src="https://raw.githubusercontent.com/arpanosso/projetofinal_Web_scraping/master/inst/ws_10.png" width="800px" style="display: block; margin: auto;" />

**10)** Rolando a aba do *Header*, verificamos o parâmetros do **body**
da requisição, destacados abaixo.

<img src="https://raw.githubusercontent.com/arpanosso/projetofinal_Web_scraping/master/inst/ws_11.png" width="800px" style="display: block; margin: auto;" />

## Download dos arquivos

Vamos testar a nossa requisição, e observamos que a tabela foi acessada.

``` r
u_base  <- "https://ape.unesp.br/orcamento_anual/ddp_tabela.php"

body_unesp <- list(
  txtunidade= 1,
  txtano= 2016,
  txtmes_inicial= 1,
  txtmes_final= 1,
  operacao_atual= "buscar_campos",
  token_ddp= ""
)

# Vamos salvar o arquivo para testar posteriormente
r_unesp <- httr::POST(
  u_base, 
  body = body_unesp,
  httr::write_disk("output/unesp.html"))

# teste do arquivo
minha_pagina_teste <- readr::read_file("output/unesp.html")
```

Seguindo as instruções, vamos checar o encoding do site

``` r
readr::guess_encoding(u_base)
#> # A tibble: 2 x 2
#>   encoding   confidence
#>   <chr>           <dbl>
#> 1 ISO-8859-1       0.39
#> 2 ISO-8859-2       0.25
```

Vamos acessar a tabela:

``` r
httr::content(r_unesp, encoding = "ISO-8859-1")  |> 
  xml2::xml_find_first("//table")
#> {html_node}
#> <table width="100%" border="0" cellpadding="2" cellspacing="2">
#> [1] <tr>\n<td valign="top"><div class="form-group">\r\n                    <l ...
```

Transformando a tabela em data.frame.

``` r
# Função para mudar o formato dos números
muda_numero <- function(x){
    as.numeric(
       stringr::str_replace(stringr::str_remove_all(x,"\\."),",",".")
       )
}


httr::content(r_unesp, encoding = "ISO-8859-1")  |> 
  xml2::xml_find_all("//table/tbody") |> 
  rvest::html_table(header = FALSE) |>   
  as.data.frame() |> 
  dplyr::mutate(
    dplyr::across(X2:X11,muda_numero)
    ) |> 
  dplyr::filter(stringr::str_starts(X1,"0")) |> 
  dplyr::mutate(
    categoria = stringr::str_sub(X1,2,2),
    categoria = dplyr::case_when(
      categoria == 1 ~ "PESSOAL_REFLEXOS",
      categoria == 2 ~ "PLANTOES",
      categoria == 3 ~ "DESPESAS_CORRENTES",
      categoria == 4 ~ "DIVIDAS_SENTENCAS",
      categoria == 5 ~ "DESPESAS_INVESTIMENTOS",
      TRUE ~ as.character(categoria)
    ),
    X1 = stringr::str_remove_all(X1,"[:digit:]|-")
   ) |> 
  dplyr::relocate(categoria) |> 
  dplyr::rename(
      despesa = X1,
      tesouro_orcamento_vigente = X2,
      convenios_orcamento_vigente = X3,
      rec_propria_orcamento_vigente= X4,
      tesouro_restos_pagar= X5,
      convenios_restos_pagar= X6,
      rec_propria_restos_pagar= X7,
      tesouro_diversos_credores= X8,
      convenios_diversos_credores= X9,
      rec_propria_diversos_credores= X10,
      total= X11
    ) |> 
  head(3)
#>          categoria                despesa tesouro_orcamento_vigente
#> 1 PESSOAL_REFLEXOS     Folha de Pagamento                 469308.56
#> 2 PESSOAL_REFLEXOS   Encargos com Pessoal                  10101.09
#> 3 PESSOAL_REFLEXOS        Auxílio Funeral                  98795.08
#>   convenios_orcamento_vigente rec_propria_orcamento_vigente
#> 1                           0                             0
#> 2                           0                             0
#> 3                           0                             0
#>   tesouro_restos_pagar convenios_restos_pagar rec_propria_restos_pagar
#> 1            154370580                      0                  6200.00
#> 2             25146047                      0                  1395.92
#> 3                    0                      0                     0.00
#>   tesouro_diversos_credores convenios_diversos_credores
#> 1                         0                           0
#> 2                         0                           0
#> 3                         0                           0
#>   rec_propria_diversos_credores        total
#> 1                             0 154846088.23
#> 2                             0  25157544.37
#> 3                             0     98795.08
```

Inpecionando a página, verificamos que a unesp tem 53 unidades, assim,
criamos a seguinte função para renomear as unidades, quando necessário:

``` r
nome_unidade <- function(x){
  dplyr::case_when(
    x==1 ~"UNESP CONSOLIDADA",
    x==2 ~"REITORIA ADMINISTRAÇÃO SUPERIOR",
    x==3 ~"REITORIA",
    x==4 ~"INSTITUTO DE FISICA TEORICA",
    x==5 ~"COORDENADORIA GERAL DE BIBLIOTECAS",
    x==6 ~"INSTITUTO DE PESQUISAS METEOROLOGICAS DE BAURU",
    x==7 ~"CENTRO DE DOCUMENTAÇÃO E MEMORIA DA UNESP",
    x==8 ~"CENTRO DE ESTUDOS AMBIENTAIS",
    x==9 ~"CENTRO DE AQUICULTURA DA UNESP",
    x==10 ~"CENTRO DE RADIO E TELEVISAO CULTURAL E EDUCATIVA",
    x==11 ~"CENTRO DE RAIZES E AMIDOS TROPICAIS",
    x==12 ~"CENTRO DE ESTUDOS DE VENENOS E ANIMAIS PEÇONHENTOS",
    x==13 ~"INSTITUTO DE POLÍTICAS PÚBLICAS E RELAÇÕES INTERNACIONAIS",
    x==52 ~"INSTITUTO DE ESTUDOS AVANÇADOS DO MAR",
    x==14 ~"FACULDADE DE ODONTOLOGIA DE ARARAQUARA",
    x==15 ~"FACULDADE DE CIÊNCIAS FARMACÊUTICAS",
    x==16 ~"FACULDADE DE CIENCIAS E LETRAS DE ARARAQUARA",
    x==17 ~"INSTITUTO DE QUÍMICA DE ARARAQUARA",
    x==18 ~"FACULDADE DE CIÊNCIAS HUMANAS E SOCIAIS",
    x==19 ~"FACULDADE DE CIÊNCIAS AGRÁRIAS E VETERINÁRIAS DE JABOTICABAL",
    x==20 ~"FAZENDA DE ENSINO E PESQUISA E PRODUCAO",
    x==21 ~"INSTITUTO DE BIOCIÊNCIAS DE RIO CLARO",
    x==22 ~"INSTITUTO DE GEOCIÊNCIAS E CIÊNCIAS EXATAS",
    x==23 ~"BOTUCATU ADMINISTRAÇÃO GERAL",
    x==24 ~"FACULDADE DE MEDICINA DE BOTUCATU",
    x==25 ~"FACULDADE DE MEDICINA VETERINARIA E ZOOTECNIA DE BOTUCATU",
    x==26 ~"FACULDADE DE CIENCIAS AGRONOMICAS DE BOTUCATU",
    x==27 ~"FAZENDA DE ENSINO, PESQUISA E PRODUCAO",
    x==28 ~"INSTITUTO DE BIOCIÊNCIAS DE BOTUCATU",
    x==29 ~"INSTITUTO DE ARTES DE SAO PAULO",
    x==30 ~"FACULDADE DE ENGENHARIA DE GUARATINGUETA",
    x==31 ~"INSTITUTO DE CIÊNCIA E TECNOLOGIA DE SÃO JOSÉ DOS CAMPOS",
    x==32 ~"FACULDADE DE CIENCIAS E LETRAS DE ASSIS",
    x==33 ~"FACULDADE DE FILOSOFIA E CIENCIAS DE MARILIA",
    x==34 ~"FACULDADE DE CIENCIAS E TECNOLOGIA DE PRESIDENTE PRUDENTE",
    x==35 ~"FACULDADE DE ODONTOLOGIA DE ARAÇATUBA",
    x==53 ~"CENTRO DE ASSISTENCIA ODONTOLOGICA A EXCEPCIONAIS",
    x==36 ~"FACULDADE DE MEDICINA VETERINÁRIA DE ARAÇATUBA",
    x==37 ~"FACULDADE DE ENGENHARIA DE ILHA SOLTEIRA",
    x==38 ~"FAZENDA DE ENSINO, PESQUISA E PRODUCAO",
    x==39 ~"INSTITUTO DE BIOCIÊNCIAS, LETRAS E CIÊNCIAS EXATAS DE SÃO JOSÉ DO RIO PRETO",
    x==40 ~"BAURU ADMINISTRAÇÃO GERAL",
    x==41 ~"FACULDADE DE ENGENHARIA DE BAURU",
    x==42 ~"FACULDADE DE CIENCIAS DE BAURU",
    x==43 ~"FACULDADE DE ARQUITETURA, ARTES E COMUNICACAO DE BAURU",
    x==44 ~"UNIDADE DO LITORAL PAULISTA",
    x==45 ~"UNIDADE DE DRACENA",
    x==46 ~"UNIDADE DE ITAPEVA",
    x==47 ~"UNIDADE DE TUPÃ",
    x==48 ~"UNIDADE DE REGISTRO",
    x==49 ~"UNIDADE DE ROSANA",
    x==50 ~"UNIDADE DE OURINHOS",
    x==51 ~"UNIDADE DE SOROCABA",
    TRUE ~ as.character(x)
  )
}
```

Agora que sabemos arrumar nossa tabela, vamos fazer o donwload de todas
as tabelas dos anos de 2016 a 2020, salvando-as em um diretório do nosso
computador..

``` r
unidades <- 1:53
anos <- 2016:2020
meses <- 1:12
caminho <- "output/unesp_despesas/"

download_unesp <- function(unidade,ano,mes,caminho){
  u_base  <- "https://ape.unesp.br/orcamento_anual/ddp_tabela.php"
  body_unesp <- list(
    txtunidade= unidade,
    txtano= ano,
    txtmes_inicial= mes,
    txtmes_final= mes,
    operacao_atual= "buscar_campos",
    token_ddp= ""
  )
  arquivo <- paste0(caminho,"/",nome_unidade(unidade),"_", mes,"_",ano,".html")
  r_unesp <- httr::POST(
    u_base, 
    body = body_unesp, 
    httr::write_disk(arquivo,overwrite = TRUE))
}
# 
# for(i in 48:53){
#    for(j in 2016:2020){ 
#      for(k in 1:12){
#        download_unesp(i,j,k,caminho)
#      }
#    }
#  }
```

Após o donwload, os arquivos serão apresentados como abaixo, dentro da
pasta:

<img src="https://raw.githubusercontent.com/arpanosso/projetofinal_Web_scraping/master/inst/ws_12.png" width="800px" style="display: block; margin: auto;" />

Os arquivos podem ser acessados no [link do google
drive](https://drive.google.com/file/d/1h8fuYAAcRZE1w8icQPTwIOI3owIs6_TR/view?usp=sharing)

## Parseando

``` r
arquivos_baixados <- fs::dir_ls(caminho)

parse_job <- function(dir){
  html <- xml2::read_html(dir, encoding = "ISO-8859-1")
  
  html |> 
    xml2::xml_find_all("//table/tbody") |> 
    rvest::html_table(header = FALSE) |>   
    as.data.frame() 
}

# gasto_unesp <- purrr::map_df(arquivos_baixados, 
#                             purrr::possibly(parse_job, data.frame() ), .id = "arquivos_baixados")
```

## Salvando em um diretório

``` r
# readr::write_rds(gasto_unesp,"data-raw/gasto_unesp.rds")
```
