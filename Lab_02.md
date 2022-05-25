
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
1. Para este laboratório, usaremos uma imagem multiespectral coletada pelo satélite Sentinel-2 da Agência Espacial Européia. O Sentinel-2 é uma missão de imagem multiespectral de alta resolução e ampla faixa que apoia os estudos de Monitoramento de Terras do Copernicus, incluindo o monitoramento de vegetação, solo e cobertura de água, bem como observação de vias navegáveis interiores e áreas costeiras. Usaremos uma imagem coletada no Parque Nacional Iguaçu, PR.

2. Vamos navegar até a área de interesse copiando o código abaixo no Editor de Código e clicando em "_Run_" (Executar). Lembre-se de que a linha que começa com // é uma nota para nós mesmos e para os outros, e não é processada (chamamos isso de comentário). Os números entre parênteses são a longitude, latitude e nível de zoom (o intervalo é de 1 a 22). 





