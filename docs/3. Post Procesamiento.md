# Post Procesamiento
## 1. Revisar si hubo errores
```python
target = "target name"
target_dir = "main_target_path/"+target
N_cores = 10 #No of cores where you have run the target
mole_wo_score = []
ix_wo_score = []
mole_ix = 0 #IX-1 of the first molecule

for i in range (N_cores):
    ruta_dock =  target_dir + "/" +str(i+1) + "/dock.out"
    with open(ruta_dock, 'r') as file:
        for line in file:
            if line.startswith("Molecule:"):
                mole_ix += 1
                mole_line = line
                name_mole = mole_line.strip().split()[1]
                ix_mole = mole_ix
            elif line.startswith(" ERROR:"):
                mole_wo_score.append(name_mole)
                ix_wo_score.append(ix_mole)

print ("Number of molecules wo/ score: " + str(len (mole_wo_score)))
print (len(ix_wo_score)) # tiene que ser el mismo numero
ix_wo_score
```

Este codigo devuelve la cantidad de moleculas que tuvieron error en la corrida, y el IX de las mismas
### volver a correr si es necesario

En el caso de tener errores, se puede hacer una segunda corrida,solo a las moleculas que fallaron, cambiando 2 parametros del archivo flex.in: <br>
1. Para crear un archivo solo con las moleculas que falaron se peude correr el siguiente codigo: <br>
```python
# Definir los números de moléculas a extraer
ix_moleculas = ix_wo_score

# Nombre del archivo .mol2 de entrada y salida
input_filename = '/PATH_TO/target_DB_docking.mol2' #base de datos completa
output_filename = '/PATH_TO/target/sub_base_datos.mol2' #base de datos de solo las moleculas que fallaron

# Leer el archivo .mol2 original
with open(input_filename, 'r') as infile:
    lines = infile.readlines()

# Crear una lista para almacenar las moléculas seleccionadas
selected_molecules = []

# Variable para controlar si estamos dentro de una molécula de interés
inside_molecule = False
molecule_number = None
molecule_data = []

# Recorrer todas las líneas del archivo
for line in lines:
    if line.startswith('@<TRIPOS>MOLECULE'):
        # Comenzamos una nueva molécula
        if molecule_data:
            # Verificamos si la molécula actual está en la lista de interés
            if molecule_number in ix_moleculas:
                selected_molecules.extend(molecule_data)
        # Reiniciar los datos de la molécula
        molecule_data = [line]
        inside_molecule = True
        molecule_number = None
    elif inside_molecule:
        molecule_data.append(line)
        if molecule_number is None and len(molecule_data) > 1:
            # Extraer el número de molécula del nombre
            parts = molecule_data[1].strip().split()
            if len(parts) > 1:
                molecule_number = int(parts[1])

# Verificar la última molécula en el archivo
if molecule_data and molecule_number in ix_moleculas:
    selected_molecules.extend(molecule_data)

# Escribir las moléculas seleccionadas en un nuevo archivo .mol2
with open(output_filename, 'w') as outfile:
    outfile.writelines(selected_molecules)

print(f'Se ha creado el archivo {output_filename} con las moléculas seleccionadas.')
```
2. Corregir el archivo flex.in cambiando tanto: *"pruning_conformer_score_cutoff"* y *"internal_energy_cutoff"* a 50000000000000000, y el nombre de la base de datos a la nueva.
3. Volver a correr el comando, solo para estas moleculas: <br>
```
dock6 -i flex.in -o dock.out
```
## 2. Unificar archivos y convertir a .sdf
Ya sea que no hubo errores, se solucionaros, o se eligio omitar las oleculas con errores, el siguiente paso es unificar los archivos de todas las carpetas separadas.<br>
Tambien, queremos convertir el out a un archivo que nos sea facil para analizar los datos, donde agregaremos el score, el ix y el nombre de la molecula. <br>
Para eso, vamos a usar el siguiente codigo: <br>
[Script para unificar y convertir a sdf](docs/scripts)

## 3. Convertir a SDF
En el caso de no hbaer usado el paso 2, porque no se corrio en diferentes carpetas o porque ya se tiene el out.mol2 en un solo archivo, para converitr de mol2 a sdf, se corre el siguiente comando por terminal:
```
obabel -imol2 out_dock6.mol2 -osdf -O out_dock6.sdf
```

