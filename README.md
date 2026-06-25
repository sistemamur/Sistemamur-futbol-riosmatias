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

# =====================================================================
# 🚀 EXTENSIÓN SISTEMA MUR PRO: MODIFICADORES AGRESIVOS Y ESTADÍSTICAS FINAS
# =====================================================================

def evaluar_contexto_pro(e1, e2, condiciones):
    # Multiplicadores base hiperinflados para forzar tendencias > 70%
    mod_atk_e1, mod_def_e1, mod_atk_e2, mod_def_e2 = 1.0, 1.0, 1.0, 1.0

    # 1. Filtro de Creencias Religiosas / Misticismo Cultural (Afecta moral y fe en la épica)
    if e1.get('creencia_religiosa_fuerte'): mod_atk_e1 *= 1.30  # +30% empuje ofensivo
    if e2.get('creencia_religiosa_fuerte'): mod_atk_e2 *= 1.30

    # 2. Efecto de Optimismo / Eliminación (Urgencia vs Desmotivación total)
    if e1.get('ya_eliminado'): m_atk_e1, m_def_e1 = mod_atk_e1 * 0.40, mod_def_e1 * 0.40  # Se cae el equipo
    if e2.get('ya_eliminado'): m_atk_e2, m_def_e2 = mod_atk_e2 * 0.40, mod_def_e2 * 0.40
    if e1.get('urgencia_clasificar') and not e1.get('ya_eliminado'): mod_atk_e1 *= 1.45  # Ultra ofensivo
    if e2.get('urgencia_clasificar') and not e2.get('ya_eliminado'): mod_atk_e2 *= 1.45

    # 3. Desgaste por Cansancio (Minutos acumulados en las piernas)
    mod_def_e1 *= (1 - (e1.get('minutos_acumulados_promedio', 180) / 2000))
    mod_def_e2 *= (1 - (e2.get('minutos_acumulados_promedio', 180) / 2000))

    # 4. Disciplina Extrema (Efecto acumulativo de Tarjetas Amarillas)
    mod_def_e1 *= (1 - (e1.get('tarjetas_amarillas_torneo', 0) * 0.08))
    mod_def_e2 *= (1 - (e2.get('tarjetas_amarillas_torneo', 0) * 0.08))

    return mod_atk_e1, mod_def_e1, mod_atk_e2, mod_def_e2

# ... [Funciones de predicción y ejemplos de jornada avanzados] ...




# =====================================================================
# 🚀 EXTENSIÓN SISTEMA MUR PRO: INTENSIDAD EXTREMA Y MARCADORES EN VIVO
# =====================================================================

def evaluar_contexto_avanzado(e1, e2):
    # Multiplicadores drásticos para forzar probabilidades > 70%
    mA1, mD1, mA2, mD2 = 1.0, 1.0, 1.0, 1.0

    # 1. Creencias Religiosas y Misticismo Cultural (+35% de convicción)
    if e1.get('fe_misticismo'): mA1 *= 1.35
    if e2.get('fe_misticismo'): mA2 *= 1.35

    # 2. Efecto de Optimismo / Eliminación Total
    if e1.get('ya_eliminado'): mA1 *= 0.30; mD1 *= 0.50
    if e2.get('ya_eliminado'): mA2 *= 0.30; mD2 *= 0.50
    if e1.get('urgencia_clasificar') and not e1.get('ya_eliminado'): mA1 *= 1.50
    if e2.get('urgencia_clasificar') and not e2.get('ya_eliminado'): mA2 *= 1.50

    # 3. Desgaste por Cansancio Extremo (Minutos en fase de grupos)
    mA1 *= (1 - (e1.get('minutos_jugados', 180) * 0.001))
    mD1 *= (1 - (e1.get('minutos_jugados', 180) * 0.0015))
    mA2 *= (1 - (e2.get('minutos_jugados', 180) * 0.001))
    mD2 *= (1 - (e2.get('minutos_jugados', 180) * 0.0015))

    # 4. Disciplina: Acumulación de Tarjetas Amarillas
    mD1 *= (1 - (e1.get('amarillas', 0) * 0.07))
    mD2 *= (1 - (e2.get('amarillas', 0) * 0.07))

    return mA1, mD1, mA2, mD2

