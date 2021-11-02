## Código de Google Earth Engine

// Recordar que el código posee elementos que se importaron pero que no aparecen acá, siendoe estos todas las geometrías, la imagen LANDSAT y la capa máscara "tempisque" para recortar el resultado.



<h1>Código con el clasificador de smileCart</h1>
<p></p>
<p></p>
<p></p>
<p></p>

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




### Código con el clasificador de randomForest




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



