## Código de Google Earth Engine

Recordar que el código posee elementos que se importaron pero que no aparecen acá, siendoe estos todas las geometrías, la imagen LANDSAT y la capa máscara "tempisque" para recortar el resultado.



<strong><h3>Código con el clasificador de smileCart</h1></strong>

´´´
var image = ee.Image(ee.ImageCollection('LANDSAT/LC08/C01/T1_SR')
    .filterBounds(roi)
    .filterDate('2020-01-01', '2020-02-28')
    .sort('CLOUD_COVER')
    .first());
Map.addLayer(image, {bands: ['B4', 'B3', 'B2'],min:0, max: 3000}, 'True colour image');

//Unir clases Fusionar.
var classNames = Bosque.merge(BodyWater).merge(Cultivo).merge(SueloDesnudo).merge(Nubes).merge(Urban).merge(Sombras_nubes);
print(classNames);
 
//Valores de reflectancia de las bandas
var bands = ['B2', 'B3', 'B4', 'B5', 'B6', 'B7'];

//Seleccionar bandas, así como las propiedades.
var training = image.select(bands).sampleRegions({
  collection: classNames,
  properties: ['landcover'],
  scale: 30
});
print(training);

//Se establece el algoritmo de clasificacion. 
var classifier = ee.Classifier.smileCart().train({ 
  features: training,
  classProperty: 'landcover', 
  inputProperties: bands
});

//Run the classification
var classified = image.select(bands).classify(classifier);
var final = classified.clip(tempisque);

//Display classification Map.center para centrar la vista.
Map.centerObject(classNames, 11);
Map.addLayer(final,
{min: 0, max: 6, palette: ['green', 'yellow', 'blue','brown','white','gray', "black"]},
'classification');


//----------------------Matriz Confusión-----------------------------------

var valNames = vbosque.merge(vBodyWater).merge(vcultivo).merge(vSueloDesnudo).merge(vNubes).merge(vUrban).merge(vSombras_Nubes);

// Validación
var validation = classified.sampleRegions({
  collection: valNames,
  properties: ['landcover'],
  scale: 30,
});
print(validation);

//
var testAccuracy = validation.errorMatrix('landcover', 'classification');

//
print('Validación de error matrix: ', testAccuracy);

//
print('Validación de precisión general: ', testAccuracy.accuracy());


//---------------------Gráficos firmas espectrales----------------
//Bandas para análisis, y el feature collection.
var subset = image.select('B[1-7]')
var samples = ee.FeatureCollection([BosqueP,CultivoP,BodyWaterP,SueloDesnudoP,NubesP, UrbanP,Sombras_nubesP]);

//Gráfico 1
var Chart1 = ui.Chart.image.regions(
 subset, samples, ee.Reducer.mean(), 10, 'Etiqueta')
 .setChartType('ScatterChart');
print(Chart1)

//Defina opciones de personalizado
var plotOptions = {
 title: 'Landsat-8 Surface reflectance spectra',
 hAxis: {title: 'Wavelength (nanometers)'},
 vAxis: {title: 'Reflectance'},
 lineWidth: 1,
 pointSize: 6,
 series: {
 0: {color: 'green'}, // Bosque
 1: {color: 'yellow'}, // Cultivo
 2: {color: 'blue'}, //BodyWater
 3: {color: 'brown'}, // SueloDesnudo
 4: {color: 'white'}, // Nubes
 5: {color: 'gray'}, // Urban
 6: {color: 'black'}, // Sombras_nubes
}};

/*Defina una lista de longitudes de onda Landsat-8 para las etiquetas del eje X. Esto 
ser revisa en los metadatos de la colección*/
var wavelengths = [443, 482, 562, 655, 865, 1609, 2201];

// Crear gráfico 2 
var Chart2 = ui.Chart.image.regions(
 subset, samples, ee.Reducer.mean(), 10, 'Etiqueta', wavelengths)
 .setChartType('ScatterChart')
 .setOptions(plotOptions);
 
// Desplegar el gráfico
print(Chart2);




<strong><h2>Código con el clasificador de randomForest</h2></strong>




// Recordar que el código posee elementos que se importaron pero que no aparecen acá, siendoe estos todas las geometrías, la imagen LANDSAT y la capa máscara "tempisque" para recortar el resultado.

var image = ee.Image(ee.ImageCollection('LANDSAT/LC08/C01/T1_SR')
    .filterBounds(roi)
    .filterDate('2020-01-01', '2020-02-28')
    .sort('CLOUD_COVER')
    .first());
