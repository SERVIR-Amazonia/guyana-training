---
layout: page
title: Data Visualization with Landsat
parent: Introduction to Remote Sensing
nav_order: 3
---

# Data Visualization with Landsat
Remote sensing data is useful for gaining insights about a particular region by extracting the values stored in image bands and combining them to calculate different types of indices. Perhaps even more powerful, however, is the ability to use imagery to visualize those calculations. This lesson will use Landsat data to calculate different vegetation health indicators and review how to properly visualize those results. 

## Objectives
1. Understand and implement the required pre-processing steps for Landsat data. 
2. Experiment with common band combinations for visualizing remote sensing data.
3. Calculate NDVI with band math using Landsat data products.
4. Gain strategies for creating visually effective and useful image products.

## Image Pre-processing
Before remote sensing imagery can be used for analysis, it must go through a number of correction steps to ensure the data is as accurate as possible for a user’s research project. A number of these steps are often carried out by the organization that provides the data before it is released:
1. **Geometric correction.** This process aligns data layers to both their correct geographic location as well as relative to each other.
2. **Solar correction.** This process adjusts pixel values to account for solar effects. The resulting imagery consists of top-of-atmosphere reflectance values.
3. **Atmospheric correction.** This process counteracts the effects of the atmosphere on the energy measurements taken by the sensor. Some of these effects include scattering and absorption.
4. **Topographic correction.** This process accounts for reflectance variations due to slope, elevation, and aspect. It is especially important for areas with rugged or mountainous terrain. 

[IMAGE landsat pre-processing]
Figure 6. Pre-processing levels for remote sensing data. Image source: Young et. al., 2017

The level of pre-processing that a user needs depends on the type of analysis being carried out. Pre-processing does impact the values, and thus the analysis results, of the data being used and has the potential to introduce additional errors into a study. It is important to use remote sensing data with the appropriate amount of pre-processing to carry out the most accurate and consistent analysis as possible. 

### Exercise 2.1 Examine differences between data products.
This exercise will underline the differences between surface reflectance (SR) and top-of-atmosphere (TOA) Landsat 8 data products.

1. Open QGIS.
2. Under “Project Templates,” select “New Empty Project.”
3. Once the project opens, select “Project > Save As…” and save your project as `intro-remote-sensing`. 
4. Once the project saves, select “Layer > Add Layer > Add Raster Layer…”
5. First, add the SR image. Click on the `...` next to the “Raster dataset(s)” field. Navigate to the `intro-rs-data` folder and double-click on `l8-sr-negril-example.tif`. [IMAGE loaded raster]
6. Press “Add”, close the window, and then save the project. [IMAGE added l8 image]





