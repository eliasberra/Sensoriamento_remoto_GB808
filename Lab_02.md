
# Sensoriamento Remoto
Lab 2 - Compreendendo combinações de bandas e visualização de imagens
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
O objetivo deste laboratório é fortalecer sua compreensão dos princípios de visualização de imagens e desenvolver habilidades práticas no mapeamento de combinações de bandas e na exploração das propriedades de reflectância dos elementos da superfície terrestre.


### Carregando uma imagem multiespectral Sentinel-2
---------
1. Para este laboratório, usaremos uma imagem multiespectral coletada pelo satélite Sentinel-2 da Agência Espacial Européia. O Sentinel-2 é uma missão de imagem multiespectral de alta resolução e ampla faixa que apoia os estudos de Monitoramento de Terras do Copernicus, incluindo o monitoramento de vegetação, solo e cobertura de água, bem como observação de vias navegáveis interiores e áreas costeiras. Usaremos uma imagem coletada no no noroeste do Paraná, sobre Porto Rico.

2. Vamos navegar até a área de interesse copiando o código abaixo no Editor de Código e clicando em "_Run_" (Executar). Lembre-se de que a linha que começa com // é uma nota para nós mesmos e para os outros, e não é processada (chamamos isso de comentário). Os números entre parênteses são a longitude, latitude e nível de zoom (o intervalo é de 1 a 22). 

