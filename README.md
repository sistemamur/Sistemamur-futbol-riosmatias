
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
predecir_partido_sistemamuriosmatias(partido_a_testear)







    

