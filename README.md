import streamlit as st
import pandas as pd
import plotly.express as px
from fpdf import FPDF

# Configuração da página
st.set_page_config(page_title="Alpha Gestão Comercial", layout="wide")

st.title("📊 Alpha Gestão: Inteligência Comercial")
st.markdown("---")

# Upload do arquivo
file = st.file_uploader("Suba sua base de dados (CSV ou Excel)", type=['csv', 'xlsx'])

if file:
    # Carregamento dos dados
    try:
        df = pd.read_csv(file) if file.name.endswith('.csv') else pd.read_excel(file)
        st.success("Dados carregados com sucesso!")
        
        # Interface de Seleção
        col1, col2 = st.columns([1, 3])
        
        with col1:
            st.subheader("Configurações")
            eixo_x = st.selectbox("Selecione a Dimensão (Eixo X)", df.columns)
            eixo_y = st.selectbox("Selecione a Métrica (Eixo Y)", df.columns)
            
        with col2:
            # Lógica de Gráfico Automático
            if pd.api.types.is_numeric_dtype(df[eixo_y]):
                if 'data' in eixo_x.lower() or pd.api.types.is_datetime64_any_dtype(df[eixo_x]):
                    fig = px.line(df, x=eixo_x, y=eixo_y, title="Tendência ao Longo do Tempo")
                else:
                    fig = px.bar(df, x=eixo_x, y=eixo_y, color=eixo_x, title="Comparativo de Performance")
                
                st.plotly_chart(fig, use_container_width=True)
                
                # Insights Rápidos
                media = df[eixo_y].mean()
                total = df[eixo_y].sum()
                st.info(f"💡 *Insight:* A média de {eixo_y} é {media:.2f} e o volume total acumulado é {total:.2f}.")
            else:
                st.warning("Selecione uma coluna numérica para o Eixo Y para gerar o gráfico.")

    except Exception as e:
        st.error(f"Erro ao processar arquivo: {e}")
else:
    st.info("Aguardando upload da planilha para análise.")
