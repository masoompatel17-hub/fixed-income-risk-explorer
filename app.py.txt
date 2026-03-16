import streamlit as st
import numpy as np
import plotly.graph_objects as go

st.set_page_config(page_title="Fixed Income Risk Explorer", layout="wide")

st.title("Fixed Income Risk Explorer")
st.caption("Interactive model showing how interest rate shocks affect a fixed income portfolio.")

# Sidebar Inputs
with st.sidebar:
    st.header("Model Inputs")

    portfolio_value = st.slider("Portfolio Value ($)", 100000, 10000000, 1000000, step=100000)
    convexity = st.slider("Convexity", 5, 100, 40)
    max_rate_shift = st.slider("Max Interest Rate Shock (%)", 1.0, 5.0, 2.0, step=0.5)

# Create grid for surface
rate_changes = np.linspace(-max_rate_shift/100, max_rate_shift/100, 60)
durations = np.linspace(1, 15, 60)

X, Y = np.meshgrid(rate_changes, durations)

# Bond sensitivity approximation
pct_change = (-Y * X) + (0.5 * convexity * (X**2))
Z = portfolio_value * pct_change

# Metrics
worst_case = np.min(pct_change) * 100
best_case = np.max(pct_change) * 100

col1, col2 = st.columns(2)
col1.metric("Worst Scenario Loss", f"{worst_case:.2f}%")
col2.metric("Best Scenario Gain", f"{best_case:.2f}%")

# 3D Surface Graph
fig = go.Figure(
    data=[
        go.Surface(
            x=X*100,
            y=Y,
            z=Z,
            colorscale="Viridis"
        )
    ]
)

fig.update_layout(
    title="Portfolio Sensitivity Surface",
    scene=dict(
        xaxis_title="Interest Rate Change (%)",
        yaxis_title="Duration",
        zaxis_title="Portfolio Value Change ($)"
    ),
    height=700
)

st.plotly_chart(fig, use_container_width=True)

st.markdown("""
### About this model
This tool visualizes how changes in interest rates interact with portfolio duration and convexity.  
Instead of a static spreadsheet calculation, the model generates a dynamic risk landscape that updates as parameters change.
""")