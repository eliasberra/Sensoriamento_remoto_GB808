
# Sensoriamento Remoto
Lab 3 - De espectros até índices, e encontrando a imagem correta
--------------

### Agradecimentos
- Google Earth Engine Team
- Earth Engine Beginning Curriculum
- Prof. Shaun Levick do laboraório [GEARS](https://www.gears-lab.com)  (Geospatial Ecology and Remote Sensing) 

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


O objetivo deste laboratório é entender uma variedade de índices espectrais e desenvolver as habilidades para calcular qualquer índice que você precisar. Antes de chegar a isso, vamos desenvolver o laboratório da semana passada e aprender como encontrar uma imagem para qualquer local geográfico de interesse.

1. Logo acima do painel Codificação está a barra de pesquisa. Procure 'Darwin' nesta barra de pesquisa GEE e clique no resultado para deslocar e ampliar o mapa para Darwin (Figura 1).


![Figura 1. Navegando para a área de interesse no Google Earth Engine](search.png)


2. Use as ferramentas de geometria para marcar um ponto no campus Casuarina da Charles Darwin University (localizado no subúrbio de Brinkin, ao norte de Rapid Creek). Depois de criar o ponto de geometria, você o verá adicionado ao seu painel Codificação como uma variável (var) sob o título Importações.


![Figura 2. Criando um ponto geométrico](geometry.png)

3. Renomeie o ponto resultante como 'campus' clicando no nome da importação (que é chamado de 'geometria' por padrão).

![Figura 3. Renomeando um ponto geométrico](campus.png)


4. Procure por 'Sentinel-2' na barra de pesquisa. Na seção de resultados, você verá 'Sentinel-2: Instrumento multiespectral (MSI), Nível-1C' - clique nele e depois clique no botão 'Importar'.

![Figura 4. Importando dados do Sentinel-2](sent2.png)


5. Após clicar em importar, o Sentinel-2 será adicionado às nossas Importações no painel Codificação como uma variável. Ele será listado abaixo do ponto de geometria do nosso campus com o nome padrão "imageCollection". Vamos renomear isso para “sent2” clicando em imageCollection e digitando “sent2”.

![Figura 5. Importando dados do Sentinel-2](sent2_2.png)

6. É importante entender que agora adicionamos acesso à coleção completa de imagens do Sentinel-2 (ou seja, todas as imagens que foram coletadas até o momento) ao nosso script. Para este exercício, não queremos carregar todas essas imagens - queremos uma única imagem livre de nuvem sobre a Universidade Charles Darwin. Dessa forma, agora podemos filtrar a coleção de imagens com alguns critérios, como tempo de aquisição, localização espacial e cobertura de nuvens.

---------

### Filtrando coleções de imagens

7. Para conseguir isso, precisamos usar um pouco de codificação. Na linguagem de programação JavaScript, duas barras invertidas (//) indicam linhas de comentários e são ignoradas nas etapas de processamento reais. Usamos // para escrever notas para nós mesmos em nosso código, para que nós (e outros que queiram usar nosso código) possamos entender por que fizemos certas coisas.

```JavaScript
// Esta é nossa primeira linha de código. Vamos definir a coleção de imagens com a qual estamos trabalhando escrevendo este comando
    var imagem = ee.Image(sent2

    // Em seguida, incluiremos um filtro para obter apenas imagens no intervalo de datas em que estamos interessados
    .filterDate("01-07-2015", "30-09-2017")

    // Em seguida, incluímos um filtro geográfico para restringir a pesquisa a imagens no local do nosso ponto
    .filterBounds(campus)

    // Em seguida, também classificaremos a coleção por uma propriedade de metadados, no nosso caso, a cobertura de nuvens é muito útil
    .sort("CLOUD_COVERAGE_ASSESSMENT")

    // Agora vamos selecionar a primeira imagem desta coleção - ou seja, a imagem mais livre de nuvens no intervalo de datas
    .primeiro());

    // E vamos imprimir a imagem no console.
    print("Uma cena do Sentinel-2:", imagem);
```

8. Você precisa copiar todo o código acima e colá-lo na caixa “Novo script” do editor de código GEE. Em seguida, clique no botão "Executar" e veja o Google fazer sua mágica... Este pedaço de código pesquisará o arquivo completo do Sentinel-2, encontrará imagens localizadas em Darwin, classificá-las de acordo com a porcentagem de cobertura de nuvens e, em seguida, retorne a imagem livre de nuvem mais recente para nós. As informações relacionadas a esta imagem serão impressas no Console, onde está listada como "Uma cena do Sentinel-2" com alguns detalhes sobre essa cena (COPERNICUS/S2/20160629T014038\_20160629T062926\_T52LFM (16 bandas)). Sabemos pelo nome da cena que foi coletado em 29 de junho de 2016.

![Figura 6. Filtrando a coleção](run.png)
---------
## Adicionando imagens à visualização do mapa
9. Agora, para realmente dar uma olhada nesta imagem, precisamos adicioná-la ao nosso ambiente de mapeamento. Antes de fazer isso, no entanto, vamos definir como queremos exibir a imagem. Vamos começar com uma representação de cores verdadeiras colando as seguintes linhas abaixo das que você já adicionou e clique em "Executar".

```JavaScript
// Defina os parâmetros de visualização em um dicionário JavaScript para renderização de cores verdadeiras. Bandas 4,3 e 2 necessárias para RGB.
    var trueColour = {
        bandas: ["B4", "B3", "B2"],
        min: 0,
        máximo: 3000
        };

  // Adicione a imagem ao mapa, usando os parâmetros de visualização.
  Map.addLayer(image, trueColour, "imagem de cor verdadeira");
```

10. Este código especifica que para uma imagem de cores verdadeiras, as bandas 4,3 e 2 devem ser usadas na composição RGB. Depois que a imagem aparecer no mapa, você poderá ampliar e explorar Darwin. Vemos grandes detalhes na imagem do Sentinel-2, que tem resolução de 10m para as bandas selecionadas. Os símbolos (+) e (-) no canto superior esquerdo do mapa podem ser usados ​​para ampliar e para fora (também possível com a roda de rolagem do mouse/trackpad). Um clique com o botão esquerdo do mouse abre a "mão" para mover a imagem ao redor. Mover o mouse sobre o botão "Camadas" no canto superior direito do painel do mapa mostra as camadas disponíveis e permite ajustar a opacidade das diferentes camadas.

![Figura 7. Adicionando uma imagem true color ao mapa](truecolour.png)

11. Para obter mais informações em locais específicos, podemos usar a ferramenta Inspector localizada no Painel do Console - guia à esquerda. Clique na guia Inspetor e, em seguida, clique na imagem na visualização do mapa. Onde quer que você clique na imagem, os valores da banda nesse ponto serão exibidos na janela do Inspetor. Clique sobre alguns tipos de manchas diferentes (campos esportivos, manguezais, oceano, praia, casas) para ver como o perfil espectral muda.


12. Agora vamos dar uma olhada em uma composição de cores falsas - precisamos trazer a banda do infravermelho próximo (banda 8) para isso. Cole as seguintes linhas abaixo das que você já adicionou e clique em "Executar".

```JavaScript
//Definir parâmetros de visualização de cores falsas.
    var falseCor = {
        bandas: ["B8", "B4", "B3"],
        min: 0,
        máximo: 3000
        };

    // Adicione a imagem ao mapa, usando os parâmetros de visualização.
    Map.addLayer(image, falseColour, "composição de cores falsas");
```

![Figura 9. Adicionando uma composição de cores falsas ao mapa](false.png)

13. Composições de cores falsas colocam a faixa do infravermelho próximo no canal vermelho, e vemos uma forte resposta ao conteúdo de clorofila nas folhas verdes. Vegetação que aparece verde escuro na cor verdadeira, aparecendo vermelho brilhante na cor falsa. Observe as variações em vermelho que podem ser vistas na vegetação que margeia Rapid Creek. Você também verá que "composição de cores falsas" foi adicionada à guia Camadas na visualização do mapa.

---------

### Calculando o NDVI

14. Em seguida, vamos calcular o índice de vegetação de diferença normalizada (NDVI) para esta imagem. O NDVI é um índice calculado a partir das bandas RED e NIR, de acordo com esta equação:

NDVI = (NIR - VERMELHO)/(NIR + VERMELHO)

Cole as seguintes linhas abaixo das que você já adicionou e clique em "Executar". Os valores do NDVI variam de 0 a 1, e quanto maior o valor mais “vigorosa” a vegetação.


```javascript
//Define a variável NDVI da equação
    var NDVI = imagem.expressão(
        "(NIR - VERMELHO) / ​​(NIR + VERMELHO)",
        {
          VERMELHO: image.select("B4"), // VERMELHO
          NIR: image.select("B8"), // NIR
          AZUL: image.select("B2") // AZUL
        });

    Map.addLayer(NDVI, {min: 0, max: 1}, "NDVI");
```

![Figura 10. Recuperando NDVI do Sentinel-2](ndvi.png)

15. Explore as diferentes partes da imagem e veja como os valores de NDVI variam com os diferentes tipos de substrato.


----
### Exercício prático

1. Procure uma imagem Sentinel-2 sem nuvens de maio, julho e setembro de 2018 coletada no Litchfield National Park (Litchfield está localizado ao sul de Darwin, perto da cidade de Batchelor, Território do Norte, Austrália).
2. Calcule o NDVI para cada uma das cenas e carregue-os na visualização do mapa.
3. Inspecione como o NDVI varia espacialmente em cada imagem e explore como os padrões no NDVI variam de acordo com a época do ano.
4. Procure uma imagem do Landsat 8 sem nuvens (USGS Landsat 8 Surface Reflectance Tier 1) de maio, julho e setembro de 2018 coletada no Litchfield National Park.
5. Lembre-se de que a posição da banda dos comprimentos de onda RED e NIR pode diferir entre os diferentes sensores. Para Landsat 8, a propriedade de metadados para cobertura de nuvens é 'CLOUD_COVER'.
6. Compare os valores de NDVI obtidos pelos dois sensores e pense por que eles podem diferir.

-------
### Obrigada

Espero que você tenha achado isso útil. Um vídeo gravado deste tutorial pode ser encontrado na [Lista de reprodução de introdução ao sensoriamento remoto do ambiente] do meu canal do YouTube (https://www.youtube.com/playlist?list=PLf6lu3bePWHDi3-lrSqiyInMGQXM34TSV) e no site do meu laboratório [GEARS] (https://www.gears-lab.com).

#### Atenciosamente, Shaun R Levick
------
