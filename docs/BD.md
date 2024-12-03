# Preparacion de las BD

Para poder optimizar las corridas y facilitar el analisis de los datos, conviene tratar la base de datos. <br>
Recordemos que el formato de la base de datos debe ser en *.mol2, en caso de tenerla como *.sdf, convertirla ocn OpenBabel:
```
obabel -isdf path_to_sdf_file.sdf -omol2 -O path_to_mol2_file.mol2
```
Para ello:
- Numeramos la base de datos con el campo "IX" de 1 a n (n = largo de la BD)
```
dir_DB = "path_to_db"
num_mole = 1
moles_name = []

with open(dir_DB, 'r') as archivo_origen:
    # Read the lines of the file
    lines = archivo_origen.readlines()

#Iterate over the lines
for i, line in enumerate(lines):
    # Strip any leading/trailing whitespace
    line = line.strip()
    
    # Check for the start of a new molecule block
    if line == "@<TRIPOS>MOLECULE":
        # Extract the molecule name from the next line
        molecule_name = lines[i+1].split()[0]
        
        # Update the molecule name with the molecule number
        new_name = f"{molecule_name} {num_mole}"
        lines[i+1] = f"{new_name}\n"
        
        # Store the updated name
        moles_name.append(new_name)
        
        # Increment the molecule number
        num_mole += 1
        
  #Write the modified lines back to the file
  with open(dir_DB, 'w') as archivo_destino:
      archivo_destino.writelines(lines)
  ```
- Dividimos la misma en tantos cores hayan disponibles:
```
def split_mol2_file(input_file, num_files):
    # Leer el archivo .mol2 y contar el número total de moléculas
    with open(input_file, 'r') as f:
        mol_count = 0
        for line in f:
            if line.startswith('@<TRIPOS>MOLECULE'):
                mol_count += 1
    
    # Calcular cuántas moléculas debe contener cada archivo dividido
    molecules_per_file = mol_count // num_files
    
    # Iterar sobre el archivo .mol2 y escribir las moléculas en grupos en los archivos divididos
    current_file_index = 1
    molecules_written = 0
    with open(input_file, 'r') as f:
        outfile = None
        for line in f:
            if line.startswith('@<TRIPOS>MOLECULE'):
                if molecules_written % molecules_per_file == 0:
                    if outfile:
                        outfile.close()
                    outfile = open(target_dir + '/' + str(current_file_index) +'/' +f"{input_file[42:-5]}_{current_file_index}.mol2", 'w')
                    current_file_index += 1
                molecules_written += 1
            if outfile:
                outfile.write(line)

    if outfile:
        outfile.close()


split_mol2_file(dir_DB, N_cores)

```
