import math

# --- 1. BASE DE DATOS MUNDIALISTA COMPLETA (48 SELECCIONES) ---
# Estructura: "Nombre_Equipo": (Goles_A_Favor_Promedio_x10, Goles_En_Contra_Promedio_x10)
BD_MUNDIAL = {
    # Grupo A
    "Mexico": (18, 9), "South Africa": (10, 14), "South Korea": (14, 11), "Czechia": (12, 12),
    # Grupo B
    "Canada": (15, 10), "Bosnia and Herzegovina": (11, 13), "Qatar": (10, 15), "Switzerland": (16, 9),
    # Grupo C
    "Brazil": (22, 7), "Morocco": (15, 9), "Haiti": (8, 18), "Scotland": (11, 14),
    # Grupo D
    "USA": (19, 10), "Paraguay": (11, 12), "Australia": (13, 11), "Turkiye": (14, 13),
    # Grupo E
    "Germany": (24, 8), "Curacao": (9, 19), "Ivory Coast": (16, 10), "Ecuador": (13, 11),
    # Grupo F
    "Netherlands": (21, 9), "Japan": (17, 10), "Sweden": (14, 12), "Tunisia": (10, 14),
    # Grupo G
    "Belgium": (19, 10), "Egypt": (13, 12), "Iran": (14, 11), "New Zealand": (9, 16),
    # Grupo H
    "Spain": (23, 7), "Cabo Verde": (11, 15), "Saudi Arabia": (12, 13), "Uruguay": (18, 10),
    # Grupo I
    "France": (25, 6), "Senegal": (14, 11), "Iraq": (11, 14), "Norway": (15, 12),
    # Grupo J
    "Argentina": (26, 5), "Algeria": (13, 12), "Austria": (14, 11), "Jordan": (10, 15),
    # Grupo K
    "Portugal": (22, 8), "DR Congo": (11, 13), "Uzbekistan": (12, 12), "Colombia": (17, 10),
    # Grupo L
    "England": (21, 8), "Croatia": (15, 11), "Ghana": (13, 13), "Panama": (10, 16)
}

# --- 2. CALCULADORA MATEMÁTICA ---
def calcular_poisson(lambda_goles, k):
    """Calcula la probabilidad de que ocurran exactamente k goles."""
    return (math.exp(-lambda_goles) * (lambda_goles ** k)) / math.factorial(k)

# --- 3. LÓGICA DE FILTROS CONTEXTUALES ORIGINALES ---
def evaluar_contexto(e1, e2, condiciones):
    mod_atk_e1, mod_def_e1, mod_atk_e2, mod_def_e2 = 1.0, 1.0, 1.0, 1.0

    # 1. Filtro Altura
    if condiciones.get('altura_ciudad', 0) >= 2500:
        mod_atk_e1 *= 1.15
        mod_atk_e2 *= 0.85

    # 2. Velocidad de la Selección
    mod_atk_e1 *= (1 + (e1.get('velocidad_promedio', 70) / 1000))
    mod_def_e2 *= (1 + (e2.get('velocidad_promedio', 70) / 1000))

    # 3. Altura física de los jugadores (Juego aéreo)
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
    if e1.get('porcentaje_jugadores_trayectoria', 0.5) > e2.get('porcentaje_jugadores_trayectoria', 0.5):
        mod_atk_e1 *= 1.05
    else:
        mod_atk_e2 *= 1.05

    return mod_atk_e1, mod_def_e1, mod_atk_e2, mod_def_e2

# --- 4. MOTOR DE PREDICCIÓN ---
def predecir_partido(nombre_e1, nombre_e2, ctx_e1, ctx_e2, condiciones):
    gf_e1, gc_e1 = BD_MUNDIAL.get(nombre_e1, (12, 12))
    gf_e2, gc_e2 = BD_MUNDIAL.get(nombre_e2, (12, 12))
    
    e1 = {'nombre': nombre_e1, 'gf_ultimos_10': gf_e1, 'gc_ultimos_10': gc_e1, **ctx_e1}
    e2 = {'nombre': nombre_e2, 'gf_ultimos_10': gf_e2, 'gc_ultimos_10': gc_e2, **ctx_e2}

    prom_f_e1, prom_c_e1 = e1['gf_ultimos_10'] / 10, e1['gc_ultimos_10'] / 10
    prom_f_e2, prom_c_e2 = e2['gf_ultimos_10'] / 10, e2['gc_ultimos_10'] / 10

    m_atk_e1, m_def_e1, m_atk_e2, m_def_e2 = evaluar_contexto(e1, e2, condiciones)

    lambda_e1 = prom_f_e1 * m_atk_e1 * prom_c_e2 * m_def_e2
    lambda_e2 = prom_f_e2 * m_atk_e2 * prom_c_e1 * m_def_e1

    prob_e1, prob_empate, prob_e2 = 0.0, 0.0, 0.0

    for g_e1 in range(7):
        for g_e2 in range(7):
            p_g_e1 = calcular_poisson(lambda_e1, g_e1)
            p_g_e2 = calcular_poisson(lambda_e2, g_e2)
            p_resultado = p_g_e1 * p_g_e2

            if g_e1 > g_e2: prob_e1 += p_resultado
            elif g_e1 == g_e2: prob_empate += p_resultado
            else: prob_e2 += p_resultado

    total = prob_e1 + prob_empate + prob_e2
    if total > 0:
        prob_e1 /= total; prob_empate /= total; prob_e2 /= total

    return prob_e1 * 100, prob_empate * 100, prob_e2 * 100

