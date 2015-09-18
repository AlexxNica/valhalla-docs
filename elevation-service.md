
## Elevation service API reference

Mapzen's elevation service is a free, open-source web API (and C++ library) that provides digital elevation model (DEM) data as the result of a query. The elevation service data has many applications when combined with other routing and naivgation data, including computing the steepness of edges or generating an elevation profile along a route. This page documents the inputs and outputs to the service.

The elevation service is in active development. You can follow the [Mapzen blog](https://mapzen.com/blog) to get updates. To report software issues or suggest enhancements, open an issue in [Skadi GitHub repository](https://github.com/valhalla/skadi/issues). If you find a unique use for the elevation service, let the developers know at [routing@mapzen.com](mailto:routing@mapzen.com)! 

#### API keys and service limits

To use Mapzen's elevation service, you must first obtain a free, Elevation Service developer API key. Sign in at https://mapzen.com/developers to create and manage your API keys.

This is a free, shared service. As such, there are limitations on the number of sampling points to prevent individual users from degrading the overall system performance. The limits are related to the number of points for which you request elevation vations, rather than the resolution of the DEM in that area. 

Limits may be increased in the future, but you can contact routing@mapzen.com if you encounter rate limit status messages and need higher limits in the meantime.

### Inputs of the elevation service

The elevation service currently has a single action, `/height?`, that can be requested. The `height` provides the elevation at a set of input locations, which are specified either as a `shape` or an `encoded_polyline`. The `shape` parameter is an ordered list of two or more locations within a JSON array, while an [encoded polyline}(https://developers.google.com/maps/documentation/utilities/polylinealgorithm?hl=en) stores multiple locations within a single string. 

The `range` parameter is optional and assumed to be `false` if omitted. However, if `range` is provided and set to `true`, both the cumulative distance and the height are returned for each point. 

Requests takes the form of `elevation.mapzen.com/height?json={}&api_key=`, where the JSON inputs inside the ``{}`` include location and shape information and the optional range parameter. Note that you must append your own [API key](https://mapzen.com/developers) to the URL, following `&api_key=` at the end.

Here is an example request of a profile request using `shape`:

    elevation.mapzen.com/height?json={"range":true,"shape":[{"lat":40.712431,"lon":-76.504916},{"lat":40.712275,"lon":-76.605259},{"lat":40.712122,"lon":-76.805694},{"lat":40.722431,"lon":-76.884916},{"lat":40.812275,"lon":-76.905259},{"lat":40.912122,"lon":-76.965694}]}&api_key=elevation-xxxxxx

This request provides shape points near Pottsville, PA. The resulting profile response displays the input shape, as well as the height and range for each shape point.

An example request using `encoded_polyline` instead might look like this:

    elevation.mapzen.com/height?json={"range":true,"encoded_polyline":"s{cplAfiz{pCa]xBxBx`AhC|gApBrz@{[hBsZhB_c@rFodDbRaG\\ypAfDec@l@mrBnHg|@?}TzAia@dFw^xKqWhNe^hWegBfvAcGpG{dAdy@_`CpoBqGfC_SnI{KrFgx@?ofA_Tus@c[qfAgw@s_Agc@}^}JcF{@_Dz@eFfEsArEs@pHm@pg@wDpkEx\\vjT}Djj@eUppAeKzj@eZpuE_IxaIcF~|@cBngJiMjj@_I`HwXlJuO^kKj@gJkAeaBy`AgNoHwDkAeELwD|@uDfC_i@bq@mOjUaCvDqBrEcAbGWbG|@jVd@rPkAbGsAfDqBvCaIrFsP~RoNjWajBlnD{OtZoNfXyBtE{B~HyAtEsFhL_DvDsGrF_I`HwDpGoH|T_IzLaMzKuOrFqfAbPwCl@_h@fN}OnI"}&api_key=elevation-xxxxxx

#### Input parameters

A location must include a latitude and longitude in decimal degrees. The coordinates can come from many input sources, such as a GPS location, a point or a click on a map, a geocoding service, and so on. External search services, such as [Pelias](https://github.com/pelias) or [Nominatum](http://wiki.openstreetmap.org/wiki/Nominatim), can be used to find places and geocode addresses, whose coordinates can be used as input to the Elevation service.

| Shape parameters | Description |
| :--------- | :----------- |
| `lat` | Latitude of the location in degrees. |
| `lon` | Longitude of the location in degrees. |

The `encoded_polyline` parameter is simply a string of polyline encoded shape.

| Encoded polyline parameters | Description |
| :--------- | :----------- |
| `encoded_polyline` | A set of encoded lat,lng pairs of a line or shape.|

The `range` parameter is simply a boolean value which controls whether or not the returned array is 1 or 2 dimensional as described below. This can be used to generate a graph along a computed route since the returned 2d array is essentially just an x (range) and y (height) for each shape point. Steepness or gradient can also be easily computed from a profile request. 

| Range parameters | Description |
| :--------- | :----------- |
| `range` | `true` or `false`. Defaulted to `false`.|

### Outputs of the elevation service

The profile results are returned with the form of shape that was supplied in the request along with a 2D array representing the x and y of each input point in the elevation profile.

| Item | Description |
| :---- | :----------- |
| `shape` | The specified shape coordinates are returned from the input request. |
| `encoded_polyline` | The specified encoded polyline coordinates are returned from the input request. |
| `range_height` | The 2D array of range (x) and height (y) per lat,lng coordinate. |
| `x coordinate` | x is the range or distance along the input locations. It is the cumulative distance along the previous lat,lng coordinates up to the current coordinate. x for the first coordinate in the shape will always be 0. |
| `y coordinate` | y is the height or elevation of the associated lat,lng pair. The height will be returned as `null` if no height data exists for a given location. |
| `height` | An array of height for the associated lat,lng coordinates. |

### Data Sources and Known Issues

Currently the underlying data sources for the service are a mix of [SRTM](http://www2.jpl.nasa.gov/srtm/), [GMTED](http://topotools.cr.usgs.gov/gmted_viewer/) and [GEBCO](http://www.gebco.net/data_and_products/gridded_bathymetry_data/) DEMs. These sets provide global coverage at varying resolutions up to approximately 30 meters. It should be noted that both SRTM and GMTED zero fill oceans and other bodies of water; in these areas we use GEBCO to provide bathymetry (as well as in regions which are not coverged by SRTM and GMTED). Many other classical DEM-related issues occur in these datasets. It is not uncommon to see large variations in elevation in areas with large buildings and other such structures. We are considering how to best integrate NED and NRCAN sources, and are always looking for better datasets. If you find any data issues or can suggest any supplemental open datasets please let us know by filing an issue in [valhalla/skadi](https://github.com/valhalla/skadi).
