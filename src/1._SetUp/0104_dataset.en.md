# Dataset Preparation

## Subsurface Velocity Structure Model

In `OpenSWPC`, a 3D inhomogeneous medium is represented as a set of
velocity discontinuities using `NetCDF`-formatted files. 
As an example of the velocity structure beneath the Japanese
Archipelago, an automatic model generation script for the [Japan
Integrated Velocity Structure Model (JIVSM)](https://taro.eri.u-tokyo.ac.jp/saigai/computerE.html#2008), developed by Koketsu et al. (2012[^Koketsu2012])  and the Headquarters for Earthquake Research Promotion in
Japan, is included. An extension of JIVSM (eJIVSM), which covers a wider area, is also provided. These velocity structure models contain the ground surface (topography and bathymetry), subsurface soil, Moho, and oceanic crust of the two subducting plates. To generate these models, the Generic Mapping Tools (GMT) is required. If the user does not use this model, the following steps may not be necessary.

[^Koketsu2012]: Koketsu, K., H. Miyake, and H. Suzuki (2012). Japan Integrated Velocity Structure Model Version 1, Proceedings of the 15th World Conference on Earthquake Engineering, Paper No. 1773.

![](../fig/jivsm_ext_section.png)
/// caption
Area of JIVSM and eJIVSM. The colored area in the map is where the original JIVSM is defined. eJIVSM is extended to the gray-shaded area via an extrapolation. The surrounding graphs show the depth sections along the lines on the map of the model.
///

To build this model, first go to  `dataset/vmodel` directory, and then execute the following shell script:
```bash
$ ./gen_JIVSM.sh
```

After a successful execution, 23 `NetCDF`-formatted files will be generated in the two model directories for `JIVSM` and `eJIVSM`. These files can be read and visualized in GMT by the `grdimage` or `grd2xyz` modules. The netcdf filename contains five integer numbers, which correspond to mass density (in kg/m${}^3$), P wavespeed (m/s), S wavespeed (m/s), $Q_P$, and $Q_S$. They indicate the material information below the discontinuity defined in the file. List files of these `NetCDF` files (`jivsm.lst` and `ejivsm.lst`) for use in the `OpenSWPC` will also be generated. 


## Station list

An example script to generate a station list file is stored in `dataset/station/gen_stlst_hinet.sh`. This script generates a formatted list of the high-sensitivity seismograph network Japan (Hi-net) provided by the National Research Institute for Earth Science and Disaster Resilience (NIED). 

To use this script, first download the station csv list from the Hi-net website following the comments in the `gen_stlst_hinet.sh` file. Then, executing this bash script will result in the station list file for `OpenSWPC`.

