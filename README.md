
# ============================================================================
# INTRODUCCIÓN: SISTEMAMURIOSMATIAS
# Sistema Avanzado de Predicción de Partidos del Mundial 2026
# ============================================================================

import pandas as pd
import numpy as np
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import LabelEncoder
import lightgbm as lgb
import warnings
warnings.filterwarnings('ignore')

print("=" * 60)
print("--- SISTEMAMURIOSMATIAS ACTIVADO ---")
print("Procesando variables físicas, históricas, geográficas y culturales...")
print("=" * 60)

# 1. GENERACIÓN DE LA BASE DE DATOS DE ENTRENAMIENTO (Simulación del Mundial)
def generar_dataset_ejemplo():
    data = {
        # Datos del Encuentro
        'partido_id': range(1, 11),
        'local': ['Argentina', 'Francia', 'Brasil', 'España', 'Alemania', 'Marruecos', 'Japón', 'Portugal', 'Uruguay', 'Inglaterra'],
        'visitante': ['Arabia Saudita', 'Australia', 'Corea del Sur', 'Costa Rica', 'Japón', 'Croacia', 'España', 'Ghana', 'Corea del Sur', 'EEUU'],
        'ciudad_sede': ['Dallas', 'Los Angeles', 'Miami', 'New York', 'Mexico City', 'Toronto', 'Vancouver', 'Atlanta', 'Houston', 'Boston'],
        
        # Historial Reciente (Últimos 15 partidos)
        'puntos_ultimos_15_local':,
        'puntos_ultimos_15_vis':,
        'goles_favor_15_local':,
        'goles_contra_15_local':,
        'goles_favor_15_vis':,
        'goles_contra_15_vis':,
        
        # Jerarquía e Historia (Rankings, títulos, experiencia en escala 1-10)
        'jerarquia_plantel_local': [9.5, 9.3, 9.4, 8.8, 8.5, 7.8, 7.5, 8.9, 8.0, 9.0],
        'jerarquia_plantel_vis': [5.0, 6.2, 6.8, 5.5, 7.5, 8.2, 8.8, 6.0, 6.8, 7.9],
        'titulos_mundiales_local':,
        'titulos_mundiales_vis':,
        
        # Estado de Eliminación Actual (0: A salvo, 1: Riesgo crítico, 2: Eliminado)
        'riesgo_eliminacion_local':,
        'riesgo_eliminacion_vis':,
        
        # Datos Físicos (Alturas en cm y Velocidades Absolutas Promedio en km/h)
        'altura_media_local': [179.5, 183.2, 180.1, 181.5, 185.0, 182.1, 177.8, 186.2, 181.2, 184.5],
        'altura_media_vis': [176.2, 184.0, 176.5, 179.0, 177.8, 183.5, 181.5, 182.0, 176.5, 182.1],
        'vel_absoluta_max_local': [35.2, 36.5, 34.9, 33.8, 33.2, 34.1, 34.5, 35.0, 33.9, 35.5], 
        'vel_absoluta_max_vis': [31.5, 32.2, 33.0, 31.8, 34.5, 33.5, 33.8, 33.1, 33.0, 33.9],
        
        # Factores Culturales y Sede
        'religion_dominante_local': ['Católica', 'Católica', 'Católica', 'Católica', 'Protestante', 'Islam', 'Sintoísmo', 'Católica', 'Católica', 'Protestante'],
        'religion_dominante_vis': ['Islam', 'Protestante', 'Cristiana', 'Católica', 'Sintoísmo', 'Católica', 'Católica', 'Cristiana', 'Cristiana', 'Protestante'],
        
        # Variable Objetivo real: 1 = Gana Local, 0 = Empate, 2 = Gana Visitante
        'resultado': [1, 1, 1, 1, 0, 2, 0, 1, 1, 1]
    }
    return pd.DataFrame(data)

df = generar_dataset_ejemplo()

