# Pronóstico mensual del homicidio intencional en Colombia: modelos estadísticos vs. aprendizaje automático

Código, datos y resultados del trabajo de grado **"Caracterización y predicción del homicidio intencional en Colombia: comparación entre modelos estadísticos y de aprendizaje automático con datos del SIEDCO"**, presentado como requisito para optar al título de Magíster en Ciencia de Datos, Universidad Ean (Facultad de Ingeniería), Bogotá, 2026.

**Autores:** Fabián Eduardo Camelo Sánchez, Eduardo José Fontalvo Romero, Bryan García Munévar
**Directora:** Carolina María Luque Zabala

## Resumen

Se construyó una serie mensual nacional de víctimas de homicidio intencional a partir de ~3,78 millones de registros del SIEDCO (Policía Nacional de Colombia), complementada con seis variables exógenas mensuales (hurto a personas, lesiones personales, violencia intrafamiliar, incautación de estupefacientes, Índice de Seguimiento a la Economía y tasa de desempleo). Sobre esa serie se entrenaron y compararon nueve modelos, tres estadísticos (SARIMA, SARIMAX, VAR) y seis de aprendizaje automático (XGBoost, LightGBM, Random Forest, LSTM, GRU, Prophet), bajo un esquema de validación temporal con control explícito de fuga de información: partición cronológica, rezago de un mes en todas las exógenas, y evaluación final frente al homicidio efectivamente observado entre enero y mayo de 2026, periodo nunca utilizado en el ajuste ni en la selección de modelos. La interpretabilidad se analizó con valores SHAP.

## Resultados principales (prueba real, enero-mayo de 2026)

| Modelo | MAE | RMSE | MASE | ME | MAPE (%) | R² |
|---|---|---|---|---|---|---|
| LightGBM | 14,62 | 16,61 | 0,19 | 1,35 | 1,25 | 0,84 |
| SARIMAX(2,0,1)(0,1,2)[12] | 31,55 | 33,16 | 0,42 | 15,79 | 2,75 | 0,37 |
| Random Forest | 31,30 | 35,84 | 0,41 | 28,88 | 2,66 | 0,26 |
| VAR HOMICIDIOS~ISE | 32,81 | 37,36 | 0,43 | 21,22 | 2,81 | 0,20 |
| SARIMA(2,0,1)(0,1,2)[12] | 41,04 | 47,41 | 0,54 | 37,33 | 3,60 | -0,30 |
| Prophet | 43,63 | 48,26 | 0,58 | 43,63 | 3,79 | -0,34 |
| XGBoost | 53,28 | 55,26 | 0,71 | 53,28 | 4,56 | -0,76 |
| LSTM | 239,92 | 243,82 | 3,18 | -239,92 | 20,83 | -33,25 |
| GRU | 295,14 | 298,03 | 3,91 | 295,14 | 25,34 | -50,17 |

ME = error medio (observado menos pronosticado). MASE se escala frente a un pronóstico ingenuo estacional (valor de 12 meses atrás). El R² se calcula sobre cinco observaciones y debe leerse solo como indicativo; los criterios principales de comparación son RMSE y MASE.

## Estructura del repositorio

```
.
├── Comparacion_Modelos_Homicidios_v13.ipynb   # Notebook principal (todo el flujo)
├── requirements.txt                            # Dependencias de Python
├── Datos/
│   ├── db_delitos_norm.csv                     # Registros SIEDCO (usado solo en el EDA)
│   ├── ts_mensual_2005_2025.csv                # Series mensuales 2005-2025 (modelos)
│   └── ts_mensual_2026.csv                     # Homicidios observados ene-may 2026 (prueba)
└── Resultados/
    ├── fig_*.png                               # Figuras exportadas por el notebook
    ├── tabla_comparacion_final_modelos.csv     # Métricas de los nueve modelos
    ├── resultados_pronosticos_2026.xlsx        # Pronósticos e intervalos por modelo (una hoja por modelo)
    
```

