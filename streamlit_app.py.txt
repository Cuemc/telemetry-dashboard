# streamlit_app.py
import streamlit as st
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

# Настройка страницы
st.set_page_config(page_title="RaceTelemetry Insights", layout="wide")
st.title("🏎️ RaceTelemetry Insights")

# --- Боковая панель ---
with st.sidebar:
    st.header("🔧 Фильтры")
    driver = st.selectbox("Пилот", ["Пилот А", "Пилот Б"])
    lap = st.slider("Круг", 1, 50, 10)
    session = st.selectbox("Сессия", ["Квалификация", "Гонка", "FP1"])
    compare = st.selectbox("Сравнить с", ["Эталон", "Пилот Б"])
    st.markdown("---")
    st.markdown("### 📤 Управление")
    if st.button("Обновить данные"):
        st.experimental_rerun()
    st.info("Данные обновлены автоматически.")

# --- Генерация тестовых данных ---
np.random.seed(lap)
distance = np.linspace(0, 5600, 1000)  # 5.6 км — типичная трасса

# Скорость: эталон + шум + деградация
speed_ref = 320 - 150 * np.sin(distance / 300)
speed_curr = speed_ref - np.random.normal(5, 4, len(distance)) + (lap - 5) * 0.4
delta_time = np.cumsum((speed_ref - speed_curr) / 3600 * 0.1)

# Температура шин (растёт с кругом)
temp_fl = 85 + lap * 2.3 + np.random.randn()

# --- Ключевые метрики ---
st.markdown("### 📊 Ключевые метрики")
col1, col2, col3, col4 = st.columns(4)
col1.metric("Время круга", f"{92.3 + lap * 0.12:.3f} с", delta=f"+{delta_time[-1]:.3f} с")
col2.metric("Delta-time", f"{delta_time[-1]:.3f} с", delta="от эталона", delta_color="inverse")
col3.metric("Темп. ППШ", f"{temp_fl:.0f}°C", delta="📈", delta_color="inverse" if temp_fl > 115 else "normal")
col4.metric("G_max (поперечное)", f"{4.1 - lap * 0.03:.2f} G")

# --- Карта трассы ---
st.markdown("### 🗺️ Траектория на трассе")
fig1, ax1 = plt.subplots(figsize=(8, 6))
track_x = 3000 + 2000 * np.cos(distance / 5600 * 2 * np.pi)
track_y = 2500 + 1500 * np.sin(distance / 5600 * 2 * np.pi)
ax1.plot(track_x, track_y, color='gray', linewidth=10, label="Трасса")
scatter = ax1.scatter(track_x[::50], track_y[::50], c=speed_curr[::50], cmap='plasma', s=20)
ax1.set_aspect('equal')
ax1.axis('off')
plt.colorbar(scatter, ax=ax1, label="Скорость (км/ч)")
st.pyplot(fig1)

# --- График скорости и дельта-тайм ---
st.markdown("### 📈 Сравнение скорости и delta-time")
fig2, ax2 = plt.subplots(figsize=(10, 4))
ax2.plot(distance, speed_ref, label="Эталон", color="green", linewidth=2)
ax2.plot(distance, speed_curr, label=f"Круг {lap}", color="red", linewidth=1.5)
ax2.set_ylabel("Скорость (км/ч)")
ax2.set_xlabel("Дистанция (м)")
ax2.legend()

ax3 = ax2.twinx()
ax3.fill_between(distance[::20], delta_time[::20], color="orange", alpha=0.3, label="Delta-time")
ax3.set_ylabel("Delta-time (с)")
ax3.legend(loc="upper right")
st.pyplot(fig2)

# --- Температура шин по кругам ---
st.markdown("### 🌡️ Температура шин (динамика)")
laps = np.arange(1, 16)
temps = 85 + 2.2 * laps + np.random.randn(15) * 3
fig3, ax4 = plt.subplots(figsize=(8, 3))
ax4.plot(laps, temps, marker='o', color='red', linewidth=2)
ax4.axhspan(90, 110, color='green', alpha=0.1, label="Оптимально")
ax4.axhspan(110, 130, color='red', alpha=0.1, label="Перегрев")
ax4.set_ylabel("Темп. ППШ (°C)")
ax4.set_xlabel("Круг")
ax4.set_xticks(laps[::2])
ax4.grid(True, alpha=0.3)
ax4.legend()
st.pyplot(fig3)

# --- Подвал ---
st.markdown("---")
st.markdown("🔧 *Система анализа телеметрии | Команда: [Твоё имя]*")