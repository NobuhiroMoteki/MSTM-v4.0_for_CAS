# MSTM-v4.0_for_CAS

## 📌 Description
This is a modified version of the fortran code Multi-Sphere T-matrix Method version 4.0 ([MSTM v4.0](https://github.com/dmckwski/MSTM)) by D.W. Mackowski for simulating the quantities measured by Complex Amplitude Sensing version 2 ([CAS-v2](https://doi.org/10.1364/OE.533776)).
The original source code `mstm-v4.0.f90` was modified and renamed as `mstm-v4.0-ampmat_fb.f90`.

The modified version outputs the complex scattering amplitudes at forward (0°) and backward(180°) directions in addition to the original output quantities (e.g., efficiency factors, scattering matrix). 



## 🚀 Installation

I recommend compiling the modified MSTM codes on Linux or Windows Subsystem for Linux (WSL). 
The following instruction was performed on a WSL enviromnent with Ubuntu 22.04. 

Install the `mpich` Fortran compiler for parallel computing using the following command:

```bash
sudo apt install mpich
```

### Compilation

Compile the modified MSTM codes using the following command:

```bash
mpif90 -o mstm-ampmat_fb-mpi mstm-intrinsics.f90 mpidefs-parallel.f90 mstm-v4.0-ampmat_fb.f90
```

## 🔧 Single MSTM execution

### 1. Prepare the Input File

Calculation conditions and input parameter values should be specified in the text file `mstm-v4.0.inp`
following the rules described in the [Mackowski's MSTM manual](./doc_Mackowski/mstm-manual-2021.pdf).

Supporting information on the parameter specification in `mstm-v4.0.inp` are given in my note "scattering theory 2, 2023/1/21" [page 1](./doc_Moteki/Moteki%20note%20scattering%20theory%202%20%202023121_page1.jpg) and [page 2](./doc_Moteki/Moteki%20note%20scattering%20theory%202%20%202023121_page2.jpg).

### 2. Execution

Execution method (example for parallel computation using 4 CPU cores):

```bash
mpiexec -n 4 ./mstm-ampmat_fb-mpi mstm-v4.0.inp
```

For small aggregate sizes (N < ~100), multi-core execution may not provide significant benefits, and single-core execution should be sufficient.

### 3. Output

The MSTM execution results are written to the specified output file in the following format. Please refer to the original MSTM manual and my note "scattering theory 2, 2023/1/21" for the meaning of each parameter.

```
**********************************************************
mstm calculation results

date, time:
20240915 170619.650

input file:
mstm-v4.0.inp
**********************************************************

input variables for run 1

sphere data input file:
temp_pos.dat

number spheres
1

length, ref index scale factors
7.5340000E+00 1.0000000E+00 0.0000000E+00

volume cluster radius, area mean sphere radius, circumscribing radius, cross section radius
1.5067994E+00 1.5068000E+00 1.5068000E+00 1.5067994E+00

sphere properties and associations
sphere host layer radius x y z ref indx
1 0 0 1.507 0.000 0.000 0.000 1.5800E+00 0.0000E+00

incident plane wave
incident alpha, beta(deg)
0.0000E+00 0.0000E+00

t matrix order:
3

layer 0 refractive index
1.0000E+00 0.0000E+00

number of plane boundaries
0

max_iterations,solution_epsilon, mie_epsilon
10000 1.0000E-06 1.0000E-06

maximum Mie order, number of equations:
3 30

mean sphere Mie extinction, absorption efficiencies
1.0525E+00 -4.4409E-16

**********************************************************
calculation results for run
1

number iterations, error, solution time
0 0.0000E+00 3.9405E-04

sphere extinction, absorption, volume absorption efficiencies (unpolarized incidence)
sphere Qext Qabs Qvabs
1 1.0525E+00 7.3969E-17 7.3969E-17

total extinction, absorption, scattering efficiencies (unpol, par, perp incidence)
1.0525E+00 7.3969E-17 1.0525E+00 1.0525E+00 7.3969E-17 1.0525E+00 1.0525E+00 7.3969E-17 1.0525E+00

scattering matrix in incident plane: 0 deg = incident direction

forward scattering amplitude matrix (BH83,Eq3.12)
( 0.5974E+00, -0.1401E+01) ( 0.5974E+00, -0.1401E+01) ( -0.3795E-17, 0.5463E-19) ( 0.1789E-16, -0.2841E-16)

backward scattering amplitude matrix (BH83,Eq3.12)
( 0.2559E+00, -0.6724E-01) ( -0.2559E+00, 0.6724E-01) ( 0.3795E-17, -0.3226E-18) ( 0.7264E-17, 0.1377E-16)
```

The last two lines show the scattering amplitudes defined according to the Eq. 3.12 of BH83 at the forward $(\theta=0°)$ and backward $(\theta=180°)$ scattering angles. The values from left to right are $S_1, S_2, S_3, S_4$ in the format $(\textnormal{Re}S, \textnormal{Im}S)$.

These $S_1, S_2, S_3, S_4$ of BH83 can be converted to the $S_{11}, S_{12}, S_{21}, S_{22}$ of MI02 (Eqs. 2.30-34) by the rules:

$S_{11} = S_{2}/(-ik)$,

$S_{12} = S_{3}/(ik)$,

$S_{21} = S_{4}/(ik)$,

$S_{22} = S_{1}/(-ik)$.

where $k$ is the wavenumber in the medium (= vacuum wavenumber × medium refractive index), and $i$ is the imaginary unit. The difference between the two definitions is explained in my note "scattering theory 2, 2023/1/21" [page 1](./doc_Moteki/Moteki%20note%20scattering%20theory%202%20%202023121_page1.jpg) and [page 2](./doc_Moteki/Moteki%20note%20scattering%20theory%202%20%202023121_page2.jpg).

For homogeneous spherical particles, I checked that the $S_1(0°), S_2(0°),S_1(180°),S_2(180°)$ values calculated by the modified MSTM code agreed with those calculated by [MieScat_Py](https://github.com/NobuhiroMoteki/MieScat_Py.git).


## 🛠️ Batch MSTM runs (for PTSA aggregates)

1. Put the aggregate files generated by the [aggregate_generator_PTSA](https://github.com/NobuhiroMoteki/aggregate_generator_PTSA.git) in the subfolder `./aggregate_ptsa_files`.
2. Run the JupyterNotebook `mstm_run_agg0_batch.ipynb` that iteratively executes MSTM calculation for each of the PTSA aggregate with parameter sweeps (with respect to complex refractive index).
3. The MSTM output file from each MSTM execution is stored in the subfolder `./mstm_out_files`.



## 📝Licence
This project is licensed under the MIT License. See the LICENSE file for details.

## 📓References
The BH83 and MI02 respectively refer to the following books:

- Bohren and Huffman 1983, Absorption and Scattering of Light by Small Particles
- Mishchenko, Travis, and Lacis 2002, Scattering, Absorption, and Emission of Light by Small Particles, 3rd electronic release.

The Complex Amplitude Sensing (particle measurement technique):

- Moteki, N. (2021). Measuring the complex forward-scattering amplitude of single particles by self-reference interferometry: CAS-v1 protocol. Optics Express, 29(13), 20688-20714, https://doi.org/10.1364/OE.423175
- Moteki, N., & Adachi, K. (2024). Measuring the polarized complex forward-scattering amplitudes of single particles in unbounded fluid flow: CAS-v2 protocol. Optics Express, 32(21), 36500-36522, https://doi.org/10.1364/OE.533776.
