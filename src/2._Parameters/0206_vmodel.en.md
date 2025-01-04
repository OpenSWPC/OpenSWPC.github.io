
# Velocity Model

## Choice of Velocity Model Type

Users can choose a uniform (`uni`), a layered homogeneous medium
(`lhm`), or a `NetCDF` (grd) file input (`grd`) velocity type. In
addition, a randomly inhomogeneous medium calculated by an external
program can be overlaid onto the model.

!!! Info "Parameters"

    **`vmodel_type`**
    : Specify the input velocity model. Choose from one of the following.

        **`’uni’`**
        :  Homogeneous medium with a free surface. The following additional parameters are required:

        **`’lhm’`**
        :  Layered Homogeneous Medium. A model with a constant physical property value for each depth layer.


        **`'lgm'`** (New in Version 25.01)
        : Linear Gradient Model: A model in which seismic wave velocity changes linearly within a layer.

        **`’grd’`**
        :   Velocity model input from `NetCDF` (GMT grd) files. 


        **`’user’`**
        :   Velocity model determined by user subroutine

Depending on the structural model chosen, the additional parameters required will differ as described below.        

###  `vmodel='uni'`

The following parameters are required:

!!! Info "Parameters"

    **`vp0`** 
    : P-wave velocity [km/s] in the uniform model.

    **`vs0`** 
    : S-wave velocity [km/s] in the uniform model.

    **`rho0`** 
    : Mass density [g/cm${}^3$] in the uniform model.

    **`qp0`** 
    : $Q_P$ of the uniform model.

    **`qs0`** 
    : $Q_S$ of the uniform model.

    **`topo0`** 
    : Topography depth in the uniform model. If this value is greater than zero, seawater is filled from $z=0$ to this depth.


### `vmodel='lhm' & 'lgm'`

In `lhm` (Layered Homogeneous Medium) and `lgm`, the interface depth and physical properties of each layer are specified in a table.

!!! Info "Parameters"


    **`fn_lhm`**
    : Medium specification file. Each line specifies the depth of
    the top of the layer, density, P-wave velocity, S-wave
    velocity, $Q_P$, and $Q_S$ below that depth. They must be
    separated by space(s) (see the following example). Lines
    starting with `#` will be neglected.
    ```C
    # depth  rho(g/cm^3)  vp(km/s)   vs(km/s)     Qp      Qs
    # -------------------------------------------------------
          0        2.300      5.50      3.14     600     300
          3        2.400      6.00      3.55     600     300
         18        2.800      6.70      3.83     600     300
         33        3.200      7.80      4.46     600     300
        100        3.300      8.00      4.57     600     300
        225        3.400      8.40      4.80     600     300
        325        3.500      8.60      4.91     600     300
        425        3.700      9.30      5.31     600     300
    ```

Both `lhm` and `lgm` are the same file (in both cases, the parameter name is `fn_lhm`), but the physical properties created from the same file are different. In the case of `lhm`, the layers below the first column are filled with uniform physical properties specified in the second to sixth columns. In contrast, in the case of `lgm`, the physical properties change linearly up to the layer one row below. However, the bottom layer (last row) behaves in the same way as `lhm`.

In order to understand these behaviors, consider the following simplified model. Although there are actually multiple values such as density, Vp, and Vs, we will explain them using a single velocity value.

In order to understand these behaviors, let us consider the following simplified model. There are actually multiple values such as density, Vp, and Vs, but we will explain them using a single wavespeed.

```c
# depth  velocity
# ----------------
      0        V1
     10        V2
     30        V3
     30        V4
     40        V5
```

The depth-dependent velocity structure created using this structural model file is as shown in the figure below.

![](../fig/lhm_lgm_layer.png)
/// caption
An example of the difference in depth-dependent velocity structure between `lhm` and `lgm`, which are created from the same structure specification file. `lhm` has uniform physical properties within a layer, whereas `lgm` has linear variation in physical properties within a layer.
///

