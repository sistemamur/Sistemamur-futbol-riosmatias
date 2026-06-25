# Sistemamur Fútbol RM- Predicción de Partidos con Inteligencia y Estadística ⚽📊

Este proyecto es un **software predictor de fútbol** desarrollado en **Python** que utiliza la **Distribución de Poisson** combinada con extracción automática de datos de internet para calcular las probabilidades matemáticas de victorias, empates y derrotas.

## 🐍 Código del Predictor Unificado (Automático + Contextual)

```python
import math
import requests

# --- 1. EXTRACCIÓN AUTOMÁTICA DE GOLES (API DEPORTIVA) ---

def obtener_estadisticas_api(nombre_equipo):
    """Busca en internet el equipo y calcula los goles de sus últimos partidos."""
    print(f"Buscando estadísticas reales de '{nombre_equipo}' en internet...")
    
    url_buscar = f"https://thesportsdb.com{nombre_equipo}"
    try:
        respuesta = requests.get(url_buscar).json()
        if not respuesta or 'teams' not in respuesta or not respuesta['teams']:
            print(f"⚠️ No se encontró el equipo '{nombre_equipo}'. Usando datos base (1.2 goles).")
            return 12, 12
        
        id_equipo = respuesta['teams'][0]['idTeam']
        url_resultados = f"https://thesportsdb.com{id_equipo}"
        datos_partidos = requests.get(url_resultados).json()
        
        goles_favor = 0
        goles_contra = 0
        partidos_contados = 0
        
        if datos_partidos and 'results' in datos_partidos and datos_partidos['results']:
            for partido in datos_partidos['results']:
                if partido['idHomeTeam'] == id_equipo:
                    goles_favor += int(partido['intHomeScore'] or 0)
                    goles_contra += int(partido['intAwayScore'] or 0)
                else:
                    goles_favor += int(partido['intAwayScore'] or 0)
                    goles_contra += int(partido['intHomeScore'] or 0)
                partidos_contados += 1
        
        if partidos_contados > 0:
            gf_final = int((goles_favor / partidos_contados) * 10)
            gc_final = int((goles_contra / partidos_contados) * 10)
            print(f"📊 Datos en vivo: {gf_final/10} goles marcados y {gc_final/10} recibidos en promedio.")
            return gf_final, gc_final
            
    except Exception:
        print("❌ Error de conexión al buscar datos. Usando valores estándar.")
        
    return 12, 12


# --- 2. CALCULADORA MATEMÁTICA Y CONTEXTO (SISTEMA MUR) ---

def calcular_poisson(lambda_goles, k):
    """Calcula la probabilidad de que ocurran exactamente k goles."""
    return (math.exp(-lambda_goles) * (lambda_goles ** k)) / math.factorial(k)


def evaluar_contexto(e1, e2, condiciones):
    mod_atk_e1, mod_def_e1, mod_atk_e2, mod_def_e2 = 1.0, 1.0, 1.0, 1.0

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


def predecir_partido_completo(nombre_e1, nombre_e2, datos_manuales_e1, datos_manuales_e2, condiciones):
    # El sistema busca los goles automáticamente en internet
    gf_e1, gc_e1 = obtener_estadisticas_api(nombre_e1)
    gf_e2, gc_e2 = obtener_estadisticas_api(nombre_e2)
    
    # Se arman los perfiles combinando los goles de internet con tus variables manuales
    e1 = {'nombre': nombre_e1, 'gf_ultimos_10': gf_e1, 'gc_ultimos_10': gc_e1, **datos_manuales_e1}
    e2 = {'nombre': nombre_e2, 'gf_ultimos_10': gf_e2, 'gc_ultimos_10': gc_e2, **datos_manuales_e2}

    prom_f_e1 = e1['gf_ultimos_10'] / 10
    prom_c_e1 = e1['gc_ultimos_10'] / 10
    prom_f_e2 = e2['gf_ultimos_10'] / 10
    prom_c_e2 = e2['gc_ultimos_10'] / 10

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

    print(f"\n🔮 --- RESULTADO DEL SISTEMA MUR UNIFICADO ---")
    print(f"Probabilidad de que gane {e1['nombre']}: {prob_e1 * 100:.2f}%")
    print(f"Probabilidad de Empate: {prob_empate * 100:.2f}%")
    print(f"Probabilidad de que gane {e2['nombre']}: {prob_e2 * 100:.2f}%")


# --- 3. CONFIGURACIÓN DEL ENCUENTRO ---

# 1. Escribe los nombres para buscar en internet (en inglés preferentemente para la API)
EQUIPO_LOCAL = "Barcelona"
EQUIPO_VISITANTE = "Real Madrid"

# 2. Configura los detalles específicos de tu modelo original
contexto_local = {
    'velocidad_promedio': 78,
    'altura_jugadores_promer': 1.76,
    'lesionados_clave_ataque': 1,
    'lesionados_clave_defensa': 0,
    'porcentaje_jugadores_trayectoria': 0.20
}

contexto_visitante = {
    'velocidad_promedio': 88,
    'altura_jugadores_promer': 1.82,
    'lesionados_clave_ataque': 0,
    'lesionados_clave_defensa': 1,
    'porcentaje_jugadores_trayectoria': 0.50
}

condiciones_partido = {
    'altura_ciudad': 0  # Cambiar si se juega en altura (ej: 3600)
}

# --- EJECUTAR ---
predecir_partido_completo(EQUIPO_LOCAL, EQUIPO_VISITANTE, contexto_local, contexto_visitante, condiciones_partido)
```






