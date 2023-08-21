# Sensoriamento Remoto
Lab 04 - Classificação de imagens - parte 1
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

O objetivo deste laboratório é entender o processo de classificação de imagens e explorar formas de transformar imagens de sensoriamento remoto em mapas de cobertura da terra.

----------

## Carregando a imagem
Vamos usar a ferramenta de desenho de ponto (ícone de lágrima) das ferramentas de geometria e desenhar um único ponto 
na região de interesse - vamos usar a cidade de Carlópolis, PR, para este exemplo. Após desenhado, 
clique na mãozinha para sair das ferramentas de desenho. Observe que uma nova variável é criada na seção de importações ('Imports'), contendo o ponto único, importado como uma Geometria. Altere o nome desta importação para "roi" - abreviação de região de interesse.

![image](https://user-images.githubusercontent.com/41900626/175430473-421e0bfc-7656-407f-a242-ffbd52765c49.png)

O primeiro passo é obter uma imagem sem nuvem para trabalhar. Faça isso importando imagens USGS Landsat 8 Surface Reflectance Tier 1, filtrando espacialmente para uma região de interesse (filterBounds), filtrando temporalmente para o intervalo de datas necessário (filterDate) e, por último, classificando por cobertura de nuvens ('CLOUD_COVER') e extraindo a cena menos nublada (first - primeira).

Em seguida, podemos executar o script abaixo para extrair nossa imagem desejada da coleção Landsat 8 e adicioná-la à visualização do mapa como uma composição de cores verdadeiras (você pode copiar e colar OU digitar manualmente o Script):

```JavaScript
//Seleciona imagem 
var imagem = ee.Image(ee.ImageCollection('LANDSAT/LC08/C02/T1_L2') 
    .filterBounds(roi)
    .filterDate('2022-01-01', '2022-06-30')
    .sort('CLOUD_COVER')
    .first()); 
//Inspeciona imagem selecionada e a mostra no mapa
print('imagem', imagem)
Map.addLayer(imagem, {bands:['SR_B4', 'SR_B3', 'SR_B2'],min:6000, max: 10000}, 'Composição cor verdadeira');
```
![image](https://user-images.githubusercontent.com/41900626/175431098-d6545b86-f7c0-4056-af33-505dc5d6f592.png)

Dê uma olhada ao redor da cena e familiarize-se com a paisagem. A composição parece ter muito brilho, não achas? Você pode testar diferentes limiares de contraste ajustando o valor de min e max.


**Nota: Lembre de ir SALVANDO o código.**

## Recortando a cena para uma área menor
Antes de prosseguirmos às etapas de classificação, vamos recortar nossa cena para uma área teste de menor tamanho a fim tornar mais rápida a etapa de classificação automática e economizar recurso computacional.
Passe o mouse na caixa 'Geometry Imports' ao lado das ferramentas de desenho de geometria e clique em '+ new layer'.![image](https://user-images.githubusercontent.com/41900626/175122195-9242e624-cf3b-4ab1-973c-1581a8aa0578.png)
Use a ferramenta de geometria 'Draw a rectangle' ![image](https://user-images.githubusercontent.com/41900626/175122290-343fa15c-cd83-4321-aaa5-56d152b35c76.png)
para desenhar um retângulo ao redor de Carlópolis representando nossa área de interesse. Desenhe um retângulo com dimensões parecidas as apresentadas na figura abaixo. Renomeie o vetor para 'recorte'.
![image](https://user-images.githubusercontent.com/41900626/175431592-88b41349-c9c9-4d0f-87c8-707b42dbd884.png)

Clique novamente na mãozinha para sair do modo desenho.

Em seguida, podemos executar o script abaixo para extrair nossa imagem recortada:

```JavaScript
//Recortar cena
var img_recorte =  imagem.clip(recorte);
Map.addLayer(img_recorte, {bands:['SR_B4', 'SR_B3', 'SR_B2'],min:6000, max: 20000}, 'Recorte');
```
![image](https://user-images.githubusercontent.com/41900626/175431844-9218ab35-457b-4f88-ada7-4a05fe8ad066.png)


Você pode habilitar/desabilitar qualquer um dos vetores na aba de 'Geometry Imports'. Você pode, da mesma forma, desabilitar as camdas na aba 'Layers'.



## Coletando dados de treinamento
1. O primeiro passo para classificar nossa imagem é coletar alguns dados de treinamento para ensinar o classificador. Queremos coletar amostras representativas de espectros de refletância para cada classe de cobertura da terra de interesse na cena recortada. Ative o 'Draw a rectangle' primeiramente: ![image](https://user-images.githubusercontent.com/41900626/184734863-20d9b073-204b-49ef-aba6-a6b24bcf00e5.png)

2. Passe o mouse na caixa 'Geometry Imports' ao lado das ferramentas de desenho de geometria e clique em '+ new layer' ![image](https://user-images.githubusercontent.com/41900626/175124178-fd317651-ebba-403b-bc2d-c9693a6698c9.png).

3. Cada nova camada (layer) a ser criada armazenará a localização do conjunto de polígonos representando uma determinada classe de cobertura da terra.
4. Vamos definir nossa primeira nova camada/classe como 'urbana'. Localize áreas (alguns pixels) representativos dessa camada em áreas urbanas ou edificadas (edifícios, estradas, estacionamentos, etc.) e clique para coletá-los adicionando polígonos na camada de geometria.
5. Colete em torno de 10 polígonos representativos e renomeie a 'geometry' para 'urbana'. Lembre de parar a aquisição clicando no símbolo da mãozinha![image]![image](https://user-images.githubusercontent.com/41900626/184735493-b28b6d65-a3db-40bc-a2ba-9847533ac11a.png)


6. Em seguida, você pode configurar a importação da geometria da classe 'urbana' clicando no símbolo da engrenagem na mesma linha em que ela se encontra ![image](https://user-images.githubusercontent.com/41900626/175432525-c8ee3afd-3265-4e20-b680-352c89888f73.png). 
Clique no ícone da engrenagem para configurá-lo, altere 'Import as' de 'Geometry' para 'FeatureCollection'. Use '+Property' para adicionar valores identificadores de cada cobertura de terra (Property = 'Uso' (de Uso da Terra) e Value = 1). As classes subsequentes terão 'Value' 2, 3, 4 e 5. 
Clique mais uma vez em '+Property' e escreva Property = 'Nome' e Value = urbana, para garantir a identificação da classe de interesse futuramente, conforme abaixo:
Quando terminar, clique em 'OK'.
![image](https://user-images.githubusercontent.com/41900626/184952065-d7acfead-0fda-4958-8279-ee1ba186446b.png)



7. Adicione mais 4 classes. Colete em torno de 10 amostras de treinamento para cada uma dessas classes: 
  - 'floresta', 
  - 'area_agricola_vegetada' (área agrícola com cultivo evidente), 
  - 'area_agricola_solo' (área agrícola sem um cultivo evidente; solo exposto) e,
  - 'agua'. 

Nota: Lembre de salvar seu código---

8. Repita a etapa 5 para cada classe de cobertura da terra que deseja incluir em sua classificação, garantindo que os pontos de treinamento se sobreponham à imagem. Você pode achar mais interessante utilizar outras combinações de bandas para melhorar a fotointerpretação das classes na composição colorida. Lembre de usar a engrenagem para configurar as geometrias, alterando o tipo para FeatureCollection e definindo o nome da propriedade como 'Uso' com valores de 2, 3, 4 e 5 para as diferentes classes. Também defina o 'Nome' com o nome da respectivas classes.
![image](https://user-images.githubusercontent.com/41900626/184743002-ae2f92f4-95d6-4295-8235-0d7b9c322be8.png)




9. Agora temos cinco classes definidas ('urbana', 'floresta', 'area_agricola_vegetada', 'area_agricola_solo' e 'agua'), mas antes de podermos usá-las para coletar dados espectrais de treinamento, precisamos mesclá-las em uma única coleção, chamada FeatureCollection. Execute a seguinte linha para mesclar as geometrias em um único FeatureCollection:

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


