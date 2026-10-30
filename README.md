# Optimizacion de Multiplicacion de Matrices y OpenMP

Este proyecto realiza un estudio experimental detallado sobre el impacto de la optimizacion de codigo a nivel de hardware y paralelismo en el algoritmo de multiplicacion de matrices cuadradas ($C = A \times B$).

Se comparan diversas estrategias de optimizacion, incluyendo reestructuracion del acceso a memoria cache, directivas de optimizacion del compilador (`gcc`) y paralelismo de memoria compartida utilizando OpenMP. El proyecto evalua metricas de tiempo de ejecucion, aceleracion (*speedup*), eficiencia paralela, uso de memoria RAM, perfilado de CPU y rendimiento en GFLOPS a traves de multiples arquitecturas de hardware.

---

## Estructura del Repositorio

```text
.
└── hpc_proyecto2com/
    └── hpc_proyecto2/
        ├── matrizsecuencial.c         # Implementacion secuencial estandar (i-j-k)
        ├── matrizfilaxfila.c          # Optimizacion por localidad espacial de cache (i-k-j)
        ├── matriz2.c                  # Variante de optimizacion secuencial secundaria
        ├── matrizhilos.c              # Version paralela utilizando OpenMP
        ├── matrizprocces.c            # Version paralela basada en multiprocesamiento
        ├── benchmark.sh               # Benchmark general para pruebas secuenciales y de memoria
        ├── run_openmp_benchmark.sh    # Pruebas automatizadas de escalabilidad OpenMP (variando hilos)
        ├── run_profile.sh             # Perfilado de execution mediante gprof y perf
        ├── run_tests.sh               # Scripts de ejecucion y validacion general
        ├── run_testshilos.sh          # Pruebas de escalabilidad especificas para hilos
        ├── run_testsproceces.sh       # Pruebas especificas para procesos
        ├── resultados_a_excel.py      # Script en Python para procesamiento de datos y graficado
        ├── resultados_M1/             # Resultados, registros de perf y CSVs obtenidos en Maquina 1
        ├── resultados_M2/             # Resultados, registros de perf y CSVs obtenidos en Maquina 2
        └── resultafos_image/          # Graficas comparativas generadas (Speedup, Eficiencia, GFLOPS, RAM)
```

---

## Enfoques de Optimizacion e Implementacion

El proyecto abarca cuatro niveles de optimizacion:

1. **Secuencial Estandar (`matrizsecuencial.c`):**
   * Algoritmo clasico $O(N^3)$ con orden de bucles `i-j-k`.
   * Presenta alto indice de fallos de cache (*cache misses*) al recorrer la matriz $B$ por columnas en el bucle interno.

2. **Optimizacion por Localidad de Cache (`matrizfilaxfila.c`):**
   * Reordenamiento del bucle a `i-k-j`.
   * Permite el acceso secuencial en memoria (contiguo) para los elementos de las matrices $A$ y $B$, maximizando la localidad espacial y aprovechando el principio de linea de cache.

3. **Optimizacion por Compilador:**
   * Aplicacion de flags del compilador `gcc` (tales como `-O3`, `-march=native`, `-ffast-math`) para autovectorizacion SIMD y reordenamiento de instrucciones a nivel de ensamblador.

4. **Paralelizacion con OpenMP (`matrizhilos.c`):**
   * Paralelizacion del bucle exterior mediante la directiva `#pragma omp parallel for`.
   * Evaluacion de escalabilidad fuerte (*strong scaling*) variando la cantidad de hilos de ejecucion (1, 2, 4, 8, etc.) y analizando las politicas de planificacion (*scheduling*).

---

## Requisitos del Sistema

* **Compilador de C:** `gcc` v9.0 o superior con soporte habilitado para OpenMP (`-fopenmp`).
* **Sistema Operativo:** GNU/Linux moderno (Ubuntu, Debian, Fedora) o WSL2 sobre Windows.
* **Herramientas de Diagnostico y Perfilado:**
  * `gprof` (incluido en `binutils`).
  * `perf` (contadores de rendimiento del kernel de Linux).
  * `/usr/bin/time` para la medicion de consumo maximo de memoria residente (RSS).
* **Entorno Python (para procesamiento de datos):**
  * `Python 3.8+`
  * Librerias: `pandas`, `matplotlib`, `numpy`.

---

## Compilacion y Ejecucion

### Compilacion Manual

Puedes compilar manualmente los binarios correspondientes a cada nivel de optimizacion:

```bash
cd hpc_proyecto2com/hpc_proyecto2

# 1. Version secuencial base sin optimizaciones
gcc -O0 matrizsecuencial.c -o bin_base

# 2. Version optimizada para memoria cache
gcc -O0 matrizfilaxfila.c -o bin_memoria

# 3. Version optimizada por compilador (-O3)
gcc -O3 matrizsecuencial.c -o bin_compilador

# 4. Version paralela con OpenMP y optimizacion de compilador
gcc -O3 -fopenmp matrizhilos.c -o bin_openmp
```

---

## Automatizacion de Experimentos y Profiling

El proyecto incluye scripts en Bash para automatizar la recoleccion de datos y mediciones de rendimiento:

### 1. Ejecucion de Benchmarks Generales
Ejecuta el conjunto de pruebas para las versiones secuenciales, de memoria y de compilador:
```bash
./benchmark.sh
```

### 2. Escalabilidad con OpenMP
Evalua el rendimiento variando el numero de hilos en la arquitectura elegida:
```bash
./run_openmp_benchmark.sh
```

### 3. Perfilado de CPU y Eventos de Hardware
Genera archivos de rastreo con `gprof` y analiza metricas de contadores de hardware (uso de instrucciones, ciclos por instruccion, fallos de cache) con `perf`:
```bash
./run_profile.sh
```

---

## Generacion de Reportes y Graficas

Los resultados crudos almacenados en los archivos CSV dentro de `resultados_M1/` y `resultados_M2/` pueden procesarse mediante el script de Python incluido para generar los graficos comparativos en `resultafos_image/`:

```bash
python3 resultados_a_excel.py
```

### Metricas Analizadas en las Graficas

* **Tiempo de Ejecucion vs. Tamaños de Matriz:** Comparativa del tiempo total en segundos frente al tamaño $N \times N$.
* **Speedup ($S_p$):** Relacion entre el tiempo secuencial base y el tiempo paralelo con $p$ hilos ($S_p = T_1 / T_p$).
* **Eficiencia Paralela ($E_p$):** Medicion del aprovechamiento de los nucleos del procesador ($E_p = S_p / p$).
* **Rendimiento en GFLOPS:** Rendimiento en miles de millones de operaciones de punto flotante por segundo.
* **Uso de Memoria RAM:** Monitoreo del espacio ocupado en memoria por matriz e impacto del footprint total.
