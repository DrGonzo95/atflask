import streamlit as st
import yfinance as yf
import pandas as pd
import numpy as np
import plotly.graph_objects as go
from ta.trend import EMAIndicator, MACD
from ta.momentum import RSIIndicator
from ta.volatility import BollingerBands
from scipy.signal import find_peaks

# Función para obtener datos históricos de una acción
def obtener_datos(ticker, periodo='2y'):
    stock = yf.Ticker(ticker)
    df = stock.history(period=periodo)
    return df

# Función para calcular indicadores técnicos
def calcular_indicadores(df):
    df['EMA20'] = EMAIndicator(close=df['Close'], window=20).ema_indicator()
    df['EMA50'] = EMAIndicator(close=df['Close'], window=50).ema_indicator()
    df['EMA200'] = EMAIndicator(close=df['Close'], window=200).ema_indicator()

    macd = MACD(close=df['Close'])
    df['MACD'] = macd.macd()
    df['MACD_Signal'] = macd.macd_signal()

    df['RSI'] = RSIIndicator(close=df['Close']).rsi()

    bollinger = BollingerBands(close=df['Close'])
    df['BB_Superior'] = bollinger.bollinger_hband()
    df['BB_Inferior'] = bollinger.bollinger_lband()
    df['BB_Media'] = bollinger.bollinger_mavg()

    return df

# Función para encontrar soportes y resistencias
def encontrar_soportes_resistencias(df, ventana=20):
    precios = df['Close'].values
    maximos_locales, _ = find_peaks(precios, distance=ventana)
    minimos_locales, _ = find_peaks(-precios, distance=ventana)

    niveles_resistencia = precios[maximos_locales]
    niveles_soporte = precios[minimos_locales]

    return niveles_soporte, niveles_resistencia, maximos_locales, minimos_locales

# Función para calcular objetivos de precio
def calcular_objetivos(df, niveles_soporte, niveles_resistencia):
    precio_actual = df['Close'].iloc[-1]
    soportes_validos = niveles_soporte[niveles_soporte < precio_actual]
    stop_loss = soportes_validos[-1] if len(soportes_validos) > 0 else precio_actual * 0.95
    resistencias_validas = niveles_resistencia[niveles_resistencia > precio_actual]
    take_profit = resistencias_validas[0] if len(resistencias_validas) > 0 else precio_actual * 1.05
    objetivo_precio = precio_actual + (take_profit - precio_actual) * 0.618
    return objetivo_precio, take_profit, stop_loss

# Resumen de análisis técnico
def resumen_tecnico(df):
    precio_actual = df['Close'].iloc[-1]
    ema_20 = df['EMA20'].iloc[-1]
    ema_50 = df['EMA50'].iloc[-1]
    ema_200 = df['EMA200'].iloc[-1]
    rsi = df['RSI'].iloc[-1]

    if precio_actual > ema_200:
        tendencia = "Alcista"
    else:
        tendencia = "Bajista"

    if rsi > 70:
        recomendacion = "Posible sobrecompra, considerar vender."
    elif rsi < 30:
        recomendacion = "Posible sobreventa, considerar comprar."
    else:
        recomendacion = "Condiciones normales, seguir evaluando."

    return tendencia, recomendacion

# Gráfico interactivo con Plotly
def graficar(df, ticker, niveles_soporte, niveles_resistencia, maximos_locales, minimos_locales, objetivo_precio, take_profit, stop_loss):
    fig = go.Figure()

    # Precio y EMAs
    fig.add_trace(go.Scatter(x=df.index, y=df['Close'], mode='lines', name='Precio', line=dict(color='blue')))
    fig.add_trace(go.Scatter(x=df.index, y=df['EMA20'], mode='lines', name='EMA 20', line=dict(color='orange')))
    fig.add_trace(go.Scatter(x=df.index, y=df['EMA50'], mode='lines', name='EMA 50', line=dict(color='green')))
    fig.add_trace(go.Scatter(x=df.index, y=df['EMA200'], mode='lines', name='EMA 200', line=dict(color='red')))

    # Bandas de Bollinger
    fig.add_trace(go.Scatter(x=df.index, y=df['BB_Superior'], mode='lines', name='Banda Superior', line=dict(color='gray', dash='dot')))
    fig.add_trace(go.Scatter(x=df.index, y=df['BB_Inferior'], mode='lines', name='Banda Inferior', line=dict(color='gray', dash='dot')))

    # Soportes y Resistencias
    fig.add_trace(go.Scatter(x=df.index[maximos_locales], y=df['Close'].iloc[maximos_locales], mode='markers', name='Resistencias', marker=dict(color='red', symbol='triangle-up')))
    fig.add_trace(go.Scatter(x=df.index[minimos_locales], y=df['Close'].iloc[minimos_locales], mode='markers', name='Soportes', marker=dict(color='green', symbol='triangle-down')))

    # Objetivos
    fig.add_hline(y=objetivo_precio, line=dict(color='purple', dash='dash'), annotation_text='Objetivo de Precio')
    fig.add_hline(y=take_profit, line=dict(color='green', dash='dash'), annotation_text='Take Profit')
    fig.add_hline(y=stop_loss, line=dict(color='red', dash='dash'), annotation_text='Stop Loss')

    fig.update_layout(title=f'Análisis Técnico de {ticker}', xaxis_title='Fecha', yaxis_title='Precio', template='plotly_dark')
    return fig

