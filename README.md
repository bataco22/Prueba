# Centro Quant B v1.2 · QRA-09 — 2026-08-30

Esta versión parte del ZIP `Centro_Quant_B_v1_1_Hotfix_Monitor_Shadow_2026-08-29` y conserva intactas las reglas operativas de Quant B.

## QRA-09 · Gestión dinámica de salida
- `B_MAIN` sigue siendo la referencia operativa y NO es modificado por QRA-09.
- QRA-09 sólo admite operaciones nuevas nacidas después de su frontera prospectiva fija (2026-08-31 02:12 UTC).
- Cada entrada elegible comparte exactamente entrada, dirección y velas con B_MAIN.
- Mantiene tres posiciones virtuales independientes: Escalera, Trailing 0.20R y Trailing 0.25R.
- Cada rama se cierra causalmente conforme llegan velas de 1 minuto y guarda `status`, `stopR`, `maxR`, `resultR` y `closedAt` dentro de `qra09` en el JSON.
- Escalera: antes de +1R conserva stop -1R; +1R→BE; +1.25R→protege +1R; +2R→+1.25R; +2.5R→+2R; después continúa por escalones.
- Trailing: antes de +1R conserva stop -1R; al alcanzar +1R pasa a BE y después sigue el máximo en pasos de 0.20R o 0.25R según la rama.
- Las ramas QRA-09 pueden cerrar aunque B_MAIN siga abierta, y el monitor continúa siguiendo la comparación sin contaminar el ledger operativo.
- Las operaciones anteriores al corte permanecen fuera de la cohorte QRA-09 aunque ya tengan datos históricos de `exitComparison`.

# Centro Quant B v1 — Challenger prospectivo congelado 2026-08-29

Este ZIP es un CLON independiente de Centro Quant A v6.11.7. No sustituye ni modifica el control. Usa claves localStorage `quantb_` para evitar mezclar el ledger con A.

Reglas B v1 congeladas:
- QRA-06 CONTRA_CONTEXTO / RIESGO_ENTRADA_TARDIA: rechaza.
- QRA-06 OBSERVAR / ESPERAR_PULLBACK: espera/rechaza la entrada actual.
- LONG + QRA-05 REANUDACION + QRA-06 TIMING_CANDIDATO: entra, objetivo 3R.
- SHORT + REANUDACION + TIMING_CANDIDATO: OFF.
- QRA-05 LATERAL o TRANSICION + QRA-06 OBSERVAR/TIMING_CANDIDATO (nunca CONTRA_CONTEXTO, tarde ni esperar pullback): rama experimental B-EXP-LATERAL, objetivo fijo 0.25R.
- Resto: no trade.
- Las señales rechazadas se guardan como shadow trades separados y se siguen contra el control 3R, sin contarlas como operaciones B.
- QRA-06 se calcula point-in-time ANTES de autorizar la entrada B.
- Entrada real sigue siendo al open causal del siguiente minuto completo.

No optimizar reglas durante la cohorte. Cortes sugeridos: 25 (bugs), 50 (preliminar), 100 (comparación A/B).

# Centro Quant v6.11.7 · QRA-06 Timing/Contexto OOS

Base operativa congelada: v6.11.6 Entrada Sincronizada.

## Qué añade QRA-06
- Capa estrictamente `research-only` y fuera de muestra.
- Observa cada operación DESPUÉS de que el control ya fue creado.
- Reconstruye 15m, 1h, 4h y 1d usando `endTime = createdAt - 1` para evitar información futura.
- Registra contexto, score/opuesto, tendencia, EMA20/50/55/200, RSI, ADX, ATR, volumen, posición en rango 20, ruptura de estructura y fase QRA-05.
- Produce una decisión VIRTUAL: `TIMING_CANDIDATO`, `ESPERAR_PULLBACK`, `RIESGO_ENTRADA_TARDIA`, `CONTRA_CONTEXTO` u `OBSERVAR`.

## Lo que NO cambia
- Score y pesos.
- Umbral de entrada.
- `scanAutoPaper()`.
- Frescura/sincronización de señal v6.11.6.
- `activateCausalTrade()`.
- Entrada causal al siguiente minuto.
- Stop, target, RR 1:3.
- Seguimiento, MFE/MAE, trailing/escalera o cierres.

QRA-06 solo escribe metadatos observacionales en `qra06Context` dentro del JSON para comparar después con el resultado real.
