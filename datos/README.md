# Datos

Todas las fuentes son públicas. Los archivos chicos ya están en el repositorio; los microdatos de
la EPH, por su tamaño, hay que descargarlos y copiarlos en `datos/eph/`.

## `eph/` (no incluido: descargar)

Microdatos del INDEC, base **de individuos**:

| Archivo esperado | Relevamiento | Lo usan |
|---|---|---|
| `personas_tot.urb_3T_2016.txt` … `personas_tot.urb_3T_2025.txt` | Encuesta Permanente de Hogares (EPH) continua, 3.er trimestre de cada año | notebooks 02 a 05 y 07 |
| `Individual_EAHU_10.dta` | Encuesta Anual de Hogares Urbanos (EAHU) 2010, en formato Stata | notebook 07 (pirámide y nacimientos de 2010) |

Las bases de la EPH se descargan desde la sección *Bases de datos* del sitio del INDEC
(<https://www.indec.gob.ar/indec/web/Institucional-Indec-BasesDeDatos>). Cada trimestre viene como
`usu_individual_T3AA.txt` (separador `;`): hay que renombrarlo a `personas_tot.urb_3T_AAAA.txt`.
Se usan las columnas originales del INDEC (`CODUSU`, `NRO_HOGAR`, `ANO4`, `AGLOMERADO`, `PROVINCIA`,
`CH03`, `CH04`, `CH06`, `CH07`, `CH08`, `CH10`, `CH12`, `CH13`, `CH14`, `NIVEL_ED`, `ESTADO`, `PP07H`,
`ITF`, `IPCF`, `PONDERA`, `PONDIH`, entre otras).

## `mortalidad/` (incluido)

*Tablas abreviadas de mortalidad, por sexo y edad. Total del país y jurisdicciones* del INDEC
(<https://www.indec.gob.ar/indec/web/Nivel4-Tema-2-42-148>), insumo del vector de mortalidad
(notebook 01). El código identifica cada tabla por el año central de su período (2000, 2009 y 2019).

| Archivo | Período | Formato |
|---|---|---|
| `mortalidad_01.csv` | 2000-2001 | `year, provincia, sex, age, n, qx` (provincia numerada como en la EPH, 1-24; `sex`: 0 varón, 1 mujer) |
| `mortalidad_10.csv` | 2008-2010 | ídem |
| `mortalidad_19.csv` | 2019 | tabla publicada, separador `;` y coma decimal (`Jurisdiccion_cod`, `Sexo`, `Edad`, `n`, `q`, …) |

## `deis/` (incluido)

Estadísticas vitales de la Dirección de Estadísticas e Información de Salud (DEIS, Ministerio de
Salud), usadas para validar la natalidad de la EPH (notebook 05):

| Archivo | Contenido |
|---|---|
| `natalidad_nacional.csv` | Tasa bruta de natalidad del total del país, por mil habitantes, 2010-2024 |
| `natalidad_2022.txt`, `natalidad_2023.txt`, `natalidad_2024.txt` | Población y nacidos vivos registrados por jurisdicción |
