# Archivos auxiliares para correr DOCK 6
### 1. Surface
La superficie se puede generar con Chimera. <br>
**a.** Cargar el target en formato **.mol2** <br>
**b.** Actions > Surface > Show <br>
**c.** Tools > ASteructure editing > write DMS <br>
<br>
output: file "surface.dms"
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

### 2. Spheres
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

### 3. Box

### 4. Grid
