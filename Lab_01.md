
# Sensoriamento Remoto
Lab 1 - Começando com Google Earth Engine (GEE)
--------------

### Agradecimentos
- Google Earth Engine Team
- Earth Engine Beginning Curriculum
- Prof. Shaun Levick do [GEARS](https://www.gears-lab.com)  Geospatial Ecology and Remote Sensing lab - 

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


## 1. The Earth Engine code editor

![Figure 1. O ambiente do GEE](https://github.com/geospatialeco/GEARS/blob/master/gee_editor.png)

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



1. Editor Panel
	- The Editor Panel is where you write and edit your Javascript code
2. Right Panel
	- Console tab for printing output.
	- Inspector tab for querying map results.
	- Tasks tab for managing long­ running tasks.
3. Left Panel
	- Scripts tab for managing your programming scripts.
	- Docs tab for accessing documentation of Earth Engine objects and methods, as well as a few specific to the Code Editor application
	- Assets tab for managing assets that you upload.
4. Interactive Map
	- For visualizing map layer output
5. Search Bar
	- For finding datasets and places of interest
6. Help Menu
	- User guide ­ reference documentation
	- Help forum ­ Google group for discussing Earth Engine
	- Shortcuts ­ Keyboard shortcuts for the Code Editor
	- Feature Tour ­ overview of the Code Editor
	- Feedback ­ for sending feedback on the Code Editor
	- Suggest a dataset. Our intention is to continue to collect datasets in our public archive
and make them more accessible, so we appreciate suggestions on which new datasets we should ingest into the Earth Engine public archive.

---------

## 2. Getting started with images

1. Navigate to Darwin and zoom in using the mouse wheel.

![Figure 2. Zoom to Darwin](navdarwin.png)


2. Clear the script workspace by selecting "Clear script" from the Reset button dropdown menu.

![Figure 3. Clear script](clearscript.png)

3. Search for “elevation” and click on the SRTM Digital Elevation Data 30m result to show the dataset description.

![Figure 4. Search for elevation data](elevsearch.png)

4. View the information on the dataset, and then click on Import, which moves the variable to the Imports section at the top of your script.

![Figure 4. View elevation datasource and import](importsrtm.png)

5. Rename the default variable name "image" to be "srtm".

![Figure 5. Rename image](renamesrtm.png)

6. Add the image object to the console by coping the script below into the code editor, and click "run" :

```JavaScript
print(srtm);
```
![Figure 6. Print SRTM](printsrtm.png)


7. Browse through the information that was printed to the console. Open the “bands” section to show the one band named “elevation”. Note that all this same information is automatically available for all variables in the Imports section.

![Figure 7. SRTM in console](bandssrtm.png)


8. Use the Map.addLayer() method to add the image to the interactive map. We will start simple, without using any of the optional parameters.

```JavaScript
Map.addLayer(srtm);
```

The displayed map will look pretty flat grey, because the default visualization parameters map the full 16­bit range of the data onto the black–white range, but the elevation range is much smaller than that in any particular location. We’ll fix it in a moment.

![Figure 8. Map SRTM](mapsrtm.png)

7. Select the Inspector tab. Then click on a few points on the map to get a feel for the elevation range in this area.

![Figure 8. Inspect SRTM](inspecsrtm.png)

8. Now you can set some more appropriate visualization parameters by adjusting the code as follows (units are in meters above sea level):

```JavaScript
Map.addLayer(srtm, {min: 0, max: 300});
```
![Figure 9. Visualise SRTM](vissrtm.png)

9. You will now be able to see variation in elevation range with low values in black and highest points in white. Layers added to the map will have default names like "Layer 1", "Layer 2", etc. To improve the readability, we can give each layer a human­-readable name, by adding a title with the syntax in the following code. Don't forget to click run.

```JavaScript
Map.addLayer(srtm, {min: 0, max: 300}, 'Elevation above sea level');
```
![Figure 10. Rename title](title2srtm.png)

10. Now the last step for today is to save your code, however before doing that it is good practice to add a some comment lines to your code reminding you of what you did and why. We add these with two forward slashes // :

```Javascript
// Print data details to console
print(srtm);

// Add the SRTM data to the interactive map
Map.addLayer(srtm)

// Add the data again, but with rescrited value ranges for better visualisation
Map.addLayer(srtm, {min: 0, max: 300})

// Add the data again, with value ranges, and a useful title for teh Layer tab
Map.addLayer(srtm, {min: 0, max: 300}, 'Elevation above sea level');
```
![Figure 11. Comment script](commentsrtm.png)

11. The next step is then to save you script by clicking "Save". It will be saved in your private repository, and will be accessible the next time you log in to Earth Engine.

![Figure 12. Comment script](savesrtm.png)

12. If you would like to experiment with different colour combinations, you can play with colour palettes as per the example below:

```Javascript
Map.addLayer(srtm, {min: 0, max: 300, palette: ['blue', 'yellow', 'red']}, 'Elevation above sea level');
```

![Figure 13. Colour scale elevation](coloursrtm.png)

13. For better visualisation we can create a hillshade view of the elevation data. Remember you can use the Layer transparency options to create draped images for colourised hillshades.

```JavaScript
var hillshade = ee.Terrain.hillshade(srtm);
Map.addLayer(hillshade, {min: 150, max:255}, 'Hillshade');
```

![Figure 14. Hillshade view](hillsrtm.png)


14. Slope works in a similar way:

```javascript
var slope = ee.Terrain.slope(srtm);
Map.addLayer(slope, {min: 0, max: 20}, 'Slope')
```

![Figure 15. Slope map](slopesrtm.png)
-------
### Thank you

I hope you found that useful. A recorded video of this tutorial can be found on my YouTube Channel's [Introduction to Remote Sensing of the Environment Playlist](https://www.youtube.com/playlist?list=PLf6lu3bePWHDi3-lrSqiyInMGQXM34TSV) and on my lab website [GEARS](https://www.gears-lab.com).

#### Kind regards, Shaun R Levick
------
