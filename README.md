# Laboratorio 4 SAR
# Nayely Araya Valerin C10561

## Introducción

En este laboratorio lo que se busca es observar la comparación de Sentinel-1 con Sentinel-2 para observar incendios y hacer sus respectivos análisis. Por lo que se va a trabajar con SAR en Sentinel-1 y otro con otro tipo de bandas que igualmente sirven para trabajar incendios con Sentinel-2, tal como se vio previamente en otro laboratorio en el cual se trabajó incendios y se utilizó el índice de vegetación NBR.

SAR es un tipo de radar, el cual ayuda a obtener imágenes de alta resolución a larga distancia. Este registra el tiempo que tarda un pulso en volver, su intensidad y fase de la microonda, esas señales de fase producen un interferograma entre dos captaciones de datos de SAR. (ESA, sf) Por lo que mejor dicho este radar envía pulsos electromagnéticos a la tierra para grabar su retorno y luego reflejarse, sin necesidad de fuentes de iluminación, por lo que como característica tiene que operar bien tanto de día como de noche, y con nubes y con lluvia. (Guimeno, 2019) Estas imágenes que genera el SAR suelen ser útiles para estudiar características del hielo y la nieve, cambios y medir flujos de hielo con la correlación de imágenes o “Speckle Tracking”. (ESA, sf)

Entonces lo que como tal devuelve una imagen SAR es un mapa con áreas iluminadas, que serían los objetos que devuelven más señal al radar, esto como puntos en las imágenes y en el caso de las que devuelven menos señal están las superficies más lisas o planas que son zonas más oscuras en la imagen. (Guimeno, 2019) Como ejemplo de algunas aplicaciones que se les puede dar a las imágenes SAR esta como la cartografía de la deforestación, seguimiento de la actividad marítima e identificación de zonas inundadas. (Menezes, sf)

## Marco Teórico

**Sentinel-1:** es un satélite con instrumentos de antenas de radar que ayuda a estudiar tanto la superficie terrestre como la oceánica y gracias a los datos radar este satélite no se ve afectado por el día o por la noche, por lo que tiene una monitorización constante. (Copernicus, 2017)

**Sentinel-2:** es un satélite con instrumento Multi Spectral Intrument el cual toma datos de alta resolución espacial para monitorizar la superficie terrestre. Además, cuenta con 13 bandas con diferentes resoluciones espectrales. (Copernicus, 2017)

**NBR:** es un índice de vegetación que se usa para calcular o identificar áreas calcinadas, por lo tanto, la vegetación saludable tiene alta reflectancia en el NIR, y el área quemada alta reflectancia en el SWIR. Este índice se suele utilizar para estudiar incendios. (Alonso, sf)

**Github:** es una plataforma que puede almacenar códigos, compartir y trabajar junto con otras personas el código. (Github, sf)

## Desarrollo

### Analisis de imagenes de inundación con Sentinel-1

Enlace de código de inundaciones Sentinel-1: 
https://code.earthengine.google.com/ccf23585a08601b9e93c2cd7136bc769 


