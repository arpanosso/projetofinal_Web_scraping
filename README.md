
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

# Aquisição de dados

**1)** Acesse o endereço <https://www2.unesp.br/>.

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

**4)** Será apresentado um campo para a escolha de uma das unidades da
universidade, no exemplo, selecione **10061 - UNESP CONSOLIDADA**.

<img src="https://raw.githubusercontent.com/arpanosso/projetofinal_Web_scraping/master/inst/ws_04.png" width="800px" style="display: block; margin: auto;" />

<br />

**5)** Novos campos serão apresentados, então, faça o preenchimento dos
mesmos. Será gerado um relatório com *infobox*’s e gráficos, contudo,
sem a tabela de dados de interesse.

<img src="https://raw.githubusercontent.com/arpanosso/projetofinal_Web_scraping/master/inst/ws_05.png" width="800px" style="display: block; margin: auto;" />

<br />

**6)** Rolando essa página até o final, pode-se observar a opção para
acesso da **Tabela Completa**, como apresentado abaixo.

<img src="https://raw.githubusercontent.com/arpanosso/projetofinal_Web_scraping/master/inst/ws_06.png" width="800px" style="display: block; margin: auto;" />

<br />

**7)** Ao selecionarmos, será apresentado a tabela com os dados para
podermos raspar, portanto a **url base** será:
“<https://ape.unesp.br/orcamento_anual/ddp_tabela.php>”.

<img src="https://raw.githubusercontent.com/arpanosso/projetofinal_Web_scraping/master/inst/ws_07.png" width="800px" style="display: block; margin: auto;" />

<br />

**8)** Para entender as requisições realizadas pela página, utilizamos a
opção **Inspecionar** do navegador, após isso, clicando **Network**, e
atualizamos a página com o **F5**. Observando o tipo `document`,
clicamos em **ddp\_tabela.php**.

<img src="https://raw.githubusercontent.com/arpanosso/projetofinal_Web_scraping/master/inst/ws_08.png" width="800px" style="display: block; margin: auto;" />

<br />

**9)** Observe que a requisição realizada foi do tipo **POST**.

<img src="https://raw.githubusercontent.com/arpanosso/projetofinal_Web_scraping/master/inst/ws_10.png" width="800px" style="display: block; margin: auto;" />

**10)** Rolando a aba do *Header*, pode-se verificar “os argumentos”
utilizados no **body** da requisição.

<img src="https://raw.githubusercontent.com/arpanosso/projetofinal_Web_scraping/master/inst/ws_11.png" width="800px" style="display: block; margin: auto;" />

## Download dos arquivos

Vamos testar a nossa requisição, e observamos que a tabela foi acessada.

``` r
u_unesp  <- "https://ape.unesp.br/orcamento_anual/ddp_tabela.php"

body_unesp <- list(
  txtunidade= 1, # unidade adminstrativa
  txtano= 2016, # ano
  txtmes_inicial= 1, # ano inicial
  txtmes_final= 1, # ano final
  operacao_atual= "buscar_campos",
  token_ddp= ""
)

# Vamos salvar o arquivo para testar posteriormente
r_unesp <- httr::POST(
  u_unesp, 
  body = body_unesp,
  httr::write_disk("output/unesp.html",
                   overwrite = TRUE))

# teste do arquivo
minha_pagina_teste <- readr::read_file("output/unesp.html")
```

Abrindo a págino baixada no navegador verificamos que a tabela está
presente.

<img src="https://raw.githubusercontent.com/arpanosso/projetofinal_Web_scraping/master/inst/ws_11_1.png" width="800px" style="display: block; margin: auto;" />

Seguindo as instruções, vamos checar o encoding do site

