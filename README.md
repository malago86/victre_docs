# VICTRE PIPELINE DOCUMENTATION

**Author**: Miguel A. Lago

## Documentation

You can find the documentation in this link: https://malago86.github.io/victre_docs/annotated.html

Report your issues and requests here: https://github.com/malago86/VICTRE_PIPELINE

## Introduction

The Victre automated pipeline currently supports the following steps:
- Mass lesion generation (using [breastMass](https://github.com/DIDSR/breastMass))
- X-Ray Projection (using [VICTRE_MCGPU](https://github.com/DIDSR/VICTRE_MCGPU))
- DBT Reconstruction ([customized code](https://github.com/DIDSR/VICTRE/tree/master/FBP%20DBT%20reconstruction%20in%20C))
- Region of Interest extraction

## Example

### Step 1: Create the pipeline object
To start a new pipeline object use this function:

```
import Victre
pline = Victre.Pipeline(ip={"cpu": CPU_IP, "gpu": GPU_IP},
                        phantom_file=PHANTOM_FILE)
```

- `ip`: dictionary with the following two fields (use `localhost` if running in local machine)
   - `gpu`: IP address for the GPU enabled node that will run the projection
   - `cpu`: IP address for a computing node
- `phantom_file`: path to a previously generated voxelized phantom in raw format

Optional parameters:
- `seed`: random seed used for the lesion insertion and ROI extraction
- `results_folder`: Path to folder to be used when saving the results
- `spectrum_file`: path to a previously generated spectrum file in raw format
- `lesion_file` Path to file containing the lesion to be inserted (in HDF5 format)
- `materials` Dictionary including the materials to be used during projection (see Constants for an example of this dictionary)
- `arguments_mcgpu` Arguments to be overridden for the projection in MCGPU
- `arguments_recon` Arguments to be overridden for the reconstruction algorithm
- `flatfield_file` Path to the flatfield file for projection

### Step 2: Insert lesions

Create the region of interest dictionary for the lesions you want to insert:

```
roi_sizes = {Constants.VICTRE_SPICULATED: [109, 109, 9],
             Constants.VICTRE_CLUSTERCALC: [65, 65, 5]}
```
Call the `insert_lesions` method:
```
pline.insert_lesions(lesion_type=Constants.VICTRE_SPICULATED,
                     n=2,
                     roi_sizes=roi_sizes)
```
- `lesion_type`: see Constants for a list of available lesion types
- `n`: number of lesions to be inserted
- `roi_sizes`: dictionary with the size of the regions of interest for each lesion

> If the number of lesions is too high or the lesion size is too large, the insertion might not be successful. 

### Step 3: Project and reconstruct

Simply call the following methods:
```
pline.project()
pline.reconstruct()
```

The process will take several minutes, depending on the speed and number of GPUs available.

### Step 4: Add absent regions of interest

If you want to extract regions of interest without any lesion, call the `add_absent_ROIs` method:

``` 
pline.add_absent_ROIs(lesion_type=Constants.VICTRE_SPICULATED,
                      n=2,
                      roi_sizes=roi_sizes)
```

Parameters and conditions are the same as for `insert_lesions`.

### Step 5: Extract ROIs

To extract the ROIs and save them, use the method `save_ROIs` with the dictionary for the `roi_sizes` to be used for each type of lesion inserted.

```
save_ROIs(roi_sizes=roi_sizes)
```

## Results

The Victre pipeline saves the results of each step this way:
- **Insertion**: Results from the lesion insertion will be saved in the `phantom` subfolder as:
  - One gzip compressed raw file containing the voxelized phantom
  - One text file (extension `.loc`) with the locations coordinates in the original phantom for each lesion and the lesion type (negative lesion types are for absent regions of interest)
- **Projection**: Results from the projection will be saved in the `results` folder in their corresponding seed folder:
  - A number of projection raw files from `projection_0001.raw` to `projection_0025.raw` (or the number of projections requested)
  - One projection corresponding to the digital mammography projection `projection_0000.raw`
  - The input and output file for the MCGPU software `input_mcgpu.in` and `output_mcgpu.out`
- **Reconstruction**: Results from the reconstruction will be saved along the results from the projection:
  - One file `reconstruction.raw` with the results from the reconstruction software for the DBT image
  - One file `projection_3000x1500pixels_25proj.raw` that is all the stacked projection slices
  - The input and output file for the reconstruction software `input_recon.in` and `output_recon.out`
- **ROI**: Results from the ROI extraction will be saved in two different ways along with the other result files:
  - All the ROIs together saved as an HDF5 file `ROIs.h5` 
  - Each ROI saved in raw format inside the `ROIs` subfolder