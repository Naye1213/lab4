# Laboratorio 4

## Desarrollo

Este es un bloque de codigo para cargar un poligono en forma de roi.

<details>
  <summary>Clic</summary>
  
  ```js
var roi = ee.FeatureCollection('projects/mtb2023-399203/assets/Palo_verde');
Map.addLayer(roi, {color: 'green'}, 'ROI');
Map.centerObject(roi, 12)
  ```
</details>

Este es un bloque de codigo para cargar la colección de imagenes en este caso de Sentinel-1, que este es de Radar.

<details>
  <summary>Clic</summary>
  
  ```js
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
  
  ```js
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
  
  ```js
// pasemos de un ImageCollection a un Image
var beforeinc = beforeinc.mosaic().clip(roi) //puedes cambiar mosaic por mean or median
var afterinc =  afterinc.mosaic().clip(roi)
print(beforeinc, 'imagen antes del incendio')
print(afterinc, 'imagen despues del incendio')

Map.addLayer( beforeinc,{bands: ['VV'], min: -15, max: -5, gamma: 1.2},  1.2}, 'antes del incendio sin speckle', 0);
  ```
</details>









