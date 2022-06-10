
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

O objetivo deste laboratório é entender uma variedade de índices espectrais e desenvolver as habilidades para calcular qualquer índice que você precisar. Antes de chegar a isso, vamos desenvolver um pouco mais o laboratório da semana passada e aprender como encontrar uma imagem para qualquer local geográfico de interesse.

1. Logo acima do painel Codificação está a barra de pesquisa. Procure 'Bandeirantes', PR, nesta barra de pesquisa GEE e clique no resultado para deslocar e ampliar o mapa para Bandeirantes.

![image](https://user-images.githubusercontent.com/41900626/173054996-57985aa9-ab7f-46e9-902b-5fb41f8c8ce8.png)


2. Use a ferramenta de geometria 'Add a marker' para marcar um ponto sobre a cidade de Bandeirantes (uma vez selecionado o marcador é só clicar sobre o mapa base). Depois de criar o ponto de geometria, você o verá adicionado ao seu painel Codificação como uma variável (var) sob o título 'Imports'.

![image](https://user-images.githubusercontent.com/41900626/173055410-18cd4d0e-55f7-48f8-9386-73e1a0410466.png)


3. Renomeie o ponto resultante como 'bandeirantes' clicando no nome da 'Imports' (que é chamado de 'geometry' por padrão).

![image](https://user-images.githubusercontent.com/41900626/173056048-64e1112e-bfbf-4fb4-9ba4-8fc3e04c17e0.png)

Nota: Você já pode salvar seu código. Salvei com o nome 'Lab3'.

4. Procure por 'Sentinel-2' na barra de pesquisa. Na seção de resultados, você verá 'Sentinel-2 MSI: MultiSpectral Instrument, Level-2A' - clique nele e depois clique no botão 'Import'.

![image](https://user-images.githubusercontent.com/41900626/173056842-c0562f7b-689b-47d6-afac-62c5dc524d8d.png)


5. Após clicar em 'Import', o Sentinel-2 será adicionado às nossas importações ('Imports') no painel de Codificação como uma variável (var). Ele será listado abaixo do ponto de geometria do cidade de Bandeirantes com o nome padrão "imageCollection" (coleção de imagens). Vamos renomeá-lo para “sent2” clicando em 'imageCollection' e digitando “sent2”.

![image](https://user-images.githubusercontent.com/41900626/173057420-4d099f26-8947-4633-add6-8fb80e124a82.png)


6. É importante entender que agora adicionamos acesso à coleção completa de imagens do Sentinel-2 (ou seja, todas as imagens que foram coletadas até o momento) ao nosso script. Para este exercício, não queremos carregar todas essas imagens - queremos uma única imagem livre de nuvem sobre a cidade de Bandeirantes. Dessa forma, agora podemos filtrar a coleção de imagens com alguns critérios, como intervalo de aquisição, localização espacial e cobertura de nuvens.

---------

### Filtrando coleções de imagens

7. Para filtrar a coleção, precisamos usar um pouco de codificação. Na linguagem de programação JavaScript, duas barras (//) indicam linhas de comentários e são ignoradas quando rodamos o processamento. Usamos // para escrever notas para nós mesmos em nosso código, para que nós (e outros que queiram usar nosso código) possamos entender por que fizemos certas coisas.

```JavaScript
// Esta é nossa primeira linha de código. Vamos definir a coleção de imagens com a qual estamos trabalhando escrevendo este comando
    var imagem = ee.Image(sent2

    // Em seguida, incluiremos um filtro para obter apenas imagens no intervalo de datas em que estamos interessados
    .filterDate("2021-07-01", "2021-09-30")

    // Em seguida, incluímos um filtro geográfico para restringir a pesquisa a imagens no local do nosso ponto
    .filterBounds(bandeirantes)

    // Em seguida, também classificaremos a coleção por uma propriedade de metadados, no nosso caso, a cobertura de nuvens é muito útil
    .sort("CLOUD_COVERAGE_ASSESSMENT")

    // Agora vamos selecionar a primeira imagem desta coleção - ou seja, a imagem mais livre de nuvens no intervalo de datas
    .first());

    // E vamos imprimir a imagem no console.
    print("Uma cena do Sentinel-2:", imagem);
```

8. Você precisa copiar todo o código acima e colá-lo na caixa do editor de código do GEE. Em seguida, clique no botão "Run" (executar) e veja o GEE fazer sua mágica... Este pedaço de código pesquisará o arquivo completo do Sentinel-2, encontrará imagens localizadas em Bandeirantes, PR, classificá-las de acordo com a porcentagem de cobertura de nuvens e, em seguida, retornará a imagem mais livre de cobertura de nuvens. As informações relacionadas a esta imagem serão impressas no Console, onde está listada como "Uma cena do Sentinel-2" com alguns detalhes sobre essa cena (COPERNICUS/S2_SR/20210810T133229_20210810T133225_T22KEV). Sabemos pelo nome da cena que foi coletado em 10 de agosto de 2021.

![image](https://user-images.githubusercontent.com/41900626/173060430-f0ffb7f9-3a3c-406d-b7c5-a90231b7ebca.png)
---------

## Adicionando imagens à visualização do mapa ('Map view')
9. Agora, para realmente dar uma olhada nesta imagem, precisamos adicioná-la ao nosso ambiente de mapeamento. Antes de fazer isso, no entanto, vamos definir como queremos exibir a imagem. Vamos começar com uma representação de cores verdadeiras colando as seguintes linhas abaixo das que você já adicionou e clique em "Run".

```JavaScript
// Defina os parâmetros de visualização em um dicionário JavaScript para renderização de cores verdadeiras. Bandas 4,3 e 2 necessárias para RGB.
    var corVerdadeira = {
        bands: ["B4", "B3", "B2"],
        min: 0,
        max: 3000
        };

  // Adicione a imagem ao mapa, usando os parâmetros de visualização.
  Map.addLayer(imagem, corVerdadeira, "imagem de cor verdadeira");
```

10. Este código especifica que para uma imagem de cores verdadeiras, as bandas 4,3 e 2 devem ser usadas na composição RGB. Depois que a imagem aparecer no mapa, você poderá ampliar e explorar Bandeirantes e a um pouco da região norte do Paraná. Vemos grandes detalhes na imagem do Sentinel-2, que tem resolução de 10 m para as bandas selecionadas. Os símbolos (+) e (-) no canto superior esquerdo do ambinete de mapa podem ser usados para aplicar diferentes níveis de zoom na cena (também possível com a roda de rolagem do mouse/trackpad). Um clique com o botão esquerdo do mouse abre a "mão" para mover a imagem ao redor. Mover o mouse sobre o botão 'Layers' (camadas) no canto superior direito do painel do mapa mostra as camadas disponíveis e permite ajustar a opacidade das diferentes camadas.

![image](https://user-images.githubusercontent.com/41900626/173065724-aeb70824-e7d5-4828-ab47-2a260f0bf240.png)


11. Para obter mais informações em locais específicos, podemos usar a ferramenta 'Inspector' localizada no Painel do Console - guia à esquerda. Clique na guia 'Inspetor' e, em seguida, clique na imagem na visualização do mapa. Onde quer que você clique na imagem, os valores da(s) banda(s) nesse ponto serão exibidos na janela do 'Inspetor'. Clique sobre alguns tipos de manchas diferentes (campos esportivos, corpo d'água, casas) para ver como o perfil espectral muda.


12. Agora vamos dar uma olhada em uma composição de cores falsas - precisamos trazer a banda do infravermelho próximo (banda 8) para isso. Cole as seguintes linhas, logo abaixo das que você já adicionou, e clique em "Run".

```JavaScript
//Definir parâmetros de visualização de cores falsas.
    var falsaCor = {
        bands: ["B8", "B4", "B3"],
        min: 0,
        max: 3000
        };

    // Adicione a imagem ao mapa, usando os parâmetros de visualização.
    Map.addLayer(imagem, falsaCor, "composição de cores falsas");
```

![image](https://user-images.githubusercontent.com/41900626/173068670-00118ab6-fbfc-4b77-9d9c-de3920556e6c.png)


13. Composições de cores falsas colocam a faixa do infravermelho próximo no canal do vermelho (R), e vemos uma forte resposta ao conteúdo de clorofila nas folhas verdes e quantidade de folhas. Vegetação que aparece verde escuro na cor verdadeira, aparece vermelho brilhante na cor falsa. Observe as variações em vermelho que podem ser vistas na vegetação que margeia Bandeirantes. Você também verá que "composição de cores falsas" foi adicionada à guia 'Layers' na visualização do mapa.

---------

### Calculando o NDVI

14. Em seguida, vamos calcular o índice de vegetação de diferença normalizada (NDVI - Normalized Difference Vegetation Index) para esta imagem. O NDVI é um índice calculado a partir das bandas VERMELHO (RED) e NIR, de acordo com esta equação:

NDVI = (NIR - VERMELHO)/(NIR + VERMELHO)

Cole as seguintes linhas abaixo das que você já adicionou e clique em "Run". Os valores do NDVI variam de 0 a 1, e quanto maior o valor mais “vigorosa” a vegetação.


```javascript
   var NDVI = imagem.expression(
        "(NIR - RED) / (NIR + RED)",
        {
          RED: imagem.select("B4"),    //  RED
          NIR: imagem.select("B8"),    // NIR
        });

    Map.addLayer(NDVI, {min: 0, max: 1}, "NDVI");
```

![image](https://user-images.githubusercontent.com/41900626/173073210-5c842e72-1109-436e-84c8-21895c4dbe48.png)

15. Explore as diferentes partes da imagem e veja como os valores de NDVI variam com os diferentes tipos de cobertura da terra.


----
### Exercício prático

1. Procure uma imagem Sentinel-2 sem nuvens de maio, julho e setembro de 2018 coletada em uma área de sua escolha.
2. Calcule o NDVI para cada uma das cenas e carregue-os na visualização do mapa.
3. Inspecione como o NDVI varia espacialmente em cada imagem e explore como os padrões no NDVI variam de acordo com a época do ano.
4. Procure uma imagem do Landsat 8 sem nuvens (USGS Landsat 8 Surface Reflectance Tier 1) de maio, julho e setembro de 2018 na sua área de estudo.
5. Lembre-se de que a posição da banda dos comprimentos de onda RED e NIR pode diferir entre os diferentes sensores. Para Landsat 8, a propriedade de metadados para cobertura de nuvens é 'CLOUD_COVER'.
6. Compare os valores de NDVI obtidos pelos dois sensores e pense por que eles podem diferir.

-------
### Obrigado

Espero que você tenha achado isso útil. 


------
