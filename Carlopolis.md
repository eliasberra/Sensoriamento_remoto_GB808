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
//mesclar as classes em uma única coleção de feições
var nomeClasses = urbana.merge(floresta).merge(area_agricola_vegetada).merge(area_agricola_solo).merge(agua);

//Imprimir a coleção de feições para inspeção
print('nome das classes', nomeClasses)

```
Repare que a coleção assume os valores e nomes que acambamos de definir no passo acima:
![image](https://github.com/eliasberra/Sensoriamento_remoto_GB808/assets/41900626/7566a82e-020d-4f43-8e2a-187ebb050220)





## Extraia os dados espectrais para treinamento do classificador

Agora podemos usar o coleção de feições que criamos para extrair os dados de reflectância espectral dentro de cada amostra. Os valores extraídos serão adicionados como novas propriedades à coleção de feições.

```javascript
//Extrair uma lista de valores por amostra, por banda espectral
var bandas = ['SR_B2', 'SR_B3', 'SR_B4', 'SR_B5', 'SR_B6', 'SR_B7'];
var treinamento = imagem_selecionada.select(bandas).sampleRegions({
  collection: nomeClasses,
  properties: ['Uso','Nome'],//cuidado: as palavras são sensíveis as letras maísuculas/minúsculas
  scale: 30
});
print('treinamento', treinamento);
```
Depois de executar o script, os dados de treinamento serão impressos no '_Console_'. Você notará que as informações de 'properties' agora mudaram e, além da classe de cobertura da terra, para cada ponto agora há um valor de refletância correspondente para cada banda da imagem.

![image](https://github.com/eliasberra/Sensoriamento_remoto_GB808/assets/41900626/068ce7c5-1b2e-41c0-a4c2-ffecbce28827)





## Treine o classificador e execute a classificação
Nesse exemplo, vamos usar o classificador CART (_Classification and Regression Trees_) (Breiman et al. 1984) para prever as classes temáticas.
Agora podemos treinar o algoritmo do classificador usando nossas amostras que devem ser, idealmente, representativas de toda a variabilidade espectral presente na cena escolhida.

```javascript
//Treinar o classificador
var classificador = ee.Classifier.smileCart().train({
      features: treinamento,
      classProperty: 'Uso',
      inputProperties: bandas
});
```

Agora, o CART definiu uma árvore de decisão para classificar os pixels da nossa imagem em uma das cinco classes no qual o CART foi treinado. Podemos executar a classificação.

```javascript
//Executar a classificação
var classificada = imagem_selecionada.select(bandas).classify(classificador);
```

Exiba os resultados usando a função de mapeamento abaixo. Você pode precisar ajustar as cores, mas se os dados de treinamento foram criados com 'urbana' = 1, 'floresta' = 2,  'area_agricola_vegetada' = 3, 'area_agricola_solo' = 4, 'agua' = 5, então o resultado será renderizado com essas classes como 'urbana'= cinza, floresta = 'verde',  'area_agricola_cultivada' = oliva, 'area_agricola_solo' = amarelo e 'agua' = azul.
Você pode consultar opções de cores para a pallete em <https://en.wikipedia.org/wiki/Web_colors>.


```javascript
//Exibir a classificação
Map.centerObject(nomeClasses, 10);
Map.addLayer(classificada,
{min: 1, max: 5, palette: ['grey', 'green', 'olive','yellow', 'blue']}, 'classificação');
```
![image](https://github.com/eliasberra/Sensoriamento_remoto_GB808/assets/41900626/2ca25caa-eb76-4aed-a159-3c707b515577)



## Validação da classificação

Tão importante quanto a classificação, é sabermos sua acurácia. Para isso, podemos validar nossa classificação com amostras indenpendentes para as classes de interesse.

1. Colete dados de validação usando a ferramenta de geometria de polígono retangular ![image](https://user-images.githubusercontent.com/41900626/175937417-b8d465e1-d5af-48e2-b5ba-958bd98e2e09.png):
  - faça isso da mesma maneira que você coletou dados de treinamento.
  - coletar amostras das mesmas cinco classes, mas renomeia-as de forma diferente, adicionando o 'v' de validação ('vUrbana', 'vFloresta', 'vArea_agricola_vegetada', 'vArea_agricola_solo', 'vAgua')
  - use os mesmos nomes e rótulos de propriedade.
  - não sobreponha os polígonos aos dados de treinamento; queremos dados independentes.
  - não exceda 5000 pixels (se não o GEE trava).
  
Vamos a um exemplo com a amostra 'vUrbana'. Colete poligonos para representar os dados de validação dentro de 'vUrbana': 
![image](https://github.com/eliasberra/Sensoriamento_remoto_GB808/assets/41900626/74195d19-068d-4df6-8e4f-251b57fcbeb7)



Configure a geometria (clicando no ícone da engrenagem), seguindo a mesma ordem de insreção dos dados de treinamento. Por exemplo, para 'vUrbana', configure para '_FeatureCollection_', clique em '_+Property_' e  adicione a propriedade '_Property_' = 'Uso'  e '_Value_' = '1' (para as demais classes escreva o valor 2,3, 4 e 5).
Clique mais uma vez em '+Property' e  adicione 'Property' = 'Nome'  e 'Value' = 'vUrbana' (para as demais classes escreva o valor  'vFloresta', 'vArea_agricola_vegetada', 'vArea_agricola_solo', 'vAgua').

![image](https://github.com/eliasberra/Sensoriamento_remoto_GB808/assets/41900626/7049f8d6-829e-4684-bc4d-9a6eac4804d4)



**Nota: Lembre de ir SALVANDO o código.**

Ao final, você deve ter adquirido amostras de validação para as cinco classes temáticas:
![image](https://github.com/eliasberra/Sensoriamento_remoto_GB808/assets/41900626/99aab88a-6ec3-403f-a6fc-b9d0f2d3f83b)


2. Mescle seus polígonos de validação em uma coleção de feições
```JavaScript
///--------------Validação------------------------------------------------
//-----------------------------------------------------------------------
//mesclar os polígonos de validação
var nomeVal = vUrbana.merge(vFloresta).merge(vArea_agricola_vegetada).merge(vArea_agricola_solo).merge(vAgua);
```

3. Amostrar os resultados da classificação para as áreas de validação
```JavaScript
//Extrair valores da classificação nos polígonos de validação
var validacao = classificada.sampleRegions({
  collection: nomeVal,
  properties: ['Uso', 'Nome'],
  scale: 30,
});
print('validação', validacao);

