# Toma-de-decisiones-con-base-en-datos-para-la-salud-empresarial
import streamlit as st
import pandas as pd
import yfinance as yf
import matplotlib.pyplot as plt

# --- Configuración de la página ---
st.set_page_config(page_title="Análisis Financiero", layout="wide")
st.title("📊 Análisis Financiero Interactivo")
st.markdown("Compará el desempeño de distintas acciones en el mercado 📈")

# --- Entrada del usuario ---
st.sidebar.header("Configuración")
tickers_input = st.sidebar.text_input(
    "Símbolos de acciones (separados por coma)",
    value="YPF,CVX,XOM"
)
fecha_inicio = st.sidebar.date_input("Fecha inicio", pd.to_datetime("2024-01-01"))
fecha_fin = st.sidebar.date_input("Fecha fin", pd.to_datetime("2025-01-01"))

# --- Procesar tickers ---
tickers = [t.strip().upper() for t in tickers_input.split(",") if t.strip()]

# --- Descargar datos ---
if st.sidebar.button("Descargar datos"):
    try:
        data = yf.download(tickers, start=fecha_inicio, end=fecha_fin)
        st.success("✅ Datos descargados correctamente")

        # --- Calcular rendimientos ---
        rend_diario = data['Close'].pct_change()
        rend_anual = (rend_diario.mean() * 252).to_frame(name="Rendimiento_Anual")

        # --- Mostrar datos ---
        st.subheader("📅 Datos Históricos")
        st.dataframe(data['Close'].tail())

        # --- Gráfico de precios ---
        st.subheader("📈 Evolución de precios de cierre")
        fig, ax = plt.subplots(figsize=(10, 5))
        data['Close'].plot(ax=ax)
        plt.xlabel("Fecha")
        plt.ylabel("Precio (USD)")
        st.pyplot(fig)

        # --- Tabla de rendimiento ---
        st.subheader("💹 Rendimientos Anuales Estimados")
        st.dataframe(rend_anual.style.format("{:.2%}"))

        # --- Gráfico de rendimientos diarios ---
        st.subheader("📊 Rendimientos diarios")
        fig2, ax2 = plt.subplots(figsize=(10, 5))
        rend_diario.plot(ax=ax2)
        plt.xlabel("Fecha")
        plt.ylabel("Rendimiento diario (%)")
        st.pyplot(fig2)

        # --- Opción de descarga ---
        st.subheader("⬇️ Descargar datos")
        excel_name = "analisis_financiero_completo.xlsx"
        with pd.ExcelWriter(excel_name) as writer:
            data.to_excel(writer, sheet_name="Datos_Completos")
            rend_diario.to_excel(writer, sheet_name="Rendimientos_Diarios")
            rend_anual.to_excel(writer, sheet_name="Resumen_Anual")

        with open(excel_name, "rb") as f:
            st.download_button(
                label="Descargar Excel con todos los datos",
                data=f,
                file_name=excel_name,
                mime="application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"
            )

    except Exception as e:
        st.error(f"❌ Error al descargar los datos: {e}")

else:
    st.info("👈 Ingresá los símbolos y presioná **Descargar datos** para comenzar.")