In the case of `lhm`, the layers below the depth of each layer are filled with the physical properties of the line. In the above example, there are two layers at a depth of 30 km, and the physical properties `V3` and `V4` are specified for each, but in the case of `lhm`, the `V3` layer is ignored because it is 0 in thickness.

In the case of `lgm`, on the other hand, the physical properties of each layer change linearly with depth up to the value of the next layer. In this case, the different physical properties of the two 30 km-deep layers can also be used to express a discontinuous surface. If the same property value (`V1`) is applied over a certain range as an following example, it behaves in the same way as `lhm`: 


```c
 0 V1
10 V1
```

In other words, `lgm` can express all the structures that can be expressed by `lhm`. However, the structure specification files for `lhm` and `lgm` are different from each other so that the same structure is expressed by `lhm` and `lgm`.


### `vmodel='grd'` 

The following additional parameters are required for structural input using `NetCDF` files.


!!! Info "Parameters"

    **`dir_grd`**
    : Directory of the velocity structure (`NetCDF`) files.

    **`fn_grdslt`**
    : List file that specifies the grd files and the associated medium. 
    Each grd file specifies the depth of the top surface of the structural boundary. 
    The unit of depth is "m".　
    Each line contains the grd filename (with a single
    or double quotation mark; recommended), density, P-wave
    velocity, S-wave velocity, $Q_P$, $Q_S$, and the layer
    number integers (0-9) separated by spaces (see following
    example). Lines starting with `#` will be neglected. The
    layer number is used to specify the source or station depth
    fit to the layer depth. The first `NetCDF` file will be
    treated as the ground surface. If the depth of the ground
    surface is deeper than zero, the depth range from $z = 0$ to
    the surface is assumed to be an ocean layer. The grid above
    the free surface is treated as an air column.
    ```C
    # grd filename          rho    vp    vs    QP   QS  sw
    # -------------------------------------------------------
    'eJIVSM_01_TAB_.grd'   1.80  1.70  0.35   119    70  0 
    'eJIVSM_02_BSM_.grd'   1.95  1.80  0.50   170   100  0 
    'eJIVSM_03_BSM_.grd'   2.00  2.00  0.60   204   120  0 
    'eJIVSM_04_BSM_.grd'   2.05  2.10  0.70   238   140  0 
    'eJIVSM_05_BSM_.grd'   2.07  2.20  0.80   272   160  0 
    'eJIVSM_06_BSM_.grd'   2.10  2.30  0.90   306   180  0 
    'eJIVSM_07_BSM_.grd'   2.15  2.40  1.00   340   200  0 
    'eJIVSM_08_BSM_.grd'   2.20  2.70  1.30   442   260  0 
    'eJIVSM_09_BSM_.grd'   2.25  3.00  1.50   510   300  0 
    'eJIVSM_10_BSM_.grd'   2.30  3.20  1.70   578   340  0 
    'eJIVSM_11_BSM_.grd'   2.35  3.50  2.00   680   400  0 
    'eJIVSM_12_BSM_.grd'   2.45  4.20  2.40   680   400  0 
    'eJIVSM_13_BSM_.grd'   2.60  5.00  2.90   680   400  0 
    'eJIVSM_14_BSM_.grd'   2.65  5.50  3.20   680   400  0 
    'eJIVSM_15_UPC_.grd'   2.70  5.80  3.40   680   400  0 
    'eJIVSM_16_LWC_.grd'   2.80  6.40  3.80   680   400  0 
    'eJIVSM_17_CTM_.grd'   3.20  7.50  4.50   850   500  0 
    'eJIVSM_18_PH2_.grd'   2.40  5.00  2.90   340   200  1 
    'eJIVSM_19_PH3_.grd'   2.90  6.80  4.00   510   300  0 
    'eJIVSM_20_PHM_.grd'   3.20  8.00  4.70   850   500  0 
    'eJIVSM_21_PA2_.grd'   2.60  5.40  2.80   340   200  2 
    'eJIVSM_22_PA3_.grd'   2.80  6.50  3.50   510   300  0 
    'eJIVSM_23_PAM_.grd'   3.40  8.10  4.60   850   500  0 
    ```

    **`node_grd`**
    : MPI node to input the `NetCDF` data. All `NetCDF` files are
    first read by this node, and then, transferred to all nodes
    via MPI data communication.


    **`topo_flatten`** 
    : Make topography variatinon flat by offsetting the medium below. 

    !!! Warning "Renaming `is_flatten` to `topo_flatten`"
        This option used to be `is_flatten` until Version 5.1, but has been renamed to avoid confusion with the `earth_flattening` option implemented in Version 5.2.