def calcular_marcador_exacto(l1, l2):
    # Traduce los índices lambda corregidos a goles enteros plausibles
    goles_l = math.floor(l1) if l1 >= 1 else (1 if l1 > 0.4 else 0)
    goles_v = math.floor(l2) if l2 >= 1 else (1 if l2 > 0.4 else 0)
    if abs(l1 - l2) < 0.15: # Forzar empate técnico en números si el λ es igual
        return goles_l, goles_l
    return goles_l, goles_v

def procesar_jornada_pro(partidos):
    print("\n🔮 --- SIMULACIÓN AVANZADA SISTEMA MUR PRO (MUNDIAL 2026) ---")
    for local, visitante, ctx_l, ctx_v in partidos:
        # Extraer estadísticas base de tu BD_MUNDIAL original
        gf_l, gc_l = BD_MUNDIAL.get(local, (12, 12))
        gf_v, gc_v = BD_MUNDIAL.get(visitante, (12, 12))
        
        # Calcular lambdas base según tu lógica original
        l_base_l = (gf_l / 10) * (gc_v / 10)
        l_base_v = (gf_v / 10) * (gc_l / 10)
        
        # Modificadores Pro extremos
        mA1, mD1, mA2, mD2 = evaluar_contexto_avanzado(ctx_l, ctx_v)
        lambda_final_l = l_base_l * mA1 * mD2
        lambda_final_v = l_base_v * mA2 * mD1
        
        # Re-calcular la matriz de Poisson de tu código para las probabilidades
        p_l, p_e, p_v = 0.0, 0.0, 0.0
        for g_l in range(7):
            for g_v in range(7):
                p_g_l = calcular_poisson(lambda_final_l, g_l)
                p_g_v = calcular_poisson(lambda_final_v, g_v)
                res = p_g_l * p_g_v
                if g_l > g_v: p_l += res
                elif g_l == g_v: p_e += res
                else: p_v += res
        
        tot = p_l + p_e + p_v
        if tot > 0: p_l /= tot; p_e /= tot; p_v /= tot
        
        # Obtener los goles finales calculados en números
        g_l, g_v = calcular_marcador_exacto(lambda_final_l, lambda_final_v)
        
        print(f"⚽ {local} vs {visitante}")
        print(f"   📊 Probabilidades -> Local: {p_l*100:.2f}% | Empate: {p_e*100:.2f}% | Visitante: {p_v*100:.2f}%")
        print(f"   🏆 Marcador numérico calculado: {local} {g_l} - {g_v} {visitante}\n")

# --- NUEVA LISTA DE PARTIDOS CON TU CONTEXTO SOLICITADO ---
jornada_pro_hoy = [
    ("Ecuador", "Germany", 
     {"fe_misticismo": True, "ya_eliminado": False, "urgencia_clasificar": True, "minutos_jugados": 180, "amarillas": 4},
     {"fe_misticismo": False, "ya_eliminado": False, "urgencia_clasificar": False, "minutos_jugados": 120, "amarillas": 1}),
     
    ("Paraguay", "Australia", 
     {"fe_misticismo": True, "ya_eliminado": False, "urgencia_clasificar": True, "minutos_jugados": 185, "amarillas": 5},
     {"fe_misticismo": False, "ya_eliminado": False, "urgencia_clasificar": True, "minutos_jugados": 170, "amarillas": 2}),
     
    ("Turkiye", "USA", 
     {"fe_misticismo": False, "ya_eliminado": True, "urgencia_clasificar": False, "minutos_jugados": 190, "amarillas": 6},
     {"fe_misticismo": True, "ya_eliminado": False, "urgencia_clasificar": True, "minutos_jugados": 110, "amarillas": 0})
]

# Ejecutar el nuevo motor Pro
procesar_jornada_pro(jornada_pro_hoy)

import random

