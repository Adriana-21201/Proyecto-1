#Proyecto 1
#Adriana Girón

import numpy as np
import matplotlib.pyplot as plt
import heapq
import time
import math
import queue

filename = "Laberinto2-3.txt"

# Función para leer el archivo con manejo de excepciones
def read_lab(filename):
    try:
        with open(filename, 'r') as file:
            matriz = []
            for line in file.readlines():
                fila = line.strip().split(',')
                try:
                    fila = [int(valor) for valor in fila]
                    matriz.append(fila)
                except ValueError:
                    print(f"Error al convertir valores en la línea: {line}")
                    return None
            return np.array(matriz)
    except FileNotFoundError:
        print(f"Archivo {filename} no encontrado.")
        return None

# Heurísticas
def heuristica_manhattan(a, b):
    return abs(a[0] - b[0]) + abs(a[1] - b[1])

def heuristica_eucladiana(a, b):
    return math.sqrt((a[0]-b[0]) ** 2 + (a[1] - b[1]) ** 2)

# Vecinos (Movimientos posibles)
def vecinos(pos, lab):
    filas, columnas = lab.shape
    mov = [(0, -1), (1, 0), (0, 1), (-1, 0)] 
    resultado = []
    for dx, dy in mov:
        x, y = pos[0] + dx, pos[1] + dy
        if 0 <= x < filas and 0 <= y < columnas and lab[x, y] != 1:
            resultado.append((x, y))
    return resultado

# Algoritmo BFS
def bfs(lab, start, end):
    print(f"Inicio: {start}, Fin: {end}")
    inicio_tiempo = time.time()
    cola = queue.Queue()
    cola.put((start, [start]))
    visitados = set()

    while not cola.empty():
        nodo, camino = cola.get()
        if nodo in visitados:
            continue
        visitados.add(nodo)
        if nodo == end:
            fin_tiempo = time.time()
            return camino, len(visitados), fin_tiempo - inicio_tiempo
        
        for vecino in vecinos(nodo, lab):
            if vecino not in visitados:
                cola.put((vecino, camino + [vecino]))

    return [], len(visitados), time.time() - inicio_tiempo

# Dibujar laberinto con Matplotlib
def draw_lab(lab, camino):
    filas, columnas = lab.shape
    img = np.zeros((filas, columnas, 3))
    for i in range(filas):
        for j in range(columnas):
            if lab[i, j] == 1:
                img[i, j] = [0, 0, 0]  # Negro
            elif lab[i, j] == 2:
                img[i, j] = [0, 1, 0]  # Verde
            elif lab[i, j] == 3:
                img[i, j] = [1, 0, 0]  # Rojo
            else:
                img[i, j] = [1, 1, 1]  # Blanco

    for (x, y) in camino:
        img[x, y] = [0, 0, 1]  # Azul

    plt.imshow(img)
    plt.axis("off")
    plt.show()

# Resolver Laberinto
def resolver_lab(alg, lab):
    start = tuple(np.argwhere(lab == 2)[0]) if (lab == 2).any() else None
    end = tuple(np.argwhere(lab == 3)[0]) if (lab == 3).any() else None

    if start is None or end is None:
        print("Error: No se encontró punto de inicio o fin")
        return

    print(f"Inicio: {start}, Fin: {end}")

    if alg == "BFS":
        camino, nodos, tiempo = bfs(lab, start, end)

    if camino:
        draw_lab(lab, camino)
        print(f"Nodos visitados: {nodos}\nLargo del camino: {len(camino)}\nTiempo: {tiempo:.4f}s")
    else:
        print("No se encontró solución al laberinto")

# Interfaz de consola
if __name__ == "__main__":
    lab = read_lab(filename)
    if lab is not None:
        resolver_lab("BFS", lab)
