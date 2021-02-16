# Detection and Measurement of the Intracellular Calcium Variation in Follicular Cells
https://www.researchgate.net/publication/265683681_Detection_and_Measurement_of_the_Intracellular_Calcium_Variation_in_Follicular_Cells

*Writer's notes are in itallics*

- The work here presents a new method for measuring the variation of intracellular calcium in follicular cells

## How does this method work?
This method is divided in two stages:

### Detection of a cell's nuclei
- This is done using watershed modified transformation *What's that?*
- During this step image segmentation is also done. *AKA: Detect each individual cell in the image and separate them*
- Automatic image segmentation can be done but its success can be affected by noise or abrupt changes in the images. If it fails you could get a segment with a partial cell or multiple cells
- Steps done during this stage:
    1. The image segmentation process is done using the Watershed transform. The contours of the cells as descriptors. 
    1. These contours are then enhanced with a morphological filter.*Check Canchola's notes* This homogenizes the luminance variation of the image *i.e. fix the image's contrast and brigthness* <br>
- *This first step basically makes the image's details **that we want to see**, a lot more understandable*

### Analysis of the fluorescence variations
- The fluorescene variations are modeled as an exponential decreasing function
- **Fluorescenece varioations are highly correlated with changes of intracellular free calcium**
- A new morphological filter (medium reconstruction process) is applied to help enhance the data for the modeling process
- *This step analyzes the features we enhanced applying an already known scientific principle*

## Calcium in cells? More likely than you think!
- Calcium is responsible for controlling many cellular processes
- Calcium acts as a second messenger triggering stuff like cell injury or death
- Calcium participates in conditions like hypertension or arrythmia
- A technique for analyzing calcium in cells is through **fluorescent microscopy**, i.e. **applying chemical fluorescents to cells that make the cells have a fluorescence effect depending on the level of calcium in them**

## What is a morphological filter?
- Based on increasing transformations:
    - An transformation is increasing if for all pairs of functions f and g, f <= g *The results of all functions in the transformation increase in value, compared to the previous function*
    - This transformation is idempotent. T(T(f)) = T(f)
    - *Too much maths for me*

## What's the watershed Transformation?
- This transform is used to segment images avoiding oversegmentation
- Uses a bunch of morphological filters
- To avoid oversegmentation an upper limit in the number of minima regions is imposed *What are minima regions? Maybe check the point below this one?*
- Some assumptions must be made to use this transformation:
    - The minima represent the center of the object (the core of the cell)
    - Gradient estimation *???*

### Marker detection
- **Regional minimum**: In a grey-scale image, a regional minimum is a connected component of pixels with uniform altitude without lower neighbors
