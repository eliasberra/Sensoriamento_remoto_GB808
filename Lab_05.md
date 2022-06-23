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

Abra seu script do laboratório da semana passada. Se você não o salvou, repita as etapas do [Lab_04](https://github.com/eliasberra/Sensoriamento_remoto_GB808/blob/99f77c491b61da623bfff5f70d4ec06269bc725a/Lab_04.md) e salve-o desta vez.

Forneci o código completo abaixo, mas lembre-se de que você precisa coletar manualmente os dados de treinamento e atribuir propriedades de cobertura da terra.

```JavaScript
//Selecionar imagem e formar uma composição colorida
var imagem = ee.Image(ee.ImageCollection('LANDSAT/LC08/C02/T1_L2') 
    .filterBounds(roi)
    .filterDate('2022-01-01', '2022-06-30')
    .sort('CLOUD_COVER')
    .first());   
Map.addLayer(imagem, {bands:['SR_B4', 'SR_B3', 'SR_B2'],min:6000, max: 20000}, 'Imagem cor verdadeira', false);

//Recortar cena
var img_recorte =  imagem.clip(recorte);
Map.addLayer(img_recorte, {bands:['SR_B4', 'SR_B3', 'SR_B2'],min:6000, max: 20000}, 'Recorte');

//mesclar as classes em uma única coleção
var nomeClasses = urbano.merge(floresta).merge(area_agricola_cultivada).merge(area_agricola_solo).merge(agua);

//Imprimir a coleção de feições
print(nomeClasses)

//Extrair uma lista de valores em cada ponto pem cada banda
var bandas = ['SR_B2', 'SR_B3', 'SR_B4', 'SR_B5', 'SR_B6', 'SR_B7'];
var treinamento = img_recorte.select(bandas).sampleRegions({
  collection: nomeClasses,
  properties: ['cobertura_terra'],
  scale: 30
});
print('treinamento', treinamento);

//Treinar o classificador
var classificador = ee.Classifier.smileCart().train({
      features: treinamento,
      classProperty: 'cobertura_terra',
      inputProperties: bandas
});

//Executar a classificação
var classificada = img_recorte.select(bandas).classify(classificador);

//Exibir a classificação
Map.centerObject(nomeClasses, 11);
Map.addLayer(classificada,
{min: 0, max: 4, palette: ['grey', 'green', 'blue','olive', 'yellow']},
'classificação');

```
![image](https://user-images.githubusercontent.com/41900626/175385609-82018207-6d8a-471c-bf3e-2a8c8fb22df7.png)



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
