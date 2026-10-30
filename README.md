# ¿Qué operadores de CallMeMaybe trabajan mal?

**Proyecto final · Análisis de datos · TripleTen · Octubre 2026**
Autor: Angel Regino

---

## 1. El problema

CallMeMaybe es un servicio de telefonía virtual. Sus **supervisores** necesitan una lista de operadores ineficaces para apoyarlos y tomar decisiones.

El proyecto tiene dos objetivos:
1. **Identificar operadores ineficaces.**
2. **Comprobar una hipótesis:** el tiempo de espera de los clientes depende de su tarifa (A, B o C).

## 2. Los datos

| Archivo | Contenido |
|---|---|
| `telecom_dataset_us` | 53 902 registros de llamadas: dirección (entrante/saliente), si fue interna, operador, si se perdió, número de llamadas y duraciones (con y sin espera). |
| `telecom_clients_us` | 732 clientes con su tarifa (A, B o C) y fecha de registro. Se une con las llamadas por `user_id`. |

## 3. Lo que hice

### Limpieza
- Eliminé las filas sin operador (~15%), porque no se le puede atribuir la llamada a nadie.
- Eliminé 4 179 duplicados y 117 nulos en `internal` (0.2%).
- Convertí las fechas a `datetime` y creé la columna **tiempo de espera** = duración total − duración de la plática.
- Resultado: **41 491 registros limpios**.

### Exploración
- El 86.6% de las llamadas son salientes, el 98.1% son externas y el 38.8% son perdidas. Como muchas perdidas son salientes que el cliente no contestó, solo evalué las **entrantes perdidas**.
- El servicio creció de casi 0 a ~12 500 llamadas diarias entre agosto y noviembre de 2019.
- La carga por operador es muy desigual (mediana de 72 llamadas). Para no juzgar con pocos datos, **solo evalué operadores con al menos 17 llamadas entrantes** (la mediana), y quedaron **380 operadores**.

### Definición de "operador ineficaz"
Usé tres métricas, cada una con un límite por percentil:

| Métrica | Es "mala" si… |
|---|---|
| % de entrantes perdidas | Más del 4.2% (percentil 90) |
| Tiempo de espera | Más de 29 s por llamada (percentil 90) |
| Llamadas salientes | Menos de 16 (percentil 10), solo para quien hace salientes |

**Regla:** un operador es ineficaz si falla en **2 o más** métricas. Los 74 operadores que solo reciben llamadas no se evalúan en salientes.

### Prueba de hipótesis
- H0: el tiempo de espera es igual entre tarifas.
- Prueba **Mann-Whitney** (elegida porque hay valores extremos), alfa = 0.05, con corrección de **Bonferroni** (0.017) por hacer 3 comparaciones.

**Herramientas:** Python (pandas, NumPy, Matplotlib, SciPy) en Jupyter Notebook

## 4. Lo que descubrí

- **18 operadores ineficaces** de 380 evaluados (4.7%). 69 tienen una sola alerta y ninguno falla en las tres métricas.
- Los casos más graves pierden entre el 10% y el 18% de sus llamadas entrantes (ej. operador 937368 con 18.2%).
- **La tarifa B hace esperar más:** 18.7 s en promedio contra 15.0 s (A) y 15.6 s (C). Las 3 comparaciones dan p-value muy por debajo de 0.017, así que **se rechaza H0**.

### Recomendaciones para los supervisores
1. **Seguimiento:** capacitar y acompañar a los 18 operadores ineficaces, empezando por los que más llamadas pierden.
2. **Tarifa B:** revisar cuántos operadores atienden a estos clientes para bajar su tiempo de espera.
3. **Monitoreo:** llevar las 3 métricas al panel de supervisores y revisarlas cada mes.