## Datos

| Archivo | Contenido | Uso en el notebook |
|---|---|---|
| `Datos/db_delitos_norm.csv` | Base consolidada del SIEDCO: ~3.785.395 registros y 18 campos (temática, año, mes, día, departamento, código DANE, municipio, zona, clase de sitio, género y grupo de edad de la víctima, arma o medio, cantidad, más campos auxiliares de la depuración de texto). Homicidios de 2003 a 2026; las demás temáticas hasta enero de 2026. | Sección 1 (análisis exploratorio) |
| `Datos/ts_mensual_2005_2025.csv` | Formato largo `Tematica, fecha_mensual, Cantidad` (fecha en `dd/mm/aaaa`). Siete series mensuales completas de enero de 2005 a diciembre de 2025: `HOMICIDIOS`, `HURTO A PERSONAS`, `LESIONES PERSONALES`, `VIOLENCIA INTRAFAMILIAR`, `INCAUTACION_KG` (SIEDCO), `ISE` y `TASA DESEMPLEO` (DANE). | Secciones 2 y 3 (modelos) |
| `Datos/ts_mensual_2026.csv` | Mismo formato (fecha en `mm/dd/aaaa`, con una fila vacía final que el notebook descarta). Contiene únicamente `HOMICIDIOS` observados de enero a mayo de 2026. | Sección 4 (conjunto de prueba real) |

Fuentes: Policía Nacional de Colombia, SIEDCO (estadística delictiva); DANE (ISE y tasa de desempleo). La variable objetivo es el número mensual de **víctimas** (suma del campo `Cantidad`), no el número de eventos. La confiabilidad de SIEDCO se contrastó contra Forensis (Medicina Legal) para 2015-2024 (correlación anual 0,955).


## Entorno y dependencias

Desarrollado en Python 3.12 sobre Windows, en Jupyter (VS Code). Versiones utilizadas en los resultados reportados:

| Componente | Versión |
|---|---|
| Python | 3.12.13 |
| NumPy | 2.0.2 |
| pandas | 2.2.2 |
| statsmodels | 0.14.6 |
| scikit-learn | 1.6.1 |
| XGBoost | 3.3.0 |
| LightGBM | 4.6.0 |
| TensorFlow / Keras | 2.20.0 |
| Prophet | 1.3.0 |
| SHAP | 0.52.0 |
| matplotlib, seaborn, openpyxl | (ver `requirements.txt`) |

Instalación:

```bash
python -m venv .venv
.venv\Scripts\activate          # Windows
# source .venv/bin/activate     # Linux / macOS
pip install -r requirements.txt
```

La fuente tipográfica de las figuras es Arial (`plt.rcParams["font.family"] = "Arial"`); en Linux/macOS sin Arial, matplotlib usará una fuente sustituta y emitirá una advertencia sin afectar los resultados.

## Cómo ejecutar

1. Ajuste las rutas de la Sección 0.3 del notebook (`RUTA_DATOS`, `RUTA_RESULTADOS`), que están definidas como rutas absolutas de Windows. Para usar rutas relativas al repositorio:
   ```python
   RUTA_DATOS = Path("Datos")
   RUTA_RESULTADOS = Path("Resultados")
   ```
2. Ejecute todas las celdas en orden (`Run All`). El orden importa para la reproducibilidad (ver más abajo).
3. Tiempo de ejecución de referencia en un equipo de escritorio sin GPU: las búsquedas de XGBoost, LightGBM y Random Forest (1.024, 1.024 y 768 modelos) toman minutos; LSTM y GRU (20 entrenamientos cada uno) y los explicadores SHAP de tipo Kernel son las etapas más lentas.
4. Para exportar el notebook ejecutado a HTML:
   ```bash
   jupyter nbconvert --to html --output-dir Resultados Comparacion_Modelos_Homicidios_v13.ipynb
   ```

## Estructura del notebook

