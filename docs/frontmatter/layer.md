# layer

- [layer](#layer)
  - [Properties](#properties)
    - [Legend](#legend)
    - [Compare](#compare)
  - [Function values](#function-values)

```yaml
id: string
name: string
type: string
description: string
zoomExtent: [int, int] | null | fn(bag)
sourceParams:
  [key]: value | fn(bag)
compare: Compare
legend: Legend
```

## Properties

> 🙋 Several layer properties support functions to provide the app with dynamic values. See [Function values](#function-values) at the end of the document for details.

**id**  
`string`  
Id of the layer. Must be unique.  
⚠️ The id of a layer must match the STAC collection.

**name**  
`string`  
Name of the layer for display purposes.

**type**  
`raster | vector`  
The type of the layer will define how the data is displayed on the map.
- ⚠️ Only raster is currently supported.

**description**  
`string`  
Brief description of the layer. Will be shown in an info box.

**zoomExtent**  
`[int, int] | fn(bag)`  
Minimum and maximum zoom values for the layer. Below the minimum zoom level the layer will not be shown, but markers will be displayed to indicate where data is available.

**sourceParams**  
`object`  
Parameters to be appended to the tile source. The values will be used as provided as query string parameters.  
These values may vary greatly depending on the layer being added but some may be:
- **rescale**  
  `[int, int] | fn(bag)`  
  Minimum and maximum value for the rescale. This value is used for the color mapping, such that the minimum value corresponds to the starting color of the color map, and the maximum value corresponds to the ending. Adjusting this value changes the underlying data values mapped to the colors allowing for the analysis of different trends.
- **colormap_name**  
  `string`  
  The colormap to use for the layer. One of https://cogeotiff.github.io/rio-tiler/colormap/#default-rio-tilers-colormaps

### Legend

**legend**  
`object`  
Legend for this layer. This is shown in the interface as a visual guide to the user. The resulting legend will depend on the selected type.

```yaml
type: categorical | gradient
min: string
max: string
stops: string[] | object[]
  - string[]
  # or
  - color: string
    label: string
```

**legend.type**  
`categorical | gradient`  

<table>
<tr>
<th>Gradient</th>
<th>Categorical</th>
</tr>
<tr>
<td>

A `gradient` legend will display a continuous color scale using the provided stops which are rendered equally spaced from each other.
</td>
<td>

A `categorical` legend will display discreet color buckets according to the defined stops.
</td>
</tr>
<tr>
<td width='50%'>

![](../media/legend-gradient.png)
</td>
<td width='50%'>

![](../media/legend-categorical.png)
</td>
</tr>
</table>

**legend.min**  
`string`  
The label for the legend’s minimum value.  
⚠️ Not used when the legend is `categorical`.

**legend.max**  
`string`  
The label for the legend’s maximum value.  
⚠️ Not used when the legend is `categorical`.

**legend.stops**  
`string[] | object[]`  
The legend stops define the colors that should be rendered.  

In the case of `gradient` legends, this should be an array of color strings:
```yaml
stops:
  - '#FF0000'
  - '#00FF00'
  - '#0000FF'
```

If the legend is `categorical`, each entry should be an object with a color and a label. These values will be displayed so that there’s a clear mapping between color and label.
```yaml
stops:
  - color: '#FF0000'
    label: Corn
  - color: '#00FF00'
    label: Wheat
  - color: '#0000FF'
    label: Barley
```

### Compare

**compare**  
`object`  
Through the compare settings it is possible to define which layer should be loaded when the comparison gets enabled.  

There are 2 ways of defining compare layers:
1. The first, and easiest, is when the layer we want to compare to, is defined as a layer of a dataset. In this case we only need to reference the `datasetId` and the `layerId` (besides the other meta properties).
```yaml
compare:
  datasetId: string
  layerId: string
```

2. The second option is to compare to a layer which is not part of a dataset but still accessible via the api as a STAC item. This option requires us to fully define a layer within the `compare` object.
```yaml
compare:
  stacCol: string
  stacFilter: object
  type: raster | vector
  zoomExtent: [int, int] | null | fn
  sourceParams:
    [key]: value | fn
```

There are additional properties which are common to both ways of defining compare layers:
```yaml
compare:
  initialActive: boolean
  mapLabel: string | fn
  datetime: string | fn
```
- ⚠️ Option 2 is not currently implemented.

**compare.initialActive**  
`boolean`  
Whether the compare feature should be enabled when the layer is toggled. If `true` the compare feature will be immediately enabled as soon as the user set the current layer as active.
 - ⚠️ Not yet implemented.

**compare.mapLabel**  
`string | fn(bag)`  
When the comparison is enabled, the user should be informed about what is being shown. Could be a static string like “Modeled vs Real” or something dynamic computed from input parameters. It is often used for operations with dates.

**compare.datetime**  
`string | fn(bag)`  
Whenever the layer we’re comparing to has a time extend it is possible to define which date to load. This can be a hardcoded value or a function which allows for dynamic date computation. An example would be comparing the current selected date with the previous year - the function accepts the selected date as input and we subtract 1 year.

**compare.datasetId**  
`string`  
Id of the dataset from which to get the layer.  
⚠️ Only used when the layer belongs to a dataset present in the application.

**compare.layerId**  
`string`  
Id of the layer we want to compare to. The layer must belong to the dataset defined through `layer.compare.datasetId`.  
⚠️ Only used when the layer belongs to a dataset present in the application.


## Function values
Several layer properties support functions to provide the app with dynamic values.  Functions are written in javascript syntax and must be prefixed with `::js`.
Example:
```yaml
property: ::js () => 'value'
```

For more complicated functions it is helpful to spread them in multiple lines. To achieve this we can use the YAML Block Style Indicator (`|` pipe).
```yaml
property: |
  ::js () => {
    const val = 'value';
    return val;
  }
```

All these functions will be resolved by the app with a parameter (`bag`) containing helpers and internal values to help determining the correct value.  
Properties of `bag`:
```js
{
  datetime: Date // The date that is currently selected
  dateFns: Object // All the functions of https://date-fns.org/
  raw: Object // The unresolved layer data - kind of a self reference.
}
```

Usage example:
```yaml
property: | 
  ::js (bag) => {
    return bag.foo ? 'bar' : 'baz'
  }
```

Example of the `compare.datetime` being dynamically set as 1 year before the selected date:
```yaml
compare:
  datetime: |
    ::js ({ datetime, dateFns }) => {
      return dateFns.sub(datetime, { years: 1 });
    }
```