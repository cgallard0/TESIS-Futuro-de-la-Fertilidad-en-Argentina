# El Futuro de la Fertilidad en Argentina

**Asimetría por Nivel Educativo y Formación de Capital Humano**

Código de la tesis de Licenciatura en Economía de **Catalina Inés Gallardo** (Universidad de
San Andrés, Departamento de Economía, 2026). Mentora: María Noelia Romero.

La fertilidad argentina cayó 34,7 % entre 2016 y 2024, de 1,84 a 1,20 hijos por mujer, y la caída
fue asimétrica por nivel educativo. La tesis combina el panel de la Encuesta Permanente de Hogares
(EPH) con modelos de *machine learning* que predicen el comportamiento reproductivo, laboral,
de pareja y educativo de cada persona, y con un motor de microsimulación estocástica que proyecta
la población urbana hasta 2075 bajo dos escenarios de fertilidad. Este repositorio reproduce todas
las figuras y tablas del documento.

---

## Estructura

```
├── README.md
├── requirements.txt
├── datos/                     insumos (ver datos/README.md)
│   ├── eph/                   microdatos EPH 2016-2025 y EAHU 2010  ← descargar del INDEC
│   ├── mortalidad/            tablas de vida del INDEC (incluidas)
│   └── deis/                  estadísticas vitales de la DEIS (incluidas)
├── notebooks/                 el pipeline completo, en orden
└── resultados/
    ├── figuras/               las 10 figuras de la tesis
    ├── tablas/
    │   ├── tesis/             cada tabla de la tesis, con su formato (una por archivo)
    │   ├── descriptivas/      detalle de la estadística descriptiva y de la validación de datos
    │   └── resultados/        series proyectadas por escenario (insumo de las figuras)
    └── intermedios/           vectores, modelos, población base y simulaciones (no versionado)
```

## Cómo correrlo

1. Crear un entorno con Python 3.12 e instalar las dependencias:
   ```bash
   pip install -r requirements.txt
   ```
2. Descargar los microdatos de la EPH y de la EAHU 2010 y copiarlos en `datos/eph/` con los
   nombres indicados en [`datos/README.md`](datos/README.md).
3. Correr los notebooks **en orden**, desde Jupyter o en la terminal (cada notebook detecta solo
   la carpeta raíz del proyecto):
   ```bash
   cd notebooks
   for nb in 0*.ipynb; do
     jupyter nbconvert --to notebook --execute --inplace --ExecutePreprocessor.timeout=-1 "$nb"
   done
   ```
   Como el notebook 06 tarda muchas horas, conviene correrlo desde la terminal y no desde el
   navegador.

Todo el pipeline es reproducible: la imputación de ingresos, los modelos y cada réplica de la
microsimulación usan semillas fijas.

## Notebooks, orden de ejecución y tiempos

Los tiempos son aproximados, medidos en una computadora portátil (10 núcleos, 16 GB de RAM). La
corrida de las simulaciones usada en la tesis tardó unas 17 horas.

| # | Notebook | Qué hace | Depende de | Tiempo |
|---|---|---|---|---|
| 01 | `01_mortalidad.ipynb` | Vector de mortalidad 2022-2075 por provincia, sexo y edad (tablas de vida 2000, 2009 y 2019, tendencia en logit, piso a la brecha por sexo). **Figura:** esperanza de vida al nacer. | — | 15 s |
| 02 | `02_poblacion_base.ipynb` | Población base 2025 desde la EPH (tercil de ingreso por adulto con imputación, clima educativo, paridad). Descarta la edad 0. | — | 10 s |
| 03 | `03_vectores_ciclo_de_vida.ipynb` | Vectores de empleo, emparejamiento (pareja, pareja ocupada, clima educativo) y educación (stock por cohorte y flujo condicionado al destino): XGBoost calibrado + *drift* por estrato. | — | 2-3 min |
| 04 | `04_vector_fertilidad.ipynb` | Ajuste de los nacimientos recientes; comparación de modelos para la cantidad de hijos; vector de fertilidad `f(X) + g(t) + desvío` en los dos escenarios; cohorte de edad 0 de 2025. **Figura:** fecundidad por edad y nivel educativo. | 01, 02, 03 | 9 min |
| 05 | `05_estadistica_descriptiva.ipynb` | Estadística descriptiva 2016-2024 con *bootstrap* por hogar (asimetría, *tempo*, contribución por edad, ingreso vs. clima educativo) y validación contra las estadísticas vitales. **Figura:** no linealidad ingreso-fecundidad. | 04 | 1 min |
| 06 | `06_simulaciones.ipynb` | Motor de microsimulación: 1.000 réplicas × 50 años en cada escenario. | 01, 03, 04 | **17-25 h** (30-45 s por réplica) |
| 07 | `07_resultados_agregacion.ipynb` | Resume las 2.000 réplicas (mediana e IC 95 %), arma las series observadas para el empalme y calcula la descendencia final por generación. | 04, 06 | 6-7 min |
| 08 | `08_figuras_resultados.ipynb` | Las 7 figuras del capítulo de Resultados. | 05, 07 | 10 s |
| 09 | `09_tablas_tesis.ipynb` | Todas las tablas de la tesis, con su formato. | 02-07 | 5 s |

Sin contar las simulaciones, el pipeline corre en unos 20 minutos. Para probarlo de punta a punta
sin esperar las simulaciones completas, se puede correr el notebook 06 con pocas réplicas, por ejemplo
`K_SIMULACIONES=5 jupyter nbconvert --execute ...` (las bandas de confianza no son
interpretables con tan pocas corridas). Si el notebook 06 se interrumpe, al volver a ejecutarlo
retoma desde la primera réplica que falta. Con la variable `DIR_SIMULACIONES` se puede guardar o
leer las simulaciones en otra carpeta (ocupan unos 2 GB).

