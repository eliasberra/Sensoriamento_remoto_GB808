# Sensoriamento Remoto
Classificação de imagens de satélite
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

O objetivo deste laboratório é entender o processo de classificação de imagens e explorar formas de transformar imagens de sensoriamento remoto em mapas de uso e cobertura da terra.

----------

## Carregando a imagem
Para carregar uma imagem, precisamos definir uma área de interesse (AOI). Nesse exercício, vamos utilizar como AOI o polígono representando o limite municipal de Carlópolis, PR, conforme disponibilizado pela FAO.

Nota: Quando iniciamos uma linha com '//', a mesma vira um comentário e não será executada. Isso serve para que possamos incluir notas sobre a finalidade das linhas de código.


```JavaScript
//------------Importar limite territorial de Carlópolis, PR, com dados da FAO 
var zeroLevel = ee.FeatureCollection("FAO/GAUL/2015/level0");// País                      
var firstLevel = ee.FeatureCollection("FAO/GAUL/2015/level1");//Estado
var secondLevel = ee.FeatureCollection("FAO/GAUL/2015/level2") ;//Município

var Carlopolis = secondLevel.filter(ee.Filter.eq('ADM2_NAME', 'Carlopolis'))
Map.addLayer(Carlopolis, {}, 'Limite Municipal')//Adicionar o limite como uma nova camada
```

