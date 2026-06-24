# Sistemamur-futbol-riosmatias
Calculadora futbolistica
# ==========================================
# sistema mur-futbol-rios matias
# ==========================================
# Método de Predicción de Probabilidades en Fútbol

def calcular_merito(w, d, gf, gc):
    """
    Calcula la variable M (Mérito) basada en los últimos 10 partidos.
    w = victorias, d = empates, gf = goles favor, gc = goles contra
    """
    rendimiento = ((w * 3) + (d * 1)) / 30.0
    factor_goles = gf / (gc + 1.0)
    return rendimiento * factor_goles

 def calcular_pf_absoluto(m, u, r):
    # Usamos un valor base (por ejemplo, 10) para evitar que los restas den negativo
    # A mejor ubicación (número menor), sumamos más puntaje.
    puntaje_posicion = max(0, 21 - u)  # Asumiendo una liga de 20 equipos
    puntaje_rival = max(0, r) 
    
    return (m * 0.5) + (puntaje_posicion * 0.2) + (puntaje_rival * 0.3)


# --- EJEMPLO DE USO RÁPIDO ---
print("--- SISTEMA MUR-FUTBOL-RIOS MATIAS ---")

# Datos de prueba para el equipo 1
M1 = calcular_merito(w=6, d=2, gf=18, gc=8) # Ejemplo: 6 victorias, 2 empates, 18 GF, 8 GC
PF1 = calcular_pf_absoluto(m=M1, u=7, r=4)    # Ubicación=7, Rival=4

# Datos de prueba para el equipo 2
M2 = calcular_merito(w=4, d=3, gf=12, gc=10) # Ejemplo: 4 victorias, 3 empates, 12 GF, 10 GC
PF2 = calcular_pf_absoluto(m=M2, u=5, r=6)    # Ubicación=5, Rival=6

# Probabilidad Final
prob_e1 = (PF1 / (PF1 + PF2)) * 100
prob_e2 = (PF2 / (PF1 + PF2)) * 100

print(f"Probabilidad Equipo 1: {prob_e1:.2f}%")
print(f"Probabilidad Equipo 2: {prob_e2:.2f}%")
