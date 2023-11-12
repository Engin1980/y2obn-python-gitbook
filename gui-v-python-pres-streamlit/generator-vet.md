---
description: >-
  V této sekci si vytvoříme jednoduchou dětskou hru - generátor vět. Ze vstupu
  (náhodná slova - podmět, přísudek, předmět) se generuje náhodná věta.
---

# Generátor vět

## Příprava projektu

Nejdříve ve složce vytvoříme soubor s projektem `sentence_generator.py` a vložíme do něj kód:

{% code title="sentence_generator.py" lineNumbers="true" %}
```python
import streamlit as st
import datetime
import random

st.set_page_config(layout="wide")

st.header("Sentence generator")
```
{% endcode %}

Poté vytvoříme spouštěcí soubor `start_sentence_generator.bat` s kódem:

{% code title="start_sentence_generator.bat" %}
```batch
.\venv\scripts\activate.bat & streamlit run camera.py
```
{% endcode %}



## Vytvoření základního rozhraní

```python
import datetime
import random

import streamlit as st
import datetime
import random

st.set_page_config(layout="wide")

st.header("Sentence generator")

ss = st.session_state
ss["refresh"] = datetime.datetime.now()

st.write(st.session_state)
st.write(ss["refresh"])


def get_random_item_from_list(lst) -> str:
    index = random.randint(0, len(lst) - 1)
    ret = lst[index]
    return ret


def generate_sentence():
    ret = (get_random_item_from_list(ss["who"])
           + " " + get_random_item_from_list(ss["what"])
           + " " + get_random_item_from_list(ss["whom"])
           + ".")
    return ret


if "initialized" not in ss:
    ss["initialized"] = True

    ss["who"] = []
    ss["what"] = [
        "skáče", "mačká", "roluje"
    ]
    ss["whom"] = ["ručník", "hrudku másla", "citrón"]
    ss["who"].append("Lampička")
    ss["who"].append("Auto")
    ss["who"].append("Tráva")

if "btnWhat" in ss:
    if ss["btnWhat"]:
        ss["what"].append(ss["txtWhat"])
        ss["txtWhat"] = ""

if "txtWhom" in ss and len(ss["txtWhom"]) > 0:
    ss["whom"].append(ss["txtWhom"])
    ss["txtWhom"] = ""

colA, colB, colC = st.columns(3)

with colA:
    st.subheader("Kdo")

    colAtxt, colAbtn = st.columns(2)

    with colAtxt:
        who = st.text_input("Nové 'kdo'", label_visibility="collapsed")
    with colAbtn:
        if st.button("Vlož"):
            ss["who"].append(who)

    for source in ss["who"]:
        st.text(source)

with colB:
    st.subheader("Co")

    colBtxt, colBbtn = st.columns(2)

    with colBtxt:
        what = st.text_input("Nove 'co'", label_visibility="collapsed", key="txtWhat")
    with colBbtn:
        st.button("Vlož", key="btnWhat")

    for source in ss["what"]:
        st.text(source)

with colC:
    st.subheader("Komu")

    what = st.text_input("Nove 'komu'", label_visibility="collapsed", key="txtWhom")

    for source in ss["whom"]:
        st.text(source)

btnGenerate = st.button("G e n e r u j")
if btnGenerate:
    st.text(generate_sentence())

```
