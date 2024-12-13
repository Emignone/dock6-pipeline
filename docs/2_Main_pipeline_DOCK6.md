# Archivos auxiliares para correr DOCK 6
## 1. Surface
La superficie se puede generar con Chimera. <br>
**a.** Cargar el target en formato **.mol2** <br>
**b.** Actions > Surface > Show <br>
**c.** Tools > ASteructure editing > write DMS <br>
<br>
output: file "surface.dms"<br>
***Importante***<br>
Si el receptor no se preparo con Chimera, se va a tener que editar el archivo surface.dms de la siguiente manera: <br>
```Python
dir_dms = "/PATH/surface.dms"

f = open (dir_dms, "r")
lines = f.read().splitlines()
new_lines = []
for line in lines:
    new_line = line.replace(line.split()[0], (line.split()[0])[0:3])
    new_lines.append(new_line)
    print(new_line)

with open(dir_dms, 'w') as archivo_destino:
    archivo_destino.writelines('\n'.join(new_lines))
```

## 2. Spheres
**Paso 1:**<br>
Para generar las esferas, necesitamos un archivo input: **"INSPH"**
```
surface.dms #input
R
X
0.0
8.0
0.1
spheres.sph #output
```
Luego, se debe correr el siguiente comando en consola:
```
insph
```
**Paso 2:**<br>
Una vez obtenidas las esferas en el archivo *"spheres.sph"*, es necesatrio solo quedarson con el cluster de esferas que estaa cerca del sitio de union.<br>
Para eso, lo que debemos hacer es correr el siguiente comando:
```
sphere_selector spheres.sph path/al/ligando.mol2 10.0
```
En este caso, se uso radio r = 10.0 A, este valor se peude cambiar. Solemos usar 10.0.

## 3. Box
Ahora hay que calcular la caja que va a delimitar el espacio de trabajo, para ello, vamos a necesitar 3 coordenadas *(x,y,z)* del centro de gravedad y el tamaño de 3 lados. Todo eso se va a volcar en el archivo *"box.in"*:
```
N
U
-8.58518227, -6.39779485, 55.4131 #Poner las coordenadas del centro de graved
23 23 28 # Poner el tamaño de los lados
box.pdb #Nombre del file output
```
¿Donde conseguir las coordenadas? Se puede usar AUtoDockTools, y seleccionar para hacer una grid. (No olvidar poner spacing = 1)<br>
[Guia ADT](adt.md) <br>

Una vez realizado eso, se puede correr el siguiente comando por temrinal de linux:
```
showbox < box.in
```

## 4. Grid
Para calcular el grid, tambien se necesita un archivo input llamado "grid.in", que consta de los siguentes campos:
```
compute_grids                             yes
grid_spacing                              0.3
output_molecule                           no
contact_score                             no
energy_score                              yes
energy_cutoff_distance                    10
atom_model                                u
attractive_exponent                       6
repulsive_exponent                        12
distance_dielectric                       yes
dielectric_factor                         4.0
allow_non_integral_charges                yes
bump_filter                               no
bump_overlap                              0.75
receptor_file                             aa2ar_rec.mol2 #si el archivo esta en otra carpeta poner el path entero
box_file                                  box.pdb #idem al path
vdw_definition_file                       /PATH_TO/dock6/parameters/vdw_AMBER_parm99.defn 
score_grid_prefix                         grid
```
Una vez que tenemos ese archivo, se puede correr el siguiente comando que genera 3 archivos de output <br>
**(importante correrlo en la computadora donde se va a correr el docking porque genera archivos binarios especificos).**
```
grid -i grid.in -o grid.out
```
Los 3 archivos que genera si funciono bien son: *grid.out, grid.bmp y grid.nrg*
## Ultimos pasos
Para poder correr el docking, es necesario preparar la base de datos, con eso nos referimos a dividirnla en N cores, Agregar IX a las moleculas y dividiro todo en diferentes carpetas. <br>
Esos pasos los podran encontrar en el siguiente archivo: [Preparacion de BD](BD.md)

