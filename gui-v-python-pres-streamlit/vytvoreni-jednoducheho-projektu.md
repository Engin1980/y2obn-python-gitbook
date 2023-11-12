# Vytvoření jednoduchého projektu

Vytvořená aplikace, mezistav:

{% code lineNumbers="true" %}
```python
import streamlit as st
import datetime


st.header("Nazdar")

name = st.text_input("Zadej jméno:")
birth_date = st.date_input("Zadej datum narození", datetime.date(2000, 12, 24),)
if st.button("Potvrdit zadání"):
    st.text(name)
```
{% endcode %}