Map.addLayer(image, {bands: ['B4', 'B3', 'B2'],min:0, max: 3000}, 'True colour image');
//Unir clases Fusionar.
var classNames = Bosque.merge(BodyWater).merge(Cultivo).merge(SueloDesnudo).merge(Nubes).merge(Urban).merge(Sombras_nubes);

print(classNames);
 
//Valores de reflectancia de las bandas
var bands = ['B2', 'B3', 'B4', 'B5', 'B6', 'B7'];

//select para que tome esas bandas, así como las propiedades de la clase
var training = image.select(bands).sampleRegions({
  collection: classNames,
  properties: ['landcover'],
  scale: 30
});
print(training);

// Establezca el algoritmo de clasificacion. 
var classifier = ee.Classifier.smileRandomForest(7).train({
  features: training,
  classProperty: 'landcover', 
  inputProperties: bands
});

//Run the classification
var classified = image.select(bands).classify(classifier);
var final= classified.clip(tempisque);

//Display classification Map.center para centrar la vista.
Map.centerObject(classNames, 11);
Map.addLayer(final,
{min: 0, max: 6, palette: ['green', 'yellow', 'blue','brown','white','gray', "black"]},
'classification');

//----------------------Matriz Confusión-----------------------------------

var valNames = vbosque.merge(vBodyWater).merge(vcultivo).merge(vSueloDesnudo).merge(vNubes).merge(vUrban).merge(vSombras_Nubes);

//
var validation = classified.sampleRegions({
  collection: valNames,
  properties: ['landcover'],
  scale: 30,
});
print(validation);

//
var testAccuracy = validation.errorMatrix('landcover', 'classification');

//
print('Validación de error matrix: ', testAccuracy);

//
print('Validación de precisión general: ', testAccuracy.accuracy());



<h2>Curvas Espectrales</h2>

