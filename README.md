# Sistemamur-futbol-riosmatias
Calculadora futbolistica
# ==========================================
# sistema mur-futbol-rios matias
# ==========================================
# Método de Predicción de Probabilidades
import math

def calcular_poisson(lambda_goles, k):
    """Calcula la probabilidad de que ocurran exactamente k goles."""
    return (math.exp(-lambda_goles) * (lambda_goles ** k)) / math.factorial(k)

def evaluar_contexto(e1, e2, condiciones):
    """
    Modifica la fuerza de ataque y defensa según variables contextuales:
    Altura, Lesiones, Velocidad, Altura de jugadores, Trayectoria y Renovación.
    """
    mod_atk_e1, mod_def_e1 = 1.0, 1.0
    mod_atk_e2, mod_def_e2 = 1.0, 1.0

    # 1. Efecto Altura de la Ciudad (Ventaja para el local si está acostumbrado)
    if condiciones['altura_ciudad'] > 2000:
        # El equipo local suele estar adaptado, el visitante sufre el desgaste
        mod_atk_e1 *= 1.15
        mod_atk_e2 *= 0.85

    # 2. Velocidad de la Selección (Transiciones rápidas)
    mod_atk_e1 *= (1 + (e1['velocidad_promedio'] - e2['velocidad_promedio']) * 0.05)
    mod_atk_e2 *= (1 + (e2['velocidad_promedio'] - e1['velocidad_promedio']) * 0.05)

    # 3. Altura física de los jugadores (Balón parado / Juego aéreo)
    if e1['altura_jugadores_prom'] > e2['altura_jugadores_prom']:
        mod_atk_e1 *= 1.05  # Más peligro en tiros de esquina/faltas
        mod_def_e2 *= 1.05  # Mayor dificultad para defender
    else:
        mod_atk_e2 *= 1.05
        mod_def_e1 *= 1.05

    # 4. Jugadores Lesionados (Resta poder ofensivo o defensivo)
    mod_atk_e1 *= (1 - (e1['lesionados_clave_ataque'] * 0.10))
    mod_def_e1 *= (1 + (e1['lesionados_clave_defensa'] * 0.10))
    mod_atk_e2 *= (1 - (e2['lesionados_clave_ataque'] * 0.10))
    mod_def_e2 *= (1 + (e2['lesionados_clave_defensa'] * 0.10))

    # 5. Trayectoria vs Renovación (Experiencia en partidos bajo presión)
    # Equilibrio entre la energía juvenil (renovados) y la jerarquía (trayectoria)
    exp_e1 = e1['porcentaje_jugadores_trayectoria'] - e1['porcentaje_plantel_renovado']
    exp_e2 = e2['porcentaje_jugadores_trayectoria'] - e2['porcentaje_plantel_renovado']
    
    if exp_e1 > exp_e2:
        mod_atk_e1 *= 1.05
    else:
        mod_atk_e2 *= 1.05

    return mod_atk_e1, mod_def_e1, mod_atk_e2, mod_def_e2

def predecir_partido_avanzado(e1, e2, condiciones):
    # Goles promedio en los últimos 10 partidos anteriores al encuentro
    prom_f_e1 = e1['gf_ultimos_10'] / 10
    prom_c_e1 = e1['gc_ultimos_10'] / 10
    prom_f_e2 = e2['gf_ultimos_10'] / 10
    prom_c_e2 = e2['gc_ultimos_10'] / 10
    
    # Obtener los modificadores del contexto
    m_atk_e1, m_def_e1, m_atk_e2, m_def_e2 = evaluar_contexto(e1, e2, condiciones)
    
    # Calcular la expectativa de gol ajustada (Lambda)
    # Tu ataque potenciado contra su defensa debilitada
    lambda_e1 = ((prom_f_e1 * m_atk_e1) + (prom_c_e2 * m_def_e2)) / 2
    lambda_e2 = ((prom_f_e2 * m_atk_e2) + (prom_c_e1 * m_def_e1)) / 2
    
    prob_e1, prob_empate, prob_e2 = 0.0, 0.0, 0.0
    
    # Simulación de matriz de goles (0 a 5 goles)
    for g_e1 in range(6):
        for g_e2 in range(6):
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
    
    print(f"--- PREDICCIÓN DE PARTIDO ---")
    print(f"Probabilidad {e1['nombre']} (Local): {(prob_e1/total)*100:.2f}%")
    print(f"Probabilidad Empate: {(prob_empate/total)*100:.2f}%")
    print(f"Probabilidad {e2['nombre']} (Visitante): {(prob_e2/total)*100:.2f}%")

# --- CONFIGURACIÓN DE LOS DATOS DE ENTRADA ---

# Datos previos al partido (Últimos 10 juegos)
seleccion_1 = {
    'nombre': 'Bolivia',
    'gf_ultimos_10': 14,
    'gc_ultimos_10': 12,
    'velocidad_promedio': 78,           # Escala 1-100 (tipo FIFA/EA Sports)
    'altura_jugadores_prom': 1.76,      # En metros
    'lesionados_clave_ataque': 1,       # Cantidad de delanteros titulares de baja
    'lesionados_clave_defensa': 0,
    'porcentaje_plantel_renovado': 0.40, # 40% jóvenes nuevos
    'porcentaje_jugadores_trayectoria': 0.20 # 20% veteranos
}

seleccion_2 = {
    'nombre': 'Brasil',
    'gf_ultimos_10': 25,
    'gc_ultimos_10': 6,
    'velocidad_promedio': 88,
    'altura_jugadores_prom': 1.82,
    'lesionados_clave_ataque': 0,
    'lesionados_clave_defensa': 1,
    'porcentaje_plantel_renovado': 0.20,
    'porcentaje_jugadores_trayectoria': 0.50
}

# Entorno geográfico del partido
condiciones_encuentro = {
    'altura_ciudad': 3600,  # La Paz (en metros)
}

# Ejecutar predicción
predecir_partido_avanzado(seleccion_1, seleccion_2, condiciones_encuentro)

