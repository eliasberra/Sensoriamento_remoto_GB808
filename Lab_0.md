# Sensoriamento Remoto
Lab 0 - Comportamento espectral dos alvos
--------------

### Agradecimentos
- Google Earth Engine Team

------

### Prerequisitos
-------------
A conclusão deste exercício de laboratório requer o uso do navegador Google Chrome e uma conta do Google Earth Engine. Se você ainda não se inscreveu - faça-o agora em uma nova guia:

[Registro da conta do Earth Engine](https://signup.earthengine.google.com/)

Uma vez registrado, você pode acessar o ambiente do Earth Engine aqui: https://code.earthengine.google.com

O Google Earth Engine usa a linguagem de programação JavaScript. Abordaremos o básico desta linguagem durante este curso. Se você quiser mais detalhes, pode ler a introdução fornecida aqui:

[JavaScript background](https://developers.google.com/earth-engine/tutorials/tutorials)

------------------------------------------------------------------------

### Objetivo
---------
O objetivo deste laboratório é fornecer uma introdução ao ambiente de processamento do GEE e realizar a leitura dos valores dos pixels dos diferentes alvos da superfície terrestre.



## 1. O editor de código do GEE

![Fig. 1. O ambiente do GEE](https://github.com/geospatialeco/GEARS/blob/master/gee_editor.png)

1. Painel do Editor (_Code editor_)
	- O Painel do Editor é onde você escreve e edita seu código Javascript.
2. Painel Direito
	- Guia do console para saída de impressão.
	- Guia Inspetor para consultar os resultados do mapa.
	- Guia Tarefas para gerenciar tarefas de longa duração.
3. Painel Esquerdo
	- Aba Scripts: para gerenciar seus scripts de programação.
	- Aba Docs: para acessar a documentação dos objetos e métodos do Earth Engine, bem como alguns específicos do aplicativo Code Editor.
	- Aba Assets: Guia de dados geoespaciais que você criou e salvou, ou carregou.
4. Mapa Interativo
	- Para visualizar a saída das camadas do mapa
5. Barra de pesquisa
	- Para encontrar conjuntos de dados e locais de interesse.
6. Menu Ajuda
	- Documentação de referência do guia do usuário
	- Grupo de ajuda do fórum do Google para discutir o Earth Engine
	- Atalhos de teclado para o Editor de código
	- Visão geral do tour de recursos do Editor de código
	- Feedback para enviar feedback sobre o Code Editor
	- Sugerir um novo conjunto de dados.
---------

## 2. Procurando uma imagem

1. Logo acima do painel Codificação está a barra de pesquisa. 
Procure 'Paranaguá', PR, nesta barra de pesquisa GEE e clique no resultado para deslocar e ampliar o mapa para Paranaguá. Amplie/reduza o zoom usando a roda (scroll) do _mouse_.
![image](https://user-images.githubusercontent.com/41900626/178794965-07dde932-44de-4acf-a6d0-58649b7bad95.png)


2. Use a ferramenta de geometria 'Add a marker' para marcar um ponto sobre a cidade de Paranaguá (uma vez selecionado o marcador é só clicar sobre o mapa base). Depois de criar o ponto de geometria, você o verá adicionado ao seu painel Codificação como uma variável (var) sob o título 'Imports'.
![image](https://user-images.githubusercontent.com/41900626/178795354-74c3042e-707d-4625-b806-5cb0f4b48141.png)

3. Renomeie o ponto resultante como 'paranagua' clicando no nome da 'Imports' (que é chamado de 'geometry' por padrão).
 ![image](https://user-images.githubusercontent.com/41900626/178795572-b59562aa-19cf-448b-9099-02586706b05b.png)
 
 Nota: Você já pode salvar seu código em ![image](https://user-images.githubusercontent.com/41900626/178795780-e672f2b2-2472-4e9c-b32c-8caef6e928da.png)
. Salvei com o nome 'Lab0'.

4. Procure por 'Landsat 5 surface' na barra de pesquisa. Na seção de resultados, você verá 'USGS Landsat 5 Level 2, Collection 2, Tier 1'.
![image](https://user-images.githubusercontent.com/41900626/178796322-90d5b518-4a66-4e4c-9417-4023fa4f59bb.png)
Clique nele e observe as importantes descrições do tipo de produto, como resolução espacial. Uma melhor visualização é alcançada clicando no canto superior direito, conforme indicado na figura.
![image](https://user-images.githubusercontent.com/41900626/178797162-dea6d5be-e7b5-4a6f-8838-39008e319578.png)

Após essa analise, volte alguns passos e clique no botão 'Import'.
![image](https://user-images.githubusercontent.com/41900626/178797596-25c93775-9dd1-48fa-b0f1-7999eda3437e.png)

5. Após clicar em 'Import', o Sentinel-2 será adicionado às nossas importações ('Imports') no painel de Codificação como uma variável (var). Ele será listado abaixo do ponto de geometria do cidade de 'paranagua' com o nome padrão "imageCollection" (coleção de imagens). Vamos renomeá-lo para “land5” clicando em 'imageCollection' e digitando “land5”.
![image](https://user-images.githubusercontent.com/41900626/178797936-98cc6d8e-2246-49b4-8825-dbea9b4c16a6.png)

6. É importante entender que agora adicionamos acesso à coleção completa de imagens do Landsat-5 (ou seja, todas as imagens que foram coletadas até o momento) ao nosso script. Para este exercício, não queremos carregar todas essas imagens - queremos uma única imagem livre de nuvem sobre a cidade de Paranaguá. Dessa forma, agora podemos filtrar a coleção de imagens com alguns critérios, como intervalo de aquisição, localização espacial e cobertura de nuvens.




### Filtrando coleções de imagens

7. Para filtrar a coleção, precisamos usar um pouco de codificação. Na linguagem de programação JavaScript, duas barras (//) indicam linhas de comentários e são ignoradas quando rodamos o processamento. Usamos // para escrever notas para nós mesmos em nosso código, para que nós (e outros que queiram usar nosso código) possamos entender por que fizemos certas coisas.
Você pode digitar manualmente o código abaixo, o que é legal para aprender a linguagem, ou simplesmente copiar todo o código acima e colá-lo na caixa do editor de código do GEE.

```JavaScript
// Esta é nossa primeira linha de código. Vamos definir a coleção de imagens com a qual estamos trabalhando escrevendo este comando
    var imagem = ee.Image(land5

    // Em seguida, incluiremos um filtro para obter apenas imagens no intervalo de datas em que estamos interessados
    .filterDate("2010-07-01", "2010-12-31")

    // Em seguida, incluímos um filtro geográfico para restringir a pesquisa a imagens no local do nosso ponto
    .filterBounds(paranagua)

    // Em seguida, também classificaremos a coleção por uma propriedade de metadados, no nosso caso, a cobertura de nuvens é muito útil
    .sort("CLOUD_COVER")

    // Agora vamos selecionar a primeira imagem desta coleção - ou seja, a imagem mais livre de nuvens no intervalo de datas
    .first());

    // E vamos imprimir a imagem no console.
    print("Uma cena do Landsat 5:", imagem);
```

8. Em seguida, clique no botão "Run" (executar) e veja o GEE fazer sua mágica... 
Este pedaço de código pesquisará o arquivo completo do Landsat-5, encontrará imagens localizadas em Paranaguá, PR (na interseção com o ponto que você escolheu, para ser mais preciso), irá classificá-las de acordo com a porcentagem de cobertura de nuvens e, em seguida, retornará a imagem mais livre de cobertura de nuvens. As informações relacionadas a esta imagem serão impressas no Console, onde está listada como "Uma cena do Landsat 5:" com alguns detalhes sobre essa cena ('Image LANDSAT/LT05/C02/T1_L2/LT05_220078_20101119 (19 bands)'). Sabemos pelo nome da cena que foi coletado em 19 de novembro de 2010.
![image](https://user-images.githubusercontent.com/41900626/178800777-2ff5c8a4-b7ba-4206-9270-31834fa590f4.png)



## Adicionando imagens à visualização do mapa ('Map view')
1. Agora, para realmente dar uma olhada nesta imagem, precisamos adicioná-la ao nosso ambiente de mapeamento (similarmente a um SIG como QGIS). Antes de fazer isso, no entanto, vamos definir como queremos exibir a imagem. Vamos começar com uma representação de cores verdadeiras colando as seguintes linhas abaixo das que você já adicionou e clique em "Run".

```JavaScript
  // Defina os parâmetros de visualização em um dicionário JavaScript para renderização de cores verdadeiras. Bandas 3,2 e 1 são  necessárias para tal.
    var corVerdadeira = {
        bands: ["SR_B3", "SR_B2", "SR_B1"],
        min: 7000,//Os valores em min: e max: definem os limiares de contraste a ser aplicado
        max: 12000
        };

  // Adicione a imagem ao mapa, usando os parâmetros de visualização.
  Map.addLayer(imagem, corVerdadeira, "imagem de cor verdadeira");
```

10. Este código especifica que para uma imagem de cores verdadeiras, as bandas 3,2 e 1 devem ser usadas na composição RGB. Depois que a imagem aparecer no mapa, você poderá ampliar e explorar Paranaguá e arredores. Os símbolos (+) e (-) no canto superior esquerdo do ambinete de mapa podem ser usados para aplicar diferentes níveis de zoom na cena (também possível com a roda de rolagem do mouse/trackpad). 
Um clique com o botão esquerdo do mouse abre a "mão" para mover a imagem ao redor. Mover o mouse sobre o botão 'Layers' (camadas) no canto superior direito do painel do mapa mostra as camadas disponíveis e permite ajustar a opacidade das diferentes camadas.
Experimente diferentes limiares de contraste alterando os valores de 'min:' (valor mínimo) e 'max:' (valor máximo).
![image](https://user-images.githubusercontent.com/41900626/178803288-9323cb8a-0a93-44e2-a42b-ff91297e7d2b.png)





