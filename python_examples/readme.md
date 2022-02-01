# Explanation of inputs (input.yml) for running simulations via Python

<img src="https://github.com/je-santos/MultiphasePorousMediaPalabos/blob/master/illustrations/LBM%20geometry%203D.png" align="middle" width="600" height="400" alt="geometry inputs explanation">

Figure 1- Geometry setup for 2-phase flow simulations


**The inputs are explained in the same sequence as the .yml file. First, the general inputs common between 1 and 2 phase, and then specific inputs for 1 phase and 2 phase simulations**

### General Inputs 
- `simulation type:` String, "1-phase" or "2-phase" choose which type of simulation you want to run
#### `input output:` (no user inputs here, just a key to organize input and output information)
- `simulation directory:` String specifying the absolute path the the simulation directory (where you are running the simulation from). Please run the command `pwd` in your simulation directory and copy the output into the input file (the path should be inside quotation marks and not have a '/' at the end). We are working on adding a simpler way of getting this info to Python!
- `input folder:` String specifying the path to where the input geometry is stored. In the examples, we provide "input/" as the input folder. 
- `output folder:` String specifying the path to where the simulation output should be saved. In the examples, we provide "tmp/" as the output folder.
#### `geometry:` (no user inputs here, just a key to organize geometry information)
- `file name:` String specifying the name of the name of the input geometry file
- `data type:` Datatype of the input geometry file (eg int8)
- `geometry size:` Please provide the x, y, and z dimensions (`Nx`, `Ny`, and `Nz` subkeys in the examples) of the input geometry as integers
#### `domain:` (no user inputs here, just a key to organize domain information)
- `geom name:` "rg_theta60_phi10"  # Name of .dat file, rename from original if you'd like. Do not include the file extension.
- `domain size:` Please provide the x, y, and z dimensions (`nx`, `ny`, and `nz` subkeys in the examples) of the input simulation domain
- `periodic boundary:` True or False, whether the boundaries are periodic in the x, y, and z directions
- `inlet and outlet layers:` Integer number of layers added to the ends of the simulation domain (in x direction). For 1-phase simulations, 1 or greater should work well. For 2-phase simulations, 4 should work well and at least 3 are highly recommended.
- `add mesh:` NOT YET IMPLEMENTED, True or False, Add A neutral mesh
- `swap xz:` True or False  # False by default
- `double geom resolution:` NOT YET IMPLEMENTED, True or False, this will double the domain resolution

### 1-Phase Simulation Inputs (1-phase inputs for `simulation` key)
- `num procs:` Integer number of processors to run on (please make sure this is compatible with your system!)
- `num geoms:` Integer total number of geometries / individual simulations, set to 1 for a single-phase simulation
- `pressure:` Pressure difference across the core, numbers on the order of 1e-4 should work well
- `max iterations:` Integer of the maximum number of iterations the simulation should run for. 
- `convergence:` Energy difference convergence criterion for the simulation. We recommend figuring out the order accuracy you need and trying that. In addition, the lower the convergence threshold the higher you should set `max iterations`. From testing, `convergence = 1e-6` and `max iterations = 1e7` will yield good, consistent results, but it could take a long time (order of hours) depending on how large the geometry is.
- `save vtks:` True or False, whether to save vtk files with velocity field

### 2-Phase Simulation Inputs (2-phase inputs for `simulation` key)
- `num procs:` Integer number of processors to run on (please make sure this is compatible with your system!)
- `restart sim:` True or False, set to False if starting a simulation. Set to True if you would like to continue from a previous saved state. 
- `rho_f1:` Initial/equilibrium density of Fluid 1
- `rho_f2:` Initial/equilibrium density of Fluid 2
- *A note on fluid densities:* The Shan-Chen method is sensitive to the density ratio of the two fluids. We have not done much testing with fluids of different density, but it is a possiblility with our model. 
- `force_f1:` Applied force to Fluid 1
- `force_f2:` Applied force to Fluid 2
- *A note on forces:* For no applied forces, set both entries to 0. If you do want to use forces, depending on your geometry and applications, this should be around 1e-3 to 1e-6. For porous media applications where capillary number is important, a smaller force should be used. There is currently no built-in function to calculate capillary number, but it can be calculated from velocity output. That being said, it may take some trial and error to get the force right for you application. 
- `pressure bc:` True or False, to use a pressure boundary condition or not
- `minimum radius:` This number is correlated to delta rho in the docs (see below). This acts as entry capillary pressure, so set 1-3 voxels lower than inscribed sphere radius to reach residual saturations
- `num pressure steps:` Integer number of pressure values the simulation should use. Set to 1 if you only want a constant pressure difference across the sample. Setting this larger than 1 will create an array of pressure differences to simulate; this is how to obtain a capillary pressure curve via drainage. The pressure difference will start at zero, and the pressure found from `minimum radius` will be the largest pressure difference across the sample. The code will create an array of pressure values from 0 to the maximum pressure with `num pressure steps` steps in between.
  
  # Initial Conditions
  fluid init: drainage  # If drainage, traditional drainage setup used. If custom, use fluid 1/2 init to do custom fluid setup
  fluid 1 init:
    x1: 1
    x2: 2
    y1: 1
    y2: 100
    z1: 1
    z2: 100
  fluid 2 init:
    x1: 3
    x2: 100
    y1: 1
    y2: 100
    z1: 1
    z2: 100
    
  fluid data:    
    Gc: 0.9
    omega_f1: 1
    omega_f2: 1
    # Wetting forces from Huang et al. 2007
    G_ads_f1_s1: -0.4
    G_ads_f1_s2: 0
    G_ads_f1_s3: 0
    G_ads_f1_s4: 0
  
  convergence: 0.01  # Convergence threshold
  convergence iter: 100  # How often to check for convergence
  max iterations: 500000  # max iterations per Pc step
  save sim: True  # Save restart files
  save iter: 5000  # How often to save restart files
  gif iter: 5000  # How often to save gifs
  vtk iter: 5000  # How often to save vtk files
  rho_f2_vtk: True  # When True, saves rho f1 and f2 vtks. If False, only saves rho f1 vtk
  print geom: True  # Create vtk of geometry at beginning
  print stl: False  # Create stl of geometry at beginning
  