# 2. INGENIERÍA DE CARACTERÍSTICAS (Métricas Cruzadas)
df['dif_puntos_15'] = df['puntos_ultimos_15_local'] - df['puntos_ultimos_15_vis']
df['dif_goles_favor'] = df['goles_favor_15_local'] - df['goles_favor_15_vis']
df['dif_goles_contra'] = df['goles_contra_15_local'] - df['goles_contra_15_vis']
df['dif_jerarquia'] = df['jerarquia_plantel_local'] - df['jerarquia_plantel_vis']
df['dif_altura'] = df['altura_media_local'] - df['altura_media_vis']
df['dif_velocidad_final'] = df['vel_absoluta_max_local'] - df['vel_absoluta_max_vis']

# Codificación de variables de texto (Ciudad, Religiones)
le_ciudad = LabelEncoder()
le_rel_loc = LabelEncoder()
le_rel_vis = LabelEncoder()

df['ciudad_encoded'] = le_ciudad.fit_transform(df['ciudad_sede'])
df['rel_local_encoded'] = le_rel_loc.fit_transform(df['religion_dominante_local'])
df['rel_vis_encoded'] = le_rel_vis.fit_transform(df['religion_dominante_vis'])

# Selección de características para el modelo
features = [
    'dif_puntos_15', 'dif_goles_favor', 'dif_goles_contra', 'dif_jerarquia', 
    'dif_altura', 'dif_velocidad_final', 'riesgo_eliminacion_local', 'riesgo_eliminacion_vis',
    'ciudad_encoded', 'rel_local_encoded', 'rel_vis_encoded', 'titulos_mundiales_local', 'titulos_mundiales_vis'
]

X = df[features]
y = df['resultado']

# 3. ENTRENAMIENTO DEL MODELO INTELIGENTE
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

model = lgb.LGBMClassifier(objective='multiclass', num_class=3, random_state=42, verbose=-1)
model.fit(X_train, y_train)

print("¡Modelo entrenado y listo con todas las variables cargadas!")

# 4. FUNCIÓN PARA EVALUAR CUALQUIER PARTIDO NUEVO
def predecir_partido_sistemamuriosmatias(datos_partido):
    df_partido = pd.DataFrame([datos_partido])
    
    # Calcular diferenciales del nuevo cruce
    df_partido['dif_puntos_15'] = df_partido['puntos_ultimos_15_local'] - df_partido['puntos_ultimos_15_vis']
    df_partido['dif_goles_favor'] = df_partido['goles_favor_15_local'] - df_partido['goles_favor_15_vis']
    df_partido['dif_goles_contra'] = df_partido['goles_contra_15_local'] - df_partido['goles_contra_15_vis']
    df_partido['dif_jerarquia'] = df_partido['jerarquia_plantel_local'] - df_partido['jerarquia_plantel_vis']
    df_partido['dif_altura'] = df_partido['altura_media_local'] - df_partido['altura_media_vis']
    df_partido['dif_velocidad_final'] = df_partido['vel_absoluta_max_local'] - df_partido['vel_absoluta_max_vis']
    
    # Mapeo de texto a números de forma segura
    df_partido['ciudad_encoded'] = le_ciudad.transform([datos_partido['ciudad_sede']])[0]
    df_partido['rel_local_encoded'] = le_rel_loc.transform([datos_partido['religion_dominante_local']])[0]
    df_partido['rel_vis_encoded'] = le_rel_vis.transform([datos_partido['religion_dominante_vis']])[0]
    
    # Extraer probabilidades
    X_pred = df_partido[features]
    probabilidades = model.predict_proba(X_pred)[0]
    
    # Mostrar resultados en pantalla
    print("\n" + "="*50)
    print(f"ANÁLISIS DE PARTIDO: {datos_partido['local']} vs {datos_partido['visitante']}")
    print(f"Sede: {datos_partido['ciudad_sede']} | Choque Cultural: {datos_partido['religion_dominante_local']} vs {datos_partido['religion_dominante_vis']}")
    print("-" * 50)
    print(f"Probabilidad Victoria {datos_partido['local']}: {probabilidades[1]*100:.2f}%")
    print(f"Probabilidad de Empate: {probabilidades[0]*100:.2f}%")
    print(f"Probabilidad Victoria {datos_partido['visitante']}: {probabilidades[2]*100:.2f}%")
    print("="*50)

