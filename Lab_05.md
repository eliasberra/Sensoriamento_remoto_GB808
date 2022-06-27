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

9. Colete dados de validação usando a ferramenta de geometria de polígono retangular ![image](https://user-images.githubusercontent.com/41900626/175937417-b8d465e1-d5af-48e2-b5ba-958bd98e2e09.png):
  - faça isso da mesma maneira que você coletou dados de treinamento.
  - use os mesmos nomes e rótulos de propriedade.
  - não sobreponha os polígonos aos dados de treinamento (pontos).
  - não exceda 5000 pixels.
  - coletar exemplos das mesmas cinco classes, mas renomeia-as de forma diferente (vUrbana, vCity, vForest, vOther)

Por exemplo, coletando poligonos para representar os dados de validação para vUrbana
  -'floresta', -'area_agricola_vegetada' (área agrícola com cultivo evidente), -'area_agricola_solo' (área agrícola sem um cultivo evidente; solo exposto) e, -'agua'.

10. Mescle seus polígonos de validação em uma coleção de recursos

```JavaScript
//Mesclar em um FeatureCollection
var valNames = vWater.merge(vCity).merge(vForest).merge(vOther);
```

11. Amostra seus resultados de classificação para suas novas áreas de validação

```JavaScript
var validação = classificado.sampleRegions({
  coleção: valNames,
  propriedades: ['cobertura do solo'],
  escala: 30,
});
imprimir(validação);
```

12. Execute a avaliação de validação usando a abordagem de matriz de erros

```JavaScript
//Compara a cobertura do solo de seus dados de validação com o resultado da classificação
var testAccuracy = validation.errorMatrix('landcover', 'classification');
// Imprime a matriz de erro no console
print('Matriz de erro de validação: ', testAccuracy);
// Imprime a precisão geral no console
print('Precisão geral da validação: ', testAccuracy.accuracy());
```
13. Exporte sua matriz de erro para análise posterior para calcular a precisão de classe individual e a precisão do usuário e do produtor.

-------
### Obrigado
