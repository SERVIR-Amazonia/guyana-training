---
layout: page
title: Part 1 - Gathering Data
parent: "Intermediate Google Earth Engine: Mangrove Use Case"
nav_order: 2
---

# Part 1 - Gathering Data

# Overview 

In this workflow, we will create an archive of Landsat imagery from Landsat missions 5 through 9, filter the image archive down to a desired time period of interest, then use the data to classify mangrove presence using a Random Forest classification model. 

Follow along by copying and pasting each code block in the lesson into your own blank script. At the end you will have the entire workflow saved to a script file on your own GEE account.

# Setting up Area of Interest

An area of interest can be uploaded from a local shapefile, drawn on the map, or derived from a pre-existing dataset in the Earth Engine catalogue.

As an example, we will look at the Food and Agriculture Organization's Global Administrative Units Layer (FAO GAUL) dataset. At the top of the code editor, type in the search bar 'FAO GAUL Global First'. We see that it is a `FeatureCollection` containing Level 1 administrative boundaries globally.

<img align="center" src="../images/gee-mangrove/faogaul.png" hspace="15" vspace="10" width="600">

Click on the `Table Schema` tab. We notice there is a useful field named 'ADM1_CODE'. We will use this property to derive our AOI. We will focus in on the Barima Waini area and the mangrove area on its coast.  Once you add the FAO GAUL data set to the map, you can find the 'ADM1_CODE' for this region by clicking on the map while in **Inspector** mode, and navigating to `Objects` > `FAO Boundaries` > `0` > `properties`.

```javascript
//--------------------------------------------------------------
// Define vector data (area of interest, aoi)
//--------------------------------------------------------------

// Define an aoi by filtering the FAO GAUL dataset
// https://developers.google.com/earth-engine/datasets/catalog/FAO_GAUL_2015_level1
var boundaries = ee.FeatureCollection("FAO/GAUL/2015/level1");
Map.addLayer(boundaries, {}, 'FAO boundaries');
var aoi = boundaries.filter(ee.Filter.eq('ADM1_CODE', 1398));

// Center the Map on the aoi object, with a specified zoom-level 
// (between 1-24)
Map.centerObject(aoi, 7);

// Add the aoi object as a layer to the map
Map.addLayer(aoi, {}, 'AOI');
```

<img align="center" src="../images/gee-mangrove/BarimaWaini_aoi.png" hspace="15" vspace="10" width="600">

However, for this section, we will create an AOI by drawing a polygon.  Comment out all the code above, and utilize the geometry drawing tools in the **Map** window to draw an area of interest. In the top-left of the **Map** window, select the Polygon draw button, then create your polygon over the coast in this region.  At the top of your script, rename it from ‘geometry’ to ‘aoi’.

<img align="center" src="../images/gee-mangrove/coast_aoi.png" hspace="15" vspace="10" width="700">

# Preprocessing Image Collections 

We always want to apply filters to `ImageCollections` as early in our workflow as we can to reduce the amount of effort the GEE servers will require. We already know the area that we'd like to pull data for (our AOI), and that we want relatively few clouds in our images, so we will apply a boundary and a cloud cover filter.

Let's do that for our first Landat `ImageCollection`, Landsat 5.

*Tip:*  Since there are differences in the amount and the order of bands on each Landsat mission, we use a dictionary (`sensorBandDictLandsatTOA`) and a list (`bandNamesLandsatTOA`) to standardize this information for us going forward using `select()` - it saves us quite a bit of typing when doing this for multiple collections. 

```javascript
//--------------------------------------------------------------
// Define raster data (Landsat data)
//--------------------------------------------------------------

// merge together all Landsat Image Collections from 
// Landsat 4 to Landsat 9 
// to make a continuous archive of data going back to the 1990s

// each Landsat imageCollection has slightly different ordering  
// of bands, so handling it with a dictionary saves us some typing

// Create a dictionary with the Landsat missions and their bands
var sensorBandDictLandsatTOA = {'L9': [1, 2, 3, 4, 5, 9, 6, 11],
                                'L8': [1, 2, 3, 4, 5, 9, 6, 11],
                                'L7': [0, 1, 2, 3, 4, 5, 7, 9],
                                'L5': [0, 1, 2, 3, 4, 5, 6, 7],
                                'L4': [0, 1, 2, 3, 4, 5, 6, 7]}

// Create a list with the band names
var bandNamesLandsatTOA = ['blue', 'green', 'red', 'nir', 
                           'swir1', 'temp', 'swir2', 'QA_PIXEL']

// Create filter by max cloud cover % in a scene
var metadataCloudCoverMax = 25

// Get Landsat 5 as an Image Collection
// by referencing the dictionary and renaming the bands
// and filtering out cloudy scenes
var lt5 = ee.ImageCollection('LANDSAT/LT05/C02/T1_TOA')
    .filterBounds(aoi)
    .filter(ee.Filter.lt('CLOUD_COVER', metadataCloudCoverMax))
    .select(sensorBandDictLandsatTOA['L5'], bandNamesLandsatTOA)

// Check one of the Image Collections
print(lt5)
```
Checking in the **Console**, we see that `lt5` is an `ImageCollection` with over 40 images in it. 

