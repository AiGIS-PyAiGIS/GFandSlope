# GFandSlope
CUDA-powered code for calculating gravitational fields and surface slope of irregular shaped small bodies.

## Requirements
### Main code
- NVIDIA CUDA Compiler: `nvcc`

### Utilities
- Python
- [PyVista](https://docs.pyvista.org)

## Compile
You will get an executable file `GFandSlope` by running `make` command with `Makefile` in `src/`. The best architecture of GPU will be selected automatically.

## Input
`GFandSlope` require three input files.

1. Configuration file
2. Shape model file
3. Point list file (optional)

### Configuration file
This file specifies initial conditions and file IO settings for the computation. A sample .ipynb notebook doesn't require to prepare a configuration file, because it create a file within the notebook script.

#### With a point list file

```
// PERIOD[hour], DENSITY[kg/m^3]
period: 12.1324
density: 1950
input_polygon: itokawa_f0049152.txt
input_points: itokawa_f0049152_center.txt
output: itokawa_49k_ro1950_P12.1324h_center.txt
gpu: 0
```

#### Without a point list file

If a point list file is not specified (`NONE`), the center of faces of polygon shape model will be referred.

```
// PERIOD[hour], DENSITY[kg/m^3]
period: 12.1324
density: 1950
input_polygon: itokawa_f0049152.txt
input_points: NONE
output: itokawa_49k_ro1950_P12.1324h.txt
gpu: 0
```

#### Items in a configuration file

Each property must be given by the following format (e.g.  "period: 12.1324") in a line.

`[Tag]: [Argument]`

Any other lines are ignored (regarded as comments). Following tags are available for the configuration.

- `input_polygon `: File name for the target shape model data processed by GFandSlope.
- `input_points `: File name for the point list data processed by GFandSlope. If the argument is `NONE`, the center of faces of polygon shape model will be referred.
- `output`: File name which is associated with the computation result data produced by GFandSlope.
- `period`: Rotation period of the model in hour. If the period is `0.0`, the centrifugal force is ignored (or no-rotation).
- `density`: Density of the model in SI derived unit (kg/m^3).
- `gpu`: An integer to designate a target device ID (available devices with their ID can be confirmed when you execute GFandSlope CUDA). If you have several available GPUs on your computer system, you can specify the target device used for the computation. Can be omitted if there is only one available device (the default is 0).

### Shape model

A simple list of vertices and faces of the model.

```
N
1 x_1 y_1 z_1
2 x_3 y_2 z_2
...
i x_i y_i z_i
...
N x_N y_N z_N
M
1 v_1 v_2 v_3
...
j v_j1 v_j2 v_j3
...
M v_M1 v_M2 v_M3
```

Where N and M are the total numbers of vertices and faces of the model, x_i, y_i and z_i are the coordinate of the i-th vertex, and v_j1, v_j2 and v_j3 are the list of vertices (IDs) forming the j-th face.

A utility `Mesh2GFandSlopeInput.py` can convert a shape model with various formats (obj, stl...) to this format.

### Point list

A simple list of points where gravitation will be computed.

```
N
1 x_1 y_1 z_1
2 x_3 y_2 z_2
...
i x_i y_i z_i
...
N x_N y_N z_N
```

A utility `GetFaceCentersOfMesh.py` can compute centers of a given shape model and output with this format.

## Execution
`GFandSlope` requires a configuration file as an argument.

```
./GFandSlope configuration_file
```

Also, it accepts multiple configuration files at once.

```
./GFandSlope configuration_file1 configuration_file2 ...
```

## Output
Output is a space-delimitated text file, starting with comments with `#`. All coordinates are described in km, and SI units for other values.

#### Output with a point list file

If a point list file is specified, the output include result at every point in the list.

- `ID`: ID of the point
- `Point.{x,y,z}`: Coordinate of the points
- `Lon` and `Lat`: Longitude and Latitude of the points
- `CRefAcc.{x,y,z}`: Centrifugal acceleration components
- `GravAcc.{x,y,z}`: Gravity acceleration components
- `TotalAcc.{x,y,z}`: Sum of the centrifugal and gravity accelerations
- `{G,R,T}potential`: Gravitational, rotational and total potentials

```
# Shape model: itokawa_f3145728.txt
# Number of Polygons: 3145728
# Point list: itokawa_f3145728_center.txt
# Number of Points: 3145728
# Density: 1950.000000 [kg/m^3]
# Rotational Period: 12.132400 [h]
# Unit: All coordinates in km, and SI units for other values.
ID Point.x Point.y Point.z Lon Lat CRefAcc.x CRefAcc.y CRefAcc.z GravAcc.x GravAcc.y GravAcc.z TotalAcc.x TotalAcc.y TotalAcc.z Gpotential Rpotential Tpotential 
1 -0.151073 0.081720 0.075153 151.589743 23.631778 -3.126433e-06 1.691179e-06 -0.000000e+00 -3.297283e-05 3.496578e-05 6.141559e-05 -2.984640e-05 3.327460e-05 6.141559e-05 -1.360889e-02 3.052619e-04 -1.391415e-02 
2 -0.151250 0.081597 0.075173 151.653933 23.625520 -3.130089e-06 1.688627e-06 -0.000000e+00 -3.304135e-05 3.486759e-05 6.142406e-05 -2.991126e-05 3.317897e-05 6.142406e-05 -1.360579e-02 3.056061e-04 -1.391140e-02 
```

#### Output without a point list file

If a point list file is not specified, the center of faces of polygon shape model will be referred and polygon specific information are included in the output.

- `ID`: ID of the polygon face/points
- `GeopotentialSlope`: Local slope in degree (computed from the normal vector and the total acceleration vector)
- `Normal.{x,y,z}`: Normal vector components
- `Area`: Area of the polygon face

```
# Shape model: itokawa_f3145728.txt
# Number of Polygons: 3145728
# Point list: NONE
# Number of Points: 3145728 (polygon centers)
# Density: 1950.000000 [kg/m^3]
# Rotational Period: 12.132400 [h]
# Unit: All coordinates in km, and SI units for other values.
ID Point.x Point.y Point.z Lon Lat CRefAcc.x CRefAcc.y CRefAcc.z GravAcc.x GravAcc.y GravAcc.z TotalAcc.x TotalAcc.y TotalAcc.z Gpotential Rpotential Tpotential GeopotentialSlope Normal.x Normal.y Normal.z Area 
1 -0.151057 0.081707 0.075103 151.591011 23.620349 -3.126088e-06 1.690903e-06 -0.000000e+00 -3.299010e-05 3.499693e-05 6.145585e-05 -2.986401e-05 3.330602e-05 6.145585e-05 -1.361305e-02 3.051873e-04 -1.391824e-02 1.034886e+01 -5.520000e-02 8.270000e-02 1.784000e-01 1.021187e-07 
2 -0.151230 0.081580 0.075120 151.655658 23.613715 -3.129675e-06 1.688282e-06 -0.000000e+00 -3.305861e-05 3.490331e-05 6.146795e-05 -2.992893e-05 3.321503e-05 6.146795e-05 -1.361040e-02 3.055154e-04 -1.391592e-02 1.229320e+01 -4.680000e-02 1.038000e-01 1.782000e-01 1.021187e-07 
```

## Utilities
Two utility python scripts available to prepare input files. They depend on [PyVista](https://docs.pyvista.org).

- `GetFaceCentersOfMesh.py`: Compute centers of a given shape model and output with this format.
- `Mesh2GFandSlopeInput.py`: Convert a shape model with various formats (obj, stl...) to this format.

## Sample .pynb notebook
A sample Jupyter Notebook file to compute gravity on a Google Colab. environment.

## Non-GPU version
Non-GPU version (CPU-based) code is available at [Shape models, their derivatives, and relating data and tools of the asteroid Ryugu used in the paper "Hayabusa2 observations of the top-shape carbonaceous asteroid 162173 Ryugu" by Watanabe et al. (2019).](https://data.darts.isas.jaxa.jp/pub/hayabusa2/paper/Watanabe_2019/)

## References
1. Kono, F., Nakasato, N., Hirata, N. & Matsumoto, K. Acceleration of Gravitation Field Analysis for Asteroids by GPU Computation. 2021 IEEE 14th Int. Symp. Embed. MulticoreMany-core Syst.--Chip (MCSoC) 00, 8–15 (2021). [doi:10.1109/mcsoc51149.2021.00010](http://doi.org/10.1109/mcsoc51149.2021.00010)
2. Hirata, N., Matsumoto K., Kono F. & Nakasato N. GPU-Accelerated Gravity-Field Computation for Irregularly Shaped Small Bodies Using CUDA. The 57th Lunar and Planetary Science Conference (LPSC), #1418, (2026).

## Acknowledgements
GFandSlope is developed as a research project by the [University of Aizu](https://u-aizu.ac.jp/en/), [Shizuoka Institute of Science and Technology](https://www.sist.ac.jp/en/) and [National Astronomical Observatory of Japan](https://www.nao.ac.jp/en/).

The project is/was supported by:

- [ARC-Space](https://arcspace.jp), the University of Aizu, Distinctive Joint Research Center
  supported by MEXT Grant Number
  JPMXP0619217839/JPMXP0622717003/JPMXP0723830458.

### Shape model

A simple list of vertices and faces of the model.

```
N
1 x_1 y_1 z_1
2 x_3 y_2 z_2
...
i x_i y_i z_i
...
N x_N y_N z_N
M
1 v_1 v_2 v_3
...
j v_j1 v_j2 v_j3
...
M v_M1 v_M2 v_M3
```

Where N and M are the total numbers of vertices and faces of the model, x_i, y_i and z_i are the coordinate of the i-th vertex, and v_j1, v_j2 and v_j3 are the list of vertices (IDs) forming the j-th face.

A utility `Mesh2GFandSlopeInput.py` can convert a shape model with various formats (obj, stl...) to this format.

### Point list

A simple list of points where gravitation will be computed.

```
N
1 x_1 y_1 z_1
2 x_3 y_2 z_2
...
i x_i y_i z_i
...
N x_N y_N z_N
```

A utility `GetFaceCentersOfMesh.py` can compute centers of a given shape model and output with this format.

## Execution
`GFandSlope` requires a configuration file as an argument.

```
./GFandSlope configuration_file
```

Also, it accepts multiple configuration files at once.

```
./GFandSlope configuration_file1 configuration_file2 ...
```

## Output
Output is a space-delimitated text file, starting with comments with `#`. All coordinates are described in km, and SI units for other values.

#### Output with a point list file

If a point list file is specified, the output include result at every point in the list.

- `ID`: ID of the point
- `Point.{x,y,z}`: Coordinate of the points
- `Lon` and `Lat`: Longitude and Latitude of the points
- `CRefAcc.{x,y,z}`: Centrifugal acceleration components
- `GravAcc.{x,y,z}`: Gravity acceleration components
- `TotalAcc.{x,y,z}`: Sum of the centrifugal and gravity accelerations
- `{G,R,T}potential`: Gravitational, rotational and total potentials

```
# Shape model: itokawa_f3145728.txt
# Number of Polygons: 3145728
# Point list: itokawa_f3145728_center.txt
# Number of Points: 3145728
# Density: 1950.000000 [kg/m^3]
# Rotational Period: 12.132400 [h]
# Unit: All coordinates in km, and SI units for other values.
ID Point.x Point.y Point.z Lon Lat CRefAcc.x CRefAcc.y CRefAcc.z GravAcc.x GravAcc.y GravAcc.z TotalAcc.x TotalAcc.y TotalAcc.z Gpotential Rpotential Tpotential 
1 -0.151073 0.081720 0.075153 151.589743 23.631778 -3.126433e-06 1.691179e-06 -0.000000e+00 -3.297283e-05 3.496578e-05 6.141559e-05 -2.984640e-05 3.327460e-05 6.141559e-05 -1.360889e-02 3.052619e-04 -1.391415e-02 
2 -0.151250 0.081597 0.075173 151.653933 23.625520 -3.130089e-06 1.688627e-06 -0.000000e+00 -3.304135e-05 3.486759e-05 6.142406e-05 -2.991126e-05 3.317897e-05 6.142406e-05 -1.360579e-02 3.056061e-04 -1.391140e-02 
```

#### Output without a point list file

If a point list file is not specified, the center of faces of polygon shape model will be referred and polygon specific information are included in the output.

- `ID`: ID of the polygon face/points
- `GeopotentialSlope`: Local slope in degree (computed from the normal vector and the total acceleration vector)
- `Normal.{x,y,z}`: Normal vector components
- `Area`: Area of the polygon face

```
# Shape model: itokawa_f3145728.txt
# Number of Polygons: 3145728
# Point list: NONE
# Number of Points: 3145728 (polygon centers)
# Density: 1950.000000 [kg/m^3]
# Rotational Period: 12.132400 [h]
# Unit: All coordinates in km, and SI units for other values.
ID Point.x Point.y Point.z Lon Lat CRefAcc.x CRefAcc.y CRefAcc.z GravAcc.x GravAcc.y GravAcc.z TotalAcc.x TotalAcc.y TotalAcc.z Gpotential Rpotential Tpotential GeopotentialSlope Normal.x Normal.y Normal.z Area 
1 -0.151057 0.081707 0.075103 151.591011 23.620349 -3.126088e-06 1.690903e-06 -0.000000e+00 -3.299010e-05 3.499693e-05 6.145585e-05 -2.986401e-05 3.330602e-05 6.145585e-05 -1.361305e-02 3.051873e-04 -1.391824e-02 1.034886e+01 -5.520000e-02 8.270000e-02 1.784000e-01 1.021187e-07 
2 -0.151230 0.081580 0.075120 151.655658 23.613715 -3.129675e-06 1.688282e-06 -0.000000e+00 -3.305861e-05 3.490331e-05 6.146795e-05 -2.992893e-05 3.321503e-05 6.146795e-05 -1.361040e-02 3.055154e-04 -1.391592e-02 1.229320e+01 -4.680000e-02 1.038000e-01 1.782000e-01 1.021187e-07 
```

## Non-GPU version
Non-GPU version (CPU-based) code is available at [Shape models, their derivatives, and relating data and tools of the asteroid Ryugu used in the paper "Hayabusa2 observations of the top-shape carbonaceous asteroid 162173 Ryugu" by Watanabe et al. (2019).](https://data.darts.isas.jaxa.jp/pub/hayabusa2/paper/Watanabe_2019/)

## References
1. Kono, F., Nakasato, N., Hirata, N. & Matsumoto, K. Acceleration of Gravitation Field Analysis for Asteroids by GPU Computation. 2021 IEEE 14th Int. Symp. Embed. MulticoreMany-core Syst.--Chip (MCSoC) 00, 8–15 (2021). [doi:10.1109/mcsoc51149.2021.00010](http://doi.org/10.1109/mcsoc51149.2021.00010)
2. Hirata, N., Matsumoto K., Kono F. & Nakasato N. GPU-Accelerated Gravity-Field Computation for Irregularly Shaped Small Bodies Using CUDA. The 57th Lunar and Planetary Science Conference (LPSC), #1418, (2026).

## Acknowledgements
GFandSlope is developed as a research project by the [University of Aizu](https://u-aizu.ac.jp/en/), [Shizuoka Institute of Science and Technology](https://www.sist.ac.jp/en/) and [National Astronomical Observatory of Japan](https://www.nao.ac.jp/en/).

The project is/was supported by:

- [ARC-Space](https://arcspace.jp), the University of Aizu, Distinctive Joint Research Center
  supported by MEXT Grant Number
  JPMXP0619217839/JPMXP0622717003/JPMXP0723830458.