# ============================================================================
# 5. SIMULADOR EN VIVO (Puedes modificar los números de aquí abajo)
# ============================================================================
partido_a_testear = {
    'local': 'Argentina', 
    'visitante': 'Arabia Saudita', 
    'ciudad_sede': 'Dallas',
    'puntos_ultimos_15_local': 38, 
    'puntos_ultimos_15_vis': 18,
    'goles_favor_15_local': 35, 
    'goles_contra_15_local': 8,
    'goles_favor_15_vis': 12, 
    'goles_contra_15_vis': 25,
    'jerarquia_plantel_local': 9.5, 
    'jerarquia_plantel_vis': 5.0,
    'titulos_mundiales_local': 3, 
    'titulos_mundiales_vis': 0,
    'riesgo_eliminacion_local': 0,  # 0 = Estable
    'riesgo_eliminacion_vis': 1,    # 1 = Cerca de quedar eliminado
    'altura_media_local': 179.5, 
    'altura_media_vis': 176.2,
    'vel_absoluta_max_local': 35.2, # Velocidad punta promedio
    'vel_absoluta_max_vis': 31.5,
    'religion_dominante_local': 'Católica', 
    'religion_dominante_vis': 'Islam'
}

# Ejecutar el sistema predictivo

import random
import time

def rastrear_predicciones_esotericas_web(equipo1, equipo2):
    """
    MODULO 1: SImula el scraping web e internet de astrólogos,
    tarotistas y videntes globales. Convierte palabras en métricas.
    """
    print("🌐 [SCRAPING] Iniciando rastreo de blogs esotéricos y redes sociales...")
    time.sleep(1) # Simula el tiempo de carga de red
    
    # Simulación de recuento de predicciones encontradas en la nube
    votos_e1 = random.randint(10, 50)
    votos_e2 = random.randint(10, 50)
    votos_empate = random.randint(5, 20)
    
    total_votos = votos_e1 + votos_e2 + votos_empate
    
    # Normalización matemática a índices entre 0.0 y 1.0
    indice_mistico_e1 = round(votos_e1 / total_votos, 2)
    indice_mistico_e2 = round(votos_e2 / total_votos, 2)
    
    print(f"✅ [SCRAPING] Rastreo completado con éxito.")
    print(f"🔮 Tendencia Mística Detectada -> {equipo1}: {indice_mistico_e1} | {equipo2}: {indice_mistico_e2}\n")
    
    return indice_mistico_e1, indice_mistico_e2