```
![image](https://user-images.githubusercontent.com/41900626/184910866-0eb5a43a-a436-4955-99e1-8b6494fa0d7b.png)



4. Executar a avaliação de validação usando a abordagem de matriz de erros (ou matriz de confusão). Além do Console, a matriz de erros é exportada como um arquivo texto (.csv) para permitir que esse resultado seja melhor trabalhado e organizado.
```JavaScript
// Construir a matriz de confusão dos dados de validação contra o resultado da classificação
var testeAcuracia = validacao.errorMatrix('Uso', 'classification');

// Imprime a matriz de erro no Console 
print('Matriz de erro de validação: ', testeAcuracia);

//Exportar para o Google Drive a tabela com a matriz de confusão
var featureCollection = ee.FeatureCollection(testeAcuracia.getInfo()
                        .map(function(element){
                        return ee.Feature(null,{prop:element})}));
Export.table.toDrive({collection: featureCollection,
    description: 'Exportar_matriz_confusao',
    folder: 'GB808',//Folder no Google Drive onde será armazenado o arquivo CSV
    fileNamePrefix: 'matrizConfusao', 
    fileFormat: 'CSV',
    selectors: (['system:index', 'prop'])
});


```
![image](https://user-images.githubusercontent.com/41900626/184911504-abb712ee-84a8-419b-954e-c5e8fcbba737.png)


Você deve ter observado que a aba 'Tasks' ficou laranja ![image](https://user-images.githubusercontent.com/41900626/176021968-ccf22719-c332-4979-aa52-34102b3ceedd.png)
Se você clicar nela, irá aparece uma tarefa (task) esperando a ser executada. Clique em 'RUN' para exportar a tabela e salvá-la no seu Google Drive. Uma vez exportada, você pode trabalhar a tabela posteriormente calculando, por exemplo, a acurácia de cada classe individualmente.
![image](https://user-images.githubusercontent.com/41900626/176022613-a803a49d-9816-4885-92a3-c64193e5bd0c.png)


5. Por fim, calcule um conjunto de medidas fornecendo indicadores da qualidade da classificação.
```JavaScript
// Calcula e imprime a acurácia geral no console
print('Acurácia geral da validação: ', testeAcuracia.accuracy());

