

def procesar_mística_esotérica(signo_dt, color_camiseta):
    """Módulo Esotérico: Calcula el impacto astral en goles (De -0.5 a +0.5 goles)"""
    modificador_astral = 0.0
    
    # 1. Tránsitos planetarios según el signo del DT (Fuego = Más energía)
    signos_fuego = ["Aries", "Leo", "Sagitario"]
    if signo_dt.capitalize() in signos_fuego:
        modificador_astral += 0.25  # Marte a favor da un plus de empuje
        
    # 2. Cromoterapia y cábalas según el color de la camiseta
    if color_camiseta.lower() in ["blanco", "celeste", "azul"]:
        modificador_astral += 0.25  # Alta vibración energética
    elif color_camiseta.lower() in ["negro", "gris"]:
        modificador_astral -= 0.25  # Bloqueo energético o Mercurio Retrógrado
        
    return modificador_astral

def sistemamur_prediccion_total(eq1, eq2, d1, d2):
    """
    SISTEMAMURIOSMATIAS v3.0 - HÍBRIDO TOTAL
    Combina: Goles (Últimos 10 PJ) + Tarjetas + Velocidad + Módulo Esotérico
    """
    print(f"--- SIMULACIÓN HÍBRIDA: {eq1} vs {eq2} ---")
    
    # 1. BASE LÓGICA: Cruce de ataque vs defensa
    goles_eq1 = (d1["gf_promedio"] + d2["gc_promedio"]) / 2
    goles_eq2 = (d2["gf_promedio"] + d1["gc_promedio"]) / 2
    
    # 2. IMPACTO DE TARJETA ROJA (Suspensiones)
    if d1["rojas_acumuladas"] > 0: goles_eq1 *= 0.85
    if d2["rojas_acumuladas"] > 0: goles_eq2 *= 0.85
        
    # 3. IMPACTO DE AMARILLAS (Penales / Tiros libres concedidos)
    goles_eq1 += (d2["amarillas_promedio"] * 0.05)
    goles_eq2 += (d1["amarillas_promedio"] * 0.05)
    
    # 4. FACTOR VELOCIDAD (Contragolpes de los últimos 10 partidos)
    dif_vel = d1["velocidad_promedio"] - d2["velocidad_promedio"]
    if dif_vel >= 1.5:   goles_eq1 += 0.4
    elif dif_vel <= -1.5: goles_eq2 += 0.4
        
    # 5. INYECCIÓN ESOTÉRICA (El destino y los astros modifican el marcador)
    mística_eq1 = procesar_mística_esotérica(d1["signo_dt"], d1["color_camiseta"])
    mística_eq2 = procesar_mística_esotérica(d2["signo_dt"], d2["color_camiseta"])
    
    goles_eq1 += mística_eq1
    goles_eq2 += mística_eq2
    
    # Marcador final redondeado (nunca menor a cero)
    marcador_eq1 = max(0, round(goles_eq1))
    marcador_eq2 = max(0, round(goles_eq2))
    
    print(f"Marcador Proyectado: {eq1} {marcador_eq1} - {marcador_eq2} {eq2}")
    
    if marcador_eq1 > marcador_eq2:
        return f"Resultado: Victoria de {eq1} 🔮"
    elif marcador_eq2 > marcador_eq1:
        return f"Resultado: Victoria de {eq2} 🔮"
    else:
        return "Resultado: Empate Clavado por los Astros ⚖️"

# ==========================================
# PRUEBA EN VIVO DEL SISTEMA COMPLETO
# ==========================================
if __name__ == "__main__":
    # Ejemplo: Paraguay vs Australia (Se juega hoy)
    paraguay = {
        "gf_promedio": 1.2, "gc_promedio": 0.8,
        "amarillas_promedio": 2.5, "rojas_acumuladas": 1, # Almirón suspendido (-15%)
        "velocidad_promedio": 7.5,
        "signo_dt": "Aries",       # Gustavo Alfaro (Fuego: +0.25 goles)
        "color_camiseta": "blanco" # Tradicional albirroja (+0.25 goles)
    }
    
    australia = {
        "gf_promedio": 1.5, "gc_promedio": 1.0,
        "amarillas_promedio": 1.8, "rojas_acumuladas": 0,
        "velocidad_promedio": 6.0, # Más lentos que Paraguay
        "signo_dt": "Tauro",       # Tierra (Neutral)
        "color_camiseta": "amarillo" # Neutral
    }
    
    analisis = sistemamur_prediccion_total("Paraguay", "Australia", paraguay, australia)
    print(analisis)
