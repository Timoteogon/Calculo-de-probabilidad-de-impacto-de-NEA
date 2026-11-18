# Cálculo de la probabilidad de impacto de un NEA
 
Estimación de la probabilidad de impacto $P_i$ del asteroide cercano a la Tierra **(29075) 1950 DA** (clase Apolo) en su posible encuentro con la Tierra en **marzo de 2880**, usando el método del plano B con linealización. El resultado se compara con el de Sentry (JPL).
 
Todo el análisis está en un solo notebook de Python: `Calculo_de_probabilidad_de_impacto_de_NEA.ipynb`.
 
[![Abrir en Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Timoteogon/Calculo-de-probabilidad-de-impacto-de-NEA/blob/main/Calculo_de_probabilidad_de_impacto_de_NEA.ipynb)
 
## Resultados
 
| | Resultado |
|---|---|
| Radio efectivo de la Tierra, $R_{\rm ef}$ | $8.13\times10^{3}$ km |
| Sigmas de $\Gamma$ en el plano B | $2.6\times10^{3}$ km y $4.0\times10^{6}$ km |
| $P_i$ (linealizado desde la solución de impacto) | $2.91\times10^{-4}$ |
| $P_i$ (nominal integrado como centro) | $0$ |
| $P_i$ de Sentry | $3.4\times10^{-4}$ |
| Escala de Palermo (con el $P_i$ linealizado) | $\approx -0.95$ |
 
El $P_i$ linealizado desde la solución de impacto es del mismo orden que el de Sentry.
 
### Advertencia
 
El resultado **depende del punto donde se linealiza**. Linealizando en otros puntos del barrido de $A_2$ ($z=0,\ 0.5,\ 1.0$) se obtienen valores entre $2\times10^{-65}$ y $3\times10^{-14}$, frente a $2.9\times10^{-4}$ en la solución de impacto ($z=1.647$). Por eso el valor se toma como el del punto más cercano a la región de impacto y **no como una estimación validada**.
 
Otras limitaciones:
 
- Solo se exploró una solución de impacto (barriendo $A_2$ con los otros seis elementos fijos).
- No se hizo un muestreo aleatorio para comprobar la linealización.
- La validación del modelo con ASSIST se hizo a 6 años, y ese error crece al extrapolar hasta 2880.
## Método
 
1. **Covarianza.** Se descarga de la API de SBDB la solución orbital con su matriz de covarianza $7\times7$ (parámetros $e, q, t_p, \Omega, \omega, i, A_2$) y se comprueba que sea simétrica y definida positiva.
2. **Validación del montaje.** Se integra con REBOUND y con ASSIST (efemérides DE440/DE441) durante 6 años y se compara con Horizons. ASSIST baja el error de posición de 635 km (REBOUND) a unos 2 km.
3. **Efecto Yarkovsky.** Se modela con el parámetro transversal $A_2$ (modelo de Marsden). Su incertidumbre es la que más desplaza la posición en 2880.
4. **Solución de impacto.** El nominal no impacta (pasa a $\approx 1.0\times10^{7}$ km). Se barre $A_2$ de $-3\sigma$ a $+3\sigma$ hasta encontrar un asteroide virtual que impacte.
5. **Plano B.** Para esa solución se calcula la posición $\vec b$ en el plano perpendicular a la velocidad relativa de llegada. Hay impacto si $|\vec b| < R_{\rm ef}$, con
   $R_{\rm ef}=R_\oplus\sqrt{1+v_{\rm esc}^2/v_\infty^2}$.
6. **Linealización.** El Jacobiano $J$ se calcula por diferencias centradas (14 integraciones) y la covarianza se lleva al plano B con $\Gamma = J\,C\,J^T$.
7. **Probabilidad.** $P_i$ es la integral de una gaussiana en el plano B sobre el disco de radio $R_{\rm ef}$, evaluada numéricamente en una malla de $401\times401$ puntos.
8. **Escala de Palermo.** Se calcula con $P_i$, la energía del impacto y el tiempo hasta el encuentro.
## Contenido del notebook
 
- Datos del NEA (elementos orbitales y parámetros físicos de SBDB)
- Matriz de covarianza y comprobaciones (autovalores, Cholesky)
- Escala de Palermo
- Velocidad y energía de impacto
- Simulación con REBOUND y validación frente a Horizons
- Validación con ASSIST
- Efecto Yarkovsky
- Barrido en $A_2$ y solución de impacto
- Plano B, linealización y estimación de $P_i$
- Chequeo de la linealización y sensibilidad a $d_{\rm ref}$
- Comparación de dos métodos para $b$ (Öpik y asíntota de dos cuerpos)
Las funciones nuevas de cada librería se explican con un ejemplo corto en el propio notebook.
 
## Requisitos
 
- Python 3
- Acceso a internet (SBDB, Horizons y la descarga de las efemérides DE441)
- Pensado para correr en **Google Colab**
Instalación (primera celda del notebook):
 
```bash
pip install -Uq pymcel "rebound==4.6.0" assist naif-de440 jpl-small-bodies-de441-n16
```
 
Otras librerías usadas: `numpy`, `matplotlib`, `spiceypy`, `requests`, `astropy`.
 
> `assist` solo funciona con la versión de `rebound` con la que fue compilado (4.6.0). Por eso todas las instalaciones van juntas en la primera celda, antes de importar nada.
 
## Cómo usarlo
 
1. Abre el notebook en Colab (botón de arriba) o súbelo manualmente.
2. Ejecuta las celdas en orden.
3. La descarga de `de441_part-2.bsp` es grande, así que la primera ejecución puede tardar.
**Tiempo de cómputo.** La sección de chequeo de linealización es lenta: cada punto cuesta 15 llamadas a `plano_B`, y cada una integra unos 850 años. Puede tardar varios minutos por punto.
 
## Referencias
 
- Ward, S. N. y Asphaug, E. (2003). Asteroid impact tsunami of 2880 March 16. *Geophysical Journal International*, 153(3), F6-F10. https://doi.org/10.1046/j.1365-246X.2003.01944.x (densidad del asteroide)
- Holman, M. J. et al. (2023). ASSIST: an ephemeris-quality test-particle integrator. *The Planetary Science Journal*, 4, 69.
- Milani, A. et al. (2002). Asteroids III (plano B y detección de impactos).
- Chesley, S. R. et al. (2002). Quantifying the risk posed by potential Earth impacts. *Icarus* (escala de Palermo).
- [JPL Small-Body Database, 29075 (1950 DA)](https://ssd.jpl.nasa.gov/tools/sbdb_lookup.html#/?sstr=29075) (elementos orbitales, covarianza y parámetros físicos)
- [JPL Sentry, 29075 (1950 DA)](https://cneos.jpl.nasa.gov/sentry/details.html#?des=29075) (probabilidad de impacto de referencia)
## Autor
 
_Tu nombre_ · Universidad de Antioquia
 
