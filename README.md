# ETP-2
# [🚨Smart Innovation Landslide Early Warning System🚨] ![Soil-erosion-feature-at-Genting-Highlands](https://github.com/user-attachments/assets/6a5ac9bf-4607-4e51-b519-8a0730b5fb58)

import streamlit as st
import pandas as pd
import numpy as np
import altair as alt
from datetime import datetime

# ----------------------------------------
# 🚨 Classify threat levels into MILD / LOW / HIGH
# ----------------------------------------
def classify_threat_level(row):
    if row['tilt_angle_deg'] > 10 or row['soil_moisture_pct'] > 90 or row['rainfall_mm'] > 40:
        return 'HIGH'
    elif row['tilt_angle_deg'] > 5 or row['soil_moisture_pct'] > 70 or row['rainfall_mm'] > 30:
        return 'LOW'
    else:
        return 'MILD'

# ----------------------------------------
# 🔧 Page Config
# ----------------------------------------
st.set_page_config(page_title="Smart Landslide Early Warning System", layout="wide")
st.title("🚨 Smart Innovation: Landslide Early Warning System")

# ----------------------------------------
# 📂 File Uploader
# ----------------------------------------
st.sidebar.header("Upload Sensor CSV File")
uploaded_file = st.sidebar.file_uploader("Choose a CSV file", type=["csv"])

# ----------------------------------------
# 📊 Upload + Processing
# ----------------------------------------
if st.sidebar.button("Upload and Process"):
    if uploaded_file:
        # Load CSV data
        df = pd.read_csv(uploaded_file, parse_dates=['timestamp'])

        # Apply classification if not already present
        if 'threat_level' not in df.columns:
            df['threat_level'] = df.apply(classify_threat_level, axis=1)

        # ----------------------------------------
        # 🎛️ Filter by threat level
        # ----------------------------------------
        st.sidebar.header("Filters")
        threat_filter = st.sidebar.multiselect(
            "Select Risk Levels:",
            ['MILD', 'LOW', 'HIGH'],
            default=['MILD', 'LOW', 'HIGH']
        )
        df_filtered = df[df['threat_level'].isin(threat_filter)]

        # ----------------------------------------
        # 📈 Latest Sensor Metrics
        # ----------------------------------------
        latest = df.iloc[-1]
        col1, col2, col3, col4 = st.columns(4)
        col1.metric("Tilt Angle (°)", f"{latest['tilt_angle_deg']:.2f}")
        col2.metric("Soil Moisture (%)", f"{latest['soil_moisture_pct']:.2f}")
        col3.metric("Rainfall (mm)", f"{latest['rainfall_mm']:.2f}")
        col4.metric("Current Risk Level", latest['threat_level'])

        # ----------------------------------------
        # 📊 Time Series Chart
        # ----------------------------------------
        st.subheader("📈 Sensor Trends Over Time")
        chart = alt.Chart(df_filtered).transform_fold(
            ['tilt_angle_deg', 'soil_moisture_pct', 'rainfall_mm']
        ).mark_line().encode(
            x='timestamp:T',
            y='value:Q',
            color='key:N'
        ).properties(
            width=900,
            height=400
        )
        st.altair_chart(chart, use_container_width=True)

        # ----------------------------------------
        # 📊 Risk Level Distribution
        # ----------------------------------------
        st.subheader("📊 Risk Level Distribution")
        threat_count = df_filtered['threat_level'].value_counts().reset_index()
        threat_count.columns = ['Risk Level', 'Count']
        st.bar_chart(threat_count.set_index('Risk Level'))

        # ----------------------------------------
        # 🧾 Data Table
        # ----------------------------------------
        st.subheader("🧾 Sensor Data Table (Last 50 Rows)")
        st.dataframe(df_filtered.tail(50), use_container_width=True)

        # ----------------------------------------
        # 📥 Download Button
        # ----------------------------------------
        st.download_button(
            "📥 Download Filtered CSV",
            data=df_filtered.to_csv(index=False),
            file_name="filtered_landslide_data.csv",
            mime='text/csv'
        )
    else:
        st.warning("⚠️ Please upload a CSV file to proceed.")