## Figuras de la tesis

| Figura | Archivo | Notebook |
|---|---|---|
| Tasa específica de fecundidad por edad y nivel educativo (Introducción) | `eda_asfr_edad_x_educacion.png` | 04 |
| No linealidad de la fecundidad: nacimientos de 2024 por decil de ingreso per cápita familiar | `barras_tradeoff_natalidad_2024.png` | 05 |
| Esperanza de vida al nacer proyectada, por sexo, 2019-2075 | `esperanza_vida.png` | 01 |
| Tasa Global de Fecundidad proyectada, Escenario Base y Alternativo | `fig_tgf_escenarios.png` | 08 |
| Estructura de la población por edad y sexo, 2010, 2025, 2050 y 2075 | `fig_piramide_evolucion_horizonte.png` | 08 |
| Tasa bruta de natalidad y de mortalidad, 2016-2075 | `fig_tbn_tbm_historico_proyectado.png` | 08 |
| Composición de la población total por grupos de edad | `fig_composicion_tramos_funcionales.png` | 08 |
| Matrícula proyectada por nivel del sistema educativo | `fig_matricula_por_tramo.png` | 08 |
| Composición educativa de la población de 25 a 65 años | `fig_composicion_educativa_adultos_25_65.png` | 08 |
| Composición de los nacimientos por clima educativo del hogar | `fig_composicion_nacimientos_clima.png` | 08 |

## Tablas de la tesis

Todas quedan en `resultados/tablas/tesis/` (notebook 09).

| Tabla | Archivo |
|---|---|
| Asimetría educativa en la caída de la TGF, 2016 y 2024 | `tab_asimetria.csv` |
| Descomposición de la edad media a la maternidad | `tab_tempo_mac.csv` |
| Variación de la MAC truncada por nivel educativo | `tab_tempo_estrato.csv` |
| Contribución de cada tramo de edad a la caída de la TGF | `tab_contribucion_edad.csv` |
| Comparación de especificaciones para la cantidad total de hijos | `tab_comparacion_modelos.csv` |
| Composición de la población base 2025 por tercil de ingreso por adulto | `tab_composicion_base.csv` |
| *Completed fertility* por generación, Escenario Base y Alternativo | `tab_generaciones.csv` |
| TBN de la EPH y de las estadísticas vitales, total del país (Apéndice) | `tab_validacion_nacional.csv` |
| TBN de la EPH y de las estadísticas vitales, por grupo de jurisdicciones (Apéndice) | `tab_validacion_externa.csv` |
| Factores anuales de ajuste de los nacimientos recientes (Apéndice) | `tab_factor_edad0.csv` |
| Mujeres de 15 a 49 años por clima educativo del hogar (Apéndice) | `tab_clima_cinco.csv` |
| Heterogeneidad del *drift* entre estratos, por vector (Apéndice) | `tab_heterogeneidad_drift.csv` |
| Métricas de validación de los modelos de clasificación (Apéndice) | `tab_validacion_vectores.csv` |
| Calibración del modelo de fertilidad (Apéndice) | `tab_calibracion_fertilidad.csv` |

Las cifras que el texto cita sin tabla propia (por ejemplo, la caída de la TGF por clima educativo
y por ingreso, o las participaciones de cada clima en los nacimientos) están en
`resultados/tablas/descriptivas/` y `resultados/tablas/resultados/`.

## Notas

- **Universo.** Todo se lee dentro de la población urbana que releva la EPH; los totales no son
  comparables con los del total del país.
- **Escenarios.** El Escenario Base estabiliza la fecundidad en un piso de 0,95 hijos por mujer;
  el Alternativo (rebote) recupera 0,25 hijos en 12 años después del mínimo. Los parámetros se
  declaran en el notebook 04.
- **Simulaciones.** El notebook 06 exporta sólo los cortes que usan las figuras y las tablas
  (población por sexo y edad, TGF sintética, paridad por cohorte, matrícula y nivel educativo por
  edad, y origen socioeducativo). Los sorteos aleatorios no dependen de qué se exporta.

## Diferencias con el documento

Reconstruido desde cero, el pipeline reproduce las figuras y las tablas de la tesis (las
simulaciones, réplica por réplica, son idénticas a las originales). Las únicas diferencias son:

- **Tabla de *completed fertility* por generación.** Generación X da 1,90 hijos (la tesis
  reporta 1,89) y *Millennials* 1,85 (tesis: 1,84), en los dos escenarios. La versión original
  del análisis usaba un panel de la EPH guardado antes de que el estrato de ingreso pasara de
  deciles de ingreso per cápita a terciles de ingreso por adulto; acá ese panel se construye con
  la definición vigente. Las generaciones simuladas no cambian.
- **Descomposición de la edad media a la maternidad.** Los valores *p* del *bootstrap* son
  0,140 (MAC original) y 0,674 (MAC truncada); la tesis reporta 0,102 y 0,610. En ningún caso la
  diferencia es significativa.
- **Texto, caída de la TGF por estrato de ingreso** (deciles 1-3, 4-7 y 8-10): el código da
  -32,4 %, -26,2 % y -35,0 % (el texto dice 31,3 %, 26,3 % y 34,1 %).
- **Redondeos.** En la tabla de factores de ajuste, `g_T` de 2017 es 1,0025 → 1,002 y `r_T`, `g_T`
  de 2019 son 1,2205 y 1,1555 → 1,220 y 1,155 (la tesis muestra 1,003, 1,221 y 1,156). En la
  validación por jurisdicciones, el cociente total con la EPH original es 0,9247 → 0,92 (tesis:
  0,93).
