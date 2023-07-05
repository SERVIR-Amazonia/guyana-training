---
layout: page
title: Challenges - Modify and Improve your Random Forest Model
parent: "Intermediate Google Earth Engine: Mangrove Use Case"
nav_order: 4
---

# Overview

You've successfully written a remote sensing workflow to train a Random Forest classification model to map Mangrove presence with the Landsat archive. Now make it your own! There are several ways to tweak the original workflow right away to change the objective and make improvments. Here are a few ideas. Collaborate with your colleagues and your instructors. Happy coding!

## Area of Interest (AOI)

We defined our AOI at the beginning of the script using a hand-drawn polygon. As discussed previously you can define an AOI in GEE in many other ways. Review Part 1  to get some ideas. 

## Date Range

After merging together our full Landsat `ImageCollection`, we filtered it on a date range (i.e. 2020-2023). Come up with another date range that you'd like to map mangroves for. Remember that we can also specify a Day of Year range in our filters on the `ImageCollection`. This is useful for areas where there is heavy cloud cover during a certain time of year, or the phenomena you want to map exhibits a specific seasonal pattern that we can sense from satellite data.

## Model Structure

There are several ways we could make our RF model more robust... 

The first is in the number of 'trees' in the Random Forest. You can change that in the `ee.Classifier.smileRandomForest()` function and observe whether any of the accuracy metrics have improved. 

## Sample Data

Beyond the model structure itself, we can also provide more and/or better reference data to the model. The first improvement would be to increase the amount of total samples. Try a number between 200 and 500 per class. 

To provide better quality reference data, we can look for another source of Mangrove presence/absence data, or make our own. Making your own will take time and expertise. 

Try to find another mangrove data source like [Global Mangrove Watch](https://www.globalmangrovewatch.org/?activeLayers=mangrove_extent&category=distribution_and_change&zoom=7.299707280073768). How could you replace the Giri et al 2011 data in the workflow with this data source?  

## Happy Coding!
Do some experimentation, collaborate with your colleagues, ask your instructor questions - Good luck!

*Tip:* The **Docs** tab is your friend. If you see a function is being used in the script, find that function in the **Docs** to see what the required arguments are. It'll help you understand how to change the pre-existing functionality. 

<img align="center" src="../images/gee-mangrove/docs.png" hspace="15" vspace="10" width="600">
