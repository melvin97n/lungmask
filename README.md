# Automated lung segmentation in CT under presence of severe pathologies

This package provides trained U-net models for lung segmentation. For now, three models are available:

- U-net(R231): This model was trained on a large and diverse dataset that covers a wide range of visual variabiliy. The model performs segmentation on individual slices, extracts right-left lung seperately includes airpockets, tumors and effusions. The trachea will not be included in the lung segmentation.

- U-net(LTRCLobes): This model was trained on a subset of the LTRC dataset. The model performs segmentation of individual lung-lobes but yields limited performance when dense pathologies are present. 

- U-net(R231CovidWeb):
[Look here for details](#COVID-19-Web)

**Examples of the two models applied**. **Left:** U-net(R231), will distinguish between left and right lung and include very dense areas such as effusions (third row), tumor or severe fibrosis (fourth row) . **Right:** U-net(LTRLobes), will distinguish between lung lobes but will not include very dense areas.

![alt text](figures/figure.png "Result examples")

For more exciting research on lung CT data, checkout the website of our research group:
https://www.cir.meduniwien.ac.at/research/lung/

## Referencing and citing
If you use this code or one of the trained models in your work please refer to:

>Johannes Hofmanninger, Forian Prayer, Jeanny Pan, Sebastian Röhrich, Helmut Prosch and Georg Langs. "Automatic lung segmentation in routine imaging is a data diversity problem, not a methodology problem". 1 2020, https://arxiv.org/abs/2001.11767

This paper contains a detailed description of the dataset used, a thorough evaluation of the U-net(R231) model, and a comparison to reference methods.

## Installation
```
pip install git+https://github.com/JoHof/lungmask
```
On Windows, depending on your setup, it may be necessary to install torch beforehand: https://pytorch.org

## Runtime and GPU support
Runtime between CPU-only and GPU supported inference varies greatly. Using the GPU, processing a volume takes only several seconds, using the CPU-only will take several minutes. To make use of the GPU make sure that your torch installation has CUDA support. In case of cuda out of memory errors reduce the batchsize to 1 with the optional argument ```--batchsize 1```

## Usage
### As a command line tool:
```
lungmask INPUT OUTPUT
```
If INPUT points to a file, the file will be processed. If INPUT points to a directory, the directory will be searched for DICOM series. The largest volume found (in terms of number of voxels) will be used to compute the lungmask. OUTPUT is the output filename. All ITK formats are supported.

Choose a model: <br/>
The U-net(R231) will be used as default. However, you can specify an alternative model such as LTRCLobes...

```
lungmask INPUT OUTPUT --modelname LTRCLobes
```

For additional options type:
```
lungmask -h
```

### As a python module:

```
from lungmask import lungmask
import SimpleITK as sitk

input_image = sitk.ReadImage(INPUT)
segmentation = lungmask.apply(input_image)  # default model is U-net(R231)
```
input_image has to be a SimpleITK object.

Load an alternative model like so:
```
model = lungmask.get_model('unet','LTRCLobes')
segmentation = lungmask.apply(input_image, model)
```

## Limitations
The model works on full slices only. The slice to process has to show the full lung and the lung has to be surrounded by tissue in order to get segmented. However, the model is quite stable to cases with a cropped field of view as long as the lung is surrounded by tissue. 

## COVID-19 Web
```
lungmask INPUT OUTPUT --modelname R231CovidWeb
```
The regular U-net(R231) model works very well for COVID-19 CT scans. However, collections of slices and case reports from the web are often cropped, annotated or encoded in regular image formats so that the original hounsfield unit (HU) values can only be estimated. The training data of the U-net(R231CovidWeb) model was augmented with COVID-19 slices that were mapped back from regular imaging formats to HU. The data was collected and prepared by MedSeg (http://medicalsegmentation.com/covid19/). While the regular U-net(231) showed very good resutls for this images there may be cases for which this model will yield slighty improved segmentations. Note that you have to map images back to HU when using images from the web. This blog post describes how you can do that (https://medium.com/@hbjenssen/covid-19-radiology-data-collection-and-preparation-for-artificial-intelligence-4ecece97bb5b)
![alt text](figures/example_covid.jpg "COVID examples")

 
