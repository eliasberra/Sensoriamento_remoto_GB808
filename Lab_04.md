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

O objetivo deste laboratório é entender o processo de classificação de imagens e explorar formas de transformar imagens de sensoriamento remoto em mapas de cobertura do solo.

----------

## Carregando a imagem

O primeiro passo é obter uma imagem sem nuvem para trabalhar. Faça isso importando imagens USGS Landsat 8 Surface Reflectance Tier 1, filtrando espacialmente para uma região de interesse (filterBounds), filtrando temporalmente para o intervalo de datas necessário (filterDate) e, por último, classificando por cobertura de nuvens ('CLOUD_COVER') e extraindo o menor cena nublada (primeiro).

Com base na semana passada, podemos usar a ferramenta de desenho de ponto (ícone de lágrima) das ferramentas de geometria e desenhar um único ponto na região de interesse - vamos usar a cidade de Cairns para este exemplo. Em seguida, 'Sair' das ferramentas de desenho. Observe que uma nova variável é criada na seção de importações, contendo o ponto único, importado como uma Geometria. Altere o nome desta importação para "roi" - abreviação de região de interesse.


![Figura 1. Navegando para Cairns](l4_cairns.png)

Em seguida, podemos executar o script abaixo para extrair nossa imagem desejada da coleção Landsat 8 e adicioná-la à visualização do mapa como um composto de cores reais:

```JavaScript
var imagem = ee.Image(ee.ImageCollection('LANDSAT/LC08/C01/T1_SR')
    .filterBounds(roi)
    .filterDate('2016-05-01', '2016-06-30')
    .sort('CLOUD_COVER')
    .primeiro());
Map.addLayer(image, {bandas: ['B4', 'B3', 'B2'],min:0, max: 3000}, 'True color image');
```

![Figura 2. Adicionando imagem à visualização do mapa](l4_layers.png)

Dê uma olhada ao redor da cena e familiarize-se com a paisagem. Você notará que a imagem é bastante escura - podemos ajustar o brilho/contraste usando as rodas de configurações para a camada que criamos na guia Camadas. Deslize o ajustador Gamma ligeiramente para a direita (de 1,0 a 1,4) para aumentar o brilho da cena.

![Figura 3. Ajuste de brilho](l4_gamma.png)

## Coletando dados de treinamento
1. O primeiro passo para classificar nossa imagem é coletar alguns dados de treinamento para ensinar o classificador. Queremos coletar amostras representativas de espectros de refletância para cada classe de cobertura do solo de interesse.
2. Usando a cena livre de nuvens como orientação, passe o mouse na caixa 'Importações de geometria' ao lado das ferramentas de desenho de geometria e clique em '+ nova camada'.
3. Cada nova camada representa uma classe dentro dos dados de treinamento, por exemplo, 'urbano'.
4. Deixe a primeira nova camada representar 'urbano'. Localize pontos na nova camada em áreas urbanas ou edificadas (edifícios, estradas, estacionamentos, etc.) e clique para coletá-los 9adicionando pontos na camada de geometria.
5. Colete 25 pontos representativos e renomeie a 'geometria' como 'urbana'.

![Figura 4. Crie e colete a classe urbana](screenshots/l4_urban.png)


5. Em seguida, você pode configurar a importação de geometria urbana (roda dentada, parte superior do script na seção de importações) da seguinte forma. Clique no ícone da roda dentada para configurá-lo, altere 'Importar como' de 'Geometria' para 'FeatureCollection'. Use a cobertura de terra 'Adicionar propriedade' e defina seu valor como 0. (As classes subsequentes serão 1, 2, 3 etc.) quando terminar, clique em 'OK'.

![Figura 5. A caixa de diálogo de geometria](screenshots/l4_cog.png)


6. Repita a etapa 5 para cada classe de cobertura do solo que deseja incluir em sua classificação, garantindo que os pontos de treinamento se sobreponham à imagem. Adicione 'água', 'floresta' e 'agricultura' em seguida - coletando 25 pontos para cada um. Use a roda dentada para configurar as geometrias, alterando o tipo para FeatureCollection e definindo o nome da propriedade como landcover com valores de 1, 2 e 3 para as diferentes classes.

![Figura 6. Adicionando classes](screenshots/l4_classes.png)

7. Agora temos quatro classes definidas (urbana, água, floresta, agricultura), mas antes de podermos usá-las para coletar dados de treinamento, precisamos mesclá-las em uma única coleção, chamada FeatureCollection. Execute a seguinte linha para mesclar as geometrias em um único FeatureCollection:

```javascript
var classNames = urban.merge(água).merge(floresta).merge(agricultura);
```

8. Imprima a coleção de recursos e inspecione os recursos.

```javascript
print(classNames)
```
![Figura 7. Classes de impressão](screenshots/l4_printclass.png)


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

