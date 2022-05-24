
# Sensoriamento Remoto
Lab 1 - Começando com Google Earth Engine (GEE)
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

[JavaScript background](https://developers.google.com/earth-engine/tutorial\_js\_01)

------------------------------------------------------------------------

### Objetivo
---------
O objetivo deste laboratório é fornecer uma introdução ao ambiente de processamento do GEE. Ao final deste exercício, você será capaz de pesquisar, localizar e visualizar uma ampla variedade de conjuntos de dados de sensoriamento remoto. Começaremos com imagens de banda única - dados de elevação da missão SRTM.


## 1. O editor de código do GEE

![Fig. 1. O ambiente do GEE](https://github.com/geospatialeco/GEARS/blob/master/gee_editor.png)

1. Painel do Editor (_Code editor_)
	- O Painel do Editor é onde você escreve e edita seu código Javascript.
2. Painel Direito
	- Guia do console para saída de impressão.
	- Guia Inspetor para consultar os resultados do mapa.
	- Guia Tarefas para gerenciar tarefas de longa duração.
3. Painel Esquerdo
	- Aba Scripts: para gerenciar seus scripts de programação.
	- Aba Docs: para acessar a documentação dos objetos e métodos do Earth Engine, bem como alguns específicos do aplicativo Code Editor.
	- Aba Assets: Guia de dados geoespaciais que você criou e salvou, ou carregou.
4. Mapa Interativo
	- Para visualizar a saída das camadas do mapa
5. Barra de pesquisa
	- Para encontrar conjuntos de dados e locais de interesse.
6. Menu Ajuda
	- Documentação de referência do guia do usuário
	- Grupo de ajuda do fórum do Google para discutir o Earth Engine
	- Atalhos de teclado para o Editor de código
	- Visão geral do tour de recursos do Editor de código
	- Feedback para enviar feedback sobre o Code Editor
	- Sugerir um novo conjunto de dados.
---------


## 2. Primeiros passos com imagens

1. Navegue até Paranaguá, PR, e amplie usando a roda (_scroll_) do mouse.
![Fig. 2. Buscando Paranaguá na barra de buscas](https://user-images.githubusercontent.com/41900626/170104019-299b4d6f-7f92-45c7-8c9e-061c298de02e.png)
![Fig. 3. Zoom em Paranaguá](https://user-images.githubusercontent.com/41900626/170104512-6f226fa6-e6cd-4a4a-87ef-8c86b9f532b6.png)


2. Limpe a área de trabalho do script selecionando "_Clear script_" no menu do botão _Reset_.
![Fig. 4. Clear script](https://user-images.githubusercontent.com/41900626/170105037-c3ab0b65-e636-435a-9bf8-77e70edc7a59.png)


3. Procure por 'elevation' e clique em 'SRTM Digital Elevation Data Version 4' para apresentar a descrição do dado.
![Fig. 5. Procure por dado de elevação](https://user-images.githubusercontent.com/41900626/170119458-246c6372-4a70-43f1-a750-d30b33b02633.png)



4. Visualize as informações sobre o conjunto de dados e clique em _Import_, que move a variável para a seção _Imports_ na parte superior do seu script.
![Fig. 6. Importando o dado de elevação](https://user-images.githubusercontent.com/41900626/170119723-c6cfe967-6ae0-4538-acde-13732b7aa6db.png)



5. Renomeie o nome da variável padrão "image" para "srtm".
![Fig. 7. Renomeando imagem](https://user-images.githubusercontent.com/41900626/170119833-b7dc0a2f-40f5-4645-b130-fbd11131266e.png)



6. Adicione o objeto (imagem nesse caso) ao console copiando o script abaixo no editor de código e clique em _run_ :
```JavaScript
print(srtm);
```

7. Navegue pelas informações que foram impressas no _Console_. Abra a seção “bands” para mostrar uma banda chamada “elevation”. Observe que todas essas mesmas informações estão automaticamente disponíveis para todas as variáveis na seção _Imports_.
![Fig. 8. SRTM no _Console_](https://user-images.githubusercontent.com/41900626/170120016-fd9c8243-b4b4-4a49-bb79-6654923b233f.png)



8. Use o método Map.addLayer() para adicionar a imagem ao mapa interativo. Começaremos simples, sem usar nenhum dos parâmetros opcionais.
```JavaScript
Map.addLayer(srtm);
```
O mapa exibido ficará bem cinza, porque os parâmetros de visualização padrão mapeiam toda a faixa de 16 bits dos dados na faixa preto-branco, mas o intervalo de elevação é muito menor do que isso em qualquer local específico. Vamos corrigi-lo em um momento.
![Fig. 9. Mapa SRTM](https://user-images.githubusercontent.com/41900626/170120188-8dacbe1b-f839-4cb6-a452-1926bd4b66d2.png)



9. Selecione a aba _Inspector_ (observe que o ponteiro do mouse assume a forma de um +). Em seguida, clique em alguns pontos no mapa para ter uma ideia dos valores da faixa de elevação nesta área.
![Fig. 10. Inspecionando o SRTM](https://user-images.githubusercontent.com/41900626/170120421-15e4fe4f-b1fe-4a2d-86ea-53eede22ba1c.png)


10. Agora você pode definir alguns parâmetros de visualização mais apropriados ajustando o código da seguinte forma (lembre-se, as unidades estão em metros acima do nível do mar):
```JavaScript
Map.addLayer(srtm, {min: 0, max: 2000});
```
![Fig. 11. Melhorando visualização do SRTM](https://user-images.githubusercontent.com/41900626/170120679-3ea69a9b-bd1c-46cf-a1ef-c7b2af8a47fc.png)


11. Agora você poderá ver a variação na faixa de elevação com valores baixos em preto e pontos mais altos em branco. As camadas adicionadas ao mapa terão nomes padrão como "Layer 1", "Layer 2", etc. Para melhorar a legibilidade, podemos dar a cada Layer um nome mais significativo, adicionando um título com a sintaxe no código a seguir. Não se esqueça de clicar em _run_.

```JavaScript
Map.addLayer(srtm, {min: 0, max: 300}, 'Elevação acima do nível do mar');```
![Fig. 12. Renomeando o título](https://user-images.githubusercontent.com/41900626/170122633-90a47204-6aec-4f7c-a23f-b451200d1e4f.png)


12. Agora, o último passo de hoje é salvar seu código, no entanto, antes de fazer isso, é uma boa prática adicionar algumas linhas de comentário ao seu código, lembrando-o do que você fez e por quê. Nós os adicionamos com duas barras //:

```Javascript
// Imprima metadados no console. 
print(srtm);

// Adicione o dado SRTM no mapa base interativo.
Map.addLayer(srtm)

// Adicione os dados novamente, mas com intervalos de valores rescritos para melhor visualização.
Map.addLayer(srtm, {min: 0, max: 2000})

// Adicione os dados novamente, com intervalos de valores e um título útil para a guia Layer
Map.addLayer(srtm, {min: 0, max: 2000}, 'Elevação acima do nível do mar');
```
![Fig. 13. Comentando o script](https://user-images.githubusercontent.com/41900626/170123437-f64ad028-ab99-48eb-95f3-5ea255b790d0.png)



13. O próximo passo é salvar seu script clicando em "_Save_" > "_Save as_". Ele será salvo em seu repositório privado e estará acessível na próxima vez que você fizer login no Earth Engine.
![Fig. 14. Salvando script](https://user-images.githubusercontent.com/41900626/170123987-def5648f-f2e9-42eb-842a-5d4804973e98.png)



14. Se você quiser experimentar diferentes combinações de cores, pode brincar com as paletas de cores conforme o exemplo abaixo:
```Javascript
Map.addLayer(srtm, {min: 0, max: 300, palette: ['blue', 'yellow', 'red']}, 'Elevação acima do nível do mar');
```
![Fig. 15. Paleta de cores](https://user-images.githubusercontent.com/41900626/170124951-9cb41e1b-468a-423b-9184-678084d71714.png)


15. Para uma melhor visualização, podemos criar uma visualização em sombreamento (_hillshade_) dos dados de elevação. Você pode usar as opções de transparência de camada para criar imagens para sombras coloridas.
```JavaScript
var hillshade = ee.Terrain.hillshade(srtm);
Map.addLayer(hillshade, {min: 150, max:255}, 'Hillshade');
```
![Fig. 16. Mapa de sombreamento '_Hillshade_'](https://user-images.githubusercontent.com/41900626/170126914-a1fc7be9-d9d4-4ca9-ac22-0ce05a04dffc.png)


16. A inclinação (_slope_) funciona de maneira semelhante:
```javascript
var slope = ee.Terrain.slope(srtm);
Map.addLayer(slope, {min: 0, max: 20}, 'Slope')
```
![Fig. 17. Mapa de inclinação](https://user-images.githubusercontent.com/41900626/170127593-80bbb245-c86d-4f11-959e-cc259d186cd4.png)


-------
### Obrigado
Espero que esse tutorial tenha sido útil.  


#### Atenciosamente, Elias F Berra
------