![alt text](https://github.com/David-young99/TP4/blob/c2d3fe172aacbc35aa3e027ab81c2acfb1ff6464/ee-chart%20(2).png)
![alt text](https://github.com/David-young99/TP4/blob/209b218a4b001439a380a99db217482519f5251c/ee-chart%20(4).png)


<h2>Descripción de los clasificadores</h2>


<h3>1. SmileCart </h3>

<p style="text-align:center;"> SmileeCart, al ser un tipo de clasificación supervisada, necesita áreas preestablecidas, o áreas de entrenamiento de los tipos de cobertura, para poder ajustar de una manera correcta los parámetros que utiliza dicho clasificador. Dado a que este algoritmo logra identificar de manera automática, y etiquetar áreas las cuales presentan características similares a las áreas de entrenamiento.  
El clasificador smileCart pertenece al grupo CART el cual es un término en general que trata de referirse a las clasificaciones pertenecientes a un árbol de decisiones. En estos lo que se realiza es tomar o identificar las relaciones existentes entre variables de una una respuesta (continua) con variables explicativas (independientes), múltiples, continuos o discretas, donde por medio de procesos, las datos o valores de píxeles en este caso, se dividen repetitivamente, forman grupos cada vez homogéneos, donde se utilizan combinaciones de variables, las cuales cada vez se distinguen mejor la variación de la variable independiente.(Ying, et al. 2020)</p>


<h3>2. RandomForest </h3>

En cuanto al algoritmo para Machine Learning “Random Forest” es utilizado en variedad de fines, pero siempre dentro del campo de análisis de datos.

Este método utiliza árboles de decisión, los cuales vienen siendo como grupos de condicionales que determinan el resultado de la clasificación de un grupo de píxeles en este caso. Estos árboles van discriminando los píxeles que cumplen con una condición, en nuestro ejercicio viene siendo las clases o categorías de cobertura. Entonces como se ve en la figura de abajo , podríamos reemplazar las preguntas de “is red?” o “is underline?” por la reflectancia de cierto rango y la otra por colores de píxeles que nosotros le “enseñamos” a estos árboles, así es como categoriza cada píxel de la imagen en alguna de las categorías. 



![alt text](https://github.com/David-young99/TP4/blob/main/Captura%20de%20pantalla%202021-11-02%20153554.jpg)

Fuente de la decripción de random forest y la figura: Yiu. T. (2019)
<h2>Comparación entre los clasificadores</h2>

Comparación:

1. El RandomForest solicita un parámetro obligatorio, siendo este un Integer mayor a 0, el determina la cantidad de árboles de decisión con la cual trabajará el algoritmo, por otra parte los parámetros que pide el SmileCart son opcionales, pues en caso de no escribirlos automáticamente se asignan valores nulos o los números necesarios para trabajar.
2. En el resultado final, hay diferencias en la clasificación, en especial las clases de cultivos, suelo desnudo, bosque y cuerpos de agua, principalmente, o son más notorios los cambios en estas. Por ejemplo, en el SmileCart se muestran algunas zonas de bosque donde el RandomForest detecta en parte, cuerpos de agua.
3. Otra diferencia entre los clasificadores es: el SmileCart muestra con mayor facilidad zonas urbanas, en cambio el RandomForest muestra en especial los núcleos más densos de las zonas urbanas, conforme nos alejamos de este, las partes urbanas las detecta como suelo desnudo o zonas de cultivo.
4. La diferencia en la validación de la precisión general difiere entre los dos clasificadores, para el caso del SmileCart es de: 0.7571428571428571 y para el caso del RandomForest: 0.7285714285714285, siendo menor en el RandomForest, como se puede observar. Esto se puede deber por la clasificación final, al usar un método de clasificación diferente, los valores entre las geometrías de clasificación y las geometrías de validación cambian para ambos métodos. 


<h2>Matrices</h2>

1. Matriz del SmileCart:


![alt text](https://github.com/David-young99/TP4/blob/8a4b42bc170787eb320efc48f43d2cb0762ff4ed/Matriz1.png)

Validación de precisión general:
0.7571428571428571

2. Matriz del RandomForest:


![alt text](https://github.com/David-young99/TP4/blob/a7144305f9bf4c850073f6e708b0569f74dd6d56/Matriz2.png)

Validación de precisión general:
0.7285714285714285


<h2>Mapas finales con su respectiva simbología</h2>


1. Mapa final de clasificación mediante SmileCart en la cuenca del Tempisque, Guanacaste, Costa Rica.
![alt text](https://github.com/David-young99/TP4/blob/main/mapa2.png)
1. Mapa final de clasificación mediante RandomForest en la cuenca del Tempisque, Guanacaste, Costa Rica.
![alt text](https://github.com/David-young99/TP4/blob/main/mapa1.png)


<h2>Conclusiones</h2>

Se observa que el Machine Learning es una herramienta que facilita el procesamiento y análisis de datos, y no solo en Geografía, estas técnicas se pueden usar también para análisis de mercado, por ejemplo predecir el comportamiento de un consumidor, en base a lo que ha podido aprender de él anteriormente (compras realizadas, búsquedas entre otros, en conjunto al Big Data).

Pero en Geografía, tiene una aplicación inmediata al procesamiento de datos, por ejemplo en este ejercicio, donde una clasificación manual tomaría días poder dar esos resultados, la clasificación podría permitirnos detectar características que no son visibles a simple vista, medir la extensión del área de una cobertura, clasificar el uso del suelo, clasificar la cobertura vegetal, planificar la expansión urbana, entre muchas otras aplicaciones. 

Siendo más específicos al ejercicio, se rescata algunos puntos del random forest, por ejemplo, en comparación al otro método de clasificación, el random forest necesita un parámetro de tipo Integer (número entero) positivo y diferente a 0, para poder funcionar, o sea es obligatorio. Este parámetro hace referencia a el número de árboles de decisión que toma el algoritmo, como se explicó anteriormente. 

El clasificador smileCart, es un logaritmo de una buena precisión, siempre y cuando las áreas de estudio sean amplias, y en las cuales, las clases se encuentren bien distribuidas sobre esta área, en nuestro caso los valores de validación rondaron entre 75% y 72%, a pesar de ser valores buenos, se pueden mejorar aumentando la precisión de los puntos de validación (en este caso se realizaron a “vista” en la imagen satelital de estudio), estos tienden a realizarse en campo en áreas ya estudiadas y verificadas propiamente. 


<h2>Bibliografía</h2>

Tu, Ying & Lang, Wei & Yu, Le & Li, Ying & Junhao, Jiang & Qin, Yawen & Wu, Jiemin & Chen, Tingting & Xu, Bing. (2020). Improved Mapping Results of 10 m Resolution Land Cover Classification in Guangdong, China Using Multi-Source Remote Sensing Data with Google Earth Engine. IEEE Journal of Selected Topics in Applied Earth Observations and Remote Sensing. 13. 10.1109/JSTARS.2020.3022210.


Yiu. T. (2019). Understanding Random Forest. Towards data science. Recuperado de: https://towardsdatascience.com/understanding-random-forest-58381e0602d2
