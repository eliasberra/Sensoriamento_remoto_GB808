# Sensoriamento Remoto
Lab 05 - Classificação de imagens - parte 2
--------------

### Agradecimentos
- Google Earth Engine Team
- Earth Engine Beginning Curriculum
- Prof. Shaun Levick do laboraório [GEARS](https://www.gears-lab.com)  (Geospatial Ecology and Remote Sensing) 

------
### Objetivo
O objetivo deste laboratório é aprofundar sua compreensão do processo de classificação de imagens e melhorar a classificação do tutorial passado.

----------

## Carregue sua classificação anterior da semana passada

Abra seu script do laboratório da semana passada. Se você não o salvou, repita as etapas do [Lab 4](https://github.com/geospatialeco/GEARS/blob/master/Intro_RS_Lab4.md) e salve-o desta vez.

Forneci o código completo abaixo, mas lembre-se de que você precisa coletar manualmente os dados de treinamento e atribuir propriedades de cobertura do solo.

```JavaScript
//Filtrar coleção de imagens para janela de tempo, localização espacial e cobertura de nuvens
var imagem = ee.Image(ee.ImageCollection('LANDSAT/LC08/C01/T1_SR')
    .filterBounds(roi)
    .filterDate('2016-05-01', '2016-06-30')
    .sort('CLOUD_COVER')
    .primeiro());

//Adiciona o composto true-clour ao mapa
Map.addLayer(image, {bandas: ['B4', 'B3', 'B2'],min:0, max: 3000}, 'True color image');

//Mesclar recursos em um FeatureCollection
var classNames = urban.merge(água).merge(floresta).merge(agricultura);

//Seleciona as bandas a serem usadas
var bandas = ['B2', 'B3', 'B4', 'B5', 'B6', 'B7'];

//Amostra os valores de refletância para cada ponto de treinamento
var treinamento = image.select(bands).sampleRegions({
  coleção: classNames,
  propriedades: ['cobertura do solo'],
  escala: 30
});

//Treina o classificador - neste caso usando uma árvore de regressão CART
var classificador = ee.Classifier.cart().train({
  características: treinamento,
  classProperty: 'landcover',
  propriedades de entrada: bandas
});

//Executa a classificação
var classificado = image.select(bandas).classify(classificador);

//Exibe o mapa de classificação
Map.centerObject(classNames, 11);
Map.addLayer(classificado,
{min: 0, max: 3, paleta: ['red', 'blue', 'green','yellow']},
'classificação');
```

![Figura 1. Mapa classificado](screenshots/l4_classified.png)

-----
## Melhorando a Classificação

Terminamos a semana passada com uma discussão sobre se estávamos ou não satisfeitos com essa classificação. Mesmo sem quaisquer dados quantitativos, estava claramente ausente em algumas regiões. Como podemos melhorar? Existem algumas opções que podemos explorar:

1. Altere o tamanho da amostra de treinamento. Amostramos apenas 25 pixels por classe. Foram muitos cliques, mas poderíamos usar polígonos em vez de pontos para amostrar mais pixels para treinamento.
2. Altere a estratégia de amostragem. Coletamos um número par de pontos por classe, mas algumas classes de cobertura do solo cobrem muito mais área do que outras. Em vez disso, poderíamos experimentar uma abordagem de amostragem estratificada.
3. Altere o classificador. Usamos um classificador CART, podemos tentar uma abordagem diferente, como uma máquina de vetor de suporte (SVM) ou abordagem randomForest (randomForest).
4. Altere as bandas. Poderíamos adicionar informações auxiliares, como dados de elevação, ou um índice derivado, como NDVI, para fornecer informações para discriminação de classe.
5. Altere a imagem. Usamos uma cena de inverno do Landsat-8. Poderíamos tentar uma cena de verão ou mudar para uma imagem do Sentinel-2.


-------
### Obrigada