# Gráfico del RSI
def graficar_rsi(df):
    fig = go.Figure()

    # Gráfico RSI
    fig.add_trace(go.Scatter(x=df.index, y=df['RSI'], mode='lines', name='RSI', line=dict(color='blue')))
    fig.add_hline(y=70, line=dict(color='red', dash='dash'), annotation_text='Sobrecompra (70)')
    fig.add_hline(y=30, line=dict(color='green', dash='dash'), annotation_text='Sobreventa (30)')

    fig.update_layout(title="RSI (Índice de Fuerza Relativa)", xaxis_title='Fecha', yaxis_title='RSI', template='plotly_dark')
    return fig

# Gráfico del MACD
def graficar_macd(df):
    fig = go.Figure()

    # Gráfico MACD
    fig.add_trace(go.Scatter(x=df.index, y=df['MACD'], mode='lines', name='MACD', line=dict(color='blue')))
    fig.add_trace(go.Scatter(x=df.index, y=df['MACD_Signal'], mode='lines', name='Señal MACD', line=dict(color='red')))

    fig.update_layout(title="MACD (Convergencia/Divergencia de la Media Móvil)", xaxis_title='Fecha', yaxis_title='MACD', template='plotly_dark')
    return fig

# Interfaz de Streamlit
st.title("Análisis Técnico de Acciones")

# Selección de ticker
ticker = st.text_input("Ingrese el Ticker del Activo (e.g., AAPL, TSLA):", value="AAPL")

# Selección de periodo
periodo = st.selectbox("Seleccionar el periodo de datos:", ["1d", "1mo", "3mo", "6mo", "1y", "2y", "5y", "10y"], index=1)

if ticker:
    df = obtener_datos(ticker, periodo)
    df = calcular_indicadores(df)
    niveles_soporte, niveles_resistencia, maximos_locales, minimos_locales = encontrar_soportes_resistencias(df)
    objetivo_precio, take_profit, stop_loss = calcular_objetivos(df, niveles_soporte, niveles_resistencia)
    tendencia, recomendacion = resumen_tecnico(df)

    # Mostrar gráfico interactivo de la acción
    fig = graficar(df, ticker, niveles_soporte, niveles_resistencia, maximos_locales, minimos_locales, objetivo_precio, take_profit, stop_loss)
    st.plotly_chart(fig)

    # Mostrar resumen técnico
    st.write(f"**Precio Actual:** ${df['Close'].iloc[-1]:.2f}")
    st.write(f"**Objetivo de Precio:** ${objetivo_precio:.2f}")
    st.write(f"**Take Profit:** ${take_profit:.2f}")
    st.write(f"**Stop Loss:** ${stop_loss:.2f}")
    st.write(f"**Tendencia:** {tendencia}")
    st.write(f"**Recomendación:** {recomendacion}")

    # Graficar y mostrar el RSI
    st.subheader("RSI (Índice de Fuerza Relativa)")
    fig_rsi = graficar_rsi(df)
    st.plotly_chart(fig_rsi)

    # Interpretación del RSI
    if df['RSI'].iloc[-1] > 70:
        st.write("**Interpretación RSI:** El activo está en sobrecompra, lo que podría indicar una posible corrección o caída en el precio.")
    elif df['RSI'].iloc[-1] < 30:
        st.write("**Interpretación RSI:** El activo está en sobreventa, lo que podría indicar una oportunidad de compra.")
    else:
        st.write("**Interpretación RSI:** El RSI se encuentra en una zona neutral, lo que sugiere que no hay señales claras para comprar o vender.")

    # Graficar y mostrar el MACD
    st.subheader("MACD (Convergencia/Divergencia de la Media Móvil)")
    fig_macd = graficar_macd(df)
    st.plotly_chart(fig_macd)

    # Interpretación del MACD
    if df['MACD'].iloc[-1] > df['MACD_Signal'].iloc[-1]:
        st.write("**Interpretación MACD:** Señal de compra, ya que el MACD está por encima de la señal.")
    else:
        st.write("**Interpretación MACD:** Señal de venta, ya que el MACD está por debajo de la señal.")