// Calcula a Acurácia do Consumidor (ou do Usuário) – AC, também conhecida como 
//especificidade e complemento do erro de comissão (1 − erro de comissão)
print('AC:', testeAcuracia.consumersAccuracy());

// Calcula a Acurácia do Produtor – AP, também conhecida como 
//especificidade e complemento do erro de omissão (1 − erro de omissão)
print('AC:', testeAcuracia.producersAccuracy());

// Calcula e imprime a estatística kappa
print('Estatística kappa:', testeAcuracia.kappa());
```

6. Essas metricas, na maioria dos casos, ficam melhor apresentadas em uma tabela. Encontre o arquivo 'matrizConfusao.csv' no seu Google Drive.
Ao abrir o arquivo, desconsidere a primeira linha e a primeira coluna.
![image](https://user-images.githubusercontent.com/41900626/185953931-1d31cee9-c634-4a3d-95db-3fd894eb1f56.png)

Após isso, organize os resultados da validação em uma tabela similar a apresentada abaixo (a tabela aabixo está no Teams e pode ser utilizada como modelo  para os cálculos):
![image](https://user-images.githubusercontent.com/41900626/184937282-f8496879-3135-4506-bbcf-f72956d3b7d6.png)
  
 Vamos à interpretação da tabela (a sua tabela terá, provavelmente, valores diferentes):
 
  Acurácia de Usuário (AC):
  Por exemplo, para a classe Área Urbana, temos uma acurácia de usuário AC=78%. Isso significa que, de todos os pixels que o classificador 'disse' (classificou) que  eram área urbana (37), ele acertou 29, ou seja, 29/37 = 78% dos casos. Ou seja, era área urbana e o classificador classificou como área urbana (_true positive_). Associado ao AC, temos o erro de comissão (ou complemento de AC); 
No exemplo, o classificador 'disse' que 8 amostras (7 de área agricola vegetada + 1 de de área agricola solo) eram Área Urbana quando, na verdade, eram outra classe (áreas agrícolas). Assim, 8/37= 22% foi o erro de comissão (que é o complemento de 78%) 

  Acurácia de Produtor (AP):
  De quantas amostras que eu indiquei que eram Área Urbana e, que, de fato, o algorítimo acertou como Área Urbana? No exemplo, indiquei um total de 30 amostras como Área Urbana (verdade de campo) e o algorítmo acertou 29, ou seja, 29/30 = 97%. O complemento, 100%-97% = 3% é o erro de omissão, ou seja, ele omitiu em 1/30 casos dizendo que essas 1 amostra não é Área Urbana, quando, na realidade, é área urbana.


O que achou dos resultados da classificação? Considera um bom mapa temático de classes de cobertura da terra?
Maiores explicações podem ser vistas em (https://solved.eco.br/avaliacao-de-acuracia-ou-concordancia/) ou (https://mapbiomas.org/analise-de-acuracia). Também, o vídeo no YouTube chamado 'Avaliação da Acurácia da Classificação' tem uma boa explicação do tópico (https://youtu.be/hoWymG_lsWw).

## Calcular a área de cada classe temática
Vamos agora calcular a área de cada classe temática que acabamos de detectar.
Para isso, é interessante realizar o cálculo em _loop_, utilizando a função 'for', a qual automatiza o processo. Caso contrário, teríamos que escrever um código para cada classe de interesse.

```JavaScript
//-----------------Calcular a área ocupada por cada classe temática--------------
for (var a = 1; a < 6; a++){//'a' vai representar a quantidade de classes temáticas (1, 2, 3, 4 e 5)
  var x = classificada.eq(a).multiply(ee.Image.pixelArea());//recupera a área de cada pixel em m²
  var stats = x.reduceRegion({//reduz a imagem de interesse em uma quantidade, nesse caso, a área total
  reducer: ee.Reducer.sum(),//soma a área de todos os pixels
  geometry: imagem.geometry(),//define a região de interesse onde realizar a soma
  maxPixels: 1e9//o número máximo de pixels a reduzir
});
  var area_class = ee.Number(stats.get('classification'));//define a área em formato numérico
  area_class = area_class.divide(10000).round();//transforma de m² para hectares (ha)
  print('pixels da classe', a, area_class, 'ha');//imprime a área no Console  
  }