rel perm:  # Parameters for 1-phase sims for rel perms
  pressure: 0.0005
  max iterations: 1000000
  convergence: 1e-6
  save vtks: True  # save velocity vtks

****

**load_savedstate:** This input gives the user the option of loading a previous incomplete simulation (True), or starting a new simulation (False).

**geometry:** There are multiple inputs required under this heading.

**file_geom:** This input asks for the name of the geometry for simulation. PALABOS requires the input geometry be in .dat file format which can be created using the [pre-processing steps](https://github.com/je-santos/MultiphasePorousMediaPalabos/tree/master/pre-processing) The geometry file should be placed in the same folder as the [2-phase simulation code](https://github.com/je-santos/MultiphasePorousMediaPalabos/tree/master/src/2-phase_LBM) or if placed elsewhere, the path should be modified to point to the geometry.

**size:** This input requires the size (in voxels) in the X, Y, and Z directions of the geometry (Figure 1).

*Note:*

1) PALABOS conducts simulations in X-direction, so please double-check X and Z directions of the geometry.
2) If the blank slices and a mesh are added in the pre-proccesing step, the original size in the X-direction would be larger.

**per:** These inputs control the periodicity for fluid 1 and fluid 2 at the boundaries in the X, Y, and Z directions.

**init:** There are multiple inputs required under this heading.

**fluid1:** This input requires the initial positions of fluid 1 (usually invading fluid). As shown in Figure 1, these inputs are x1 to x2, y1 to y2, and z1 to z2 given in orange color. Fluid 1 has one edge as the YZ plane at x=x1 at inlet side of the geometry and the second edge at x=x2 (which is also the left edge of fluid interface at t=0). As the entire fluid in the Y-Z space is filled with fluid 1 between x=x1 and x=x2, the Y and Z direction limits are the geometry limits.

**fluid2:** This input requires the initial positions of fluid 2 (usually defending fluid). As shown in Figure 1, these inputs are x1 to x2, y1 to y2, and z1 to z2 given in blue color. Fluid 2 has one edge as the YZ plane at x=x1 (which is the right edge of fluid interface at t=0) and the second edge at x=x2 at outlet side of the geometry. As the entire fluid in the Y-Z space is filled with fluid 1 between x=x1 and x=x2, the Y and Z direction limits are again the geometry limits.

**fluids:** There are multiple inputs required under this heading.

**Gc:** Interparticle (cohesion) force. This input controls the fluid-fluid interfacial tension. This value assures phase separation. A stable separation is reached by Gc > 1/(rho_f1+rho_f2). A value of 0.9 is recommended.

**omega_f1 and omega_f2:** This parameter is used to calculate the kinematic viscosity of fluid 1 and 2, using:  v=( 1 / omega_fi - 0.5 ) / c^2.

**force_f1 and force_f2** If this term is different than zero, a driving force will be added to each fluid (i.e. gravitational) in the x-direction. The pressure boundary conditions are suggested to be turned off and periodicity should be enabled (to reach a steady-state).

**Wetting forces:**
(G_ads_f1_s1, G_ads_f1_s2, G_ads_f1_s3, G_ads_f1_s4): These terms refer to the interaction force between the fluids and the solid walls. This code has the option to add 4 different wetting conditions ( 4 different solid surfaces ), but more could be added with ease. In the 3D image, the voxels labeled with 1, 3, 5, 6 are assigned G_ads_f1_s1, G_ads_f1_s2, G_ads_f1_s3, G_ads_f1_s4, respectively (2 is reserved for inside solids, 4 for the neutral-wet mesh and 0 for the fluids). The contact angle is calculated as: cos(theta) = 4*G_ads_f1_si/( Gc*( rho_f1-rho_d ) )

<img src="https://latex.codecogs.com/svg.latex?\Large&space;cos(\theta)=\frac{4G_{ads_{f1,si}}}{G_c(rho_{f1}-rho_d)}" title="\Large x=\frac{-b\pm\sqrt{b^2-4ac}}{2a}" />

