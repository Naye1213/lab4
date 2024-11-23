# Laboratorio 4

## Desarrollo

Este es un bloque de codigo para cargar una capa tipo poligono en forma de roi.
JavaScript:
  type: programming
  tm_scope: source.js
  ace_mode: javascript
  codemirror_mode: javascript
  codemirror_mime_type: text/javascript
  color: "#f1e05a"
  aliases:
  - js
    
<details>
  <summary>Clic</summary>
  
``` js
var roi = ee.FeatureCollection('projects/mtb2023-399203/assets/Palo_verde');
Map.addLayer(roi, {color: 'green'}, 'ROI');
Map.centerObject(roi, 12)
```
</details>


Este es un bloque de codigo para cargar la colección de imagenes en este caso de Sentinel-1, que este es de SAR.

<details>
  <summary>Clic</summary>
  
``` js
//Coleccion de imagenes de Sentinel-1
var s1 = ee.ImageCollection('COPERNICUS/S1_GRD')
        //.filter(ee.Filter.listContains('transmitterReceiverPolarisation', 'VV','VH'))
        .filter(ee.Filter.eq('instrumentMode', 'IW'))
        .filter(ee.Filter.eq('orbitProperties_pass', 'DESCENDING')) // puede ajustar a ASCENDING
        .filterBounds(roi)
```
</details>


En este caso ambos conjunto de codigos son para filtrar imagenes por fecha, y tenemos 2 ya que queremos imagenes de una fecha antes e imagenes de una fecha despues del incendio.

<details>
  <summary>Clic</summary>
  
``` js
// Filtro de imagenes por fecha
var beforeinc = s1.filterDate('2023-04-01', '2023-04-28')
print(beforeinc,'imagenes disponibles antes del incendio')
/* puede observar que para este rango de fechas tenemos 2 imagenes disponibles
aunque del mismo día
*/
//Imagenes luego del incendio
var afterinc = s1.filterDate('2023-05-10', '2023-06-01')
print(afterinc,'imagenes disponibles despues del incendio')
```
</details>


Aqui en este bloque lo que se esta haciendo es pasando toda la colección de imagenes que tenemos de las fechas deseadas a una sola imgen. En la parte inferior donde sale Map.addLayer, eso es para poder vizualizar en el mapa una de las imagenes que acabamos de transformar.

<details>
  <summary>Clic</summary>
  
``` js
// pasemos de un ImageCollection a un Image
var beforeinc = beforeinc.mosaic().clip(roi) //puedes cambiar mosaic por mean or median
var afterinc =  afterinc.mosaic().clip(roi)
print(beforeinc, 'imagen antes del incendio')
print(afterinc, 'imagen despues del incendio')

Map.addLayer( beforeinc,{bands: ['VV'], min: -15, max: -5, gamma: 1.2},  1.2}, 'antes del incendio sin speckle', 0);
```
</details>


En este bloque lo que hacemos es corregir el ruido que hay en las imagenes SAR.

<details>
  <summary>Clic</summary>
  
``` js
//filtro para reducir el speckle (pixeles de colores aleatorios)
var SMOOTHING_RADIUS = 50;
var beforeinc = beforeinc.focal_mean(SMOOTHING_RADIUS, 'circle', 'meters');
var afterinc = afterinc.focal_mean(SMOOTHING_RADIUS, 'circle', 'meters');
```
</details>


En este otro bloque se colocan unos parametros para poder vizualizar las imagenes SAR

<details>
  <summary>Clic</summary>
  
``` js
//Parametros de visualizacion
var visualization = {
  bands: ['VH'],  // podemos ajustar la banda a VV
  min: -20,
  max: -5,
};
```
</details>


Aqui vizualizamos en el mapa las imagenes con las correciones y demás que hemos hecho.

<details>
  <summary>Clic</summary>
  
``` js
//DVisualicemos las imagenes
Map.addLayer( beforeinc,visualization, 'antes del incendio',0);
Map.addLayer(afterinc, visualization, 'despues del incendio',0);
```
</details>


En este bloque se combinan las bandas que contengan las dos imagenes que tenemos y de esa manera se crea una nueva imagen combinada

<details>
  <summary>Clic</summary>
  
``` js
//Unamos las bandas del antes y despues en un solo image
var coll = beforeinc.addBands(afterinc)
print(coll, 'coleccion junta')

Map.addLayer(coll,imageVisParam, 'Sentinel-1')
```
</details>


Aqui lo que se hace es seleccionar dos bandas de las imagenes y se calcula una nueva banda que se agrega a la colección de imagenes, y ya luego de pone a vizualizar el resultado en el mapa.

<details>
  <summary>Clic</summary>
  
``` js
var change = coll.expression ('VH / VH_1', {
    'VH': coll.select ('VH'),  // ajuste las bandas como considere
    'VH_1': coll.select ('VH_1')})
    .toDouble().rename('change');

Map.addLayer(change, {min: 0,max:2},'Raster de cambio', 0);
print(change, 'cambio')
```
</details>


Aqui empezamos a trabajar con Sentinel-2, por lo que primero hacemos el emascaramiento de nubes para las imagenes del Sentinel-2.

<details>
  <summary>Clic</summary>
  
``` js
// Sentinel-2 cloud masking
function cloudMask(image){
  var scl = image.select('SCL');
  var mask = scl.eq(3).or(scl.gte(7).and(scl.lte(10)));
  return image.updateMask(mask.eq(0));
}
```
</details>











