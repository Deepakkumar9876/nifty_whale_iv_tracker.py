# nifty_whale_iv_tracker.py
code = '''
import streamlit as st
import pandas as pd
import requests
from datetime import datetime

st.set_page_config(layout="wide")
st.title("🧊 Nifty Options Whale Tracker with IV & OI Zones")

@st.cache_data(ttl=30)
def fetch_option_chain():
    headers = {"User-Agent": "Mozilla/5.0"}
    url = "https://www.nseindia.com/api/option-chain-indices?symbol=NIFTY"
    session = requests.Session()
    session.get("https://www.nseindia.com", headers=headers)
    response = session.get(url, headers=headers)
    data = response.json()["records"]["data"]

    option_data = []
    for item in data:
        strike = item.get("strikePrice")
        ce = item.get("CE", {})
        pe = item.get("PE", {})
        if ce:
            option_data.append({
                "Strike": strike,
                "Type": "CE",
                "Volume": ce.get("totalTradedVolume", 0),
                "OI": ce.get("openInterest", 0),
                "Price": ce.get("lastPrice", 0),
                "IV": ce.get("impliedVolatility", 0),
                "Updated": datetime.now().strftime("%H:%M:%S")
            })
        if pe:
            option_data.append({
                "Strike": strike,
                "Type": "PE",
                "Volume": pe.get("totalTradedVolume", 0),
                "OI": pe.get("openInterest", 0),
                "Price": pe.get("lastPrice", 0),
                "IV": pe.get("impliedVolatility", 0),
                "Updated": datetime.now().strftime("%H:%M:%S")
            })
    return pd.DataFrame(option_data)

df = fetch_option_chain()

lot_size = 75
threshold = 75000
df_whales = df[df["Volume"] >= threshold]
df_whales = df_whales[df_whales["IV"] < 15]
df_whales = df_whales.sort_values(by="Volume", ascending=False)

st.markdown("### 🐳 Confirmed Whale Trades (Volume ≥ 1000 lots and IV < 15%)")
if not df_whales.empty:
    st.dataframe(df_whales, use_container_width=True)
else:
    st.info("No confirmed whale trades yet. Try refreshing in a few seconds.")

st.markdown("### 🔍 Top 5 OI Strikes (Support & Resistance Zones)")
df_ce = df[df["Type"] == "CE"].sort_values(by="OI", ascending=False).head(5)
df_pe = df[df["Type"] == "PE"].sort_values(by="OI", ascending=False).head(5)

col1, col2 = st.columns(2)
with col1:
    st.markdown("#### 🔴 Resistance (Calls with High OI)")
    st.dataframe(df_ce[["Strike", "OI", "Price", "IV"]], use_container_width=True)
with col2:
    st.markdown("#### 🟢 Support (Puts with High OI)")
    st.dataframe(df_pe[["Strike", "OI", "Price", "IV"]], use_container_width=True)

st.caption("Auto-refresh every 30 seconds. Use this to track where big players are positioned.")
'''

with open("nifty_whale_iv_tracker.py", "w") as f:
    f.write(code)

print("✅ Dashboard code saved. Now upload this file to Streamlit Cloud.")
