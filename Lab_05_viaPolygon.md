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
![image](https://user-images.githubusercontent.com/41900626/184905996-8fa3b600-0ed8-4762-84cd-d5035340f8ca.png)

Execute novamente sua classificação. Você deve estar visualizando algo parecido com a figura abaixo:
![image](https://user-images.githubusercontent.com/41900626/184953593-a423368b-44f2-4d23-abd0-69acbfb526d9.png)


## Validação da classificação

1. Colete dados de validação usando a ferramenta de geometria de polígono retangular ![image](https://user-images.githubusercontent.com/41900626/175937417-b8d465e1-d5af-48e2-b5ba-958bd98e2e09.png):
  - faça isso da mesma maneira que você coletou dados de treinamento.
  - use os mesmos nomes e rótulos de propriedade.
  - não sobreponha os polígonos aos dados de treinamento; queremos dados independentes.
  - não exceda 5000 pixels.
  - coletar amostras das mesmas cinco classes, mas renomeia-as de forma diferente ('vUrbana', 'vFloresta', 'vArea_agricola_vegetada', 'vArea_agricola_solo', 'vAgua')

Por exemplo, coletando poligonos para representar os dados de validação dentro de vUrbana: 
![image](https://user-images.githubusercontent.com/41900626/184908712-6aabf151-31ad-4a55-aa9c-bb6e108485a2.png)


Configure a geometria (clicando no ícone da engrenagem), seguindo a ordem dos dados de treinamento. Por exemplo, para 'vUrbana', configure para 'FeatureCollection', clique em '+Property' e  adicione a propriedade 'Property' = 'Uso'  e 'Value' = '1' (para as demais classes escreva valor 2,3, 4 e 5).
Clique mais uma vez em '+Property' e  adicione 'Property' = 'Nome'  e 'Value' = 'vUrbana'

![image](https://user-images.githubusercontent.com/41900626/184953877-df13171b-7c4f-4394-ba07-49e15876f39a.png)


Nota: Lembre de ir salvando o código.

Ao final, você deve ter adquirido amostras de validação para as cinco classes temáticas:
![image](https://user-images.githubusercontent.com/41900626/184909657-6788ea15-d573-4ca7-a574-ee1e7e4eac60.png)


  

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

6. Essas metricas, na mioria dos casos, ficam melhor apresentadas em uma tabela. Encontre o arquivo 'matrizConfusao.csv' no seu Google Drive e organize os resultados da validação em uma tabela similar a apresentada abaixo:
![image](https://user-images.githubusercontent.com/41900626/184937282-f8496879-3135-4506-bbcf-f72956d3b7d6.png)
  
 Vamos à interpretação da tabela (a sua tabela terá, provavelmente, valores diferentes):
 
  Acurácia de Usuário (AC):
  Por exemplo, para a classe Área Urbana, temos uma acurácia de usuário AC=78%. Isso significa que, de todos os pixels que o classificador 'disse' (classificou) que  eram área urbana (37), ele acertou 29, ou seja, 29/37 = 78% dos casos. Ou seja, era área urbana e o classificador classificou como área urbana (_true positive_). Associado ao AC, temos o erro de comissão (ou complemento de AC); 
No exemplo, o classificador 'disse' que 8 amostras (7 de área agricola vegetada + 1 de de área agricola solo) eram Área Urbana quando, na verdade, eram outra classe (áreas agrícolas). Assim, 8/37= 22% foi o erro de comissão (que é o complemento de 78%) 

  Acurácia de Produtor (AP):
  De quantas amostras que eu indiquei que eram Área Urbana e, que, de fato, o algorítimo acertou como Área Urbana? No exemplo, indiquei um total de 30 amostras como Área Urbana (verdade de campo) e o algorítmo acertou 29, ou seja, 29/30 = 97%. O complemento, 100%-97% = 3% é o erro de omissão, ou seja, ele omitiu em 1/30 casos dizendo que essas 1 amostra não é Área Urbanda, quando, na relaidade, é área urbana.


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
No QGIS, clique com o botão direito em 'Classificada' > 'Simbologia' e separe e nomeie as classes conforme abaixo. Escolha as cores para as classes.
![image](https://user-images.githubusercontent.com/41900626/184946688-0f53723a-e705-41d8-bf32-946abba6cf20.png)








-------
### Obrigado