``` r
readr::guess_encoding(u_unesp)
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
httr::content(r_unesp, encoding = "ISO-8859-1")  |> 
  xml2::xml_find_first("//table/tbody") |> 
  rvest::html_table(header = FALSE) |>   
  tibble::as_tibble() |> 
  head(3)
#> # A tibble: 3 x 11
#>   X1          X2      X3    X4    X5       X6    X7    X8    X9    X10   X11    
#>   <chr>       <chr>   <chr> <chr> <chr>    <chr> <chr> <chr> <chr> <chr> <chr>  
#> 1 0101 - Fol~ 469.30~ 0,00  0,00  154.370~ 0,00  6.20~ 0,00  0,00  0,00  154.84~
#> 2 0102 - Enc~ 10.101~ 0,00  0,00  25.146.~ 0,00  1.39~ 0,00  0,00  0,00  25.157~
#> 3 0103 - Aux~ 98.795~ 0,00  0,00  0,00     0,00  0,00  0,00  0,00  0,00  98.795~
```

Inspecionando a página, verificamos que a unesp tem 53 unidades, assim,
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
computador.

``` r
unidades <- 1:53
anos <- 2016:2020
meses <- 1:12
parametros <- expand.grid(unidades,anos,meses)
caminho <- "output/unesp_despesas/"

download_unesp <- function(indice){
  unidade =as.numeric(parametros[indice,1])
  ano = as.numeric(parametros[indice,2])
  mes = as.numeric(parametros[indice,3])
  
  body_unesp<-list(
    txtunidade=unidade ,
    txtano= ano,
    txtmes_inicial= mes,
    txtmes_final= mes,
    operacao_atual= "buscar_campos",
    token_ddp= ""
  )
  
  arquivo <- paste0("output/unesp_despesas/",unidade,"_", mes,"_",ano,".html")
  r_unesp <- httr::POST(
    "https://ape.unesp.br/orcamento_anual/ddp_tabela.php", 
    body = body_unesp, 
    httr::write_disk(arquivo,overwrite = TRUE))
  return(arquivo)
}

# vamos testar com 3 arquivos
i <- 1:3
tictoc::tic()
purrr::map(i, purrr::possibly(download_unesp, ""))
#> [[1]]
#> [1] "output/unesp_despesas/1_1_2016.html"
#> 
#> [[2]]
#> [1] "output/unesp_despesas/2_1_2016.html"
#> 
#> [[3]]
#> [1] "output/unesp_despesas/3_1_2016.html"
tictoc::toc()
#> 10.92 sec elapsed
```

Vamos utilizar multisession

``` r
future::plan("multisession")
tictoc::tic()
furrr::future_map(i, purrr::possibly(download_unesp, ""))
#> [[1]]
#> [1] "output/unesp_despesas/1_1_2016.html"
#> 
#> [[2]]
#> [1] "output/unesp_despesas/2_1_2016.html"
#> 
#> [[3]]
#> [1] "output/unesp_despesas/3_1_2016.html"
tictoc::toc()
#> 7.86 sec elapsed
```

Agora vamos adicionar a barra de progesso e realizar o download de todos
os arquivos.

``` r
# Definindo todas as iterações
i <- 1:nrow(parametros)

# Craindo a função maybe_
maybe_download_unesp_prog <- function(indice, prog){
  prog()
  f <- purrr::possibly(download_unesp, "")
  f(indice)
}

# Rodando com a barra de progresso
progressr::with_progress({
  prog <- progressr::progressor(along = i)
  furrr::future_map(i, maybe_download_unesp_prog, prog=prog)
})
```

<img src="https://raw.githubusercontent.com/arpanosso/projetofinal_Web_scraping/master/inst/ws_13.png" width="800px" style="display: block; margin: auto;" />

Após o download, os arquivos serão apresentados como abaixo:

<img src="https://raw.githubusercontent.com/arpanosso/projetofinal_Web_scraping/master/inst/ws_12.png" width="800px" style="display: block; margin: auto;" />