![Esta imagen demuestra el resultado del Sentinel-1 antes de la inundación.](https://github.com/Naye1213/lab4/blob/7f14cd651060b0b96adcfb18ee20cdc49ff5a369/Antes.png)
 Esta imagen demuestra el resultado del Sentinel-1 antes de la inundación.



![Esta imagen es después de la inundación.](https://github.com/Naye1213/lab4/blob/e705bab55c0d905edaccc4b12945fec97297aacc/Despues.png)
Esta imagen es después de la inundación.

En estas dos imágenes se puede detectar leves cambios en toda la parte izquierda, la cual se ve más plana, mostrando a su vez más reflectancia y dando a entender que ahí hubo un cambio en cierto momento por lo que se podría decir que probablemente el lugar sufrió alguna afección de inundación afectando gran parte del espacio en estudio para este caso. Por el contrario, la parte derecha, que muestra zonas altas con mucha rugosidad, no se notan cambio alguno, por lo que se puede decir que esta parte no tuve afecciones de inundación, por lo que se mantuvo prácticamente igual.

### Explicación de parte por parte que hace cada bloque de codigo en el codigo de incendios para Sentinel-1 y Sentinel-2.

Este es un bloque de codigo para cargar una capa tipo poligono en forma de roi.
    
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


En esta parte lo que se hizo fue cargar la coleccion de imagenes de Sentinel-2.

<details>
  <summary>Clic</summary>
  
``` js
var s2 = ee.ImageCollection("COPERNICUS/S2_SR_HARMONIZED").filterBounds(roi) //s2 fue el nombre que le coloque a la coleccion que filtre.
  .filterDate('2023-01-01', '2023-12-31') //Defina el rango de fechas.
  .filterBounds(roi) // filtro de area.
  .map(cloudMask) // aca ejecutamos el enmascador de nubes que programamos antes. 
  print(s2) 
```
</details>


En estos dos bloques lo que se hizo fue filtrar imagenes de una fecha antes del incendio y otras imagenes con fecha posterior al incendio.

<details>
  <summary>Clic</summary>
  
``` js
// Filtros de la coleccion ANTES del incendio
var antes = s2.filter(ee.Filter.or(
 ee.Filter.date('2023-04-01', '2023-04-28')))
print(antes, 'antes del incendio s2');

// Filtros de la coleccion DESPUES del incendio
var despues = s2.filter(ee.Filter.or(
 ee.Filter.date('2023-05-10', '2023-06-01')))
print( despues, 'despues del incendio s2');
```
</details>


Lo que hace esta parte del codigo es que por medio de las imagenes que tenemos antes y las de despues se creea un nuevo conjunto de estas imagenes, pero mejoradas con la expresición mosaic.

<details>
  <summary>Clic</summary>
  
``` js
var antes2 = antes.mosaic().clip(roi) //puedes cambiar mosaic por mean or median
var despues2 =  despues.mosaic().clip(roi)
print(antes, 'imagen antes del incendio')
print(despues, 'imagen despues del incendio')

Map.addLayer( antes2,{bands: ['B4', 'B3', 'B2'], min: 354.3920564417735, max: 1282.2158558183125, gamma: 1.2}, 'antes del incendio s2', 0);
```
</details>


Aqui lo que se hace es calcular el NBR el cual es un indice de vegetación, que es utilizado para casos de incendios, este a su vez se calcula con las bandas B8 y B12, siendo la B8 el infrarrojo cercano y la B12 el infrarrojo de onda corta. Lo que se busca con esto es ver la humedad de la vegetación y asi detectar cuales son las areas quemadas, por su falta de agua.

<details>
  <summary>Clic</summary>
  
``` js
var preNBR = antes2.normalizedDifference(['B8', 'B12']).rename('nbr');
var postNBR = despues2.normalizedDifference(['B8', 'B12']).rename('nbr');
print(preNBR)
```
</details>


Estos bloques son solamente para vizualisar el NBR de las imagenes.

<details>
  <summary>Clic</summary>
  
``` js
Map.addLayer(preNBR, 
{bands: ['nbr'], min: 0.018902123252200934, max:0.7007203002942072 , gamma: 1.2}, 'nbr antes'); 
//Visualizacion
Map.addLayer(postNBR, 
{bands: ['nbr'], min: 0.018902123252200934, max:0.7007203002942072 , gamma: 1.2}, 'nbr despues'); 
```
</details>


En estos bloques lo que se busca calcular es la diferencia que hay entre antes del incendio y despues, para ver las variaciones que existen en la vegetación.

<details>
  <summary>Clic</summary>
  
``` js
// The result is called delta NBR or dNBR
var dNBR_unscaled = preNBR.subtract(postNBR);

// Scale product to USGS standards
var dNBR = dNBR_unscaled.multiply(1000);

// Add the difference image to the console on the right
print("Difference Normalized Burn Ratio: ", dNBR);
```
</details>


Con estas dos lineas podemos vizualisar en el mapa los resultados en una escala de grises.

<details>
  <summary>Clic</summary>
  
``` js
var grey = ['white', 'black'];

Map.addLayer(dNBR, {min: -1000, max: 1000, palette: grey}, 'dNBR greyscale');
```
</details>


Estas partes lo que hacen es clasificar los resultados por colores que van desde lo más quemado a lo que no se quemo, posteriormente solo se vizualisa en el mapa los resultados de esto.

<details>
  <summary>Clic</summary>
  
``` js
var sld_intervals =
  '<RasterSymbolizer>' +
    '<ColorMap type="intervals" extended="false" >' +
      '<ColorMapEntry color="#ffffff" quantity="-500" label="-500"/>' +
      '<ColorMapEntry color="#7a8737" quantity="-250" label="-250" />' +
      '<ColorMapEntry color="#acbe4d" quantity="-100" label="-100" />' +
      '<ColorMapEntry color="#0ae042" quantity="100" label="100" />' +
      '<ColorMapEntry color="#fff70b" quantity="270" label="270" />' +
      '<ColorMapEntry color="#ffaf38" quantity="440" label="440" />' +
      '<ColorMapEntry color="#ff641b" quantity="660" label="660" />' +
      '<ColorMapEntry color="#a41fd6" quantity="2000" label="2000" />' +
    '</ColorMap>' +
  '</RasterSymbolizer>';

// Add the image to the map using both the color ramp and interval schemes.
Map.addLayer(dNBR.sldStyle(sld_intervals), {}, 'dNBR classified');
```
</details>

### Analisis de resultados de indendios de Sentinel-1 y Sentinel-2

Enlace de código de incendios de Sentinel-1 y de Sentinel-2: 
https://code.earthengine.google.com/e8baf14ecdeed9da6ab7cf49c3e7cc1d 


![Esta imagen demuestra el resultado del Sentinel-1 antes del incendio.](https://github.com/Naye1213/lab4/blob/99e6711c91709566654e69bf6380e843c00e69d7/Antes.%20S1.png)
 Esta imagen demuestra el resultado del Sentinel-1 antes del incendio.


![Esta imagen demuestra el resultado del Sentinel-1 despues del incendio.](https://github.com/Naye1213/lab4/blob/b36d400c7d42a304da9f225a5a060cb0c91b3510/Despues.%20S1.png)
 Esta imagen demuestra el resultado del Sentinel-1 despues del incendio.


## Conclusiones



## Referencias

ESA. (s.f) “Radar de apertura sintética (SAR)” Recuperado de: https://www.esa.int/SPECIALS/Eduspace_Global_ES/SEMVKXF64RH_0.html 

Guimeno, N. (2019) “¿Qué es un SAR?” Recuperado de: https://www.inta.es/INTA/es/blogs/ceit/BlogEntry_1554121012176 

Menezes, E. (sf) “Explorar imágenes satelitales SAR” Recuperado de: https://learn.arcgis.com/es/projects/explore-sar-satellite-imagery/ 

Copernicus (2017) “Sentinel-1, -2 y -3” Recuperado de: https://www.inta.es/INTA/gl/blogs/copernicus/BlogEntry_1507278650016 

Alonso. (s.f) “Los 6 Índices de Vegetación para completar el NDVI” Recuperado de: https://mappinggis.com/2020/07/los-6-indices-de-vegetacion-para-completar-el-ndvi/ 

Github. (s.f) “Acerca de GitHub y Git” Recuperado de: https://docs.github.com/es/get-started/start-your-journey/about-github-and-git 










