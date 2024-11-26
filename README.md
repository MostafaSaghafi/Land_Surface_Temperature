# Land Surface Temperature (LST) using Landsat-8
calculate and visualize Land Surface Temperature (LST) time series data using Landsat 8 Surface Reflectance Tier 2 imagery:

---

### **1. Import the Landsat 8 Surface Reflectance Tier 2 image collection**
```javascript
var LC8 = ee.ImageCollection('LANDSAT/LC08/C02/T1_L2')  
  .filterBounds(geometry)  
  .filterDate('2013-01-01', '2023-12-31');  
```
- **`ee.ImageCollection('LANDSAT/LC08/C02/T1_L2')`**: This imports the Landsat 8 Surface Reflectance Tier 2 dataset, which contains calibrated surface reflectance data and thermal bands (e.g., `ST_B10` for thermal infrared).
- **`.filterBounds(geometry)`**: Filters the collection to include only images that intersect with the specified `geometry`. The `geometry` is a predefined region of interest (ROI) that could be a point, polygon, or other spatial feature.
- **`.filterDate('2013-01-01', '2023-12-31')`**: Restricts the collection to images acquired between January 1, 2013, and December 31, 2023.

---

### **2. Define a function to calculate Land Surface Temperature (LST)**
```javascript
function calculateLST(image) {  
  // Extract the thermal band (Band 10)  
  var thermalBand = image.select('ST_B10');  
```
- **`function calculateLST(image)`**: This function takes an individual Landsat 8 image as input and calculates the LST for that image.
- **`image.select('ST_B10')`**: Extracts the thermal band (`ST_B10`) from the image. This band measures the radiance emitted by the Earth's surface, which is used to calculate brightness temperature.

---

#### **a. Calculate brightness temperature in Kelvin**
```javascript
  var brightnessTemp = thermalBand.multiply(0.00341802).add(149.0).subtract(273.15); // Convert to Celsius  
```
- **`thermalBand.multiply(0.00341802).add(149.0)`**: Converts the raw digital number (DN) values of the thermal band into brightness temperature in Kelvin using the calibration constants provided in the Landsat 8 metadata. The formula is:
  ```javascript

  T = {DN} * {gain} + {offset}
  ```
  
  where:
  - Gain = `0.00341802`
  - Offset = `149.0`
- **`.subtract(273.15)`**: Converts the brightness temperature from Kelvin to Celsius by subtracting 273.15.

---

#### **b. Calculate LST using emissivity**
```javascript
  var emissivity = 0.986; // Typical value for most land covers  
  var lst = brightnessTemp.divide(emissivity);  
```
- **`emissivity = 0.986`**: The emissivity is a measure of how efficiently a surface emits thermal radiation. A value of `0.986` is typical for most land covers (e.g., vegetation, soil).
- **`brightnessTemp.divide(emissivity)`**: Corrects the brightness temperature using the emissivity value to calculate the Land Surface Temperature (LST). The formula is:
  ```javascript
  LST = {Brightness Temperature} / {Emissivity}
  ```
---

#### **c. Add the LST as a new band**
```javascript
  return image.addBands(lst.rename('LST'));  
}
```
- **`lst.rename('LST')`**: Renames the computed LST band to `'LST'` for clarity.
- **`image.addBands()`**: Adds the LST band to the original image, so the image now contains all original bands (e.g., reflectance bands) plus the new `'LST'` band.

---

### **3. Apply the LST calculation to the image collection**
```javascript
var lstCollection = LC8.map(calculateLST);  
```
- **`LC8.map(calculateLST)`**: Applies the `calculateLST` function to each image in the `LC8` image collection. The result is a new image collection (`lstCollection`) where each image includes the calculated LST band.

---

### **4. Generate time series data**
```javascript
var landsatchart1 = ui.Chart.image.series({
  imageCollection: lstCollection.select(['LST']),
  region: geometry,
  reducer: ee.Reducer.max(),
  scale: 30,
})
```
- **`ui.Chart.image.series()`**: Creates a time series chart from the image collection.
- **`imageCollection: lstCollection.select(['LST'])`**: Specifies that the chart should use only the `'LST'` band from the `lstCollection`.
- **`region: geometry`**: The region of interest (`geometry`) over which the LST values are calculated.
- **`reducer: ee.Reducer.max()`**: Specifies that the maximum LST value within the `geometry` should be used for each image. Other reducers like `mean` or `median` could also be used.
- **`scale: 30`**: The spatial resolution (in meters) at which the LST values are calculated. Landsat 8 has a 30-meter resolution for most bands.

---

#### **Set chart type and options**
```javascript
.setChartType('LineChart')
  .setOptions({
    title: 'Time Series Land Surface Temperature (2013-2023)',
    hAxis: {title: 'Date'},
    vAxis: {title: 'Temperature (K)'},
    series: {LST: {color: 'orange'}}
});
```
- **`.setChartType('LineChart')`**: Specifies that the chart should be a line chart.
- **`.setOptions()`**: Customizes the chart's appearance:
  - **`title: 'Time Series Land Surface Temperature (2013-2023)'`**: Sets the chart title.
  - **`hAxis: {title: 'Date'}`**: Labels the horizontal axis as "Date."
  - **`vAxis: {title: 'Temperature (K)'}`**: Labels the vertical axis as "Temperature (K)" (in Kelvin).
  - **`series: {LST: {color: 'orange'}}`**: Sets the color of the LST line to orange.

---

### **5. Display the chart**
```javascript
print(landsatchart1);
```
- **`print(landsatchart1)`**: Displays the LST time series chart in the GEE interface. The chart shows how the maximum LST within the `geometry` changes over time from 2013 to 2023.

---

### **Summary**
1. Loads Landsat 8 imagery for a specific region (`geometry`) and time range (2013–2023).
2. Calculates Land Surface Temperature (LST) for each image using the thermal band and emissivity correction.
3. Creates a time series chart of maximum LST values within the region of interest.
4. Displays the chart, allowing users to analyze temporal trends in LST over the specified period.