## 5. Correr el docking
Una vez se tienen todos los archivos antes mencionados, y las bases de datos divididas en la cantidad de cores disponibles, se pueden preparar los archivos finales.<br>
Es necesario el archivo "flex.in", que recomiendo crear dentro de n carpetas (n = cantidad de cores) un archivo diferente para poder paralelizar.<br>
"Flex.in": <br>

```
conformer_search_type                                        flex
write_fragment_libraries                                     no
user_specified_anchor                                        no
limit_max_anchors                                            no
min_anchor_size                                              40
pruning_use_clustering                                       yes
pruning_max_orients                                          1000
pruning_clustering_cutoff                                    200
pruning_conformer_score_cutoff                               50000000.0
pruning_conformer_score_scaling_factor                       1.0
use_clash_overlap                                            no
write_growth_tree                                            no
use_internal_energy                                          yes
internal_energy_rep_exp                                      12
internal_energy_cutoff                                       50000000.0
ligand_atom_file                                             DB_n.mol2 #path de la base de datos (n = numero del core)
limit_max_ligands                                            no
skip_molecule                                                no
read_mol_solvation                                           no
calculate_rmsd                                               no
use_database_filter                                          no
orient_ligand                                                yes
automated_matching                                           yes
receptor_site_file                                           selected_spheres.sph #Path a selected spheres
max_orientations                                             750
critical_points                                              no
chemical_matching                                            no
use_ligand_spheres                                           no
bump_filter                                                  no
score_molecules                                              yes
contact_score_primary                                        no
grid_score_primary                                           yes
grid_score_rep_rad_scale                                     1
grid_score_vdw_scale                                         1
grid_score_es_scale                                          1
grid_score_grid_prefix                                       grid #si los archivos grid estan dentro de la carpeta dejar tal que asi, sino agregar por delante el path a todfos los grid
minimize_ligand                                              yes
minimize_anchor                                              yes
minimize_flexible_growth                                     yes
use_advanced_simplex_parameters                              no
minimize_flexible_growth_ramp                                no
simplex_max_cycles                                           1
simplex_score_converge                                       0.1
simplex_cycle_converge                                       1.0
simplex_trans_step                                           1.0
simplex_rot_step                                             0.1
simplex_tors_step                                            10.0
simplex_anchor_max_iterations                                500
simplex_grow_max_iterations                                  500
simplex_grow_tors_premin_iterations                          0
simplex_random_seed                                          0
simplex_restraint_min                                        no
atom_model                                                   all
vdw_defn_file                                                /home/emilia/Downloads/dock6/parameters/vdw_AMBER_parm99.defn
flex_defn_file                                               /home/emilia/Downloads/dock6/parameters/flex.defn
flex_drive_file                                              /home/emilia/Downloads/dock6/parameters/flex_drive.tbl
ligand_outfile_prefix                                        flex.out #output file
write_orientations                                           no
num_scored_conformers                                        1
rank_ligands                                                 no
```
Dentro de cada carpeta el archivo es el mismo, solo cambia la linea *"ligand_atom_file"*, donde se debe poner la subdivision de la BD correspondiente a dicho core. <br>

Una vez que contamos con todos, procederemos a correr los n archivos "flex.in" con el siguiente comando por terminal:
```
dock6 -i flex.in -o dock.out
```
Para automatizarlo, se puede utilizar el siguente codigo:
```python
from subprocess import Popen, PIPE
for i in range(1,13): #de core 1 a core n+1 poner
    proc= Popen(['/home/ccgroup/DOCK6/dock6/bin/dock6','dock6','-i','flex.in','-o','dock.out'],cwd='/home/ccgroup/Emilia/fa10/' + str(i)) #cwd es el path a cada carpeta individual con el flex.in, en mi caso numero las carpetas de 1 a n
```
