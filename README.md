import streamlit as st
import pandas as pd
from datetime import datetime

st.set_page_config(page_title="Painel Médico", layout="wide")

@st.cache_data
def load_data():
    return pd.read_excel("dados_pacientes.xlsx")

df = load_data()

st.title("📋 Painel de Procedimentos Médicos")

medicos = df['Médico'].dropna().unique()
medico_escolhido = st.selectbox("👨‍⚕️ Selecionar Médico", medicos)
df = df[df['Médico'] == medico_escolhido]

col1, col2, col3, col4 = st.columns(4)
with col1:
    status_opcao = st.multiselect("📌 Status", [
        "AGENDADO", "REALIZADO", "AGUARDANDO AGENDAMENTO", "REVALIDA", 
        "AUTORIZADO", "NEGADO", "ORÇAMENTO", "EM ANÁLISE"
    ])
with col2:
    tipo_guia = st.multiselect("📄 Tipo de Guia", ["SP/SADT", "INTERNACAO"])
with col3:
    tipo_atendimento = st.multiselect("⏱️ Tipo de Atendimento", ["ELETIVO", "URGENCIA"])
with col4:
    numero_guia = st.text_input("🔢 Buscar por Número da Guia")

col5, col6 = st.columns(2)
with col5:
    data_agendamento = st.date_input("📅 Data do Agendamento", value=None)
with col6:
    data_validade = st.date_input("📆 Data da Validade da Guia", value=None)

opme = st.text_input("🧰 Buscar por OPME (opcional)")

if status_opcao:
    df = df[df['Status'].isin(status_opcao)]
if tipo_guia:
    df = df[df['Tipo Guia'].isin(tipo_guia)]
if tipo_atendimento:
    df = df[df['Tipo Atendimento'].isin(tipo_atendimento)]
if numero_guia:
    df = df[df['Número Guia'].astype(str).str.contains(numero_guia)]
if data_agendamento:
    df = df[pd.to_datetime(df['Data Agendamento']).dt.date == data_agendamento]
if data_validade:
    df = df[pd.to_datetime(df['Data Validade Guia']).dt.date == data_validade]
if opme:
    df = df[df['OPME'].astype(str).str.contains(opme, case=False, na=False)]

st.markdown("### 📊 Resultados")
st.dataframe(df, use_container_width=True)

st.download_button(
    label="📥 Baixar dados filtrados",
    data=df.to_excel(index=False),
    file_name=f"dados_{medico_escolhido.replace(' ', '_')}.xlsx",
    mime="application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"
)
