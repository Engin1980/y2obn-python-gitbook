---
description: >-
  Streamlit je prostředí pro tvorbu jednoduchého UI pro Python přes webový
  prohlížeč. Zde si ukážeme základ jeho použití.
---

# Vytvoření jednoduchého projektu

## Představení

Streamlit ([https://streamlit.io/](https://streamlit.io/)) je prostředí pro tvorbu uživatelského rozhraní v programovacím jazyce Python. Streamlit rozhraní tvoří přes prostředí webového prohlížeče, díky čemuž je platformě nezávislý. Pro jeho běh je tedy třeba mít nainstalovaný Python a webový prohlížeč.

Instalaci steamlit jsme si již ukázali - přes instalační službu `pip` instalaci provedeme jednoduchým příkazem `pip install streamlit`.

Po instalaci lze streamlit spustit příkazem (z příkazové řádky) `streamlit run <název_py_souboru>`, kde poslední část příkazu udává název python kódu, který se má vykonávat. T

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

## Úvodní seznámení s tvorbou UI

Streamlit UI tvoříme vkládáním komponent (ovládacích prvků) do stránky. Nejdříve je třeba v prostředí Python naimportovat knihovnu streamlit (typicky jako `st`) a následně lze do stránky vkládat prvky příkazy typu `st.<...prvek...>`. Nejjednodušší příklad vkládající do stránky nadpis:

{% code title="main.py" lineNumbers="true" %}
```python
import streamlit as st

st.header("My first app")
```
{% endcode %}

Nyní aplikaci můžeme spustit `streamlit run main.py` a otevře se okno prohlížeče s nápisem "My first app".

Pokud budeme chtít vytvořit jednoduché textové pole pro zadání jména a tlačítka pro potvrzení:

<pre class="language-python" data-title="main.py" data-line-numbers><code class="lang-python">import streamlit as st
<strong>
</strong><strong>st.text_input("Zadej své jméno:")
</strong><strong>st.button("Potvrdit zadání")
</strong></code></pre>

Do streamlit můžeme vkládat různé ovládací prvky - tlačítka, zaškrtávací pole, textová pole, numerická pole, tabulky, grafy, obrázky a spoustu dalších. Jejich soupis i popis použití lze nalézt v dokumentaci ([https://docs.streamlit.io/](https://docs.streamlit.io/)) v sekci _Streamlit Library -> API Reference_.

Nejběžnější prvky:

<table><thead><tr><th width="228"></th><th></th></tr></thead><tbody><tr><td>st.text</td><td>Vypíše volný text (nápis)</td></tr><tr><td>st.text_input</td><td>Zobrazí okno pro zadání textu</td></tr><tr><td>st.number_input</td><td>Zobrazí okno pro zadání čísla</td></tr><tr><td>st.checkbox</td><td>Zobrazí zaškrtávací pole</td></tr><tr><td>st.slider</td><td>Zobrazí posunovátko</td></tr><tr><td>st.date_input</td><td>Zobrazí pole pro zadání data</td></tr><tr><td>st.time_input</td><td>Zobrazí pole pro zadání času</td></tr><tr><td>st.button</td><td>Zobrazí tlačítko</td></tr><tr><td>st.header</td><td>Zobrazí nadpis</td></tr><tr><td>st.subheader</td><td>Zobrazí podnadpis</td></tr><tr><td>st.title</td><td>Zobrazí titulek</td></tr><tr><td>st.latex</td><td>Zobrazí matematickou rovnici ve formátu TeX</td></tr></tbody></table>

Pokud chceme pracovat s hodnotami prvků, máme dva způsoby. Jednodušší a běžnější je při tvorbě prvku si vrátit hodnotu vytvořeného prvku do proměnné a dále pracovat s touto proměnnou:

<pre class="language-python" data-title="main.py" data-line-numbers><code class="lang-python">import streamlit as st
<strong>
</strong><strong>txt_name = st.text_input("Zadej své jméno:")
</strong><strong># st.button("Potvrdit zadání")
</strong>st.text("Vaše jméno je " + txt_name + ".")
</code></pre>

U vstupních prvků (textové pole, zaškrtávací seznam atp.) je v poměnné uložena hodnota daného prvku. U tlačítek se ukládá příznak `True`, pokud bylo tlačítko stisknuto:

<pre class="language-python" data-title="main.py" data-line-numbers><code class="lang-python">import streamlit as st
<strong>
</strong>txt_name = st.text_input("Zadej své jméno:")
<strong>btn_confirm = st.button("Potvrdit zadání")
</strong><strong>if btn_confirm:
</strong><strong> st.text("Vaše jméno je " + txt_name + ".")
</strong></code></pre>

## Příklad: Kvadratická rovnice

Vytvoříme jednoduchý formulář pro výpočet kvadratické rovnice. Nejdříve vytvoříme zadávací prvky, dále tlačítko a na stisk tlačítka vypočítáme výsledek:

{% code title="main.py" lineNumbers="true" %}
```python
import math
import streamlit as st

st.header("Výpočet kvadratické rovnice")

st.text("Program pro výpočet kvadratické rovnice. Důležité vzorce:")
st.latex("a*x^2 + b*x + c = 0")
st.latex("d = b^2 - 4*a*c")
st.latex("x1,x2 = \\frac{-b \\pm \\sqrt(d)}{2*a}")

a = st.number_input("Zadejte A:")
b = st.number_input("Zadejte B:")
c = st.number_input("Zadejte C:")

btn_calculate = st.button("Vypočítat")
if btn_calculate:
    d = b * b - 4 * a * c
    if d < 0:
        st.text("Kvadratická rovnice nemá řešení v oboru reálných čísel.")
    elif d == 0:
        x = (-b) / (2 * a)
        st.text("Řešením je x = " + str(x))
    else:
        x1 = (-b - math.sqrt(d)) / (2 * a)
        x2 = (-b + math.sqrt(d)) / (2 * a)
        st.text("Řešením je x1 = " + str(x1) + ", x2 = " + str(x2))
```
{% endcode %}

Na řádcích 11-13 vytváříme vstupní textová pole a jejich hodnoty si ukládáme do proměnných a/b/c. Na řádku 15 vytváříme tlačítko. Pokud je tlačítko stisknuto (řádek 16), tak počítáme diskriminant (řádek 17). Pokud je menší než nula, vypíšeme zprávu, že řešení neexistuje (řádek 19). Pokud je diskriminant nula, vypočítáme a vypíšeme řešení (řádek 21, 22). Pro kladný diskriminant vypíšeme obě řešení (řádky 24-26). Všimněme si ve výpisu povinného převedení číselné hodnoty na řetězec přes funkci `str(...)` (řádky 22, 26).

## Příklad: Výpočet počtu dní života