### `vmodel='user'`

Any kind of velocity structure can be used by modifying a user subroutine defined in `src/swpc_*/m_vmodel_user.F90`. Note that **recompilation of the code
is necessary** if this Fortran file is modified. It is also easy to read in arbitrary parameters from a parameter file or to add parameters. Please refer the
comments in the file for the details. 


## On Treatments of Air and Seawater Layer

In `OpenSWPC`, the air column has a mass density of $\rho=0.001$ [g/cm${}^3$], 
velocities of $V_P = V_S = 0$ [km/s], and intrinsic
attenuation parameters of $Q^P = Q^S = 10^{10}$. 
The air column is treated as a
vacuum with no seismic wave propagation (with zero velocities). However,
**the mass density in the air column must not be zero to avoid division by zero**. 

In the ocean column, $\rho= 1.0$ [g/cm${}^3$], $V_S = 0.0$ [km/s],
and $Q^P = Q^S = 10^6$ are assumed. 
The attenuation of underwater sound waves is known to be very small, so we have introduced a very large $Q$ values that are virtually unattenuated. 
The P-wave velocity is $V_P=1.5$ km/s, but in the version 5.2 and later, the depth-dependent Munk's profile which is defined the equation below can be used by setting `munk_profile = .true.`. 

\begin{align*}
&V_P(z) = 1.5 \times \left[ 1.0 + 0.00737 \left( z_b - 1 + e^{-zb} \right) \right] \text{[m/s]}\\
&z_b = 2(z-1.3) /1.3
\end{align*}

where the unit of $z$ is km. This profile contains a minima which corresponds to the SOFAR channel at a depth of 1300 m. 

In the free
surface and seafloor, the reduced order of the finite difference is
performed according to Okamoto and Takenaka (2005[^OT2005]) and Maeda and Furumura (2003[^MF2003]). 
These discontinuities are automatically detected as boundaries that change $\mu$ and $\lambda$ from zero to a finite value.