<img align="center" src="../images/gee-mangrove/print_lt5.png" hspace="15" vspace="10" width="500">

Let's do the same thing for Landsat 7, 8, and 9.

```javascript
// Landsat 7
var le7 = ee.ImageCollection('LANDSAT/LE07/C02/T1_TOA')
    .filterBounds(aoi)
    .filter(ee.Filter.lt('CLOUD_COVER', metadataCloudCoverMax))
    .select(sensorBandDictLandsatTOA['L7'], bandNamesLandsatTOA)

// Landsat 8 
var lc8 = ee.ImageCollection('LANDSAT/LC08/C02/T1_TOA')
    .filterBounds(aoi)
    .filter(ee.Filter.lt('CLOUD_COVER', metadataCloudCoverMax))
    .select(sensorBandDictLandsatTOA['L8'], bandNamesLandsatTOA)

// Landsat 9
var lc9 = ee.ImageCollection('LANDSAT/LC09/C02/T1_TOA')
    .filterBounds(aoi)
    .filter(ee.Filter.lt('CLOUD_COVER', metadataCloudCoverMax))
    .select(sensorBandDictLandsatTOA['L9'], bandNamesLandsatTOA)
```

Next, we want to apply some functions to each Landsat scene in a collection. In the first function, we will mask clouds and cloud shadows using the `QA_PIXEL` band that is included in every Landsat scene. The `QA_PIXEL` band is a bitmask generated in the Landsat processing center before it is distributed to the end-user. It has a lot of useful information contained in it. 

<img align="center" src="../images/gee-mangrove/qa_pixel.png" hspace="15" vspace="10" width="700">

We use the cloud and cloud shadow bits for this function.

```javascript
//--------------------------------------------------------------
// Preprocess images (cloud masking and index calculating)
//--------------------------------------------------------------

// do time series pre-processing using functions that are applied
// every image of the collection

// Create a cloud masking function
function cloudShadowMask(image) {
  // Get the correct bits
  // Bit 3 is cloud, bit 4 is cloud shadow
  var cloudShadowBitMask = (1 << 4);
  var cloudsBitMask = (1 << 3);
  // Get the pixel QA band
  var qa = image.select('QA_PIXEL');
  // Set both flags equal to zero, indicating clear conditions
  var mask = qa.bitwiseAnd(cloudShadowBitMask).eq(0)
             .and(qa.bitwiseAnd(cloudsBitMask).eq(0));
  return ee.Image(image).updateMask(mask);
}
```
In the second function we generate several spectral indices from the pre-existing spectral bands in our Landsat scenes.  All of these indeces can help in identifying mangroves, and index values range from -1 to +1.

**NDVI:** Normalized Difference Vegetation Index - quantifies vegetation by measuring the difference between near-infrared (which vegetation strongly reflects) and red light (which vegetation absorbs)

**LSWI:** Land Surface Water Index - calculated as a normalized ratio between near infrared (NIR) and short-wave infrared (SWIR), is sensitive to vegetation and soil water content

**NDMI:** Normalized Difference Moisture Index - used to determine vegetation water content (calculated as a ratio between the NIR and SWIR values in traditional fashion)

**MNDWI:** Modified Normalized Difference Water Index - uses green and SWIR bands for the enhancement of open water features (diminishes built-up area features that are often correlated with open water in other indices)