**rho_f1:** This input takes the initial density of fluid 1 throughout the geometry.

**rho_f2:** This input takes the initial density of fluid 2 throughout the geometry.

*Note:* A stable value for both densities is 2. High density ratios tend to be numerically unstable.

**pressure_bc:** This input asks if a pressure gradient will be applied in the geometry. Please use True for un-steady state flow simulations and False for steady state calculations.

**rho_f1_i:** This input takes the initial density of fluid 1 at the inlet pressure boundary and is kept constant.

**rho_f2_i:** This input takes the initial density of fluid 2 at the outlet pressure boundary at the beginning of the simulation.

**rho_f2_f:** This input takes the final density of fluid 2 at the outlet pressure boundary at the end of the simulation. The difference between the inlet and the outlet pressure boundaries decides the capillary pressure.

**rho_d:** This input takes the dissolved density of one phase in the other (both fluid1 and fluid2). The default value may be kept 0.06.

**drho_f2:** This input takes the decrement in the pressure of fluid 2 at the outlet pressure boundary at every step (capillary pressure change) in the simulation. A range of 0.01 to 0.1 may be input depending on balance between sensitivity / computational time, as smaller decrement will require a longer time but will have greater sensitivity to measure change in fluid movement.

The pressure in the Shan-Chen model is calculated as:

<img src="https://latex.codecogs.com/svg.latex?\Large&space;P(x)=\frac{\rho_1(x)+\rho_2(x)}{3}+G_c\frac{\rho_1(x)\rho_2(x)}{3}" title="\Large x=\frac{-b\pm\sqrt{b^2-4ac}}{2a}" />

often, the second term can be neglected because its many orders of magnitude smaller than the first.

To calculate the capillary pressure of the system we use:

<img src="https://latex.codecogs.com/svg.latex?\Large&space;P_c=P_{nw}-P_{w}" title="\Large x=\frac{-b\pm\sqrt{b^2-4ac}}{2a}" />

if we substitute the expression to calculate the pressure, we get:

<img src="https://latex.codecogs.com/svg.latex?\Large&space;P_c=\frac{2}{3}-\frac{(2-\Delta\rho)}{3}=\frac{\Delta\rho}{3}" title="\Large x=\frac{-b\pm\sqrt{b^2-4ac}}{2a}" />

where the first term represents the pressure at the inlet and consequently, the second term is the pressure at the outlet.

Comparing this expression with the Young-Laplace equation for a capillary tube (circular cross section), we finally get:

<img src="https://latex.codecogs.com/svg.latex?\Large&space;\Delta\rho=\frac{6\sigma}{r}cos(\theta)" title="\Large x=\frac{-b\pm\sqrt{b^2-4ac}}{2a}" />

For the example shown above, we performed a buble test where the interfacial tension (sigma) showed a value of 0.15. Substituting that, for a non-wetting condition of G_ads_f1 = -0.4 (156.4 degrees), we get:

<img src="https://latex.codecogs.com/svg.latex?\Large&space;\Delta\rho\approx\frac{0.825}{r}" title="\Large x=\frac{-b\pm\sqrt{b^2-4ac}}{2a}" />



**output:** There are multiple inputs required under this heading.

**out_folder:** This input takes the name of the folder where all the output files will be stored.

**save_sim:** This input asks if user wants to save the simulation lattices after every capillary pressure decrement for both fluid 1 and fluid 2. The saved files are large (> 1 GB) but are overwritten after every decrement and may be used to restart the simulation from that step.

**convergence:** This input takes the value of the convergence criterion for the simulation. The convergence value is inversely proportional to the computational time and the accuracy. See the examples for best practices.

**it_max:** This input takes the value of maximum iterations allowed at  if the convergence is not reached. Roughly, in our supercomputer (LoneStar5) we can get 1M its ruuning in 96 cores for two days for a 200x200x200 domain. This is useful in case one would like to save the simulation dats files before the max run time (in our case, 2 days).

**it_conv:** This input takes the value of number of iterations after which to check if convergence criterion is satisfied. This takes a certain computational overhead, since general statistics have to be computed.

**it_gif:** This input takes the value of number of iterations after which 2D images of the geometry crossection (.gif) showing the density and current fluid configuration are to be saved.

**rho_vtk:** This input asks if 3D geometry files (.vtk) for both fluids 1 and 2 are to be saved: True, or only for for Fluid 1: False.

**it_vtk:** This input takes the  number of iterations after which the 3D geometry files (.vtk) showing the density and current fluid configuration are to be saved. If this number is greater than 100000 no vtk files will be output (which keeps the output lighter and the run faster).

**print_geom:** This input asks if a 3D geometry file (.vtk) is to be saved at the beginning of the simulation.

**print_stl:** This input asks if a 3D geometry file (.stl) is to be saved at the beginning of the simulation. This file may be used for 3D printing the geometry or for viewing.

*Note:* The .vtk files may be viewed in [PARAVIEW](https://www.paraview.org/).