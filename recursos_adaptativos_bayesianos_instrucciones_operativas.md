# Especificación operativa para IA

## Propósito

Este documento sirve para que una IA implemente recursos educativos adaptativos bayesianos de forma fiable.  
Es una especificación operativa breve.  
Si hace falta fundamento teórico, ejemplos o justificación matemática, consulta `documentacion_evaluacion_adaptativa_bayesiana.md`.  
Si hubiera conflicto entre ambos documentos, prevalece este.

## Instrucción de uso

Adjunta este documento a la IA y usa este prompt:

> Lee el documento adjunto e implementa el recurso siguiendo exactamente sus reglas operativas.

## Flujo obligatorio

1. Antes de implementar, comprueba si tienes esta información:
   - tema;
   - curso o edad;
   - objetivo de aprendizaje;
   - tipo de recurso;
   - finalidad principal;
   - número de hipótesis o niveles;
   - tipo de interacción;
   - número aproximado de preguntas o pasos;
   - salida final esperada.
2. Si falta información esencial, pregunta solo por lo imprescindible.
3. Si la información ya está dada, no repitas preguntas y pasa a implementar.
4. El resultado debe ser una página web estática autocontenida en un único archivo `HTML + CSS + JavaScript`, salvo que se pida explícitamente otro formato.

## Reglas de diseño

- El recurso no debe ser lineal si la finalidad exige adaptación.
- Cada respuesta del alumno debe modificar el estado estimado del sistema.
- La adaptación puede afectar a:
  - la siguiente pregunta;
  - la dificultad;
  - el tipo de actividad;
  - la explicación;
  - la ayuda o las pistas;
  - el itinerario;
  - el momento de finalizar.
- El resultado final debe ser pedagógico, no solo una puntuación.

## Estado del alumno

- Representa el estado del alumno como una distribución de probabilidad sobre `n` hipótesis.
- Si no hay información previa fiable, usa distribución uniforme.
- Si las hipótesis son jerárquicas, asígnales valores `theta` ordenados y centrados.
- Si el docente no fija valores, usa una escala simétrica centrada en 0.

## Actualización bayesiana

Tras cada respuesta:

1. calcula la verosimilitud de esa respuesta bajo cada hipótesis;
2. multiplica prior por verosimilitud;
3. normaliza;
4. usa el resultado como nuevo estado.

Esto debe hacerse después de cada interacción relevante.

## Verosimilitudes

- Si las hipótesis representan niveles ordenados de dominio, usa IRT 3PL.
- Fórmula recomendada:

`P(acierto | H_i, q) = c_q + (1 - c_q) / (1 + exp(-a * (theta_i - b_q)))`

- Usa por defecto:
  - `a = 1.5`
  - `c_q = 1 / m_q` si hay azar
  - `c_q = 0` si no lo hay
- Si el alumno falla:

`P(fallo | H_i, q) = 1 - P(acierto | H_i, q)`

- Si las hipótesis no son jerárquicas, no uses IRT logística. Define verosimilitudes diagnósticas específicas.
- No uses tablas fijas globales si cada pregunta puede generar sus propias verosimilitudes.

## Preguntas y actividades

Cada pregunta o interacción autocorregible debe tener, cuando proceda:

- texto o enunciado;
- dificultad `b_q`;
- número de opciones `m_q`;
- categoría o concepto;
- criterio de corrección;
- ayuda o pista opcional;
- explicación o retroalimentación.

Si el recurso es procedural o tutorial, las interacciones pueden no ser preguntas clásicas, pero deben seguir siendo autocorregibles o evaluables de forma explícita.

## Selección adaptativa

Para cada candidata disponible:

1. calcula la probabilidad marginal de acierto;
2. simula posterior si hay acierto;
3. simula posterior si hay fallo;
4. calcula la entropía esperada posterior;
5. calcula la ganancia esperada de información.

Selecciona la candidata con mayor ganancia esperada de información.

Si varias son prácticamente equivalentes:

- rompe empates con aleatorización;
- favorece categorías o conceptos menos repetidos.

No uses selección determinista simple en empates.

## Criterio de parada

Detén la sesión cuando se cumplan criterios razonables de cierre, por ejemplo:

- entropía por debajo de `H_stop`;
- hipótesis más probable por encima de `p_min`;
- mínimo de preguntas ya cumplido;
- no quedan preguntas útiles;
- la mejor pregunta restante aporta muy poca información;
- se alcanza el máximo práctico.

Reglas mínimas:

- no cierres demasiado pronto;
- no alargues artificialmente la sesión cuando la utilidad marginal ya es baja;
- si la incertidumbre sigue siendo alta, el resultado debe indicarse como provisional.

## Itinerarios por etapas

Si el recurso tiene fases, técnicas o etapas sucesivas:

- distingue entre estimación global y estimación local por etapa;
- no promociones una etapa usando solo la creencia global acumulada;
- decide la superación de la etapa con evidencia generada dentro de esa etapa.

Para dar una etapa por superada, conviene exigir al menos:

- confianza local suficiente;
- entropía local suficientemente baja;
- y un mínimo explícito de rendimiento observado en esa etapa.

Ejemplo razonable:

- `p_min = 0.80`
- mínimo de `60 %` de aciertos en la etapa

Si el alumno repite una etapa:

- reinicia la estimación local de esa etapa, salvo justificación pedagógica explícita en contra.

Terminar una etapa no implica necesariamente haberla superado.

## Recuperación y refuerzo

- Si el alumno muestra dificultades, el sistema debe poder:
  - ofrecer pistas;
  - mostrar explicación;
  - proponer refuerzo;
  - cambiar el tipo de actividad;
  - reducir temporalmente la dificultad;
  - permitir reintento o repaso.
- Si muestra dominio, puede:
  - avanzar;
  - ampliar;
  - aumentar complejidad;
  - reducir ayuda.

## Resultado final

La salida final debe incluir, según el tipo de recurso:

- diagnóstico o nivel estimado;
- grado de confianza;
- dificultades detectadas;
- fortalezas observadas;
- recomendación pedagógica;
- siguiente paso.

Si es un itinerario o actividad de aprendizaje, añade además:

- recorrido seguido;
- etapas superadas o no;
- ayudas usadas;
- áreas a reforzar.

No devuelvas solo una nota o etiqueta.

## Restricciones de implementación

- La interfaz debe ser comprensible para alumnado y profesorado.
- Debe mostrar con claridad el progreso y la retroalimentación.
- Si usas fórmulas visibles, acompáñalas de interpretación legible.
- Evita depender de backend si no se ha pedido.
- Evita preguntas abiertas largas sin corrección automática fiable.

## Valores por defecto recomendados

Si el docente no especifica parámetros:

- `n = 3` hipótesis o niveles;
- `a = 1.5`;
- `p_min = 0.80`;
- mínimo de preguntas: entre `4` y `6`;
- máximo práctico: entre `10` y `20`, según el tipo de recurso.

## Qué no debe hacer la IA

- No confundir una explicación teórica con una regla obligatoria de implementación.
- No usar la creencia global para promocionar etapas locales en itinerarios.
- No cerrar el recurso sin justificar la certeza alcanzada.
- No presentar como firme un resultado con incertidumbre alta.
- No limitar la salida a una puntuación desnuda.
