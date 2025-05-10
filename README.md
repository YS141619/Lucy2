import streamlit as st
import pandas as pd
import plotly.express as px

# 데이터 로드
@st.cache_data
def load_data():
    df = pd.read_csv("world-happiness-2024.csv")
    df.columns = [col.replace('Explained by: ', '').replace(' ', '_').lower() for col in df.columns]
    df = df.rename(columns={'country_name': 'Country', 'ladder_score': 'Happiness_Score'})
    return df

df = load_data()

# 제목
st.title("🌍 2024 World Happiness Dashboard")
st.markdown("📊 **세계 행복지수 데이터를 다양한 방식으로 시각화한 대시보드입니다.**")

# 탭 구성
tab1, tab2, tab3 = st.tabs(["🌐 세계지도 시각화", "🏆 상위 국가 그래프", "📈 상관관계 분석"])

# 🌐 탭1: 행복 점수 세계지도
with tab1:
    st.subheader("국가별 행복 점수 지도")
    fig_map = px.choropleth(
        df,
        locations="Country",
        locationmode="country names",
        color="Happiness_Score",
        hover_name="Country",
        color_continuous_scale="YlGnBu",
        title="2024 세계 행복 점수"
    )
    st.plotly_chart(fig_map, use_container_width=True)

# 🏆 탭2: 행복 점수 상위 10개국 그래프
with tab2:
    st.subheader("행복 점수 상위 10개국")
    top10 = df.sort_values("Happiness_Score", ascending=False).head(10)
    fig_bar = px.bar(
        top10,
        x="Happiness_Score",
        y="Country",
        orientation="h",
        color="Happiness_Score",
        color_continuous_scale="Blues",
        title="행복 점수 상위 10개국"
    )
    st.plotly_chart(fig_bar, use_container_width=True)

# 📈 탭3: 행복 요소 간 상관관계 분석
with tab3:
    st.subheader("행복 점수와 요인 간 관계 보기")
    numeric_cols = ["Happiness_Score", "log_gdp_per_capita", "social_support",
                    "healthy_life_expectancy", "freedom_to_make_life_choices",
                    "generosity", "perceptions_of_corruption"]

    selected_x = st.selectbox("X축 변수", numeric_cols, index=1)
    selected_y = st.selectbox("Y축 변수", numeric_cols, index=0)

    fig_scatter = px.scatter(
        df,
        x=selected_x,
        y=selected_y,
        text="Country",
        trendline="ols",
        title=f"{selected_x} vs {selected_y}"
    )
    st.plotly_chart(fig_scatter, use_container_width=True)

    st.markdown("📌 선형 추세선을 통해 변수 간 관계를 시각적으로 파악할 수 있습니다.")