| Sección | Contenido |
|---|---|
| 0. Configuración | Librerías, semilla, estilo gráfico, rutas, carga de los tres datasets, funciones compartidas (métricas MAE/RMSE/MASE/ME/MAPE/R², pruebas Ljung-Box/Jarque-Bera/ARCH, proyección de exógenas futuras, tabla y gráfico de pronóstico). |
| 1. Análisis exploratorio | Serie mensual 2015-2025; distribución por género, zona y arma o medio. |
| 2. Modelos estadísticos | Descomposición aditiva y subseries estacionales; SARIMA(2,0,1)(0,1,2)[12] (orden fijado a partir de la identificación Box-Jenkins); SARIMAX con búsqueda sobre las 64 combinaciones de exógenas (con `enforce_stationarity`/`enforce_invertibility` y descarte de ajustes no convergentes); VAR con selección de rezagos, prueba de causalidad de Granger y siete combinaciones de variables; síntesis de los tres. |
| 3. Modelos de aprendizaje automático | Conjunto supervisado (12 rezagos de la serie, exógenas rezagadas un mes, mes calendario); búsqueda conjunta de exógenas e hiperparámetros con validación 2024-2025; reajuste sobre el histórico completo; pronóstico recursivo ene-may 2026; intervalos aproximados; SHAP. XGBoost, LightGBM, Random Forest, LSTM, GRU, Prophet. |
| 4. Comparación | Tabla de métricas y pronósticos de los nueve modelos, gráfico comparativo, ranking y exportación a CSV/XLSX. |

## Decisiones metodológicas clave

- **Partición temporal.** Entrenamiento hasta diciembre de 2023, validación enero de 2024 a diciembre de 2025 (24 meses, `FECHA_CORTE_VAL = "2024-01-01"`), prueba real enero-mayo de 2026. La configuración ganadora de cada modelo se elige por RMSE de validación y luego se reajusta sobre todo el histórico hasta diciembre de 2025.
- **Control de fuga de información.** Todas las exógenas entran con rezago de un mes; el conjunto de prueba no participa en ninguna decisión de ajuste o selección.
- **Exógenas futuras.** No se dispone de valores de las exógenas para 2026, por lo que se proyectan por persistencia (último valor observado, diciembre de 2025). La función `preparar_exogenas_futuras()` admite reemplazarlas por proyecciones reales.
- **Pronóstico recursivo.** En los modelos de ML, la predicción de cada mes alimenta los rezagos del mes siguiente.
- **Intervalos.** SARIMA, SARIMAX, VAR y Prophet entregan intervalos nativos del 95 %. Para XGBoost, LightGBM, Random Forest, LSTM y GRU el intervalo es una aproximación: pronóstico puntual ± 1,96 × desviación estándar de los residuales de validación, constante en el horizonte.
- **Ventanas de entrenamiento.** Los modelos estadísticos se ajustan sobre `y_full` (2015-2025, 132 observaciones). Los modelos de ML reciben `wide_hist.loc[:"2025-12-01"]`, es decir, la serie completa disponible desde 2005; verifique el rango efectivo con `X.index.min()` al construir el conjunto supervisado.
- **Búsqueda de exógenas en LSTM, GRU y Prophet.** Por costo computacional no se recorren las 64 combinaciones, sino un subconjunto informado: sin exógenas, todas las exógenas y las combinaciones ganadoras de los modelos de árboles (`combos_dl`, `combos_prophet`).
- **Identificación Box-Jenkins.** La malla de 54 órdenes SARIMA y la búsqueda de 640 modelos SARIMAX (10 órdenes × 64 combinaciones) se realizaron originalmente en R; el notebook fija el orden resultante y replica en Python la búsqueda de exógenas para ese orden.

## Reproducibilidad y semillas

La Sección 0.1 fija `SEED = 42` y llama a `np.random.seed(SEED)` y `tf.random.set_seed(SEED)`. El alcance real de esa semilla difiere por modelo:

