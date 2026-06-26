#Simulador Místico de Partidos.
import math

# 1. BASE DE DATOS MÍSTICA (Simulación de predicciones recolectadas)
PREDICCIONES_MUNDIAL = {
    "USA_vs_BOS": {
        "tarotistas": [
            {"nombre": "Brujo Astral", "cartas_e1": ["El Sol", "La Fuerza"], "goles_e1": 2, "goles_e2": 0, "fuerza_astral": 85},
            {"nombre": "Zafiro Tarot", "cartas_e1": ["La Torre", "El Loco"], "goles_e1": 1, "goles_e2": 1, "fuerza_astral": 50}
        ],
        "radiestesia": [
            {"nombre": "Maestro Péndulo", "oscilacion_e1": 78, "oscilacion_e2": 45, "goles_e1": 3, "goles_e2": 1},
            {"nombre": "Varillas Sanadoras", "oscilacion_e1": 80, "oscilacion_e2": 30, "goles_e1": 2, "goles_e2": 0}
        ]
    }
}

# 2. MOTOR DE PREDICCIÓN (Algoritmo que consolida las energías)
def calcular_marcador_exacto(partido_id):
    if partido_id not in PREDICCIONES_MUNDIAL:
        print("Partido no registrado en el cosmos.")
        return

    datos = PREDICCIONES_MUNDIAL[partido_id]
    
    total_goles_e1_tarot = sum(t["goles_e1"] for t in datos["tarotistas"])
    total_goles_e2_tarot = sum(t["goles_e2"] for t in datos["tarotistas"])
    fuerza_tarot_total = sum(t["fuerza_astral"] for t in datos["tarotistas"]) / len(datos["tarotistas"])
    
    total_goles_e1_varillas = sum(r["goles_e1"] for r in datos["radiestesia"])
    total_goles_e2_varillas = sum(r["goles_e2"] for r in datos["radiestesia"])
    energia_e1_varillas = sum(r["oscilacion_e1"] for r in datos["radiestesia"]) / len(datos["radiestesia"])
    energia_e2_varillas = sum(r["oscilacion_e2"] for r in datos["radiestesia"]) / len(datos["radiestesia"])

    cant_tarot = len(datos["tarotistas"])
    cant_varillas = len(datos["radiestesia"])
    total_fuentes = cant_tarot + cant_varillas

    marcador_final_e1 = round((total_goles_e1_tarot + total_goles_e1_varillas) / total_fuentes)
    marcador_final_e2 = round((total_goles_e2_tarot + total_goles_e2_varillas) / total_fuentes)
    
    alineacion_cosmica = (fuerza_tarot_total + (energia_e1_varillas - energia_e2_varillas + 50)) / 2

    print("==================================================")
    print("🔮 SISTEMA MURIOS MATÍAS - SIMULADOR MÍSTICO 🔮")
    print(f"Análisis consolidado para el partido: {partido_id}")
    print("==================================================")
    print(f"📊 Fuentes analizadas: {cant_tarot} Tarotistas | {cant_varillas} Radiestesistas")
    print(f"✨ Alineación cósmica del Equipo 1: {alineacion_cosmica:.1f}%")
    print("--------------------------------------------------")
    print(f"🏁 MARCADOR EXACTO PREDECIDO: Equipo 1 [{marcador_final_e1}] - [{marcador_final_e2}] Equipo 2")
    print("==================================================")

if __name__ == "__main__":
    calcular_marcador_exacto("USA_vs_BOS")
