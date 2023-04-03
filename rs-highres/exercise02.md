---
layout: page
title: Exercise 2 Working on GEE with Planet imagery
parent: Remote Sensing with High Resolution Imagery
nav_order: 4
---

## Exercise 2: Working on Google Earth Engine (GEE) with Planet imagery

### Retrieving and analyzing global tropical regions data into GEE: NICFI access for Planet

The main objective of the NICFI Program is to provide support to reduce and reverse the loss of tropical forests, contribute to combating climate change, conserve biodiversity, contribute to the growth, restoration and improvement of forests, and facilitate sustainable development, all for non-commercial purposes.

The available NICFI Planet datasets we have are:

**Basemaps**
● Hosted directly on the GEE so you can access the basemap as you would any other open source data
● Currently, only analytical basemaps are available
● Organized into 3 regional image collections: America, Africa and Asia
● Source scenes for basemaps are not available in GEE
● There are 3 steps:
    ○ Have a GEE account
    ○ Sign Planet NICFI Terms and Conditions
    ○ Request access to NICFI basemaps on the GEE

**Daily imagery**
● Works with Order API, i.e. not hosted by GEE
● Only PSScene4Band and PSOrthoTiles are available for download on GEE
● Currently, only clipping is supported.
● There are 3 steps:
    ○ Have a GEE account integrated with Google Cloud Platform (GCP)
    ○ Create a GCP and enable the Google Earth Engine API
    ○ Give a Planet service account an EE resource writer role on GCP

Now we are going to create and visualize NICFI composites.  First let’s go to the Google Earth Engine (GEE) code environment (https://code.earthengine.google.com/) , login and create a new script as we did in the previous workshop. You can use the same repository that you already created. You can find the entire code to follow in this script link:

https://code.earthengine.google.com/252398e06c93fd18c6288ce9bee8f236

<img align="center" src="../images/rs-highres/19_gee.png" hspace="15" vspace="10" width="600">