# --- 5. CONFIGURACIÓN COMPLETA DE LA JORNADA DE HOY ---
# Parejas reales del 25 de junio + variables contextuales específicas
jornada_hoy = [
    ("Curacao", "Ivory Coast", 
     {"velocidad_promedio": 75, "altura_jugadores_promer": 1.78, "lesionados_clave_ataque": 0, "lesionados_clave_defensa": 0, "porcentaje_jugadores_trayectoria": 0.40}, 
     {"velocidad_promedio": 82, "altura_jugadores_promer": 1.83, "lesionados_clave_ataque": 1, "lesionados_clave_defensa": 0, "porcentaje_jugadores_trayectoria": 0.60}),
    
    ("Ecuador", "Germany", 
     {"velocidad_promedio": 84, "altura_jugadores_promer": 1.79, "lesionados_clave_ataque": 0, "lesionados_clave_defensa": 1, "porcentaje_jugadores_trayectoria": 0.35}, 
     {"velocidad_promedio": 80, "altura_jugadores_promer": 1.85, "lesionados_clave_ataque": 0, "lesionados_clave_defensa": 0, "porcentaje_jugadores_trayectoria": 0.70}),
    
    ("Tunisia", "Netherlands", 
     {"velocidad_promedio": 72, "altura_jugadores_promer": 1.81, "lesionados_clave_ataque": 0, "lesionados_clave_defensa": 0, "porcentaje_jugadores_trayectoria": 0.50}, 
     {"velocidad_promedio": 85, "altura_jugadores_promer": 1.84, "lesionados_clave_ataque": 2, "lesionados_clave_defensa": 1, "porcentaje_jugadores_trayectoria": 0.65}),
    
    ("Japan", "Sweden", 
     {"velocidad_promedio": 89, "altura_jugadores_promer": 1.74, "lesionados_clave_ataque": 0, "lesionados_clave_defensa": 0, "porcentaje_jugadores_trayectoria": 0.45}, 
     {"velocidad_promedio": 76, "altura_jugadores_promer": 1.87, "lesionados_clave_ataque": 1, "lesionados_clave_defensa": 0, "porcentaje_jugadores_trayectoria": 0.55}),
    
    ("Turkiye", "USA", 
     {"velocidad_promedio": 79, "altura_jugadores_promer": 1.82, "lesionados_clave_ataque": 1, "lesionados_clave_defensa": 0, "porcentaje_jugadores_trayectoria": 0.40}, 
     {"velocidad_promedio": 86, "altura_jugadores_promer": 1.78, "lesionados_clave_ataque": 0, "lesionados_clave_defensa": 0, "porcentaje_jugadores_trayectoria": 0.30}),
    
    ("Paraguay", "Australia", 
     {"velocidad_promedio": 74, "altura_jugadores_promer": 1.80, "lesionados_clave_ataque": 0, "lesionados_clave_defensa": 0, "porcentaje_jugadores_trayectoria": 0.60}, 
     {"velocidad_promedio": 78, "altura_jugadores_promer": 1.82, "lesionados_clave_ataque": 0, "lesionados_clave_defensa": 1, "porcentaje_jugadores_trayectoria": 0.50})
]

# --- EJECUCIÓN ---
print("🔮 --- PREDICCIONES MUNDIALISTAS AUTOMÁTICAS: SISTEMA MUR ---")
for local, visitante, ctx_l, ctx_v in jornada_hoy:
    l, e, v = predecir_partido(local, visitante, ctx_l, ctx_v, {'altura_ciudad': 0})
    print(f"⚽ {local} vs {visitante} -> Local: {l:.2f}% | Empate: {e:.2f}% | Visitante: {v:.2f}%")