| Modelo | Fuente de aleatoriedad | Semilla | Reproducibilidad esperada |
|---|---|---|---|
| SARIMA, SARIMAX, VAR | Ninguna (máxima verosimilitud determinista) | No aplica | Exacta |
| XGBoost, LightGBM, Random Forest | Submuestreo de filas/columnas, bootstrap | `random_state=SEED` en cada ajuste (incluido en la malla de hiperparámetros) | Exacta, independiente del orden de ejecución |
| LSTM, GRU | Inicialización de pesos, dropout, orden de lotes | Solo la semilla global de TensorFlow fijada una vez al inicio; no se reinicia antes de cada `fit` ni del reajuste final | Reproducible únicamente ejecutando el notebook completo en orden; ejecutar una celda de forma aislada o en otro orden cambia los resultados. Además, TensorFlow en CPU no garantiza determinismo bit a bit sin `tf.config.experimental.enable_op_determinism()` |
| SHAP (KernelExplainer en LSTM/GRU) | Muestreo del fondo y de la muestra explicada; permutaciones internas | `random_state=SEED` en `shap.sample` y `DataFrame.sample`; las permutaciones internas usan el generador global de NumPy | Estable en ejecución ordenada; puede variar ligeramente fuera de orden |
| Prophet | Ajuste MAP determinista; los intervalos se obtienen por simulación (`uncertainty_samples`) con el generador global de NumPy | Sin semilla propia | Pronóstico puntual exacto; los límites del intervalo dependen del estado del generador global (ejecución en orden) |

Los resultados publicados en la tesis corresponden a una ejecución completa y ordenada del notebook con estas condiciones. Si se desea determinismo estricto en las redes recurrentes en ejecuciones futuras, añada al inicio de las celdas de LSTM y GRU:

```python
keras.utils.set_random_seed(SEED)
tf.config.experimental.enable_op_determinism()
```

Tenga en cuenta que ese cambio puede producir cifras distintas a las reportadas, precisamente porque las corridas originales no lo incluían.

## Resultados exportados

- `Resultados/tabla_comparacion_final_modelos.csv`: métricas de los nueve modelos sobre enero-mayo de 2026.
- `Resultados/resultados_pronosticos_2026.xlsx`: hoja `Comparacion_final` con las métricas y una hoja por modelo con el pronóstico mensual, sus límites del 95 % y el método de proyección de exógenas (`exog_metodo`).
- `Resultados/fig_*.png`: figuras del análisis exploratorio (`fig_1_*`), de los modelos estadísticos (`fig_2_*`), de los modelos de ML con sus gráficos SHAP y de pronóstico (`fig_3_*`) y la comparación final (`fig_4_2_comparacion_todos_los_modelos.png`).

## Limitaciones

Serie corta para redes recurrentes profundas; proyección de exógenas por persistencia; conjunto de prueba de cinco meses, sobre el cual el R² y la significancia de las diferencias entre modelos son poco estables; escala nacional agregada, sin desagregación territorial; intervalos aproximados para los modelos de ML; registros administrativos de denuncia, sujetos a rezagos de consolidación en los meses más recientes.

## Uso previsto

El pronóstico es nacional y agregado, orientado al monitoreo estratégico mensual. No fue diseñado ni validado para asignar recursos o vigilancia entre territorios ni para focalizar personas o comunidades. Los datos son conteos agregados; no contienen información personal.

## Cómo citar

Camelo Sánchez, F. E., Fontalvo Romero, E. J., y García Munévar, B. (2026). *Caracterización y predicción del homicidio intencional en Colombia: comparación entre modelos estadísticos y de aprendizaje automático con datos del SIEDCO* [Trabajo de grado de maestría, Universidad Ean]. Repositorio: `https://github.com/FabianLoco77/pronostico_homicidios_colombia`

## Licencia

Los datos del SIEDCO y del DANE se redistribuyen conforme a las condiciones de uso de sus fuentes oficiales; consulte la Policía Nacional de Colombia (estadística delictiva) y el DANE para los términos aplicables.