[^OT2005]: Okamoto, T., and H. Takenaka (2005). Fluid-solid boundary implementation in the velocity-stress finite- difference method, _Zisin 2_, _57_, 355–364, doi:10.4294/zisin1948.57.3_355. (in Japanese with English Abstract) [(article link)](https://doi.org/10.4294/zisin1948.57.3_355)

[^MF2003]: Maeda, T., and T. Furumura (2013), FDM simulation of seismic waves, ocean acoustic waves, and tsunamis based on tsunami-coupled equations of motion, _Pure Appl. Geophys._, _170_(1-2), 109–127, doi:10.1007/s00024-011-0430-z. [(article link)](https://doi.org/10.1007/s00024-011-0430-z)

!!! Info "Parameters"

    **`is_ocean`** 
    : Seawater mode. When `.true.` (default), the surface of the sea (the shallowest surface) is filled with sea water from $z=0$. When `.false.`, the surface of the sea is all air, and the sea bed is treated as a free surface.

    **`munk_profile`** &nbsp; **(new in version 5.2)**
    : If this value is `.true.`, the Munk's profile with minima is applied in the seawater layer. Otherwise a constant value of 1.5 km/s is used. 


## Small-Scale Random Inhomogeneity

Users may overlay small-scale velocity inhomogeneities with specified
power-law spectra on the background velocity models of `’uni’`, `’lhm’`,
and `’grd’`. The small-scale velocity inhomogeneity $\xi$ is defined by
external files. From the average velocities $V_{P0}$, $V_{S0}$, and
$\rho_0$, the fluctuated velocities and density are given as

$$
\begin{align}
  V_P = V_{P0} \left( 1 + \xi \right) , \, V_S = V_{S0} \left( 1 + \xi \right) , \,
  \rho = \rho_0 \left( 1 + \nu \xi \right)
\end{align}
$$

where $\nu=0.8$ is a scaling parameter based on a laboratory experiment (Sato et al., 2012[^SFM2012]).

[^SFM2012]: Sato, H., M. C. Fehler, and T. Maeda (2012), _Seismic Wave Propagation and Scattering in the Heterogeneous Earth: Second Edition_, Springer Berlin Heidelberg, Berlin, Heidelberg, doi:10.1007/978-3-642-23029-5.

The random media files are generated by a separated program. See [this section](../3._Tools/0304_rmedia.en.md) for details. 
The random media are given as two- or three-dimensional `NetCDF` files.
At each grid location, the velocity fluctuation $\xi(I,J,K)$ is defined.
The code automatically reads the corresponding volume from the file; It
is not necessary to decompose the `NetCDF` files into parts for parallel
computation. If the computational size (`Nx, Ny, Nz`) is larger than the
random media file size, the media is used repeatedly by applying a
circular boundary condition. The simulation codes do not care if the
grid sizes of the simulation and the input random media file are
identical.


!!! Info "Parameters"

    **`vmodel_type`**
    : Velocity structure model type specification.

        **`'uni_rmed'`**
        : Uniform homogeneous structure + random medium

        **`'lhm_rmed'`**
        : Layered homogeneous medium + random medium

        **`'lgm_rmed'`**
        : Linear Gradient Model + random medium

        **`'grd_rmed'`**
        : Structure input using `NetCDF` files + random medium

    **`dir_rmed`**
    : A directory name for storing the random media data files.



Note that users may use the 3D random media files for 2D calculation. 

### `vmodel='uni_rmed'`

The following parameter is required in addition to the parameters used
in `vmodel=’uni’`.

!!! Info "Parameters"
    **`fn_rmed0`**
    : Name of the random medium file.

In this model, the average velocity will be fluctuated based on the
input `fn_rmed0`.

### `vmodel='lhm_rmed' & 'lgm_rmed'`

In this model, the small-scale velocity fluctuation is applied to every
layer defined by `vmodel=’lhm’`. It is possible to assign different
random velocity models at different layers.

The following parameter is substituted in `fn_lhm`:

!!! Info "Parameters"

    **`fn_lhm_rmed`**
    : List file of the velocity structure.
    The list file has a similar format to `fn_lhm`; it contains the
    filenames of the random media files in the rightmost column as in the
    following example.
    ``` c
    # depth  rho(g/cm^3)  vp(km/s)   vs(km/s)     Qp      Qs     fn_rmed
    # ----------------------------------------------------------------------
         0        2.300      5.50      3.14     600     300     rmedia1.nc
         3        2.400      6.00      3.55     600     300     rmedia1.nc
        18        2.800      6.70      3.83     600     300     rmedia2.nc
        33        3.200      7.80      4.46     600     300     rmedia2.nc
        100       3.300      8.00      4.57     600     300     -
        225       3.400      8.40      4.80     600     300     -
        325       3.500      8.60      4.91     600     300     -
        425       3.700      9.30      5.31     600     300     -
    ```
    In this example, the layers starting from depths of 0 km and 3 km have
    fluctuations defined in `rmedia1.nc`, and the layers from 18 km and 33
    km are defined in `rmedia2.nc`. For the layer deeper than 100 km, a
    dummy filename (`-`) is given. In this case (i.e., there is no file
    found), a fluctuation will not be given. The dummy filename is mandatory
    in this case.


### `vmodel='grd_rmed'`

When overlaying the random fluctuations to the layers defined by the
model of `vmodel=’grd’`, it is possible to assign different random media
to different layers. The starting depth of the velocity fluctuation can
be either the free surface or depths defined by a layer.

The filename of the velocity fluctuation is given by the following
parameter:


!!! Info "Parameters"

    **`fn_grdlst_rmed`**
    : A list file that specifies the velocity layer and the random
    fluctuation files for each layer.
    The list file has two additional columns at the right: the filename of
    the random medium and the reference layer number.
    ```C
    # grd filename          rho    vp    vs   QP   QS  sw  fn_rmed      ref
    # -----------------------------------------------------------------------
    'eJIVSM_01_TAB_.grd'   1.80  1.70  0.35   119    70  0 'rmed3d_1.nc' 0
    'eJIVSM_02_BSM_.grd'   1.95  1.80  0.50   170   100  0 'rmed3d_1.nc' 0
    'eJIVSM_03_BSM_.grd'   2.00  2.00  0.60   204   120  0 'rmed3d_1.nc' 0
    'eJIVSM_04_BSM_.grd'   2.05  2.10  0.70   238   140  0 'rmed3d_1.nc' 0
    'eJIVSM_05_BSM_.grd'   2.07  2.20  0.80   272   160  0 'rmed3d_1.nc' 0
    'eJIVSM_06_BSM_.grd'   2.10  2.30  0.90   306   180  0 'rmed3d_1.nc' 0
    'eJIVSM_07_BSM_.grd'   2.15  2.40  1.00   340   200  0 'rmed3d_1.nc' 0
    'eJIVSM_08_BSM_.grd'   2.20  2.70  1.30   442   260  0 'rmed3d_1.nc' 0
    'eJIVSM_09_BSM_.grd'   2.25  3.00  1.50   510   300  0 'rmed3d_1.nc' 0
    'eJIVSM_10_BSM_.grd'   2.30  3.20  1.70   578   340  0 'rmed3d_1.nc' 0
    'eJIVSM_11_BSM_.grd'   2.35  3.50  2.00   680   400  0 'rmed3d_1.nc' 0
    'eJIVSM_12_BSM_.grd'   2.45  4.20  2.40   680   400  0 'rmed3d_1.nc' 0
    'eJIVSM_13_BSM_.grd'   2.60  5.00  2.90   680   400  0 'rmed3d_1.nc' 0
    'eJIVSM_14_BSM_.grd'   2.65  5.50  3.20   680   400  0 'rmed3d_1.nc' 0
    'eJIVSM_15_UPC_.grd'   2.70  5.80  3.40   680   400  0 'rmed3d_1.nc' 0
    'eJIVSM_16_LWC_.grd'   2.80  6.40  3.80   680   400  0 'rmed3d_3.nc' 0
    'eJIVSM_17_CTM_.grd'   3.20  7.50  4.50   850   500  0 'rmed3d_3.nc' 0
    'eJIVSM_18_PH2_.grd'   2.40  5.00  2.90   340   200  1 'rmed3d_2.nc' 18
    'eJIVSM_19_PH3_.grd'   2.90  6.80  4.00   510   300  0 'rmed3d_2.nc' 18
    'eJIVSM_20_PHM_.grd'   3.20  8.00  4.70   850   500  0 'rmed3d_3.nc' 18
    'eJIVSM_21_PA2_.grd'   2.60  5.40  2.80   340   200  2 'rmed3d_2.nc' 21
    'eJIVSM_22_PA3_.grd'   2.80  6.50  3.50   510   300  0 'rmed3d_2.nc' 21
    'eJIVSM_23_PAM_.grd'   3.40  8.10  4.60   850   500  0 'rmed3d_3.nc' 21
    ```

The reference layer number defines the reference depth plane of the
random media. If this number is zero, the depth grid number of the
computational model is directly used to assign the random media. This is
exactly the same as the behavior of the `uni_rmed` or `lhm_rmed` models.
If the nonzero value of the reference layer number `NR` is specified,
the depth of the `NR` layer is treated as the base plane. The depth grid
of the random medium is measured from this depth. Introducing this
reference plane, the inclined random media according to the velocity
discontinuity (such as the plate boundary) can be specified. In the
above example, the 18th and 21st layers are treated as the references of
18--20th and 21--23th layers, respectively.

### Truncation of Velocity Fluctuations

If the magnitude of the velocity fluctuation becomes too large, there
can be a spot with non-physical velocity, such as negative velocity or a
velocity too large for the Earth medium. The simulation may be unstable
under the following conditions:

1.  The fluctuated velocity $V=(1+\xi)V_0$ exceeds the stability
    condition for cases with $\xi>0$.

2.  The velocity has unrealistic negative values for cases with
    $\xi<-1.0$.

3.  The mass density has negative values for cases with $\xi<-1.25$.

To avoid such situations, `OpenSWPC` automatically limits the range of
the fluctuated velocity to `vcut`$\le v \le \times v_\text{max}$,
where `vcut` is an input parameter and $v_\text{max}$ is the maximum
possible velocity derived from the stability condition.

In addition, the following parameter controls the minimum density.

!!! Info "Parameters"
    **`rhomin`**
    : Minimum mass density in g/cm${}^3$. (1.0 g/cm${}^3$ by default.)


## Truncation of Low-Velocity Layer

If the existing ground structure model is used as it is, the $S$ wave velocity near the ground surface in particular may become extremely low, and in some cases the wavelength condition may not be satisfied. It is possible to modify the structure model each time, but in OpenSWPC, the low velocity can be truncated automatically using the following parameters.

!!! Infor "Parameters"

    **`vcut`**
    : Cut-off velocity. For the `’lhm’` or `’grd’` models, a velocity
    slower than this value will be overwritten by the `vcut` value. This
    parameter is used to avoid wavelengths that are too short and
    violate the wavelength condition (the wavelength is recommended to
    be longer than 5-10 grids). This substitution will not be performed
    in the oceanic area.


The behavior of `vcut` is somewhat complex in order to maintain the Vp/Vs speed ratio. The `vcut` decision is made for the input file (`fn_lhm`, `fn_grd`, etc.).


## Earth-flattening transformation

OpenSWPC performs calculations in the Cartesian coordinate system, but when the propagation distance exceeds several hundred kilometers, the spherical effect of the Earth cannot be ignored.

To address this, the depth of the velocity structure input is treated as the depth $z_s$ from the Earth's surface, which is approximated by a sphere, and the Earth-flattening transformation (e.g., Aki and Richards, Box 9.2) is implemented to convert it to an approximately equivalent horizontal stratified structure.

!!! Info "Parameters"

    ** `earth_flattening`**  (New in Version 5.2)
    : If this option is `.true.`, OpenSWPC performs the transformation the Cartesian coordinate to pseudo-spherical coordinate by means of the Earth-flattening tranformation 

When this option is specified, the depth $z_s$ on the sphere is converted to the depth $z_f$ in the Cartesian coordinate system as follows.

$$
\begin{align}
z_f = R \log \frac{R}{R-z_s} \quad \Leftrightarrow \quad 
z_s = R \left\{1 - \exp \left[ - \frac{z_f}{R} \right] \right\}
\end{align}
$$

where $R$ is the radius of the Earth. The wavespeeds and mass density corresponding to the depth is converted as follows.

$$
\begin{align}
& \alpha_f = \frac{R}{R-z_s} \alpha_s = \exp\left[\frac{z_f}{R}\right] \alpha_s
\\
& \beta_f = \frac{R}{R-z_s} \beta_s = \exp\left[\frac{z_f}{R}\right] \beta_s
\\
& \rho_f = \left(\frac{R-z_s}{R}\right)^{m+2} \rho_s = \exp\left[-(m+2)\frac{z_f}{R}\right] \rho_s
\end{align}
$$

Here, $m$ is an integer, and $m=3$ is used for SH waves, and $m=-2$ is used for $P$-$SV$ waves and 3D.
