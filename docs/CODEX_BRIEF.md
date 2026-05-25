# Brief para Codex: herramienta bioeconómica para índices de selección en bovinos de carne

## Contexto general

Se desea diseñar una herramienta/software para construir índices de selección en bovinos de carne a partir de objetivos económicos definidos para una raza o programa de evaluación genética. El objetivo no es copiar iGenDec, sino usarlo como referencia técnica para diseñar una versión propia, más simple, trazable y adaptada a trabajo dentro de una misma raza.

El usuario trabaja en genética y mejoramiento animal, con experiencia en evaluaciones genéticas, DEPs/EBVs, modelos mixtos, BLUPF90 y programación en R. La herramienta debería pensarse inicialmente como un motor reproducible, probablemente en R, y luego eventualmente como una aplicación Shiny.

## Antecedentes revisados

Se discutieron tres líneas de antecedentes:

1. Trabajos de Urioste, Pravia y colaboradores sobre objetivos de selección e índices económicos en bovinos de carne, incluyendo modelos bioeconómicos para sistemas de Uruguay.
2. Presentación del software OS, orientado a estimar objetivos de selección, valores económicos e índices de selección en ganado de carne.
3. Repositorios de iGenDec:
   - `blgolden/igendec`: interfaz web.
   - `blgolden/iGenDecModel`: modelo de simulación y programa `starter`.

Los forks disponibles en esta cuenta son:

- `alanmaxsp/igendec`
- `alanmaxsp/iGenDecModel`

## Diagnóstico metodológico

El enfoque de iGenDec es similar al de Urioste/Pravia en el corazón bioeconómico: ambos usan un sistema productivo-económico explícito para estimar valores económicos marginales. Sin embargo, no son idénticos.

### Enfoque Urioste/Pravia/OS

- Define primero el sistema productivo.
- Identifica caracteres del objetivo de selección.
- Calcula valores económicos marginales a partir de una función de beneficio.
- Distingue entre caracteres del objetivo y criterios disponibles como EPD/DEP.
- Usa teoría formal de índices de selección y matrices genéticas para derivar los coeficientes del índice.
- Permite estimar respuestas esperadas y ganancias económicas.

### Enfoque iGenDec

- Simula un sistema comercial de bovinos de carne.
- Corre un escenario basal.
- Aplica un `bump` o incremento en componentes genéticos del toro.
- Estima MEV como diferencia entre el retorno neto medio del escenario perturbado y el basal.
- Usa esos MEV para aplicarlos a EPDs.
- En el código de `starter/main.go`, el MEV se calcula como `mev.mev = m - bmean`.
- Multiplica por 2 los MEV cuando los exporta para aplicarlos a EPD, porque EPD = EBV / 2.

## Decisiones conceptuales tomadas

1. La herramienta debe trabajar inicialmente dentro de una raza o población racial definida.
2. No interesa incluir heterosis ni composición racial en el núcleo inicial.
3. La heterosis de iGenDec se considera innecesaria para esta primera versión.
4. La herramienta no debe ser una simple calculadora de ponderaciones subjetivas de DEPs.
5. Debe partir de un objetivo económico explícito.
6. Debe separar claramente:
   - caracteres del objetivo;
   - criterios de selección disponibles;
   - parámetros económicos;
   - parámetros genéticos;
   - coeficientes finales aplicables a DEPs.
7. Debe manejar explícitamente la escala EBV versus DEP/EPD.
8. Debe guardar escenarios de forma reproducible.
9. Debe permitir comparar objetivos alternativos, por ejemplo maternal, terminal o ciclo completo.

## Arquitectura sugerida

Se propone no modificar directamente iGenDecModel, sino diseñar una herramienta nueva tomando iGenDec como referencia conceptual y computacional.

Arquitectura inicial sugerida en R:

```r
define_scenario()
define_economics()
define_objective_traits()
define_selection_criteria()
simulate_profit()
estimate_economic_values()
build_selection_index()
convert_ebv_to_dep_scale()
calculate_trait_emphasis()
apply_index_to_catalog()
compare_indexes()
export_report()
```

## Flujo conceptual mínimo

```text
1. Definir sistema productivo.
2. Definir precios y costos.
3. Definir caracteres del objetivo.
4. Calcular beneficio basal.
5. Perturbar cada carácter objetivo o componente genético.
6. Calcular valores económicos marginales.
7. Incorporar matriz genética entre caracteres objetivo y criterios disponibles.
8. Derivar coeficientes del índice.
9. Convertir coeficientes a escala DEP si corresponde.
10. Aplicar el índice a una base de animales.
11. Generar ranking, énfasis relativo y reporte.
```

## Fórmulas centrales

Objetivo agregado de selección:

```text
H = v'a
```

Índice de selección:

```text
I = b'EBV
```

o en escala DEP:

```text
I = b_DEP'DEP
```

Valor económico marginal por diferencias finitas:

```text
v_i ≈ [Π(x_i + Δ) - Π(x_i)] / Δ
```

Enfoque iGenDec simplificado:

```text
MEV_i = mean(Π_bumped_i) - mean(Π_base)
```

Conversión de escala:

```text
DEP = EBV / 2
```

Por lo tanto, si el coeficiente fue estimado sobre escala EBV, para aplicarlo directamente a DEP se debe considerar la conversión correspondiente, como hace iGenDec al multiplicar por 2 la salida para EPD.

Énfasis relativo sugerido:

```text
emphasis_i = |MEV_i| * σ_Ai / sum_j(|MEV_j| * σ_Aj)
```

## Alcance inicial recomendado

Primera versión: sistema simple de cría o ciclo completo, sin heterosis.

Caracteres candidatos:

- tasa de preñez o destete;
- facilidad de parto;
- peso al nacimiento;
- peso al destete directo;
- peso al destete materno;
- peso final o peso de venta;
- peso adulto de vaca;
- carcasa, si hay datos disponibles.

Criterios disponibles:

- DEP PN;
- DEP PD directo;
- DEP PD materno;
- DEP peso final;
- DEP circunferencia escrotal;
- DEP facilidad de parto;
- DEP AOB/grasa/carcasa, si existen.

## Recomendación técnica

No replicar todo iGenDecModel. Implementar primero un motor pequeño, explícito y testeable en R. Luego agregar complejidad. La prioridad inicial debe ser reproducibilidad, claridad de supuestos y separación entre objetivo económico y criterio de selección.

## Tarea inicial para Codex

Diseñar la estructura inicial de un paquete R o prototipo de motor bioeconómico para índices de selección en bovinos de carne, sin heterosis, con:

1. estructuras de datos para escenarios productivos;
2. estructuras para caracteres objetivo y criterios disponibles;
3. función de beneficio basal;
4. cálculo de valores económicos marginales por diferencias finitas;
5. manejo explícito de escala EBV/DEP;
6. cálculo de énfasis relativo;
7. aplicación del índice a una tabla de DEPs;
8. ejemplos mínimos reproducibles;
9. pruebas unitarias simples.

El código debe ser modular, documentado y pensado para que posteriormente pueda tener una interfaz Shiny.
