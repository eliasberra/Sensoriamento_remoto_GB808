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

O primeiro passo é obter uma imagem sem nuvem para trabalhar. Faça isso importando imagens USGS Landsat 8 Surface Reflectance Tier 1, filtrando espacialmente para uma região de interesse (filterBounds), filtrando temporalmente para o intervalo de datas necessário (filterDate) e, por último, classificando por cobertura de nuvens ('CLOUD_COVER') e extraindo a cena menos nublada (first - primeira).

Com base na semana passada, podemos usar a ferramenta de desenho de ponto (ícone de lágrima) das ferramentas de geometria e desenhar um único ponto na região de interesse - vamos usar a cidade de Apucarana, PR, para este exemplo. Em seguida, clique na mãozinha para sair das ferramentas de desenho. Observe que uma nova variável é criada na seção de importações ('Imports'), contendo o ponto único, importado como uma Geometria. Altere o nome desta importação para "roi" - abreviação de região de interesse.

![image](https://user-images.githubusercontent.com/41900626/175110172-408ef3bd-8eb2-4e7e-8fdb-5cb37ac035f8.png)


Em seguida, podemos executar o script abaixo para extrair nossa imagem desejada da coleção Landsat 8 e adicioná-la à visualização do mapa como um composto de cores reais:

```JavaScript
var imagem = ee.Image(ee.ImageCollection('LANDSAT/LC08/C02/T1_L2') 
    .filterBounds(roi)
    .filterDate('2022-01-01', '2022-06-30')
    .sort('CLOUD_COVER')
    .first());   
Map.addLayer(imagem, {bands:['SR_B4', 'SR_B3', 'SR_B2'],min:6000, max: 10000}, 'Imagem cor verdadeira');
```
![image](https://user-images.githubusercontent.com/41900626/175113459-c2752997-5f80-4d75-8e56-5a1921977137.png)


Dê uma olhada ao redor da cena e familiarize-se com a paisagem. Você pode testar diferentes limiares de contraste ajustando o valor de min e max.



## Recortando a cena para uma área menor
Antes de prosseguirmos às etapas de classificação, vamos recortar nossa cena para uma área teste de menor tamanho a fim tornar mais rápida a etapa de classificação automática.
Passe o mouse na caixa 'Geometry Imports' ao lado das ferramentas de desenho de geometria e clique em '+ new layer'.![image](https://user-images.githubusercontent.com/41900626/175122195-9242e624-cf3b-4ab1-973c-1581a8aa0578.png)
Use a ferramenta de geometria 'Draw a rectangle' ![image](https://user-images.githubusercontent.com/41900626/175122290-343fa15c-cd83-4321-aaa5-56d152b35c76.png)
para desenhar um retângulo ao redor de Apucarana representando nossa área de interesse. Desenhe um retângulo com dimensões parecidas as apresentadas na figura abaixo. Renomei o vetor para 'recorte'.
![image](https://user-images.githubusercontent.com/41900626/175122567-8cd8085a-c78c-4543-9de3-7a11169b5153.png)
Clique novamente na mãozinha para sair do modo desenho.

Em seguida, podemos executar o script abaixo para extrair nossa imagem recortada:

```JavaScript
//Recortar cena
var img_recorte =  imagem.clip(recorte);
Map.addLayer(img_recorte, {bands:['SR_B4', 'SR_B3', 'SR_B2'],min:6000, max: 20000}, 'Recorte');
```
![image](https://user-images.githubusercontent.com/41900626/175123110-27fa7196-5fd9-4acd-931c-524d9fe1bc47.png)

Você pode habilitar/desabilitar qualquer um dos vetores na aba de 'Geometry Imports'.



## Coletando dados de treinamento
1. O primeiro passo para classificar nossa imagem é coletar alguns dados de treinamento para ensinar o classificador. Queremos coletar amostras representativas de espectros de refletância para cada classe de cobertura do solo de interesse.
2. Usando a cena livre de nuvens como orientação, passe o mouse na caixa 'Geometry Imports' ao lado das ferramentas de desenho de geometria e clique em '+ new layer' ![image](https://user-images.githubusercontent.com/41900626/175124178-fd317651-ebba-403b-bc2d-c9693a6698c9.png).

3. Cada nova camada (layer) a ser criada irá representar, agora, uma classe temática para o conjunto de dados de treinamento.
4. Vamos definir nossa primeira nova camada/classe como 'urbano'. Localize pontos representativos dessa camada em áreas urbanas ou edificadas (edifícios, estradas, estacionamentos, etc.) e clique para coletá-los adicionando pontos na camada de geometria.
5. Colete em torno de 30 pontos representativos e renomeie a 'geometria' como 'urbana'.
![image](https://user-images.githubusercontent.com/41900626/175125644-60957f00-c770-456b-8854-22a65140db6a.png)



5. Em seguida, você pode configurar a importação da geometria da classe urbana clicando no símbolo da engrenagem na mesma linha em que ela se encontra (![image](https://user-images.githubusercontent.com/41900626/175126735-c6bc88e9-c6b3-47b5-871f-342c364d8c16.png)
). Clique no ícone da engrenagem para configurá-lo, altere 'Import as' de 'Geometry' para 'FeatureCollection'. Use '+Property' para adicionar valores identificadores de cada cobertura de terra (nome = Cobertura_terra) e defina seu valor como 0. (As classes subsequentes serão 1, 2, 3 etc.) quando terminar, clique em 'OK'.

![image](https://user-images.githubusercontent.com/41900626/175126255-ab240ac8-9f5c-4653-8d1e-0c2cbdccbbf9.png)



6. Adicione mais 4 classes: 'floresta', 'area_agricola_cultivada' (área agrícola com cultivo evidente), 'area_agricola_solo' (área agrícola sem um cultivo evidente) e 'agua'. Colete as amostras de treinamento (~30).
7. Repita a etapa 5 para cada classe de cobertura do solo que deseja incluir em sua classificação, garantindo que os pontos de treinamento se sobreponham à imagem. Você pode achar mais interessante utilizar outras combinações de bandas para melhorar a fotointerpretação das classes na composição colorida. Lembre de usar a engrenagem para configurar as geometrias, alterando o tipo para FeatureCollection e definindo o nome da propriedade como Cobertura_terra com valores de 1, 2, 3 e 4 para as diferentes classes.
![image](https://user-images.githubusercontent.com/41900626/175130852-808b0b57-29d1-40de-8896-5520190547d7.png)



8. Agora temos cinco classes definidas ('urbano', 'floresta', 'area_agricola_cultivada', 'area_agricola_solo' e 'agua'), mas antes de podermos usá-las para coletar dados de treinamento, precisamos mesclá-las em uma única coleção, chamada FeatureCollection. Execute a seguinte linha para mesclar as geometrias em um único FeatureCollection:

```javascript
//mesclar as classes em uma única coleção
var nomeClasses = urbano.merge(floresta).merge(area_agricola_cultivada).merge(area_agricola_solo).merge(agua);
```

9. Imprima a coleção de recursos e inspecione os recursos.

```javascript
//Imprimir a coleção de feições
print(nomeClasses)
```
![image](https://user-images.githubusercontent.com/41900626/175132079-a751b042-a999-4b7a-8f34-119c1c713261.png)


## Crie os dados de treinamento

Agora podemos usar o FeatureCollection que criamos para detalhar a imagem e extrair os dados de refletância para cada ponto, de cada banda. Criamos dados de treinamento sobrepondo os pontos de treinamento na imagem. Isso adicionará novas propriedades à coleção de recursos que representam valores de banda de imagem em cada ponto:

```javascript
var bandas = ['B2', 'B3', 'B4', 'B5', 'B6', 'B7'];
var treinamento = image.select(bands).sampleRegions({
  coleção: classNames,
  propriedades: ['cobertura do solo'],
  escala: 30
});
imprimir(treinamento);
```

Depois de executar o script, os dados de treinamento serão impressos no console. Você notará que as informações de 'propriedades' agora mudaram e, além da classe de cobertura do solo, para cada ponto agora há um valor de refletância correspondente para cada banda da imagem.

![Figura 8. Imprimindo dados de treinamento](screenshots/l4_training.png)


## Treine o classificador e execute a classificação

Agora podemos treinar o algoritmo do classificador usando nossos exemplos de como são as diferentes classes de cobertura do solo de uma perspectiva multiespectral.

```javascript
var classificador = ee.Classifier.cart().train({
  características: treinamento,
  classProperty: 'landcover',
  propriedades de entrada: bandas
});
```

O próximo passo é aplicar esse conhecimento de nosso treinamento ao restante da imagem - o uso foi aprendido em nossa coleção supervisionada para informar decisões sobre a qual classe outros pixels devem pertencer.

```javascript
//Executa a classificação
var classificado = image.select(bandas).classify(classificador);
```

Exiba os resultados usando a função de mapeamento abaixo. Você pode precisar ajustar as cores, mas se os dados de treinamento foram criados com urbano=0, água=1, floresta=2 e agricultura=3 - então o resultado será renderizado com essas classes como amarelo, azul e verde, respectivamente.


```javascript
//Exibir classificação
Map.centerObject(classNames, 11);
Map.addLayer(classificado,
{min: 0, max: 3, paleta: ['red', 'blue', 'green','yellow']},
'classificação');
```



![Figura 9. Mapa classificado](screenshots/l4_classified.png)


## Examine seus resultados

Parabéns - sua primeira classificação de cobertura de terra! Mas.....
- Você está feliz com a classificação?
- Como poderia ser melhorado?
- Tente adicionar algumas classes extras para categorias de cobertura de terra que mostram sinais de confusão

Veremos como refinar isso e discutiremos limitações e caminhos para melhorias na próxima semana.

-------
### Obrigado