def simular_sistema_murios_esoterico_completo(equipo1, equipo2, datos_deportivos, datos_misticos_base):
    """
    MODULO 2: Algoritmo de calibración híbrido.
    Fusión: 70% Parámetros Científicos / 30% Parámetros Esotéricos.
    """
    print("=" * 60)
    print("--- SISTEMAMURIOSMATIAS EN EJECUCIÓN (VERSIÓN HÍBRIDA) ---")
    print("=" * 60)
    
    # 1. Ejecutar el rastreador en internet para actualizar las variables astrales
    ind_web_e1, ind_web_e2 = rastrear_predicciones_esotericas_web(equipo1, equipo2)
    
    # 2. Procesamiento de Variables Lógicas (Peso: 70% del total)
    # Goles (30%), Jerarquía (25%), Gestión de Presión (15%), Físico (15%), Sede (15%)
    score_logico_e1 = (datos_deportivos['goles_e1'] * 0.30) + \
                      (datos_deportivos['titulos_e1'] * 0.25) + \
                      ((100 - datos_deportivos['presion_e1']) * 0.15) + \
                      (datos_deportivos['fisico_e1'] * 0.15) + \
                      (datos_deportivos['sede_e1'] * 0.15)

    score_logico_e2 = (datos_deportivos['goles_e2'] * 0.30) + \
                      (datos_deportivos['titulos_e2'] * 0.25) + \
                      ((100 - datos_deportivos['presion_e2']) * 0.15) + \
                      (datos_deportivos['fisico_e2'] * 0.15) + \
                      (datos_deportivos['sede_e2'] * 0.15)

    # 3. Procesamiento de Variables Esotéricas (Peso: 30% del total)
    # Numerología (20%), Rastreo Web / Tránsito Astral (40%), Vibración Color (40%)
    score_mistico_e1 = (datos_misticos_base['num_fundacion_e1'] * 0.20) + \
                       (ind_web_e1 * 0.40) + \
                       (datos_misticos_base['vibracion_color_e1'] * 0.40)

    score_mistico_e2 = (datos_misticos_base['num_fundacion_e2'] * 0.20) + \
                       (ind_web_e2 * 0.40) + \
                       (datos_misticos_base['vibracion_color_e2'] * 0.40)

    # 4. Fusión y Calibración Final del Algoritmo
    final_e1 = (score_logico_e1 * 0.70) + (score_mistico_e1 * 0.30)
    final_e2 = (score_logico_e2 * 0.70) + (score_mistico_e2 * 0.30)

    # Convertir a porcentajes limpios
    total_puntos = final_e1 + final_e2
    prob_e1 = (final_e1 / total_puntos) * 100
    prob_e2 = (final_e2 / total_puntos) * 100
    
    # Cálculo matemático para el factor de Empate Técnico
    prob_empate = abs(prob_e1 - prob_e2) * 0.35
    prob_e1_final = round(prob_e1 - (prob_empate / 2), 2)
    prob_e2_final = round(prob_e2 - (prob_empate / 2), 2)
    prob_empate_final = round(prob_empate, 2)

    # 5. Output Estructurado de Resultados
    print(f"📊 --- RESULTADO DE LA PROYECCIÓN --- 📊")
    print(f"🏟️ Partido: {equipo1} vs {equipo2}")
    print(f"🟢 Probabilidad Victoria {equipo1}: {prob_e1_final}%")
    print(f"⚪ Probabilidad de Empate: {prob_empate_final}%")
    print(f"🔵 Probabilidad Victoria {equipo2}: {prob_e2_final}%")
    print("=" * 60)


# ============================================================================
# CONFIGURACIÓN DE DATOS (Mundial 2026: Costa de Marfil vs Curazao)
# ============================================================================

datos_deportivos_partido = {
    'goles_e1': 2.1, 'goles_e2': 0.8,       # Goles promedio últimos 15 partidos
    'titulos_e1': 3, 'titulos_e2': 0,        # Copas oficiales de su confederación
    'presion_e1': 30, 'presion_e2': 85,      # % de nerviosismo/riesgo de eliminación
    'fisico_e1': 92, 'fisico_e2': 68,        # Rendimiento en velocidad y despliegue
    'sede_e1': 50, 'sede_e2': 50             # Sede neutral (Filadelfia)
}

datos_misticos_partido = {
    'num_fundacion_e1': 9, 'num_fundacion_e2': 5,    # Reducción numerológica pitagórica
    'vibracion_color_e1': 0.85, 'vibracion_color_e2': 0.60  # Energía cromática del uniforme
}

# Ejecución automática del sistema unificado
simular_sistema_murios_esoterico_completo(
    equipo1="Costa de Marfil", 
    equipo2="Curazao", 
    datos_deportivos=datos_deportivos_partido, 
    datos_misticos_base=datos_misticos_partido
)






    

