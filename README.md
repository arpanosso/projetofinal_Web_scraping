
<!-- README.md is generated from README.Rmd. Please edit that file -->

# Projeto Final - R4DS2

### Aluno: Alan R. Panosso

### Data: 26/07/2021

<!-- badges: start -->
<!-- badges: end -->

Projeto final apresentado os instrutores Julio Trecenti e Caio Lente da
Curso-R como parte das exigências para a finalização do curso de Web
scraping.

Objetivo do projeto é criar um produto de dados a partir do portal de
transparência da UNESP. Além a aquisição e parseamento da informação,
serão apresentados os tratamentos iniciais e a construção de alguns
gráficos e apresentação de resultados.

# Aquisição de dados

**OBS**: Para melhor acompanhamento do material, utilize o navegador
Google Chrome:

**1)** Acesse o endereço <https://www2.unesp.br/>

<img src="https://raw.githubusercontent.com/arpanosso/projetofinal_Web_scraping/master/inst/ws_01.png" width="800px" style="display: block; margin: auto;" />

<br />

**2)** Clique em *Transparência* e em seguida selecione *Transparência
Unesp*.

<img src="https://raw.githubusercontent.com/arpanosso/projetofinal_Web_scraping/master/inst/ws_02.png" width="800px" style="display: block; margin: auto;" />

<br />

**3)** Na próxima página, no setor *GESTÃO*, acesse **Despesas pagas**,
para acessar os dados da execussão orçamentária da universidade nos
últimos anos.

<img src="https://raw.githubusercontent.com/arpanosso/projetofinal_Web_scraping/master/inst/ws_03.png" width="800px" style="display: block; margin: auto;" />

<br />

**4)** Será apresentado um campo para selecionar uma das unidade da
universidade, no exemplo, selecione **10061 - UNESP CONSOLIDADA**.

<img src="https://raw.githubusercontent.com/arpanosso/projetofinal_Web_scraping/master/inst/ws_04.png" width="800px" style="display: block; margin: auto;" />

<br />

**5)** Novos campos serão apresentados, então, fizemos o preenchimento
dos mesmos, como apresentado abaixo, para verificar os filtros
disponíveis. O app apresenta um relatório com *infobox*’s e gráficos,
contudo, sem a tabela de dados de interesse.

<img src="https://raw.githubusercontent.com/arpanosso/projetofinal_Web_scraping/master/inst/ws_05.png" width="800px" style="display: block; margin: auto;" />

<br />

**6)** Rolando a página até o final, observamos a opção para acesso da
**Tabela Completa**, e selecionamos essa opção.

<img src="https://raw.githubusercontent.com/arpanosso/projetofinal_Web_scraping/master/inst/ws_06.png" width="800px" style="display: block; margin: auto;" />

<br />

**7)** Será apresentado a tabela com os dados, ou seja, encontramos a
nossa **url base**:
“<https://ape.unesp.br/orcamento_anual/ddp_tabela.php>”, pois essa é a
tabela que necessitamos.

<img src="https://raw.githubusercontent.com/arpanosso/projetofinal_Web_scraping/master/inst/ws_07.png" width="800px" style="display: block; margin: auto;" />

<br />

**8)** Para entender as requisições realizadas, utilizamos a opção de
Inspecionar “Ctrl + Shift + I”, em **Network**, atualizamos a página com
o **F5** e ordenamos os processos por tipo (Type). Observando o tipo
“document”, clicamos em **ddp\_tabela.php**.

<img src="https://raw.githubusercontent.com/arpanosso/projetofinal_Web_scraping/master/inst/ws_08.png" width="800px" style="display: block; margin: auto;" />

<br />

**9)** A requisição realizada foi do tipo **POST**.

<img src="https://raw.githubusercontent.com/arpanosso/projetofinal_Web_scraping/master/inst/ws_10.png" width="800px" style="display: block; margin: auto;" />

**10)** Rolando a aba do *Header*, verificamos o parâmetros do **body**
da requisição, destacado abaixo.

<img src="https://raw.githubusercontent.com/arpanosso/projetofinal_Web_scraping/master/inst/ws_11.png" width="800px" style="display: block; margin: auto;" />
