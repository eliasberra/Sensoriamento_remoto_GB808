# Sensoriamento Remoto
Lab 05 - Validação da classificação e avaliação da acurácia
--------------

### Agradecimentos
- Google Earth Engine Team
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

O objetivo deste laboratório é aprender como avaliar os resultados de classificação de imagens e conduzir uma avaliação da acurácia usando dados de validação independentes.

----------

## Recapitulando
Vamos continuar de onde paramos no último laboratório.
Abra o código do último laboratório no GEE. Faça uma pequena edição, por exemplo, adicione uma linha a mais. Acesse 'Save' e 'Save as' para salvar uma cópia do código. Dê um nome ao arquivo.
![image](https://user-images.githubusercontent.com/41900626/175935827-bdb54136-6de9-4357-a197-9b1160a7ee2e.png)

Execute novamente sua classificação. Você deve estar visualizando algo parecido com a figura abaixo:
![image](https://user-images.githubusercontent.com/41900626/175936358-28294555-d2b7-4d7c-ae41-6ac0b0d378fe.png)


## Validação da classificação

1. Colete dados de validação usando a ferramenta de geometria de polígono retangular ![image](https://user-images.githubusercontent.com/41900626/175937417-b8d465e1-d5af-48e2-b5ba-958bd98e2e09.png):
  - faça isso da mesma maneira que você coletou dados de treinamento.
  - use os mesmos nomes e rótulos de propriedade.
  - não sobreponha os polígonos aos dados de treinamento (pontos); queremos dados independentes.
  - não exceda 5000 pixels.
  - coletar amostras das mesmas cinco classes, mas renomeia-as de forma diferente ('vUrbana', 'vFloresta', 'vArea_agricola_vegetada', 'vArea_agricola_solo', 'vAgua')

Por exemplo, coletando poligonos para representar os dados de validação dentro de vUrbana: 
![image](https://user-images.githubusercontent.com/41900626/175941484-d0724dc3-8b75-4618-b038-a004948a6f60.png)

Configure a geometria (clicando no ícone da engrenagem), seguindo a ordem dos dados de treinamento. Por exemplo, para 'vUrbana', configure para 'FeatureCollection', adicione a propriedade 'cobertura_terra' e valor = 0 (para as demais classes escreva valor 1,2,3 e 4).
![image](https://user-images.githubusercontent.com/41900626/175942534-a534d5b4-a343-445e-a86a-5896eaeb8bab.png)

Nota: Lembre de ir salvando o código.
Ao final, você deve ter adquirido amostras para as cinco classes de validação:
![image](https://user-images.githubusercontent.com/41900626/175948116-243de285-0749-4d6f-97dd-c2111f749dae.png)

  

2. Mescle seus polígonos de validação em uma coleção de feições
```JavaScript
//--------------Validação--------------------------------

//mesclar os polígonos de validação
var nomeVal = vUrbana.merge(vFloresta).merge(vArea_agricola_vegetada).merge(vArea_agricola_solo).merge(vAgua);
```

3. Amostrar os resultados da classificação para as áreas de validação
```JavaScript
//Extrair valores da classificação nos polígonos de validação
var validacao = classificada.sampleRegions({
  collection: nomeVal,
  properties: ['cobertura_terra'],
  scale: 30,
});
print('validação', validacao);
```
![image](https://user-images.githubusercontent.com/41900626/175950839-df5f6b58-a555-423c-9a8d-9d619b87d4ff.png)


4. Executar a avaliação de validação usando a abordagem de matriz de erros (ou matriz de confusão). Além do Console, a matriz de erros é exportada como um arquivo texto (.csv) para permitir que esse resultado seja melhor trabalhado e organizado.
```JavaScript
// Construir a matriz de confusão dos dados de validação contra o resultado da classificação
var testeAcuracia = validacao.errorMatrix('cobertura_terra', 'classification');

// Imprime a matriz de erro no console e exporta tabela
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
![image](https://user-images.githubusercontent.com/41900626/176021853-48d4451b-2eca-4312-8e40-6e15118aa6e3.png)

Você deve ter observado que a aba 'Tasks' ficou laranja ![image](https://user-images.githubusercontent.com/41900626/176021968-ccf22719-c332-4979-aa52-34102b3ceedd.png)
Se você clicar nela, irá aparece uma tarefa (task) esperando a ser executada. Clique em 'RUN' para exportar a tabela e salvá-la no seu Google Drive. Uma vez exportada, você pode trabalhar a tabela posteriormente calculando, por exemplo, a acurácia de cada classe individualmente.
![image](https://user-images.githubusercontent.com/41900626/176022613-a803a49d-9816-4885-92a3-c64193e5bd0c.png)


5. Por fim, calcule um conjunto de medidas fornecendo indicadores da qualidade da classificação
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

[A acurácia global, é uma medida percentual da quantidade de amostras corretamente classificadas, em relação ao total de amostras disponíveis.

A acurácia do consumidor (AC), ou do usuário, é a medida de acurácia do ponto de vista do usuário/consumidor do mapa. Aqui, a acurácia basicamente diz ao usuário com que frequência a classe do mapa estará realmente presente no solo ou em sua referência. É também conhecida como medida de confiabilidade (reliability). O erro de comissão (EC) é complemento de AC e nos informa a quantidade de classificações associadas a falsos positivos ou falsos alarmes.

A acurácia do produtor (AP) é a acurácia do ponto de vista do criador/produtor do mapa. Esta é a frequência com que as características reais no solo são mostradas corretamente no mapa classificado ou a probabilidade de uma determinada cobertura/uso da terra ser classificada como tal no mapa. A AP é complementada pelo erro de omissão (EO), que nos informa a quantidade percentual de erros de omissões, ou seja, classes que deveriam ter sido detectadas, mas que por alguma razão foram omitidas da classificação.] (https://solved.eco.br/avaliacao-de-acuracia-ou-concordancia/)


6. Encontre o arquivo 'matrizConfusao.csv' no seu Google Drive e organize os resultados da validação em uma tabela similar a apresentada abaixo:
![image](https://user-images.githubusercontent.com/41900626/178567779-a0a63829-30c8-4e25-8e8b-70a81c9278f9.png)
O que achou dos resultados da classificação? Considera um bom mapa temático de classes de cobertura da terra?


## Calcular a área de cada classe temática
Vamos agora calcular a área de cada classe temática que acabamos de detectar.
Para isso, é interessante realizar o cálculo em _loop_, utilizando a função 'for', a qual automatiza o processo. Caso contrário, teríamos que escrever um código para cada classe de interesse.

```JavaScript
//-----------------Calcular a área ocupada por cada classe temática--------------
for (var a = 0; a < 5; a++){//'a' vai representar a quantidade de classes temáticas (0, 1, 2, 3 e 4)
  var x = classificada.eq(a).multiply(ee.Image.pixelArea());//recupera a área de cada pixel em m²
  var stats = x.reduceRegion({//reduz a imagem de interesse em uma quantidade, nesse caso, a área total
  reducer: ee.Reducer.sum(),//soma a área de todos os pixels
  geometry: imagem.geometry(),//define a região de interesse onde realizar a soma
  maxPixels: 1e9//o número máximo de pixels a reduzir
});
  var area_class = ee.Number(stats.get('classification'));//define a área em formato numérico
  area_class = area_class.divide(10000);//transforma de m² para hectares (ha)
  print('pixels da classe', a, area_class, 'ha');//imprime a área no Console  
  }
```
Ao final, a área de cada classe deverá aparecer no Console.
![image](https://user-images.githubusercontent.com/41900626/180432057-529b9496-a7c5-4894-b85a-ab5dfa125127.png)


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
Na janela que abre, clique em _Run_. Você pode modificar os parâmetros se julgar necessário.
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





-------
### Obrigado


