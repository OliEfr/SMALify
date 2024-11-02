# SMALify

<img src="docs/badja_result.gif">

This repository contains an implementation for performing 3D animal (quadruped) reconstruction from a monocular image or video. The system adapts the pose (limb positions) and shape (animal type/height/weight) parameters for the SMAL deformable quadruped model, as well as camera parameters until the projected SMAL model aligns with 2D keypoints and silhouette segmentations extracted from the input frame(s).

The code can be thought of as a modernization of the fitting code used in [Creatures Great and SMAL](https://arxiv.org/abs/1811.05804) paper; Chainer/ChumPy has been replaced with PyTorch, OpenDR replaced with PyTorch3D etc. However, I have also included some recent innovations from the [Who Left the Dogs Out?](https://arxiv.org/abs/2007.11110) such as the inclusion of limb scaling parameters, and an improved shape prior.

The aim of this repository is to provided demonstrative fitting code to benefit computer vision researchers but also those working in animal/veterinary science. In either case, I'd be delighted to hear from you!

## Installation
1. Clone the repository **with submodules** and enter directory
   ```
   git clone --recurse-submodules https://github.com/benjiebob/SMALify
   cd SMALify
   ```
   Note: If you don't clone with submodules you won't get the sample data from BADJA/StanfordExtra/SMALST.
    
2. Install dependencies, particularly [PyTorch (cuda support recommended)](https://pytorch.org/), [Pytorch3D](https://github.com/facebookresearch/pytorch3d). Check [requirements.txt](https://github.com/benjiebob/SMALify/blob/master/requirements.txt) for full details.

3. Download [BADJA videos](https://drive.google.com/file/d/1ad1BLmzyOp_g3BfpE2yklNI-E1b8y4gy/view?usp=sharing) and unzip to `badja_extra_videos.zip`. 

4. Inspect the directory paths in [config.py](https://github.com/benjiebob/SMALify/blob/master/config.py) and make sure they match your system.

## QuickStart: Running the Fitter

- Run on a sample video from the [BADJA](https://github.com/benjiebob/BADJA) dataset.
      <img src="docs/badja_opt.gif">
   - Run the python script
      ```
      python smal_fitter/optimize_to_joints.py
      ```
   - Inspect the SMALify/checkpoint directory to inspect the progress of the fitter. 
      - The output files stX_epY means, stage X and iteration Y of the fitter.
      - The final output is named st10_ep0 by (a slightly lazy) convention. 
      - The other files have the following meaning

         | Extension  | Explanation |
         | ------------- | ------------- |
         | .png  | Image Visualization  |
         | .ply  | Mesh file, can be viewed in e.g. [MeshLab](https://www.meshlab.net/)  |
         | .pkl  | Pickle file, contains the latest model/camera parameters |

   - Create a video with the final fits
      - The generate_video.py function loads the exported .pkl files generated during the fitting process and exports the data. This is generally usful if your .pkl files are created using alternative methods, e.g. Who Left the Dogs Out? (coming soon!) or your own research. 
      - Set CHECKPOINT_NAME in [config.py](https://github.com/benjiebob/SMALify/blob/master/config.py) to be the name of the output directory in SMALify/checkpoints.
      - By default the code will load the final optimized meshes, indicated by EPOCH_NAME = "st10_ep0". If you want to generate a video from intermediate results, set this to some different stage/iteration. 
      - Run the video generation script, which exports the video to SMALify/exported
         ```
         python smal_fitter/generate_video.py
         ```
      - Create a video using e.g. [FFMPEG](https://ffmpeg.org/):
         ```
         cd exported/CHECKPOINT_NAME/EPOCH_NAME
         ffmpeg -framerate 2 -pattern_type glob -i '*.png' -pix_fmt yuv420p results.mp4
         ```
- Fit to an image from [StanfordExtra](https://github.com/benjiebob/StanfordExtra) dataset.
   <img src="docs/stanfordextra_opt.gif">
   - Edit the [config.py](https://github.com/benjiebob/SMALify/blob/master/config.py) file to make load a StanfordExtra image instead of a BADJA video sequence:
      ```
      # SEQUENCE_OR_IMAGE_NAME = "badja:rs_dog"
      SEQUENCE_OR_IMAGE_NAME = "stanfordextra:n02099601-golden_retriever/n02099601_176.jpg"
      ```
   - Run the python script:
      ```
      python smal_fitter/optimize_to_joints.py
      ```
## Running on alternative data
### Alternative BADJA/StanfordExtra sequences:
- Follow the instructions for [BADJA](https://github.com/benjiebob/BADJA) or [StanfordExtra](https://github.com/benjiebob/StanfordExtra).
- Open the [config.py](https://github.com/benjiebob/SMALify/blob/master/config.py) file and make the following changes

   | Config Setting  | Explanation | Example |
   | ------------- | ------------- | ------------- |
   | SEQUENCE_OR_IMAGE_NAME  | Used to refer to your sequence/image  | badja:rs_dog
   | SHAPE_FAMILY            | Choose from 0: Cat, 1: Canine (e.g. Dog), 2: Equine (e.g. Horse), 3: Bovine (e.g. Cow), 4: Hippo |  1 |
   | IMAGE_RANGE  | Number of frames to process from the sequence. Ignored for StanfordExtra.  | [1,2,3] or range(0, 10) |
   | WINDOW_SIZE  | For video sequences, the number of frames to fit into a batch. Alter depending on GPU capacity. | 10 |
   
### Running on your own data
The first job is to generate keypoint/silhouette data for your input image(s). I recommend using [LabelMe](https://github.com/wkentaro/labelme), which is fantastic software that makes annotating keypoints / silhouettes efficient. 
   
- Install the software, and then load the joint annotation execute
   ```
   labelme --labels labels.txt --nosortlabels
   ```
- Next, generate the silhouette annotations
   ```
   # TODO
   ```
- TODO: Write script to load labelme files

### Building your own quadruped deformable model
If you want to represent an animal quadruped category which isn't covered by the SMAL model (e.g. perhaps you want to reconstruct rodents/squirrels), you can use the [fitter_3d](https://github.com/benjiebob/SMALify/tree/master/fitter_3d) tool. The basic idea is to fit the existing SMAL model to a collection of 3D artist meshes (you can download online) and thereby learn a new shape space. More information is given in the [README](https://github.com/benjiebob/SMALify/blob/master/fitter_3d/README.md).

## Improving performance and general tips and tricks
- For some sequences, it may be necessary to fiddle with the weights applied to each part of the loss function. These are defined in [config.py](https://github.com/benjiebob/SMALify/blob/master/config.py) in an OPT_WEIGHTS settings. The values weight the following loss components:

| Loss Component  | Explanation | Tips for Loss Weight
| ------------- | ------------- | ------------- |
| 2D Keypoint Reprojection  | Project the SMAL model with latest parameters and compare projected joints to input 2D keypoints | If your model limbs don't match well with the input keypoints after fitting, it may be worth increasing this. |
| 3D Shape Prior | Used to constrain the 3D shapes to be 'animal-like'. Note that (unlike equivalent human approaches that use mocap etc.) only artist data is used for this. | If your reconstructed animals don't look like animals, try increasing this. |
| 3D Pose Prior  | Used to contains the 3D poses to be anatomically plausible. | If your reconstructed animals have limb configurations which are very unreasonable, e.g. legs in strange places, try increasing this. |
| 2D Silhouette  | Project the SMAL model with latest parameters and compare rendered silhouette to input 2D silhouette. | If the shape of your reconstructed animal doesn't match well (e.g. maybe it's too thin?), try increasing this. |
| Temporal  | Constrain the change in SMAL parameters between frames. (Only for videos) | If your limbs move unnaturally between video frames, try adapting this. |

   Note that to avoid poor local minima, the optimization proceeds over multiple stages and the weights vary at each stage. For example, only once an approximate camera location has been found should there be 2D joint loss, and only once an approximate set of limb positions have been found should there be a 2D silhouette loss.

- To improve efficiency it is very likely that the number of iterations (again a setting in OPT_WEIGHTS) can be reduced for many sequences. I've err'd on the side of caution for this release by running many more iterations than probably needed.

## Acknowledgements
This repository owes a great deal to the following works and authors:
- [SMAL](http://smal.is.tue.mpg.de/); Zuffi et al. designed the SMAL deformable quadruped template model and have been wonderful for providing advice throughout my animal reconstruction PhD journey.
- [SMPLify](http://smplify.is.tue.mpg.de/); Bogo et al. provided the basis for our original ChumPY implementation and inspired the name of this repo.
- [SMALST](https://github.com/silviazuffi/smalst); Zuffi et al. provided a PyTorch implementations of the SMAL skinning functions which have been used here.

If you find this fitting code and/or BADJA dataset useful for your research, please consider citing the following paper:

```
@inproceedings{biggs2018creatures,
  title={{C}reatures great and {SMAL}: {R}ecovering the shape and motion of animals from video},
  author={Biggs, Benjamin and Roddick, Thomas and Fitzgibbon, Andrew and Cipolla, Roberto},
  booktitle={ACCV},
  year={2018}
}
```

if you make use of the limb scaling parameters, or Unity shape prior (on by default for the dog shape family) or the [StanfordExtra](https://github.com/benjiebob/StanfordExtra) dataset please cite [Who Left the Dogs Out? 3D Animal Reconstruction with Expectation Maximization in the Loop](https://arxiv.org/abs/2007.11110):

```
@inproceedings{biggs2020wldo,
  title={{W}ho left the dogs out?: {3D} animal reconstruction with expectation maximization in the loop},
  author={Biggs, Benjamin and Boyne, Oliver and Charles, James and Fitzgibbon, Andrew and Cipolla, Roberto},
  booktitle={ECCV},
  year={2020}
}
```

## Contribute
Please create a pull request or submit an issue if you would like to contribute.

## Licensing
(c) Benjamin Biggs, Oliver Boyne, James Charles, Andrew Fitzgibbon and Roberto Cipolla. Department of Engineering, University of Cambridge 2020

As of 02-NOV-2024, this dataset is now MIT licensed. Enjoy!

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the “Software”), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED “AS IS”, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

