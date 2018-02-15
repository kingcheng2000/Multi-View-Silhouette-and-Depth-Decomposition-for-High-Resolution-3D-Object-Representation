# 3D Object Super-Resolution
This is a repository to reproduce the method from the paper "3D Object Super-Resolution". The code here allows one to train models to accuractly and efficently upsample voxelized objects to high resolutions, and to reconstruct voxelized objects from single RGB images at high resolutions.


<p align="center">
  <img  src="images/SR.png" width="512" >
</p>

Example super-resolution results, when increasing from 32^3 resolution to 256^3 resolution. The first row is the low resolution inputs and the second row is the corresponing high resolution predictions. 

## Data Production
 To produce the data needed to train and test the moethods of this project we have the 'data_prep.py' script. This will donwload object CAD from the core classes of the ShapeNet dataset, convert the objects to voxel objects, exctract othographic depth maps, render the obejcts as images, and split all the data into training, valiation of test sets. This script makes use of the binvoxer executable, so first call 
 ```bash
sudo chmod 777 binvox 
```
Blender is also needed for this project so please ensure it is installed before biggining. 
 ```bash
sudo apt install blender
```
By default this scripts downloads the full chair class, to uspcale from 32^3 to 256^3 resolution, and renders 10 objects for each object. To achive this call: 
 ```bash
python data_prep.py
```
As an example to futher understand how to customize the data, to produced 1000 plane objects for 64 -> 128 resolution increase, render 5 images per plane call, and not supress error messages call:
 ```bash
python data_prep.py  -o plane -no 1000 -ni 5 -hi 128 -l 64 -debug True 
```

## Super-Resolution
The first element of this repo is the 3D super-resolution method. For this method 6 primary orthographic depth maps(ODM) are extracted from the low resolution object, and passed through our image super resolution method to produce a prediction for the corresponding high resolution object's 6 primary OMDs. The predicted ODM's are then used to carve away at the low resolution object(upsampled ot the higher resolution using nearest nighbor interpolation), to produce an estimate for the high resolution object. 

![Diagram](images/SRMethod.png?raw=true "Title")
Intuitive Diagram to understand the 3D super-resolution method. 

The image super-resolution technique used to predict high resolution odms, makes use to two deep convolutional neural networks. The first network, outlined in 'depth.py', estimates only the new depths into the known surface of the low resolution object  within a predefined range. The output of this network is added the corresponing low resolution odms(upsampled to the higher resolution), to produce a complete estimate for the depths to the new objects surface. The second network, outlined in 'occupancy.py", predicts an occupancy map for the high resolution odm, basically predicting silhouettes for the predicted object. The outputs of these two networks are combined to produce a rough estimate of the high reoslution ODM, and then smoothed to produce a final prediction. 

![Diagram](images/DepthPipeLine.png?raw=true "Title")
Intuitive Diagram to understand the 3D super-resolution method. 

 - To train the depth map prediction network call: 

```bash
python depth.py
```

 - To train the occupancy map prediction call: 

```bash
python occupancy.py
```

- To evaluate the Super Resolution prediction call: 

```bash
python SREval.py
```
These all assume the default chair classes is being increased from resolution 32 to resolution 256. To alter this call each script with the -h argument to view how to change each parameter. The two networks should not need more then 100 epochs to train fully, and graphs are created and saved in the '/plots/' directory should you wish to stop training early. The 'SREval.py' script will show predicted high resolution objects one at a time along side their input object and their ground truth high resolution object using meshlab. If you do not have meshlab installed call: 

```bash
sudo apt-get install meshlab
```



## Reference:
please cite my paper: ,if you use this repo for research 