def simular_partido_en_vivo(local, visitante, lambda_l, lambda_v):
    print(f"🏁 ¡Arranca el partido en vivo: {local} vs {visitante}!")
    goles_l, goles_v = 0, 0
    
    # Simulación minuto a minuto (simplificada a 9 bloques de 10 minutos)
    for bloque in range(1, 10):
        minuto = bloque * 10
        # Probabilidad de gol en este bloque (Poisson fraccionado)
        if random.random() < (lambda_l / 9):
            goles_l += 1
            print(f"⚽ ¡GOOOL de {local}! Minuto {minuto}. Marcador: {local} {goles_l} - {goles_v} {visitante}")
        if random.random() < (lambda_v / 9):
            goles_v += 1
            print(f"⚽ ¡GOOOL de {visitante}! Minuto {minuto}. Marcador: {local} {goles_l} - {goles_v} {visitante}")
            
        # EVENTO EXCLUSIVO: Alerta de cansancio crítico en vivo
        if bloque == 7: # Minuto 70
            print(f"⏳ Minuto 70: El cansancio acumulado empieza a pasar factura en las defensas...")
            lambda_l *= 1.2  # El partido se rompe y se vuelve más caótico (más goles)
            lambda_v *= 1.2

    print(f"🏁 Fin del partido. Resultado definitivo: {local} {goles_l} - {goles_v} {visitante}\n")


import math

# --- 1. BASE DE DATOS MUNDIALISTA ---
BD_MUNDIAL_ES = {
    "México": (1.6, 1.2), "Sudáfrica": (1.1, 1.3), "Brasil": (2.1, 0.9), "Alemania": (2.0, 1.0),
    "Costa de Marfil": (1.4, 1.1), "Ecuador": (1.3, 1.2), "Paraguay": (1.3, 1.2), "Australia": (1.3, 1.2)
}

# --- 2. NUEVA LÓGICA DE EQUILIBRIO ---
def evaluar_contexto_equilibrado(e1, e2):
    mA1, mD1, mA2, mD2 = 1.0, 1.0, 1.0, 1.0
    # Filtro 1: Calidad de Plantel y Recambios
    recambio_e1 = e1.get('calidad_recambios', 5)
    bajas_e1 = e1.get('lesionados', 0)
    mA1 *= (1 - max(0, (bajas_e1 * 0.08) - (recambio_e1 * 0.01)))
    # Filtro 2: Idiosincrasia y Resiliencia
    if e1.get('idiosincrasia_resiliente', False): mD1 *= 1.05
    # Filtro 3: Mitigador de Fatiga
    mA1 *= (1 - (e1.get('minutos_jugados', 180) * 0.0003))
    return mA1, mD1, mA2, mD2

# --- 3. MOTOR DE PROCESAMIENTO ---
def procesar_jornada_equilibrada(partidos):
    print("🔮 --- PREDICCIONES EQUILIBRADAS ---")
    for local, visitante, ctx_l, ctx_v in partidos:
        prom_f_l, prom_c_l = BD_MUNDIAL_ES.get(local, (1.2, 1.2))
        prom_f_v, prom_c_v = BD_MUNDIAL_ES.get(visitante, (1.2, 1.2))
        mA1, mD1, mA2, mD2 = evaluar_contexto_equilibrado(ctx_l, ctx_v)
        lambda_l = prom_f_l * mA1 * prom_c_v * mD2
        lambda_v = prom_f_v * mA2 * prom_c_l * mD1
        print(f"⚽ {local} vs {visitante} -> Marcador sugerido: {math.floor(lambda_l)} - {math.floor(lambda_v)}")

# --- 4. CONFIGURACIÓN ---
jornada_hoy = [
    ("Ecuador", "Alemania", 
     {"calidad_recambios": 6, "lesionados": 1, "idiosincrasia_resiliente": True, "minutos_jugados": 180}, 
     {"calidad_recambios": 9, "lesionados": 0, "idiosincrasia_resiliente": True, "minutos_jugados": 180})
]
procesar_jornada_equilibrada(jornada_hoy)



