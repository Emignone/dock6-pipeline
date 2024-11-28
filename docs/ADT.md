# Guia sobre como obtener las box dimensions con ADT y el centro de masa
En esta guia van a poder encontrar como obtener las dimensiones de la caja y el centro de masa del ligando

## Pasos para calcular centor de masa:
1. Abrir ICM <br>
2. Cargar el ligando <br>
3. Correr el comando:
   ```
    Mean(Xyz(a_ligand))
   ```

## Pasos para calcular box dimensions:
1. Abrir ADT<br>
2. File > read Molecule (seleccionar ambos ligando y proteina)<br>
3. Grid > Grid box...<br>
Luego de este paso veran lo siguiente:<br>
<p align="center"> 
</p>
<p align="center">
    <img src="imgadt.png" alt="img" />
</p>

5. Seleccionar spacing = 1, completar el centro de masa y ajustar las coordenadas x,y,z hasta que la caja tenga un tamaño aceptable. (siempre el ligando dentro de la misma)
