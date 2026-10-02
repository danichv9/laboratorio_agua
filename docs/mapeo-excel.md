# Mapeo del Excel

Origen: `modelo_tratamiento_aguas_bacterias_dinamicas.xlsx`, extraído sin modificar. Se conservan valores y fórmulas de referencia en `src/model/excel-source.json`. La app no ejecuta fórmulas de un archivo cargado.

| Hoja y celda | Parámetro | Valor original / unidad |
|---|---|---|
| Parámetros!B4 | YH | 0,6 kg SSTV/kg DBO₅ |
| Parámetros!B5 | YAOB+YNOB | 0,15 kg SSTV/kg N; dividido 2/3 y 1/3 |
| Parámetros!B8 | oxygenAOB+oxygenNOB | 4,57 kg O₂/kg N; dividido 3,43 y 1,14 |
| Parámetros!B9 | oxygenCarbon | 1 kg O₂/kg DBO₅ |
| Parámetros!B10 | alkalinityFactor | 7,14 kg CaCO₃/kg NH₄-N |
| Parámetros!B22 | muHmax | 4 d⁻¹ |
| Parámetros!B23 | muAOBmax | 0,8 d⁻¹; extensión de μN a AOB |
| Parámetros!B24 | bH | 0,06 d⁻¹ |
| Parámetros!B25 | bAOB y bNOB | 0,08 d⁻¹ |
| Parámetros!B26 | KOH | 0,2 mg O₂/L |
| Parámetros!B27 | KOAOB | 0,5 mg O₂/L |
| Parámetros!B28 | thetaH | 1,04 |
| Parámetros!B29 | thetaAOB y thetaNOB | 1,07 |
| Parámetros!B30 | SRT | 15 d |
| Parámetros!B31 | XH inicial | 100 kg → 16,129 mg/L con V original |
| Parámetros!B32 | XAOB+XNOB inicial | 5 kg → 0,80645 mg/L; fracción AOB editable |
| Parámetros!B33 | V | 6200 m³ |
| Parámetros!B34 | Tref | 20 °C |
| Dinámica bacteriana!C37 | Q | 5000 m³/d |
| Dinámica bacteriana!D37 | BODin | 250 mg/L |
| Dinámica bacteriana!E37 | BOD inicial | 20 mg/L |
| Dinámica bacteriana!F37 | NH4in | 30 mg N/L |
| Dinámica bacteriana!G37 | NH4 inicial | 2 mg N/L |
| Dinámica bacteriana!H37 | DOset y DO inicial | 2 mg O₂/L |
| Dinámica bacteriana!I37 | temperature | 20 °C |

La inicialización añadida usa NO₂=0 y NO₃=28 mg N/L para representar el efluente previamente nitrificado; no proceden del Excel. KLa=30 d⁻¹ es una hipótesis; el modo original es DO impuesto. NO₂/NO₃ de entrada=0. Nuevos parámetros KBOD, KNH4, KNO2, muNOBmax, KONOB, YAOB, YNOB y splitAOB están centralizados y son editables.

Las filas 37–97 de Dinámica bacteriana proporcionan día, fase, caudal, DBO entrada/salida, NH₄ entrada/salida, DO y temperatura. Cambios de fase: día 0 Inicial; 1 Arranque; 16 Baja DBO; 26 Bajo NH₄; 36 Limitación O₂; 44 Parada; 51 Reinicio. La secuencia completa conserva incluso condiciones diarias; los escenarios individuales aplican su fase desde el día 0. Los efluentes de esas filas solo se usan para reproducción Excel; el reactor avanzado calcula sus propios efluentes.

## Fórmulas verificadas

Dinámica!J38: `min(YH·Q·max(DBOin−DBOout,0)/1000·DO/(KOH+DO), μHmax·θH^(T−Tref)·DO/(KOH+DO)·Hprev)`.

K38: `(bH+1/SRT)Hprev`. L38: `max(0,Hprev+crecimiento−pérdidas)`. M38: `H·1000/V`. N/O/P/Q repiten el esquema para una sola población nitrificante y sus parámetros. La reproducción mantiene exactamente este paso diario y el clamp original; el reactor avanzado tiene integración conservativa distinta.

Operación diaria!I: Q·max(DBOin−DBOout,0)/1000. J: Q·max(NH4in−NH4out,0)/1000. L: I·factor carbonáceo. M: J·4,57. N: L+M. O: J·7,14. P: I·YH+J·YA. Q: b general·MLSS/1000·fracción SSTV·V. Estas filas por fecha no corresponden temporalmente a los días de Dinámica; la vista las presenta por separado.

Parámetros B6, B7, B11 y B12 se conservan en los datos fuente pero no sustituyen bH/bN en el reactor dinámico. La biomasa activa inicial no se deduce del MLSS operativo.

## Importación y errores

Se requieren las tres hojas por nombre exacto, parámetros numéricos en las celdas conocidas y condiciones iniciales en fila 37. Los valores numéricos guardados de fórmulas pueden leerse, pero jamás se evalúa su expresión. Fórmulas sin caché aparecen como «Sin caché» en comparaciones. Si falta una entrada obligatoria, se rechaza con hoja/celda. Cambiar la estructura del libro requiere adaptar `excelImport.ts`. El lector reconoce el formato del modelo, no cualquier hoja arbitraria de aguas residuales.