Ao clicar em '_Run_', o limite de Carlópolis deve estar aparecendo na área de visualizador de mapas:
![image](https://github.com/eliasberra/Sensoriamento_remoto_GB808/assets/41900626/89a4e6c4-e719-4cfb-bb6f-5606930c41d3)



Agora vamos obter uma imagem sem nuvem de Carlópolis. Faça isso importando imagens USGS Landsat 9 Surface Reflectance Tier 1, filtrando espacialmente para uma região de interesse (filterBounds), filtrando temporalmente para o intervalo de datas necessário (filterDate) e, por último, classificando por cobertura de nuvens ('CLOUD_COVER') e extraindo a cena menos nublada (first()).

```JavaScript
//------------Selecionar imagem do satélite Landsat 9 e recortar para Carlópolis
var imagem_selecionada = 
      ee.ImageCollection('LANDSAT/LC09/C02/T1_L2') 
    .filterBounds(Carlopolis)
    .filterDate('2022-05-01', '2022-06-30')//data inicial e final
    .sort('CLOUD_COVER')
    .first()    
    .clip(Carlopolis)//recorta para Carlópolis

//Inspeciona imagem selecionada, mostrando no Console
print('imagem_selecionada', imagem_selecionada)    

```

Para melhor visualizar a imagem selecionada, podemos utilizar diferentes combinações de bandas espectrais e aplicar diferentes limiares de contraste.

```JavaScript
//----Montar duas composições coloridas    
//composição cor-verdadeira
var corVerdadeira = {bands:['SR_B4',//Banda do vermelho
                    'SR_B3',//Banda do verde
                    'SR_B2'],//Banda do azul
                    min:7000, max: 20000};//Contraste da composição  

//composição falsa-cor
var falsaCor = {bands:['SR_B6',//Infravermelho médio
                    'SR_B5',//Infravermelho próximo 
                    'SR_B4'],//Vermelho
                    min:7000, max: 25000};     
                    
                    
Map.addLayer(imagem_selecionada, corVerdadeira, 'Carlopolis-cor verdadeira');//Adiciona imagem no visualizador de mapas
Map.addLayer(imagem_selecionada, falsaCor, 'Carlopolis-falsa cor');

```
![image](https://github.com/eliasberra/Sensoriamento_remoto_GB808/assets/41900626/f80bebdf-9ae4-413b-bd5e-ec1f01b7c571)




Dê uma olhada ao redor da cena e familiarize-se com a paisagem. Na aba de '_Layers_', você pode ativar e desativar as diferentes camadas de dados geoespaciais.


**Nota: Lembre de ir salvando o código.**


## Coletando dados de treinamento
1. O primeiro passo para classificar nossa imagem é coletar alguns dados de treinamento para ensinar o classificador. Queremos coletar amostras representativas de espectros de refletância para cada classe de cobertura da terra de interesse na cena recortada. Vamos definir o número de classes e o número de amostras;

Vamos classificar Carlópolis em 5 classes temáticas: 
  - 'urbana', 
  - 'floresta', 
  - 'area_agricola_vegetada' (área agrícola com cultivo evidente), 
  - 'area_agricola_solo' (área agrícola sem um cultivo evidente e/ou solo exposto) e,
  - 'agua'. 

Vamos coletar em torno de 10 amostras de treinamento para cada uma dessas classes.


Ative o 'Draw a rectangle' primeiramente: ![image](https://user-images.githubusercontent.com/41900626/184734863-20d9b073-204b-49ef-aba6-a6b24bcf00e5.png). O retângulo chamado 'geometry' é criado e deve estar aparecendo na parte superior do editor de código. Ele deve estar visível na aba '_Geometry Imports_' ![image](https://github.com/eliasberra/Sensoriamento_remoto_GB808/assets/41900626/d77fa5b5-e811-427d-8166-07e159d5aa72).

3. Vamos começar a coleta para a classe 'urbana'. Localize áreas (alguns pixels, a amostra não pode ser menor que o tamanho de um pixel) representativas dessa camada em áreas urbanas ou edificadas (edifícios, estradas, estacionamentos, etc.) e clique para coletá-los adicionando polígonos na camada de geometria.
5. Colete os 10 polígonos representativos e renomeie a '_geometry_' para 'urbana'. Lembre de parar a aquisição clicando no símbolo da mãozinha!
![image](https://github.com/eliasberra/Sensoriamento_remoto_GB808/assets/41900626/8025f475-b59c-491a-902b-91684b183b36)



6. Em seguida, configure a importação da geometria da classe 'urbana' clicando no símbolo da engrenagem na mesma linha em que ela se encontra ![image](https://github.com/eliasberra/Sensoriamento_remoto_GB808/assets/41900626/645313a0-ad18-4c7c-b099-150777b0403a).


Clique no ícone da engrenagem para configurá-lo, altere '_Import as_' de '_Geometry_' para '_FeatureCollection_'. Use '_+Property_' para adicionar valores identificadores de cada cobertura de terra (_Property_ = 'Uso' (de Uso da Terra) e _Value_ = 1). As classes subsequentes terão '_Value_' = 2, 3, 4 e 5. 
Clique mais uma vez em '_+Property_' e escreva '_Property_' = 'Nome' e '_Value_' = 'urbana', para garantir a identificação da classe de interesse futuramente, conforme abaixo.
Quando terminar, clique em 'OK'.
![image](https://github.com/eliasberra/Sensoriamento_remoto_GB808/assets/41900626/7a452e73-c5d1-41f3-9214-85cda9f4e3cc)


Repita o procedimento para as outras 4 classes: 'floresta', 'area_agricola_vegetada' (área agrícola com cultivo evidente), 'area_agricola_solo' (área agrícola sem um cultivo evidente e/ou solo exposto) e,  'agua'. Para criar novos polígonos, clique em '_+new layer_' ![image](https://github.com/eliasberra/Sensoriamento_remoto_GB808/assets/41900626/178fbc87-312e-4d91-b61e-0bf3a412752c).
Você pode achar mais interessante utilizar outras combinações de bandas para melhorar a fotointerpretação das classes na composição colorida. Lembre de usar a engrenagem para configurar as geometrias, alterando o tipo para '_FeatureCollection_' e definindo o nome da propriedade como 'Uso' com valores de 2, 3, 4 e 5 para as diferentes classes. Também defina o 'Nome' com o nome da respectivas classes.
![image](https://github.com/eliasberra/Sensoriamento_remoto_GB808/assets/41900626/8b485137-0009-4f06-93fd-2e117902ee3a)




9. Agora temos cinco classes definidas ('urbana', 'floresta', 'area_agricola_vegetada', 'area_agricola_solo' e 'agua'), mas antes de podermos usá-las para coletar dados espectrais para treinar nosso classificador, precisamos mesclá-las em uma única coleção de feições, chamada _FeatureCollection_:

```javascript
//mesclar as classes em uma única coleção
var nomeClasses = urbana.merge(floresta).merge(area_agricola_vegetada).merge(area_agricola_solo).merge(agua);
```

10. Imprima a coleção de feições e as inspecione no Console.

```javascript
//Imprimir a coleção de feições para inspeção
print('nome das classes', nomeClasses)
```
![image](https://user-images.githubusercontent.com/41900626/184744424-400a3a38-8a47-46c1-8ed0-bf0dccca6061.png)




## Extraia os dados espectrais para treinamento do classificador

Agora podemos usar o FeatureCollection que criamos para extrair os dados de reflectância para cada ponto amostral, de cada banda. Criamos dados de treinamento sobrepondo os polígonos de treinamento na imagem. Isso adicionará novas propriedades à coleção de feições que representará os valores das bandas espectrais de cada polígono:

```javascript
//Extrair uma lista de valores em cada ponto em cada banda
var bandas = ['SR_B2', 'SR_B3', 'SR_B4', 'SR_B5', 'SR_B6', 'SR_B7'];
var treinamento = img_recorte.select(bandas).sampleRegions({
  collection: nomeClasses,
  properties: ['Uso','Nome'],//essa propriedade deve ser escrita da mesma forma como escrita no passo 5 e 7
  scale: 30
});
print('treinamento', treinamento);
```
Depois de executar o script, os dados de treinamento serão impressos no Console. Você notará que as informações de 'properties' agora mudaram e, além da classe de cobertura da terra, para cada ponto agora há um valor de refletância correspondente para cada banda da imagem.

![image](https://user-images.githubusercontent.com/41900626/184747281-6f90ec3b-f346-49a2-9696-2a0783a3c1e1.png)






## Treine o classificador e execute a classificação
Nesse exemplo, vamos usar o classificador CART (Classification and Regression Trees) (Breiman et al. 1984) para prever as classes temáticas.
Agora podemos treinar o algoritmo do classificador usando nossas amostras que devem ser, em teoria, representatias das diferentes classes de cobertura da terra de uma perspectiva multiespectral.

```javascript
//Treinar o classificador
var classificador = ee.Classifier.smileCart().train({
      features: treinamento,
      classProperty: 'Uso',
      inputProperties: bandas
});
```

O próximo passo é aplicar esse conhecimento de nosso treinamento ao restante da imagem - usando o que foi aprendido em nossa coleção supervisionada (ou seja, nós supervisionamos a coleta das amostras) para auxiliar o classificador a decidir sobre qual classe os outros pixels devem pertencer.


```javascript
//Executa a classificação
var classificada = img_recorte.select(bandas).classify(classificador);
```

Exiba os resultados usando a função de mapeamento abaixo. Você pode precisar ajustar as cores, mas se os dados de treinamento foram criados com urbana = 1, floresta = 2,  area_agricola_vegetada = 3, area_agricola_solo = 4, agua = 5, - então o resultado será renderizado com essas classes como urbanoa= cinza, floresta = verde,  area_agricola_cultivada = oliva, area_agricola_solo = amarelo e água = azul.
Você pode consultar opções de cores para a pallete em <https://en.wikipedia.org/wiki/Web_colors>.


```javascript
//Exibir a classificação
Map.centerObject(nomeClasses, 10);
Map.addLayer(classificada,
{min: 1, max: 5, palette: ['grey', 'green', 'olive','yellow', 'blue']}, 'classificação');
```
![image](https://user-images.githubusercontent.com/41900626/175436317-53ad2840-1bbe-4fc5-9d61-9abcf9671f24.png)



## Examine seus resultados

Parabéns - você fez sua primeira classificação de cobertura da terra! Mas.....
- Você está feliz com a classificação?
- Como poderia ser melhorado?
- Tente adicionar algumas classes extras para categorias de cobertura de terra que mostram sinais de confusão

Veremos como refinar isso e discutiremos limitações e caminhos para melhorias no próximo tutorial.

Lembre de salvar seu script, pois o mesmo será utilizado no próximo tutorial.

-------

## Melhorando a Classificação

Aqui devemos pensar se estamos satisfeitos ou não com essa classificação. Mesmo sem quaisquer dados quantitativos, podemos perceber erros em algumas regiões. Como podemos melhorar? Existem algumas opções que podemos explorar:

1. Altere o tamanho da amostra de treinamento. Amostramos apenas ~30 pixels por classe. Foram muitos cliques, mas poderíamos usar polígonos em vez de pontos para amostrar mais pixels para treinamento.
2. Altere a estratégia de amostragem. Coletamos um mesmo número de pontos por classe, mas algumas classes de cobertura da terra cobrem muito mais área do que outras. Em vez disso, poderíamos experimentar uma abordagem de amostragem estratificada.
3. Altere o classificador. Usamos um classificador CART, podemos tentar uma abordagem diferente, como uma Support Vector Machine(SVM) ou a abordagem randomForest.
4. Altere as bandas. Poderíamos adicionar informações auxiliares, como dados de elevação, ou um índice derivado, como NDVI, para fornecer informações para discriminação de classe.
5. Altere a imagem. Usamos uma cena de verão do Landsat-8. Poderíamos tentar uma cena de inverno ou mudar para uma imagem do Sentinel-2.

### Referência
L., Breiman, J.H. Friedman, R.A. Olshen, and C.J. Stone. 1984. Classification and Regression Trees.
Boca Raton, FL: Wadsworth; Taylor & Francis Group

### Obrigado


