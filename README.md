# Sistemamur-futbol-riosmatias
Calculadora futbolistica
# Sistemamur-futbol-riosmatias

Calculadora futbolística para la predicción de partidos utilizando modelos estadísticos y variables contextuales.

## 🐍 Código del Predictor

```python
import math

# --- CALCULADORA FUTBOLÍSTICA ---

def calcular_poisson(lambda_goles, k):
    """Calcula la probabilidad de que ocurran exactamente k goles."""
    return (math.exp(-lambda_goles) * (lambda_goles ** k)) / math.factorial(k)


def evaluar_contexto(e1, e2, condiciones):
    mod_atk_e1 = 1.0
    mod_def_e1 = 1.0
    mod_atk_e2 = 1.0
    mod_def_e2 = 1.0

    # 1. Efecto Altura de la Ciudad
    if condiciones.get('altura_ciudad', 0) >= 2500:
        mod_atk_e1 *= 1.15
        mod_atk_e2 *= 0.85

    # 2. Velocidad de la Selección
    mod_atk_e1 *= (1 + (e1.get('velocidad_promedio', 70) / 1000))
    mod_def_e2 *= (1 + (e2.get('velocidad_promedio', 70) / 1000))

    # 3. Altura física de los jugadores
    if e1.get('altura_jugadores_promer', 1.75) > e2.get('altura_jugadores_promer', 1.75):
        mod_atk_e1 *= 1.05
        mod_def_e2 *= 1.05
    else:
        mod_atk_e2 *= 1.05
        mod_def_e1 *= 1.05

    # 4. Jugadores Lesionados
    mod_atk_e1 *= (1 - (e1.get('lesionados_clave_ataque', 0) * 0.1))
    mod_def_e1 *= (1 - (e1.get('lesionados_clave_defensa', 0) * 0.1))
    mod_atk_e2 *= (1 - (e2.get('lesionados_clave_ataque', 0) * 0.1))
    mod_def_e2 *= (1 - (e2.get('lesionados_clave_defensa', 0) * 0.1))

    # 5. Trayectoria vs Renovación
    exp_e1 = e1.get('porcentaje_jugadores_trayectoria', 0.5)
    exp_e2 = e2.get('porcentaje_jugadores_trayectoria', 0.5)

    if exp_e1 > exp_e2:
        mod_atk_e1 *= 1.05
    else:
        mod_atk_e2 *= 1.05

    return mod_atk_e1, mod_def_e1, mod_atk_e2, mod_def_e2


def predecir_partido_avanzado(e1, e2, condiciones):
    prom_f_e1 = e1.get('gf_ultimos_10', 10) / 10
    prom_c_e1 = e1.get('gc_ultimos_10', 10) / 10
    prom_f_e2 = e2.get('gf_ultimos_10', 10) / 10
    prom_c_e2 = e2.get('gc_ultimos_10', 10) / 10

    m_atk_e1, m_def_e1, m_atk_e2, m_def_e2 = evaluar_contexto(e1, e2, condiciones)

    lambda_e1 = prom_f_e1 * m_atk_e1 * prom_c_e2 * m_def_e2
    lambda_e2 = prom_f_e2 * m_atk_e2 * prom_c_e1 * m_def_e1

    prob_e1 = 0.0
    prob_empate = 0.0
    prob_e2 = 0.0

    for g_e1 in range(7):
        for g_e2 in range(7):
            p_g_e1 = calcular_poisson(lambda_e1, g_e1)
            p_g_e2 = calcular_poisson(lambda_e2, g_e2)
            p_resultado = p_g_e1 * p_g_e2

            if g_e1 > g_e2:
                prob_e1 += p_resultado
            elif g_e1 == g_e2:
                prob_empate += p_resultado
            else:
                prob_e2 += p_resultado

    total = prob_e1 + prob_empate + prob_e2
    if total > 0:
        prob_e1 /= total
        prob_empate /= total
        prob_e2 /= total

    print(f"--- PREDICCIÓN DE PARTIDO AVANZADO ---")
    print(f"Probabilidad de que gane {e1['nombre']}: {prob_e1 * 100:.2f}%")
    print(f"Probabilidad de Empate: {prob_empate * 100:.2f}%")
    print(f"Probabilidad de que gane {e2['nombre']}: {prob_e2 * 100:.2f}%")


# --- CONFIGURACIÓN DE LOS DATOS DE ENTRADA ---

seleccion_1 = {
    'nombre': 'Bolivia',
    'gf_ultimos_10': 14,
    'gc_ultimos_10': 12,
    'velocidad_promedio': 78,
    'altura_jugadores_promer': 1.76,
    'lesionados_clave_ataque': 1,
    'lesionados_clave_defensa': 0,
    'porcentaje_plantel_renovado': 0.40,
    'porcentaje_jugadores_trayectoria': 0.20
}

seleccion_2 = {
    'nombre': 'Brasil',
    'gf_ultimos_10': 25,
    'gc_ultimos_10': 6,
    'velocidad_promedio': 88,
    'altura_jugadores_promer': 1.82,
    'lesionados_clave_ataque': 0,
    'lesionados_clave_defensa': 1,
    'porcentaje_plantel_renovado': 0.20,
    'porcentaje_jugadores_trayectoria': 0.50
}

condiciones_encuentro = {
    'altura_ciudad': 3600
}

# --- EJECUTAR PREDICCIÓN ---
predecir_partido_avanzado(seleccion_1, seleccion_2, condiciones_encuentro)
```