```
Ao final, a área de cada classe deverá aparecer no Console.
![image](https://user-images.githubusercontent.com/41900626/184939542-480e8373-697c-4580-8357-52173784323a.png)
Por exemplo, a classe 'Área urbana', valor = 0, nesse tutorial atingiu uma área total de 7.685 ha.


## Exportar mapa para impressão
O GEE é excelente para processamento digital de imagens, mas não é o mais indicado para preparação de mapas para impressão. Para isso, vamos utilizar o QGIS.
Então, precisamos exportar a imagem classificada para, depois, importá-la no QGIS.

```JavaScript
//-----------------Preparar mapa temático para impressão--------------
// --- Exportar imagem classificada para o Google Drive---
Export.image.toDrive({image: classificada,//indica qual imagem será exportada 
                    description: 'Exportar_classificada', //descrição da tarefa que aparecerá em Tasks
                    folder: 'GB808',//pasta no Google Drive onde será salva a imagem.
                    fileNamePrefix: 'Classificada',//nome da imagem a ser salva                     
                    scale: 30,//resolução espacial que será salva a imagem 
                    });
```
A tarefa de exportação irá aparecer na aba 'Tasks'.
![image](https://user-images.githubusercontent.com/41900626/180433984-1c2e8922-20f6-4af7-844a-07c94f30fb1d.png)

Em 'Exportar_classificada', clique em _Run_.
Na janela que abre, clique em _Run_. Você pode modificar os parâmetros se julgar necessário. No momento, deixe os parametros como estão.
![image](https://user-images.githubusercontent.com/41900626/180451150-ecbbcee8-8be8-41b1-a10b-6dd8af4c8b30.png)

Pronto, a imagem classificada está salva no seu Google Drive (![image](https://user-images.githubusercontent.com/41900626/180434989-c67e7765-7ddd-4e5d-8371-ea3a3e51ce0d.png)) e pode ser importada no QGIS.



## Preparando mapa para impressão no QGIS
Baixe a imagem 'Classificada.tif' para uma pasta no seu computador.
Abra o QGIS e importe a imagem.
Acesse as 'Propriedades' da imagem e observe o item 'Sistema de Referência de Coordenadas (SRC)'
![image](https://user-images.githubusercontent.com/41900626/180450171-288ace23-719a-4570-a818-78c9b54a67bb.png)
Como se pode ver, o SRC está como  'EPSG:32622 - WGS 84 / UTM zone 22N'. O que aparenta ser, em um primiero momento, um erro, é uma metodologia adotada pela distribuidora das cenas Landsat (USGS) para otimizar alguns processamentos. Veja mais informações na matéria ['Why do Landsat scenes in the Southern Hemisphere display negative UTM values?'](https://www.usgs.gov/faqs/why-do-landsat-scenes-southern-hemisphere-display-negative-utm-values).

Deixe o SRC como está.
Agora vamos identificar as classes com nomes que tragam um significado claro para o leitor do mapa.
No QGIS, clique com o botão direito em 'Classificada' > 'Simbologia' e classifique as classes. Primeiramente, elimine a classe '0' que representa a borda da imagem sem valores significativos (no data), conforme abaixo. 
![image](https://user-images.githubusercontent.com/41900626/184964806-5639b40d-7c4c-42bd-8a39-eb40cad23b7e.png)

Agora, você deve estar visualizando as classes de 1 a 5. Agora substitua os números na coluna 'Rótulo' pelos respectivos nomes das classes temáticas. Também, modifique as cores das classes. 
![image](https://user-images.githubusercontent.com/41900626/184965940-0477c61e-ba6f-48b3-b70f-e63f530efb86.png)

Agora, você pode preparar um layout para imprimir o mapa:
![image](https://user-images.githubusercontent.com/41900626/184966380-52e6737a-af60-459a-a905-bbe394eda386.png)

Se você não lembra como utilizar o layout de impressão, você pode consultar vídeos no YouTube como por exemplo ['Como fazer o Layout de um mapa no QGIS?'](https://youtu.be/WVeMSAxt3O4) ou revisar os tutoriais de Cartografia Temática.



-------
### Obrigado
Espero que este tutorial tenha sido útil.














































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