```Javascript
//Navegue até a área de intresse
Map.setCenter(132.5685, -12.6312, 8);
```
![image](https://user-images.githubusercontent.com/41900626/171919936-de4802ae-6f9a-466b-90e6-5a9187975466.png)

3. Agora que estamos no lugar certo, vamos escolher uma imagem do Sentinel-2 usando o código abaixo. Copie e cole no Editor de Código e clique em "Run". Copernicus refere-se à missão do satélite, S2 é a abreviação de Sentinel-2, e o número longo 20180422T012719_20180422T012714_T52LHM refere-se a uma imagem específica, definida por uma data, hora e um caminho e linha da órbita do satélite. Escolhi uma única imagem para os propósitos deste laboratório, mas abordaremos a pesquisa de imagens para áreas e datas específicas em um estágio posterior.

```Javascript
// `Selecione uma imagem específica do arquivo Sentinel-2
var sent2 = ee.Image("COPERNICUS/S2_SR/20220125T134211_20220125T134211_T21KZQ");
``

4. Se o código não retornou nenhum erro, a imagem foi encontrada com sucesso no arquivo. Para verificar novamente, vamos executar a linha abaixo para imprimir as informações da imagem (metadados) no Console. Depois que as informações forem carregadas no Console, você poderá clicar nas pequenas setas suspensas ao lado de "Imagem" e "bandas" para ver mais detalhes sobre a estrutura da banda e o formato de nomenclatura.

```Javascript
// Imprima os detalhes da imagem no Console
print(sent2);
```
![image](https://user-images.githubusercontent.com/41900626/171924580-4d4503b0-11c4-451c-875c-9239a9722dd8.png)


5. Podemos ver nas informações do Console que a imagem contém várias bandas, chamadas B1, B2, B3 etc. Para descobrir quais comprimentos de onda essas bandas representam, vamos usar a barra de pesquisa para obter mais informações. Digite "Sentinel-2" na barra de pesquisa e você o verá na lista de resultados.

![image](https://user-images.githubusercontent.com/41900626/171925748-c02b2e81-0eb1-4279-8026-ac3e5eee4cbd.png)

6. Clique em "Sentinel-2 MSI: MultiSpectral Instrument, Level-2A" para abrir o painel de informações. A tabela fornecida é muito útil para obter uma visão geral rápida das bandas disponíveis, seus comprimentos de onda e resoluções espaciais.

![image](https://user-images.githubusercontent.com/41900626/171925955-3d213e36-7085-40a0-b226-2c158b67befc.png)

7. Agora, antes de prosseguirmos, salve seu script atual clicando no menu suspenso do botão 'Save' e selecionando "Save as". Salve-o no repositório do seu curso para que você possa voltar a ele a qualquer momento e de qualquer dispositivo com um navegador da web.


8. Voltando à nossa imagem, as bandas 2,3 e 4 são as bandas do azul, verde e vermelho, respectivamente. Portanto, se desejarmos visualizar uma composição em cores verdadeiras da imagem (composição RGB), precisamos colocar a Banda 4 no canal vermelho (R), a Banda 3 no canal verde (G) e a Banda 2 no canal azul (B). Podemos fazer isso com o código abaixo - observe cuidadosamente a sintaxe para especificar o arranjo da banda.


```Javascript
//Composição em cores verdadeiras
Map.addLayer(sent2,{bands:['B4','B3','B2']});
```
![image](https://user-images.githubusercontent.com/41900626/171928440-c9a7eb9d-6069-4178-aad2-921c36cf92c6.png)

9. Depois de executar a linha de código anterior, podemos ver uma imagem sendo carregada no visualizador de mapas, mas está completamente escura. Isso ocorre porque não especificamos nenhum parâmetro de visualização (contraste da imagem). Os valores de refletância para produtos Sentinel-2 variam de 0 a ~3000 nessa cena, então vamos especificar isso em nosso código como mostrado abaixo (observando que todos os parâmetros de visualização estão dentro dos colchetes {}):
```Javascript
Map.addLayer(sent2,{bands:['B4','B3','B2'], min:0, max:3000});
```
![image](https://user-images.githubusercontent.com/41900626/171930090-a6a016da-fa2d-4505-b4b4-1698b8fea372.png)

10. Essa nova composição não te parece melhor? Esta é uma visão semelhante ao que veríamos olhando pela janela de um avião - e é por isso que a chamamos de composição de cores verdadeiras. Todas as três bandas usadas na criação deste composto ocorrem na porção visível do espectro eletromagnético.

11. Aumente um pouco mais o zoom usando a roda do mouse. Essas imagens são um recurso fantástico para mapeamento e monitoramento ambiental. As bandas do espectro visível estão na resolução espacial de 10 m, e o tempo de revisita da constelação de satélites é a cada 6 dias nesta região. Obrigado ESA!

![image](https://user-images.githubusercontent.com/41900626/171930848-293d32fb-b4b6-4a9f-98b2-b2e622bc3f0d.png)

12. Antes de prosseguirmos, vamos limpar um pouco nosso código e adicionar comentários. Vamos dar títulos às camadas na visualização do mapa para que possamos saber qual é qual na guia de camada. Podemos colar as linhas abaixo sobre as duas anteriores.

```Javascript
// Adiciona uma composição RGB em cores verdadeiras ao mapa, sem contraste
Map.addLayer(sent2,{bandas:['B4','B3','B2']}, "Preto");

// Adiciona uma composição RGB em cores verdadeiras ao mapa, com contraste
Map.addLayer(sent2,{bands:['B4','B3','B2'], min:0, max:3000}, "Cores verdadeiras");
```
![image](https://user-images.githubusercontent.com/41900626/171932283-072feeb0-3ddd-466e-b4da-9662888dd0d7.png)


13. Se olharmos para a tabela de comprimentos de onda do Sentinel-2, podemos ver que a Banda 8 está no espectro NIR (infravermelho próximo). Portanto, para mapear uma composição RGB em falsa cor, precisamos colocar a Banda 8 no canal vermelho (R), mover a Banda 4 para o canal verde (G) e mover a Banda 3 para o canal azul (B). A imagem resultante agora mostra a vegetação fotossinteticamente ativa em vermelho vibrante.

```Javascript
// Adicione uma composição RGB ao mapa, usando a banda NIR como a cor falsa
Map.addLayer(sent2,{bands:['B8','B4','B3'], min:0, max:3000}, "Cor falsa");
```
![image](https://user-images.githubusercontent.com/41900626/171933695-53357dc5-f07c-4fad-b9b8-f662de8bdac2.png)


14. Agora você pode navegar pela cena e alternar entre as visualizações de cores verdadeiras e cores falsas usando a guia de camadas (Layers). Observe atentamente como as diferentes partes da cena são representadas nessas diferentes visualizações - e explore como alguns elementos da paisagem, como clareiras florestais, saltam mais claramente na composição de cores falsas.



15. Para realmente construir sua compreensão de como diferentes comprimentos de onda interagem com superfícies, agora vamos carregar bandas individuais sequencialmente, de comprimentos de onda mais curtos a mais longos. Para exibir a Banda 1, o código é o seguinte:

```JavaScript
// Adicione a Banda 1 ao mapa
Map.addLayer(sent2,{bands:['B1'], min:0, max:3000}, "B1");
```
![image](https://user-images.githubusercontent.com/41900626/172051074-b52cbe61-1b02-4ab9-8d6f-bb8a007e72a7.png)


16. Para exibir mais as bandas individualmente, o código é o mesmo, por ex.:
```JavaScript
// Adicione mais algumas bandas ao mapa
Map.addLayer(sent2,{bands:['B4'], min:0, max:3000}, "B4");
Map.addLayer(sent2,{bands:['B8'], min:0, max:3000}, "B8");
Map.addLayer(sent2,{bands:['B12'], min:0, max:3000}, "B12");
```
17. Use a guia de camadas (Layers) para ativar e desativar as bandas na visualização de exibição do mapa. Observe quais elementos da paisagem aparecem mais claros e mais escuros à medida que você altera o número da banda (e, portanto, passa de comprimentos de onda mais curtos para mais longos).


------
### Exercício prático

1. Carregue outra imagem do Sentinel-2 para a mesma área de interesse. O código abaixo contém o identificador de imagem para uma imagem coletada em agosto de 2018.


```JavaScript
// Selecione uma imagem específica do Sentinel-2 do arquivo
var sent2inv = ee.Image("COPERNICUS/S2_SR/20220125T134211_20220125T134211_T21KZQ");
```
2. Visualize esta imagem em cores verdadeiras e cores falsas, usando a composição RGB.

3. Compare a imagem de agosto com a de janeiro de 2022. O que mudou e por quê?

4. Até agora, exploramos apenas duas opções de visualização (cores verdadeiras e cores falsas), mas existem muitas outras combinações RGB possíveis:
     - Cor natural: 4 3 2
     - Infravermelho de cor falsa: 8 4 3
     - Falsa cor urbana: 12 11 4
     - Agricultura: 11 8 2
     - Penetração atmosférica: 12 11 8A
     - Vegetação saudável: 8 11 2
     - Terra/Água: 8 11 4
     - Cores naturais com remoção atmosférica: 12 8 3
     - Infravermelho de ondas curtas: 12 8 4
     - Análise de vegetação: 11 8 4

5. Experimente as combinações listadas acima e pense no motivo pelo qual podemos querer usá-las.

6. Use esta imagem ("COPERNICUS/S2_SR/20200127T131241_20200127T131235_T23KLP") para explorar um cenário mais urbano na região metropolitana da cidade de São Paulo.

-------
### Obrigado
Espero que tenha achado esse tutorial útil.
Um vídeo gravado do tutorial original, em inglês, pode ser encontrado no canal do YouTube do Prof. Shaun Levick (https://www.youtube.com/playlist?list=PLf6lu3bePWHDi3-lrSqiyInMGQXM34TSV).