*Tip:* Here is a great resource published by the University of Bonn for finding indeces for many different purposes: [https://www.indexdatabase.de/](https://www.indexdatabase.de/)

```javascript
// Create function to calculate indices
// NDVI: (NIR-Red)/(NIR+Red)
// LSWI: (NIR-SWIR1)/(NIR+SWIR1)
// NDMI: (SWIR2-Red)/(SWIR2+Red)
// MNDWI: (Green-SWIR2)/(Green+SWIR2)
// We use the GEE function normalizedDifference, expressed as: (b1-b2)/(b1+b2)
function calculateIndices(img){
  var ndvi = img.normalizedDifference(['nir', 'red']).rename('ndvi');
  var lswi = img.normalizedDifference(['nir', 'swir1']).rename('lswi');
  var ndmi = img.normalizedDifference(['swir2', 'red']).rename('ndmi');
  var mndwi = img.normalizedDifference(['green', 'swir2']).rename('mndwi');
  var withIndices = img.addBands(ndvi).addBands(lswi)
                    .addBands(ndmi).addBands(mndwi);
  return withIndices
}
```

Now let's apply the functions to every image in each Landsat collection using `.map()` and see the result.

```javascript
// Apply pre-processing functions to each image collection
// including cloud masking and index calculating
var lt5_preprocessed = lt5.map(cloudShadowMask)
                              .map(calculateIndices);
var le7_preprocessed = le7.map(cloudShadowMask)
                              .map(calculateIndices);
var lc8_preprocessed = lc8.map(cloudShadowMask)
                              .map(calculateIndices);
var lc9_preprocessed = lc9.map(cloudShadowMask)
                              .map(calculateIndices);                                 
                              
//--------------------------------------------------------------
// Visualize a non-processed and pre-processed images
//--------------------------------------------------------------

// Select first non-processed image
var firstNonProcessed = lc8.first();

// Define visualization parameters
var visParamNonProcessed = {
  bands: ['red', 'green', 'blue'],
  min: 0,
  max: 0.2
};

// Add image to map
Map.addLayer(firstNonProcessed, 
            visParamNonProcessed, 
            'First non-processed image');

// Select first pre-processed image
var firstPreProcessed = lc8_preprocessed.first();

// Define visualization parameters
var visParamPreProcessed = {
  bands: ['red', 'green', 'blue'],
  min: 0,
  max: 0.2
};

// Add image to map
Map.addLayer(firstPreProcessed, 
            visParamPreProcessed, 
            'First pre-processed image');
```

<img align="center" src="../images/gee-mangrove/nonprocessed.png" hspace="15" vspace="10" width="600">

Toggle between the two image layers to see the result of the cloud masking. You can `Inspect` each image on the map to see that the preprocessed image has a different set of spectral bands than the non-processed image.

Let's merge our preprocessed Landsat `ImageCollection`s together into one. Finally, we must decide on a specific date range with which we'll train our Mangrove model. Keep this date range for now, and we'll experiment later on.

```javascript
//--------------------------------------------------------------
// Make full Landsat archive in specific date range
//--------------------------------------------------------------

// Merge preprocessed Landsat collections together
var mergedLandsat = lt5_preprocessed
.merge(le7_preprocessed)
.merge(lc8_preprocessed)
.merge(lc9_preprocessed)

// Filter merged collection on date range of interest
// by making a filter object
var dateFilter = ee.Filter.date('2020-01-01','2023-01-01')

// apply filter to merged collections
var landsatFiltered = mergedLandsat.filter(dateFilter)
print(landsatFiltered)

// Add images to the map
Map.addLayer(landsatFiltered,visParamPreProcessed,
             'Landsat Collection 2020-2023', false)
```

Code Checkpoint: [https://code.earthengine.google.com/0064c2b2764e781d4f8f1abf9c59f762](https://code.earthengine.google.com/0064c2b2764e781d4f8f1abf9c59f762)

We now have an `ImageCollection` consisting of data from multiple Landsat sensors for your area and date range of interest. 

# Image Compositing

We've now got an `ImageCollection` consisting of multiple Landsat missions that captured data in our desired date range. To train a mangrove classification model, we'll reduce the size of our input data by making a composite of the `ImageCollection`. Let's make a median composite for this demonstration and clip it to our AOI.

*Note:*  At a given pixel in your composite, if every single image in your `ImageCollection` was masked in that location (due to preprocessing in our case), then the composite will also be masked there. This can be remedied by more nuanced preprocessing algorithms and filters for the `ImageCollection`, but is beyond the scope of this demonstration.

```javascript
//--------------------------------------------------------------
// Create a composite image
//--------------------------------------------------------------
// Use the following functions to compare different aggregations: 
// .min(); .max(); .mean(); .median()

// We will work with the Median composite.
var composite = landsatFiltered.median().clip(aoi);

// Add composite to the map.
Map.addLayer(composite, visParamPreProcessed, 'Median Composite');
```

<img align="center" src="../images/gee-mangrove/median_composite.png" hspace="15" vspace="10" width="600">

At this point, sometimes it is a good idea to export the input data you have processed for your models. Importing pre-computed data products from a GEE asset will reduce the computational effort required to train and apply models in GEE. It is not necessary for this demonstration with such a small AOI, but bear that in mind for future work. If you decide to export your composite to a GEE asset, please change the `assetId` path to one of your own asset folders. 

```javascript
//--------------------------------------------------------------
// Export composite to Drive or Asset
//--------------------------------------------------------------

// Export composite to Google Drive.
Export.image.toDrive({
  image: composite.toFloat(),
  description: 'ToDrivemedianCompositeLandsat_2020-23',
  fileNamePrefix: 'medianCompositeLandsat_2020-23',
  region: aoi,
  scale: 30,
  maxPixels: 1e13
});

// Export composite as a GEE Asset.
// change assetId to a folder inside your user folder (e.g. users/kwoodward/trinidad-tobago/)
Export.image.toAsset({
  image: composite,
  description: 'ToAssetmedianCompositeLandsat_2020-23',
  assetId: 'users/ebihari/GuyanaWS/images/medianCompositeLandsat_2020-23',
  region: aoi,
  scale: 30,
  maxPixels: 1e13
});

// Re-import median composite.
var asset = ee.Image('users/ebihari/GuyanaWS/images/medianCompositeLandsat_2020-23');
Map.addLayer(asset, visParamPreProcessed, 'Asset');
```

Code Checkpoint: [https://code.earthengine.google.com/05e2efede37db5059fd732c1760c9852](https://code.earthengine.google.com/05e2efede37db5059fd732c1760c9852)

Continue onto Part 2 to finish your workflow.