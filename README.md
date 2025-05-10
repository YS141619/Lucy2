import streamlit as st
import pandas as pd
import plotly.express as px
import json
import os

# 데이터 로드
@st.cache_data
def load_data():
    file_path = "korea_population_2024.csv"
    if not os.path.exists(file_path):
        st.error(f"❌ 파일이 존재하지 않습니다: `{file_path}`")
        return pd.DataFrame()

    df = pd.read_csv(file_path)
    df.columns = [col.strip().replace(' ', '_') for col in df.columns]
    df = df.rename(columns={'Region': 'Region', 'Population': 'Population'})
    return df

@st.cache_data
def load_geojson():
    file_path = "skorea-provinces-geo.json"
    if not os.path.exists(file_path):
        st.error(f"❌ 지도 파일이 존재하지 않습니다: `{file_path}`")
        return None

    with open(file_path, encoding="utf-8") as f:
        geojson = json.load(f)
    return geojson

# 데이터 로딩
df = load_data()
geojson = load_geojson()

if df.empty or geojson is None:
    st.stop()

# 제목
st.title("🇰🇷 2024 대한민국 시도별 인구 Dashboard")
st.markdown("📊 **시도별 인구 데이터를 시각화한 대시보드입니다.**")

# 탭 구성
tab1, tab2, tab3 = st.tabs(["🗺️ 시도별 인구 지도", "🏆 상위 인구 시도", "📈 인구와 요인 간 상관분석"])

# 🗺️ 탭1: 시도별 인구 지도
with tab1:
    st.subheader("시도별 인구수 지도")
    fig_map = px.choropleth(
        df,
        geojson=geojson,
        locations="Region",
        featureidkey="properties.name",
        color="Population",
        color_continuous_scale="YlOrRd",
        hover_name="Region",
        title="2024 시도별 인구수 지도"
    )
    fig_map.update_geos(fitbounds="locations", visible=False)
    st.plotly_chart(fig_map, use_container_width=True)

# 🏆 탭2: 인구 상위 10개 시도
with tab2:
    st.subheader("인구수 상위 10개 시도")
    top10 = df.sort_values("Population", ascending=False).head(10)
    fig_bar = px.bar(
        top10,
        x="Population",
        y="Region",
        orientation="h",
        color="Population",
        color_continuous_scale="Blues",
        title="상위 인구 시도"
    )
    st.plotly_chart(fig_bar, use_container_width=True)

# 📈 탭3: 상관관계 분석
with tab3:
    st.subheader("인구수와 다른 요인 간 상관관계")
    numeric_cols = [col for col in ["Population", "Area", "Senior_Rate"] if col in df.columns]

    selected_x = st.selectbox("X축 변수", numeric_cols, index=1)
    selected_y = st.selectbox("Y축 변수", numeric_cols, index=0)

    fig_scatter = px.scatter(
        df,
        x=selected_x,
        y=selected_y,
        text="Region",
        trendline="ols",
        title=f"{selected_x} vs {selected_y}"
    )
    st.plotly_chart(fig_scatter, use_container_width=True)

    st.markdown("📌 선형 추세선을 통해 변수 간 관계를 시각적으로 파악할 수 있습니다.")
