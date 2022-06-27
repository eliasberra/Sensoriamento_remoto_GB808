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
  - coletar exemplos das mesmas cinco classes, mas renomeia-as de forma diferente ('vUrbana', 'vFloresta', 'vArea_agricola_vegetada', 'vArea_agricola_solo', -'vAgua')

Por exemplo, coletando poligonos para representar os dados de validação dentro de vUrbana: 
![image](https://user-images.githubusercontent.com/41900626/175941484-d0724dc3-8b75-4618-b038-a004948a6f60.png)

Configure a geometria (clicando no ícone da engrenagem), seguindo a ordem dos dados de treinamento. Por exemplo, para 'vUrbana', configure para 'FeatureCollection', adicione a propriedade 'cobertura_terra' e valor = 0 (para as demais classes escreva valor 1,2,3 e 4).
![image](https://user-images.githubusercontent.com/41900626/175942534-a534d5b4-a343-445e-a86a-5896eaeb8bab.png)

Nota: Lembre de ir salvando o código.
Ao final, você deve ter adquirido amsotras para as cinco classes de validação:
![image](https://user-images.githubusercontent.com/41900626/175948116-243de285-0749-4d6f-97dd-c2111f749dae.png)

  

10. Mescle seus polígonos de validação em uma coleção de recursos

```JavaScript
//--------------Validação--------------------------------

//mesclar os polígonos de validação
var nomeClasses = vUrbana.merge(vFloresta).merge(vArea_agricola_vegetada).merge(vArea_agricola_solo).merge(vAgua);
```

11. Amostra seus resultados de classificação para suas novas áreas de validação

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


12. Execute a avaliação de validação usando a abordagem de matriz de erros

```JavaScript
//Compare a cobertura da terra de seus dados de validação contra o resultado da classificação
var testeAccuracia = validacao.errorMatrix('cobertura_terra', 'classification');
// Imprime a matriz de erro no console
print('Matriz de erro de validação: ', testeAccuracia);
// Imprime a precisão geral no console
print('Acurácia geral da validação: ', testeAccuracia.accuracy());
```
![image](https://user-images.githubusercontent.com/41900626/175952844-d2b46b79-14f7-4e71-9410-72eb0f45f992.png)

13. Exporte sua matriz de erro para análise posterior para calcular a precisão de classe individual e a precisão do usuário e do produtor.

-------
### Obrigado
