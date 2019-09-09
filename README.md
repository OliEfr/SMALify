WARNING: This public respository is still in beta... There has been a considerable rewrite (particularly swapping ChumPy with Neural Mesh Renderer, using PyTorch etc.) since the original paper. I am bringing this up to speed in my free moments, but for now, please use at your own risk :)

# Creatures Great and SMAL
Fitting code used as part of the [Creatures Great and SMAL](https://arxiv.org/abs/1811.05804) paper.

## Installation
1. Clone the repository with submodules and enter directory
   ```
   git clone --recurse-submodules https://github.com/benjiebob/CreaturesGreatAndSMAL
   cd CreaturesGreatAndSMAL
    
2. Install the Neural Mesh Renderer submodule (which originated from my [fork](https://github.com/benjiebob/neural_renderer)), which includes a render_points function

   ```
   pip install -e neural_renderer
   ```

3. Download texture map (from smal/dog_texture.pkl) and a version of SMAL 2017 converted to NumPy (smal_CVPR2017_np.pkl) from [my Google Drive](https://drive.google.com/open?id=1gPwA_tl1qrKiUkveE8PTsEOEMHtTw8br) and place under the smal folder

4. Install dependencies, particularly [PyTorch (with cuda support)](https://pytorch.org/)

5. Create a checkpoints directory

   ```
   mkdir smal_fitter/checkpoints
   ```

6. Run the python3 script
   ```
   cd smal_fitter
   python optimize_to_joints.py
   ```

## Usage
I am currently running a hyperparameter sweep for the rs_dog sequence. Will upload this & a results gif as soon as I can.

To run on your own videos, you require silhouette segmentations and joint annotations. View the example BADJA data as a starting point. You may then also need to slightly tune the hyperparameter weights, particularly if your sequence depicts fast motion or contains lots of rotation. 

Set the shape_family id (according to the animal type) for best results.

### Acknowledgements
If you find this fitting code useful for your research, please consider citing the following paper:

```
@inproceedings{biggs2018creatures,
  title={{C}reatures great and {SMAL}: {R}ecovering the shape and motion of animals from video},
  author={Biggs, Benjamin and Roddick, Thomas and Fitzgibbon, Andrew and Cipolla, Roberto},
  booktitle={ACCV},
  year={2018}
}
```

## Contribute
Please create a pull request or submit an issue if you would like to contribute.
