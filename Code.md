# NDVI
- Running this in GEE produces multimpe NDVI inspector tools for Somalia
       

#### Javascript
```javascript
/*/NDVI Inspector
*/ 

/////////////////////////////////////////////////////////////////////////////////
/////////////////////SET UP FUNCTIONS AND DATA TO USE//////////////////////////
/////////////////////////////////////////////////////////////////////////////////

/*get country data - in this case we chose the country of Somalia. However, it would
simple for a user to change the country adn thus, run this analysis anywhere around 
the globe*/
var Somalia = ee.FeatureCollection("ft:1tdSwUL7MVpOauSgRzqVTOwdfy17KDbw-1d9omPw")
                .filterMetadata('Country','equals','Somalia');

var somalia = Somalia.geometry();

//*add Somalia to map, and set map on country                
Map.addLayer(Somalia, {color:'C0C0C0'}, 'default display');
Map.setCenter(47.72,6.05,5);

// Band names
var S2_BANDS = [ 'B2',  'B3',   'B4', 'B8', 'B11',   'B12',   'B1', 'B10'];
var STD_NAMES = ['blue', 'green', 'red', 'nir', 'swir1', 'swir2', 'cb',  'cirrus'];

/*create a simple cloud score based on example code from EE. This is a method for 
finding the least cloud pixel over an image. The function takes the minimum value for 
the cirrus clouds, and creates a new band in the imagery called cloud score. This is then
used when creating a quality mosaic so that the least cloudy pixel, and all of ther values 
associated with that image for that pixel, are used to create a new image. */
var cloudScore = function(img) {
  img = img.select(S2_BANDS, STD_NAMES);
  // A helper to apply an expression and linearly rescale the output.
  var rescale = function(img, exp, thresholds) {
    return img.expression(exp, {img: img})
        .subtract(thresholds[0]).divide(thresholds[1] - thresholds[0]);
  };
  var score = ee.Image(1.0);
  score = score.min(rescale(img, 'img.cirrus', [0, 0.1]));
  score = score.min(rescale(img, 'img.cb', [0.5, 0.8]));
  return score.min(rescale(img.normalizedDifference(['green', 'swir1']), 'img', [0.8, 0.6]));
};

///ndvi function. This is the function used to get the greenest pixels for reach image. 
//the function uses Band 8 and Band 4, the Near Infared and the red bands. 
function ndvi(sentImg){
      var NDVI = sentImg.select('B8').subtract(sentImg.select('B4'))
      .divide(sentImg.select('B8')).add(sentImg.select('B4'));
      return NDVI}


/////////////////////////////////////////////////////////////////////////////////
/////////////////////Image collection for each month//////////////////////////
/////////////////////////////////////////////////////////////////////////////////
var JAN2016 = ee.ImageCollection('COPERNICUS/S2').select("B.*")
    .filterDate('2016-1-01', '2016-1-31')
    .filterBounds(Somalia);
var FEB2016 = ee.ImageCollection('COPERNICUS/S2').select("B.*")
    .filterDate('2016-2-1', '2016-2-27')
    .filterBounds(Somalia);
var MAR2016 = ee.ImageCollection('COPERNICUS/S2').select("B.*")
    .filterDate('2016-3-1', '2016-3-31')
    .filterBounds(Somalia);
var APR2016 = ee.ImageCollection('COPERNICUS/S2').select("B.*")
    .filterDate('2016-4-1', '2016-4-30')
    .filterBounds(Somalia);
var MAY2016 = ee.ImageCollection('COPERNICUS/S2').select("B.*")
    .filterDate('2016-5-1', '2016-5-31')
    .filterBounds(Somalia);
var JUN2016 = ee.ImageCollection('COPERNICUS/S2').select("B.*")
    .filterDate('2016-6-1', '2016-6-30')
    .filterBounds(Somalia);
var JUL2016 = ee.ImageCollection('COPERNICUS/S2').select("B.*")
    .filterDate('2016-7-1', '2016-7-31')
    .filterBounds(Somalia);
var AUG2016 = ee.ImageCollection('COPERNICUS/S2').select("B.*")
    .filterDate('2016-8-1', '2016-8-31')
    .filterBounds(Somalia);
var SEP2016 = ee.ImageCollection('COPERNICUS/S2').select("B.*")
    .filterDate('2016-9-1', '2016-9-30')
    .filterBounds(Somalia);
var OCT2016 = ee.ImageCollection('COPERNICUS/S2').select("B.*")
    .filterDate('2016-10-1', '2016-10-31')
    .filterBounds(Somalia);
var NOV2016 = ee.ImageCollection('COPERNICUS/S2').select("B.*")
    .filterDate('2016-11-1', '2016-11-30')
    .filterBounds(Somalia);
var DEC2016 = ee.ImageCollection('COPERNICUS/S2').select("B.*")
    .filterDate('2016-12-1', '2016-12-30')
    .filterBounds(Somalia);
var JAN2017 = ee.ImageCollection('COPERNICUS/S2').select("B.*")
    .filterDate('2016-1-1', '2016-1-30')
    .filterBounds(Somalia);
var FEB2017 = ee.ImageCollection('COPERNICUS/S2').select("B.*")
    .filterDate('2017-2-1', '2017-2-28')
    .filterBounds(Somalia);
print(FEB2017);


//JAN
//for each month, apply the simple cloud score from EE 
var JAN2016sc = ee.ImageCollection(JAN2016).select("B.*")
    .map(function(img) {
      img = img.divide(10000);
      // Invert the cloudscore so 1 is least cloudy, and rename the band.
      var score = cloudScore(img);
      score = ee.Image(1).subtract(score).select([0], ['cloudscore']);
      return img.addBands(score);
    });

//apply quality mosaic
var JAN2016M = ee.ImageCollection(ndvi(JAN2016sc
    .qualityMosaic('cloudscore')));
    
//convert to image
var JAN20161 = ee.Image(JAN2016M.iterate(function (newImage, img) {
  return ee.Image(img).addBands(newImage)}, ee.Image([])));

//clip, select bands, and set time
var JAN2016MN  = JAN20161.clip(Somalia).select( ['B8'],['NDVI'])
                .set( 'system:time_start','2016-01-01','system:time_end', '2016-01-31');

//FEB
var FEB2016sc = ee.ImageCollection(FEB2016).select("B.*")
    .map(function(img) {
      img = img.divide(10000);
      // Invert the cloudscore so 1 is least cloudy, and rename the band.
      var score = cloudScore(img);
      score = ee.Image(1).subtract(score).select([0], ['cloudscore']);
      return img.addBands(score);
    });

//apply quality mosaic
var FEB2016M = ee.ImageCollection(ndvi(FEB2016sc
    .qualityMosaic('cloudscore')));
    
//convert to image
var FEB20161 = ee.Image(FEB2016M.iterate(function (newImage, img) {
  return ee.Image(img).addBands(newImage)}, ee.Image([])));

//clip, select bands, and set time
var FEB2016MN  = FEB20161.clip(Somalia).select( ['B8'],['NDVI'])
                .set( 'system:time_start','2016-02-01','system:time_end', '2016-02-28');

//march
var MAR2016sc = ee.ImageCollection(MAR2016).select("B.*")
    .map(function(img) {
      img = img.divide(10000);
      // Invert the cloudscore so 1 is least cloudy, and rename the band.
      var score = cloudScore(img);
      score = ee.Image(1).subtract(score).select([0], ['cloudscore']);
      return img.addBands(score);
    });
//apply quality mosaic
var MAR2016M = ee.ImageCollection(ndvi(MAR2016sc
    .qualityMosaic('cloudscore')));
//convert to image
var MAR20161 = ee.Image(MAR2016M.iterate(function (newImage, img) {
  return ee.Image(img).addBands(newImage)}, ee.Image([])));

//clip, select bands, and set time
var MAR2016MN  = MAR20161.clip(Somalia).select( ['B8'],['NDVI'])
                .set( 'system:time_start','2016-03-01','system:time_end', '2016-03-31');

//APRIL
var APR2016sc = ee.ImageCollection(APR2016).select("B.*")
    .map(function(img) {
      img = img.divide(10000);
      // Invert the cloudscore so 1 is least cloudy, and rename the band.
      var score = cloudScore(img);
      score = ee.Image(1).subtract(score).select([0], ['cloudscore']);
      return img.addBands(score);
    });
//apply quality mosaic
var APR2016M = ee.ImageCollection(ndvi(APR2016sc
    .qualityMosaic('cloudscore')));
//convert to image
var APR20161 = ee.Image(APR2016M.iterate(function (newImage, img) {
  return ee.Image(img).addBands(newImage)}, ee.Image([])));

//clip, select bands, and set time
var APR2016MN  = APR20161.clip(Somalia).select( ['B8'],['NDVI'])
                .set( 'system:time_start','2016-04-01','system:time_end', '2016-04-30');

//MAY
var MAY2016sc = ee.ImageCollection(MAY2016).select("B.*")
    .map(function(img) {
      img = img.divide(10000);
      // Invert the cloudscore so 1 is least cloudy, and rename the band.
      var score = cloudScore(img);
      score = ee.Image(1).subtract(score).select([0], ['cloudscore']);
      return img.addBands(score);
    });
//apply quality mosaic
var MAY2016M = ee.ImageCollection(ndvi(MAY2016sc
    .qualityMosaic('cloudscore')));
//convert to image
var MAY20161 = ee.Image(MAY2016M.iterate(function (newImage, img) {
  return ee.Image(img).addBands(newImage)}, ee.Image([])));
//clip, select bands, and set time
var MAY2016MN  = MAY20161.clip(Somalia).select( ['B8'],['NDVI'])
                .set( 'system:time_start','2016-05-01','system:time_end', '2016-05-30');

//JUN    
var JUN2016sc = ee.ImageCollection(JUN2016).select("B.*")
    .map(function(img) {
      img = img.divide(10000);
      // Invert the cloudscore so 1 is least cloudy, and rename the band.
      var score = cloudScore(img);
      score = ee.Image(1).subtract(score).select([0], ['cloudscore']);
      return img.addBands(score);
    });
//apply quality mosaic
var JUN2016M = ee.ImageCollection(ndvi(JUN2016sc
    .qualityMosaic('cloudscore')));
//convert to image
var JUN20161 = ee.Image(JUN2016M.iterate(function (newImage, img) {
  return ee.Image(img).addBands(newImage)}, ee.Image([])));
//clip, select bands, and set time
var JUN2016MN  = JUN20161.clip(Somalia).select( ['B8'],['NDVI'])
                .set( 'system:time_start','2016-06-01','system:time_end', '2016-06-30');

//JULY
var JUL2016sc = ee.ImageCollection(JUL2016).select("B.*")
    .map(function(img) {
      img = img.divide(10000);
      // Invert the cloudscore so 1 is least cloudy, and rename the band.
      var score = cloudScore(img);
      score = ee.Image(1).subtract(score).select([0], ['cloudscore']);
      return img.addBands(score);
    });
//apply quality mosaic
var JUL2016M = ee.ImageCollection(ndvi(JUL2016sc
    .qualityMosaic('cloudscore')));
//convert to image
var JUL20161 = ee.Image(JUL2016M.iterate(function (newImage, img) {
  return ee.Image(img).addBands(newImage)}, ee.Image([])));
//clip, select bands, and set time
var JUL2016MN  = JUL20161.clip(Somalia).select( ['B8'],['NDVI'])
                .set( 'system:time_start','2016-07-01','system:time_end', '2016-07-30');

//AUGUST
var AUG2016sc = ee.ImageCollection(AUG2016).select("B.*")
    .map(function(img) {
      img = img.divide(10000);
      // Invert the cloudscore so 1 is least cloudy, and rename the band.
      var score = cloudScore(img);
      score = ee.Image(1).subtract(score).select([0], ['cloudscore']);
      return img.addBands(score);
    });
//apply quality mosaic
var AUG2016M = ee.ImageCollection(ndvi(AUG2016sc
    .qualityMosaic('cloudscore')));
//convert to image
var AUG20161 = ee.Image(AUG2016M.iterate(function (newImage, img) {
  return ee.Image(img).addBands(newImage)}, ee.Image([])));
//clip, select bands, and set time
var AUG2016MN  = AUG20161.clip(Somalia).select( ['B8'],['NDVI'])
                .set( 'system:time_start','2016-08-01','system:time_end', '2016-08-31');

//SEPTEMBER
var SEP2016sc = ee.ImageCollection(SEP2016).select("B.*")
    .map(function(img) {
      img = img.divide(10000);
      // Invert the cloudscore so 1 is least cloudy, and rename the band.
      var score = cloudScore(img);
      score = ee.Image(1).subtract(score).select([0], ['cloudscore']);
      return img.addBands(score);
    });
//apply quality mosaic
var SEP2016M = ee.ImageCollection(ndvi(SEP2016sc
    .qualityMosaic('cloudscore')));
//convert to image
var SEP20161 = ee.Image(SEP2016M.iterate(function (newImage, img) {
  return ee.Image(img).addBands(newImage)}, ee.Image([])));
//clip, select bands, and set time
var SEP2016MN  = SEP20161.clip(Somalia).select( ['B8'],['NDVI'])
                .set( 'system:time_start','2016-09-01','system:time_end', '2016-09-31');

//OCTOBER
var OCT2016sc = ee.ImageCollection(OCT2016).select("B.*")
    .map(function(img) {
      img = img.divide(10000);
      // Invert the cloudscore so 1 is least cloudy, and rename the band.
      var score = cloudScore(img);
      score = ee.Image(1).subtract(score).select([0], ['cloudscore']);
      return img.addBands(score);
    });
//apply quality mosaic
var OCT2016M = ee.ImageCollection(ndvi(OCT2016sc
    .qualityMosaic('cloudscore')));
//convert to image
var OCT20161 = ee.Image(OCT2016M.iterate(function (newImage, img) {
  return ee.Image(img).addBands(newImage)}, ee.Image([])));
//clip, select bands, and set time
var OCT2016MN  = OCT20161.clip(Somalia).select( ['B8'],['NDVI'])
                .set( 'system:time_start','2016-10-01','system:time_end', '2016-10-31');

//NOVEMBER
var NOV2016sc = ee.ImageCollection(NOV2016).select("B.*")
    .map(function(img) {
      img = img.divide(10000);
      // Invert the cloudscore so 1 is least cloudy, and rename the band.
      var score = cloudScore(img);
      score = ee.Image(1).subtract(score).select([0], ['cloudscore']);
      return img.addBands(score);
    });
//apply quality mosaic
var NOV2016M = ee.ImageCollection(ndvi(NOV2016sc
    .qualityMosaic('cloudscore')));
//convert to image
var NOV20161 = ee.Image(NOV2016M.iterate(function (newImage, img) {
  return ee.Image(img).addBands(newImage)}, ee.Image([])));
//clip, select bands, and set time
var NOV2016MN  = NOV20161.clip(Somalia).select( ['B8'],['NDVI'])
                .set( 'system:time_start','2016-11-01','system:time_end', '2016-11-31');


//DECEMBER
var DEC2016sc = ee.ImageCollection(DEC2016).select("B.*")
    .map(function(img) {
      img = img.divide(10000);
      // Invert the cloudscore so 1 is least cloudy, and rename the band.
      var score = cloudScore(img);
      score = ee.Image(1).subtract(score).select([0], ['cloudscore']);
      return img.addBands(score);
    });
//apply quality mosaic
var DEC2016M = ee.ImageCollection(ndvi(DEC2016sc
    .qualityMosaic('cloudscore')));
//convert to image
var DEC20161 = ee.Image(DEC2016M.iterate(function (newImage, img) {
  return ee.Image(img).addBands(newImage)}, ee.Image([])));
//clip, select bands, and set time
var DEC2016MN  = DEC20161.clip(Somalia).select( ['B8'],['NDVI'])
                .set( 'system:time_start','2016-12-01','system:time_end', '2016-12-31');

//JAN   
var JAN2017sc = ee.ImageCollection(JAN2017).select("B.*")
    .map(function(img) {
      img = img.divide(10000);
      // Invert the cloudscore so 1 is least cloudy, and rename the band.
      var score = cloudScore(img);
      score = ee.Image(1).subtract(score).select([0], ['cloudscore']);
      return img.addBands(score);
    });
//apply quality mosaic
var JAN2017M = ee.ImageCollection(ndvi(JAN2017sc
    .qualityMosaic('cloudscore')));
//convert to image
var JAN20171 = ee.Image(JAN2017M.iterate(function (newImage, img) {
  return ee.Image(img).addBands(newImage)}, ee.Image([])));
//clip, select bands, and set time
var JAN2017MN  = JAN20171.clip(Somalia).select( ['B8'],['NDVI'])
                .set( 'system:time_start','2017-01-01','system:time_end', '2017-01-31');

//feburary   
var FEB2017sc = ee.ImageCollection(FEB2017).select("B.*")
    .map(function(img) {
      img = img.divide(10000);
      // Invert the cloudscore so 1 is least cloudy, and rename the band.
      var score = cloudScore(img);
      score = ee.Image(1).subtract(score).select([0], ['cloudscore']);
      return img.addBands(score);
    });
//apply quality mosaic
var FEB2017M = ee.ImageCollection(ndvi(FEB2017sc
    .qualityMosaic('cloudscore')));
//convert to image
var FEB20171 = ee.Image(FEB2017M.iterate(function (newImage, img) {
  return ee.Image(img).addBands(newImage)}, ee.Image([])));
//clip, select bands, and set time
var FEB2017MN  = FEB20171.clip(Somalia).select( ['B8'],['NDVI'])
                .set( 'system:time_start','2017-02-01','system:time_end', '2017-02-28');

/////////////////////////////////////////////////////////////////////////////////
/////////////////////********************************//////////////////////////
/////////////////////////////////////////////////////////////////////////////////
var all = ee.ImageCollection([JAN2016MN,FEB2016MN,MAR2016MN,APR2016MN,
          MAY2016MN,JUN2016MN,JUL2016MN,AUG2016MN,SEP2016MN,
          OCT2016MN,NOV2016MN,DEC2016MN,JAN2017MN,FEB2017MN]);
          

// Combine the mean and standard deviation reducers.
//var reducers = ee.Reducer.mean();//.combine({
//  reducer2: ee.Reducer.stdDev(),
//  sharedInputs: true,

 
//var reducer = ee.Reducer.stdDev();

var min = all.min();

//var std = all.stdDev();

var mean = all.mean();

Map.addLayer(min, {palette:'ff2e19,ffaf1f,fbff17,1bff22,ffffff'}, 'NDVI Min');
Map.addLayer(mean, {palette:'ff2e19,ffaf1f,fbff17,1bff22,ffffff'}, 'NDVI Mean');

//Create an NDVI chart. This is adapted directly from 
//https://code.earthengine.google.com/5b0bb20c142e7479e2b7ec2c8466da8b
//the EE example Two Chart Inspector
var indexChart = ui.Chart.image.doySeries({
 imageCollection: all,
 region: Somalia,
 startDay: 1,
 endDay: 350,
 scale: 300
});

print(indexChart);

//Map.addLayer(all, {palette:'077E4F, 12F154, F0F112, F19B12, F11912'}, 'Somlaia NDVI');

// Create a panel to hold our widgets.
var panel = ui.Panel();
panel.style().set('width', '300px');

// Create an intro panel with labels.
var intro = ui.Panel([
  ui.Label({
    value: 'Inspector',
    style: {fontSize: '20px', fontWeight: 'bold'}
  }),
  ui.Label('Click a point within Somalia.')
]);
panel.add(intro);

// Create panels to hold lon/lat values.
var lon = ui.Label();
var lat = ui.Label();
panel.add(ui.Panel([lon, lat], ui.Panel.Layout.flow('horizontal')));
//add aditional maps

// Register a callback on the default map to be invoked when the map is clicked.
Map.onClick(function(coords) {
  // Update the lon/lat panel with values from the click event.
  lon.setValue('lon: ' + coords.lon.toFixed(2)),
  lat.setValue('lat: ' + coords.lat.toFixed(2));

  // Add a red dot for the point clicked on.
  var point = ee.Geometry(ee.Geometry.Point(coords.lon, coords.lat).buffer(70000));
  var dot = ui.Map.Layer(point, {color: 'FF0000'});
  Map.layers().set(1, dot);

  // Create an NDVI chart.
  var ndviChart = ui.Chart.image.series(all, point, ee.Reducer.mean(), 500);
  ndviChart.setOptions({
    title: 'NDVI Over Time',
    vAxis: {title: 'NDVI'},
    hAxis: {title: 'date', format: 'YYYY-MM-DD', gridlines: {count: 7}},
  });
  panel.widgets().set(2, ndviChart);

});

Map.style().set('cursor', 'crosshair');

// Add the panel to the ui.root.
ui.root.insert(0, panel);


```