Os arquivos podem ser acessados no [link do google
drive](https://drive.google.com/drive/folders/1QVm49MaT8P2cVt5iBs-x2PsYie-BM58M?usp=sharing)

## Parseando

``` r
arquivos_baixados <- fs::dir_ls(caminho)

parse_job <- function(dir){
  html <- xml2::read_html(dir, encoding = "ISO-8859-1")
  html |> 
    xml2::xml_find_first("//table/tbody") |> 
    rvest::html_table(header = FALSE) |>   
    tibble::as_tibble() 
}

gasto_unesp <- furrr::future_map_dfr(arquivos_baixados, 
                             purrr::possibly(parse_job, data.frame())
                             , .id = "arquivos_baixados")
```

Salvando em um diretório

``` r
readr::write_rds(gasto_unesp,"data-raw/gasto_unesp.rds")
```

Deixando a tabela mais bonitinha

``` r
# Função para mudar o formato dos números
muda_numero <- function(x){
    as.numeric(
       stringr::str_replace(stringr::str_remove_all(x,"\\."),",",".")
       )
}

# carregando a base de dados e arrumando a tabela
gasto_unesp <- readr::read_rds("data-raw/gasto_unesp.rds") |>    
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
    arquivos_baixados = fs::path_file(arquivos_baixados),
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
    )
dplyr::glimpse(gasto_unesp)
#> Rows: 50,635
#> Columns: 13
#> $ categoria                     <chr> "DESPESAS_CORRENTES", "DESPESAS_CORRENTE~
#> $ arquivos_baixados             <chr> "10_10_2016.html", "10_10_2016.html", "1~
#> $ despesa                       <chr> "  Diárias", "  Utilidade Pública", "  L~
#> $ tesouro_orcamento_vigente     <dbl> 4026.00, 15975.95, 5944.41, 15672.60, 20~
#> $ convenios_orcamento_vigente   <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0~
#> $ rec_propria_orcamento_vigente <dbl> 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, ~
#> $ tesouro_restos_pagar          <dbl> 0.00, 0.00, 0.00, 0.00, 0.00, 0.00, 5241~
#> $ convenios_restos_pagar        <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0~
#> $ rec_propria_restos_pagar      <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0~
#> $ tesouro_diversos_credores     <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0~
#> $ convenios_diversos_credores   <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0~
#> $ rec_propria_diversos_credores <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0~
#> $ total                         <dbl> 4026.00, 15975.95, 5944.41, 15672.60, 20~
```

Deixando a tabela com os nomes das unidade e criando uma coluna com a
data

``` r
gasto_unesp |> 
  dplyr::mutate(
    arquivos_baixados = stringr::str_remove(arquivos_baixados, ".html")
  ) |> 
  tidyr::separate(arquivos_baixados, into = c("unidade","mes","ano"),
                  sep="_", convert = TRUE) |> 
  dplyr::mutate(
    unidade = nome_unidade(unidade),
    data = lubridate::make_date(month = mes, year = ano)) |> 
  dplyr::glimpse()
#> Rows: 50,635
#> Columns: 16
#> $ categoria                     <chr> "DESPESAS_CORRENTES", "DESPESAS_CORRENTE~
#> $ unidade                       <chr> "CENTRO DE RADIO E TELEVISAO CULTURAL E ~
#> $ mes                           <int> 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, ~
#> $ ano                           <int> 2016, 2016, 2016, 2016, 2016, 2016, 2016~
#> $ despesa                       <chr> "  Diárias", "  Utilidade Pública", "  L~
#> $ tesouro_orcamento_vigente     <dbl> 4026.00, 15975.95, 5944.41, 15672.60, 20~
#> $ convenios_orcamento_vigente   <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0~
#> $ rec_propria_orcamento_vigente <dbl> 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, ~
#> $ tesouro_restos_pagar          <dbl> 0.00, 0.00, 0.00, 0.00, 0.00, 0.00, 5241~
#> $ convenios_restos_pagar        <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0~
#> $ rec_propria_restos_pagar      <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0~
#> $ tesouro_diversos_credores     <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0~
#> $ convenios_diversos_credores   <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0~
#> $ rec_propria_diversos_credores <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0~
#> $ total                         <dbl> 4026.00, 15975.95, 5944.41, 15672.60, 20~
#> $ data                          <date> 2016-10-01, 2016-10-01, 2016-10-01, 201~
```
