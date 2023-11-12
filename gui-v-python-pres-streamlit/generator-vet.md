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

{% hint style="info" %}
Budeme potřebovat celou šířku stránky. Streamlit v základu generuje obsah pouze do úzkého sloupce na stránce. Pro vynucení využití celé šířky zadáváme příkaz dle řádku 5.
{% endhint %}

Poté vytvoříme spouštěcí soubor `start_sentence_generator.bat` s kódem:

{% code title="start_sentence_generator.bat" %}
```batch
.\venv\scripts\activate.bat & streamlit run camera.py
```
{% endcode %}

## Session-state - jak uchovávat data

Před samotnou tvorbou aplikace se musíme seznámit s principem práce se stránkou a uchovávání dat.

Při spuštění aplikace projde streamlit kód stránky (náš `sentence_generator.py`) a vygeneruje odpovídající kód, který se zobrazí v prohlížeči. V případě, že některý z prvků na stránce zjistí svou změnu (například se stiskne tlačítko), provede se obnovení stránky a při tomto obnovení se  znovu projde kód stránky (tedy opět soubor `sentence_generator.py`) a opět se vygeneruje kód stránky spolu s odpovídajícími změnami. Jak tedy pozná streamlit, že stránká prochází znovu a má zachovat data?

V stránce se mezi jednotlivými obnoveními udržuje tzv. slovník - dvojice hodnot "klíč-hodnota", která obsahuje informace o všech prvcích na stránce. Při prvním procházení je slovník prázdný, ale při akcí vyvolanou na stránce (například při stisku tlačítka "x") se do slovníku uloží informace o akci (např. u tlačítka se uloži dvojice "x=True") a při opětovném procházení stránky se již stránka kreslí s ohledem na hodnoty ve slovníku.

Hodnoty slovníku dostaneme přes příkaz `st.session_state`. Abychom nemuseli opakovat takto dlouhý příkaz, uložíme si slovník do proměnné `ss`:&#x20;

<pre class="language-python" data-line-numbers><code class="lang-python">ss = st.session_state
<strong>ss["refresh"] = datetime.datetime.now()
</strong><strong>if "initialized" not in ss:
</strong><strong>    ss["initialized"] = True
</strong><strong>st.write(st.session_state)
</strong><strong>
</strong><strong>st.button("Stiskni mne", key="btn")
</strong></code></pre>

Na řádku 1 vložíme hodnotu do proměnné `ss`. Na řádku 2 vložíme do slovníku pod klíč `refresh` hodnotu aktuálního datumu a času. Na řádku 3 se ptáme, zda již byl slovník inicializován - pokud ne, jedná se o první průchod stránkou. Pokud ano, inicializaci přeskočíme. Na řádku 4 vložíme informaci, že inicializace proběhla (později tento kód rozšíříme). Nakonec, na řádku 5 vypíšeme slovník do stránky (později můžeme tento příkaz zakomentovat, aby nám nekazil vzhled výsledné stránky).  Na řádku 7 přidáme do stránky tlačítko, kterému přiřadíme klíč "btn".

{% hint style="info" %}
Hodnotu `ss["refresh"]` jsme vložili jen jako informaci o obnovení stránky. Díky výpisu této hodnoty vidíme, kdy přesně proběhne obnovení stránky (které jinak může být díky rychlosti prohlížeče téměř nespatřitelné).
{% endhint %}

Uvidíme, že při prvním spuštění stránky se zobrazí:

```
{
"initialized":true
"refresh":"datetime.datetime(2023, 11, 12, 18, 6, 24, 397697)"
}
```

Pokud stiskneme tlačítko, zobrazí se:

```
{
"initialized":true
"refresh":"datetime.datetime(2023, 11, 12, 18, 6, 58, 89161)"
"btn":true
}
```

Po stisku tlačítka se tedy uložila informace o tlačítku - `"btn":true`.

{% hint style="info" %}
V příkladech dříve i později jsme používali/budeme používat prvky bez přiřazeného klíče `key`. V tom případě dostane prvek klíč vygenerovaný automaticky. Prvky s automatickým klíčem však neuvidíme ve výpisu získaného přes `st.write(st.session_state)`. Přesto však existují.

V tom případě typicky hodnotu daného prvku používáme přes proměnnou, do které jsme vytvořený prvek vložíli. Tedy například bez klíče provedeme:

```
btn = st.button("a")
if btn:
    ...
```

S klíčem bude podobný kód vypadat takto:

```
st.button("a", key="btn")
if st.session_state["btn"]:
    ...
```

Použití klíče umožňuje jednodušší práci s prvkem, pokud s ním potřebujeme dělat nějaké složitější operace - ukážeme si dále.
{% endhint %}

## Inicializace stránky

Nyní, když víme, jak se uchovávají data, můžeme provést inicializaci stránky. Budeme potřebovat tři listy - pro podmět, přísudek a předmět, vazbu "kdo - co - komu/čemu". Nazveme je `who`, `what` a `whom`.

```python
if "initialized" not in ss:
    ss["initialized"] = True
    ss["who"] = ["Pračka", "Jezevec"]
    ss["what"] = []
    ss["whom"] = []
```

Do první kolekce jsme vložili i nějaké výchozí hodnoty (můžete rozšířit později), aby hned bylo na stránce co zobrazovat.

## Vytvoření základního rozhraní

Nyní vytvoříme základní rozhraní aplikace. Budeme pořebovat tři sloupce (pro kdo-co - komu/čemu). Pod tím zobrazíme tlačítko na generování věty.

<pre class="language-python" data-title="sentence_generator.py" data-line-numbers><code class="lang-python">import streamlit as st
import datetime
import random

st.set_page_config(layout="wide")

# session state
ss = st.session_state
ss["refresh"] = datetime.datetime.now()
if "initialized" not in ss:
    ss["initialized"] = True
    ss["who"] = ["Pračka", "Jezevec"]
    ss["what"] = []
    ss["whom"] = []
st.write(st.session_state)

st.header("Sentence generator")

colA, colB, colC = st.columns(3)

with colA:
    st.subheader("Kdo")
<strong>    for word in ss["who"]:
</strong><strong>        st.text(word)
</strong><strong>
</strong><strong>with colB:
</strong><strong>    st.subheader("Co")
</strong><strong>
</strong><strong>with colC:
</strong><strong>    st.subheader("Komu/Čemu")
</strong><strong>
</strong><strong>btn_generate = st.button("G e n e r u j   v ě t u")
</strong></code></pre>

Na řádcích 7-15 se staráme o session-state (vysvětleno výše). Na řádku 17 vkládáme nadpis. Na řádku 19 vytváříme 3 sloupce. Do každého sloupce vložíme hlavičku (řádky 22, 27, 30). Do prvního sloupce navíc vložíme i výpis všech položek, které již jsou zadány - procházíme kolekci `ss["who"]` (řádek 23) a vypisujeme každé slovo (řádek 24). Nakonec vytvoříme tlačítko (které zatím nebude nic dělat) - řádek 32.

Následně si ukážeme vkládání slov - pro každý sloupec trošku jiným způsobem.

## První sloupec - vkládání slova

U prvního sloupce budeme chtít vytvořit vedle sebe textové pole a tlačítko pro vložení. Do kódu (ve výše uvedeném výpisu mezi řádky 22 a 23) vložíme opět vygenerování sloupců a jejich obsahu:

{% code lineNumbers="true" %}
```python
colAtxt, colAbtn = st.columns(2)
with colAtxt:
    who = st.text_input("Nové 'kdo'", label_visibility="collapsed")
with colAbtn:
    btn_who = st.button("Vlož")
    if btn_who:
        ss["who"].append(who)
```
{% endcode %}

Na řádku 1 vytvoříme 2 sloupce - txt a btn. V prvním bude textové pole, v druhém tlačítko. Do prvního tedy vložíme textové pole nazvané `who` (a potlačíme zobrazení jeho "labelu") - řádek 3. Do druhého sloupce vložíme tlačítko `btn_who` (řádek 5). Zároveň ihned na řádku 6 kontrolujeme, zda tlačítko někdo nestiskl (pokud by se nejednalo o první procházení kódu) - pokud stiskl, vezmeme hodnotu z `who` a vložíme ji do listu hodnot `ss["who"]` - řádek 7.

{% hint style="info" %}
Většina prvků v streamlit má u sebe připojen "label" - nápis, který říká, co je daný prvek zač nebo co s ním má uživatel provádět. My tento nápis nechceme zobrazit, proto jeho zobrazení potlačíme parametrem `label_visibility="collapsed"`. Možné hodnoty jsou visible/hidden/collapsed, můžete si vyzkoušet jejich chování.
{% endhint %}

Když nyní kód spustíme, zadáme do textového pole text a stiskneme tlačítko, obsah textového pole se přiřadí do seznamu hodnot.

Nevýhodou tohoto postup je, že:

* se stránka obnoví 2x - poprvé, když kurzor opustí textové pole, a podruhé, když stisknete tlačítko. Povšimněte si změny hodnot `ss["refresh"]`, pokud textové pole opustíte například stiskem klávesy Tab;
* se obsah textového pole nesmaže po vložení hodnoty.

Obě tyto nevýhody zkusíme vyřešit v dalším sloupci.

Kompletní aktuální kód `sentence_generator.py`:

{% code title="sentence_generator.py" lineNumbers="true" %}
```python
import streamlit as st
import datetime
import random

st.set_page_config(layout="wide")

# session state
ss = st.session_state
ss["refresh"] = datetime.datetime.now()
if "initialized" not in ss:
    ss["initialized"] = True
    ss["who"] = ["Pračka", "Jezevec"]
    ss["what"] = []
    ss["whom"] = []
st.write(st.session_state)

st.header("Sentence generator")

colA, colB, colC = st.columns(3)

with colA:
    st.subheader("Kdo")

    colAtxt, colAbtn = st.columns(2)
    with colAtxt:
        who = st.text_input("Nové 'kdo'", label_visibility="collapsed")
    with colAbtn:
        btn_who = st.button("Vlož")
        if btn_who:
            ss["who"].append(who)

    for word in ss["who"]:
        st.text(word)

with colB:
    st.subheader("Co")

with colC:
    st.subheader("Komu/Čemu")

btn_generate = st.button("G e n e r u j   v ě t u")
```
{% endcode %}

## Druhý sloupec - vkládání slova

V tomto sloupci přidáme smazání hodnoty textového pole po stisku tlačítka.

Jak jsme si řekli, pokud se změní hodnota textového pole, tato změna se projeví v `session_state`. Naším úkolem bude nyní podchytit tuto změnu nahoře na stránce - při překreslování stránky se podíváme, zda nebylo stisknuto tlačítko, a pokud ano, vezmeme si hodnotu z odpovídajícího textového pole, vložíme ji do seznamu slov a následně textovému poli smažeme obsah.

Nejdříve potřebujeme vytvořit textové pole a tlačítko:

{% code lineNumbers="true" %}
```python
with colB:
    st.subheader("Co")

    colBtxt, colBbtn = st.columns(2)
    with colBtxt:
        st.text_input("Nové 'kdo'", label_visibility="collapsed", key="txt_what")
    with colBbtn:
        btn_who = st.button("Vlož", key="btn_what")

    for word in ss["what"]:
        st.text(word)
```
{% endcode %}

Upravujeme sloupec B. Vložíme do něj opět dva pod-sloupce (řádek 4). Do prvního vložíme txtové pole (řádek 6), nevracíme si však jeho hodnotu (všimněte si, že tam není žádné "what=st.text\_input(...)"), nýbrž však textovému poli přiřadíme pevně klíč - `key="txt_what"`.  (Nemůžeme použít klíč "what", protože ten už používáme por kolekci.) Obdobně postupujeme u tlačítka - řádek 8. Nakonec nezapomeneme doplnit vypsání všech hodnot se seznamu "what".

Pokračovat budeme obsloužením stisku tlačítka. Tlačítko máme v klíču `btn_what`, proto nahoře (!) v kódu před vykreslením prvků provedeme obsloužení události tlačítka (pokud nastala) - kód vkládáme před vložením hlavičky `st.header(...)`:

{% code lineNumbers="true" %}
```python
if "btn_what" in ss and ss["btn_what"]:
    ss["what"].append(ss["txt_what"])
    ss["txt_what"] = ""
```
{% endcode %}

Řádek 1 zkontroluje, zda se v `session_state` nachází klíč tlačítka a zda je pravdivý - pokud ano, tlačítko bylo stisknuto. V tom případě přidáme text textového pole do listu slov (řádek 2) a textové pole vyprázdníme (řádek 3).

Aktuální výsledný kód:

{% code title="sentence_generator.py" lineNumbers="true" %}
```python
import streamlit as st
import datetime
import random

st.set_page_config(layout="wide")

# session state
ss = st.session_state
ss["refresh"] = datetime.datetime.now()
if "initialized" not in ss:
    ss["initialized"] = True
    ss["who"] = ["Pračka", "Jezevec"]
    ss["what"] = []
    ss["whom"] = []
st.write(st.session_state)

if "btn_what" in ss and ss["btn_what"]:
    ss["what"].append(ss["txt_what"])
    ss["txt_what"] = ""

st.header("Sentence generator")

colA, colB, colC = st.columns(3)

with colA:
    st.subheader("Kdo")

    colAtxt, colAbtn = st.columns(2)
    with colAtxt:
        who = st.text_input("Nové 'kdo'", label_visibility="collapsed")
    with colAbtn:
        btn_who = st.button("Vlož")
        if btn_who:
            ss["who"].append(who)

    for word in ss["who"]:
        st.text(word)

with colB:
    st.subheader("Co")

    colBtxt, colBbtn = st.columns(2)
    with colBtxt:
        st.text_input("Nové 'co'", label_visibility="collapsed", key="txt_what")
    with colBbtn:
        btn_who = st.button("Vlož", key="btn_what")

    for word in ss["what"]:
        st.text(word)


with colC:
    st.subheader("Komu/Čemu")

btn_generate = st.button("G e n e r u j   v ě t u")

```
{% endcode %}

## Třetí sloupec - vkládání slova

Pokud jste pozorně sledovali průběh obnovení stránky a chování textových polí, zjistili jste, že už při opuštění textového pole dojde k obnovení stránky, kdy se do session-state uloží klíč s hodnotou textového pole. Proto pro přidání položky nepotřebujeme tlačítko - stačí, když uživatel potvrdí (například Enterem) zadání hodnoty do textového pole.

Připravíme tedy třetí sloupec - nyní již velmi jednoduše:

{% code lineNumbers="true" %}
```python
with colC:
    st.subheader("Komu/Čemu")
    st.text_input("Nové 'komu/čemu'", label_visibility="collapsed", key="txt_whom")
    for word in ss["whom"]:
        st.text(word)
```
{% endcode %}

Na řádku 2 vložíme nadpis sloupce, na řádku 3 vytvoříme textové pole a na řádcích 4-5 vypisujeme obsah kolekce "whom".

Nyni přidáme obslužný kód - kdykoliv se změni (potvrdí) text textového pole, proběhne obnovení stránky. Při obnovení stránky (stejně jako v přechozím případě) zachytáváme změnu - tentokrát však pouze to, zda v textovém poli je nějaký smysluplný text (kód přidáváme opět těsně před hlavičku stránky `st.header`):

{% code lineNumbers="true" %}
```python
if "txt_whom" in ss and len(ss["txt_whom"]) > 0:
    ss["whom"].append(ss["txt_whom"])
    ss["txt_whom"] = ""
```
{% endcode %}

Na řádku 1 kontrolujeme, zda je v session-state klíč textového pole "txt\_whom" a zároveň, zda hodnota v tomto klíči je text delší než 0. Pokud ano, text vložíme do kolekce (řádek 2) a obsah textového pole smažeme (řádek 3).

Výsledný kód této části:

{% code title="sentence_generator.py" overflow="wrap" %}
```python
import streamlit as st
import datetime
import random

st.set_page_config(layout="wide")

# session state
ss = st.session_state
ss["refresh"] = datetime.datetime.now()
if "initialized" not in ss:
    ss["initialized"] = True
    ss["who"] = ["Pračka", "Jezevec"]
    ss["what"] = []
    ss["whom"] = []
st.write(st.session_state)

if "btn_what" in ss and ss["btn_what"]:
    ss["what"].append(ss["txt_what"])
    ss["txt_what"] = ""

if "txt_whom" in ss and len(ss["txt_whom"]) > 0:
    ss["whom"].append(ss["txt_whom"])
    ss["txt_whom"] = ""

st.header("Sentence generator")

colA, colB, colC = st.columns(3)

with colA:
    st.subheader("Kdo")

    colAtxt, colAbtn = st.columns(2)
    with colAtxt:
        who = st.text_input("Nové 'kdo'", label_visibility="collapsed")
    with colAbtn:
        btn_who = st.button("Vlož")
        if btn_who:
            ss["who"].append(who)

    for word in ss["who"]:
        st.text(word)

with colB:
    st.subheader("Co")

    colBtxt, colBbtn = st.columns(2)
    with colBtxt:
        st.text_input("Nové 'co'", label_visibility="collapsed", key="txt_what")
    with colBbtn:
        btn_who = st.button("Vlož", key="btn_what")

    for word in ss["what"]:
        st.text(word)


with colC:
    st.subheader("Komu/Čemu")
    st.text_input("Nové 'komu/čemu'", label_visibility="collapsed", key="txt_whom")
    for word in ss["whom"]:
        st.text(word)

btn_generate = st.button("G e n e r u j   v ě t u")

```
{% endcode %}

## Generování náhodné věty na stisk tlačítka

Poslední, nyní již jednoduchou částí bude generování věty na stisk tlačítka. Pod kód tlačítka přidáme jednoduché zachycení jeho stisku:

{% code lineNumbers="true" %}
```python
btn_generate = st.button("G e n e r u j   v ě t u")
if btn_generate:
    who_word = get_random_word(ss["who"])
    what_word = get_random_word(ss["what"])
    whom_word = get_random_word(ss["whom"])
    if who_word is None or what_word is None or whom_word is None:
        st.text("Nejdříve je třeba zadat slovíčka. Nelze generovat větu.")
    else:
        st.text(who_word + " " + what_word + " " + whom_word + ".")
```
{% endcode %}

Pokud bylo tlačítko stisknuto (řádek 2), z každého listu vygenerujeme jedno slovo (řádky 3,4,5). Pokud je některé ze slov prázdné (podmínka řádek 6), vypíšeme, že nelze větu vygenerovat. V opačném případě vypíšeme složenou větu (řádek 9).

Pro fungování kódu potřebujeme funkci `get_random_word(...)`, kterou používáme na řádcích 3,4,5. Tuto funkci vložíme úplně na  začátek kódu třídy, pod příkazy `import`:

{% code lineNumbers="true" %}
```python
def get_random_word(list):
    if len(list) == 0:
        ret = None
    else:
        index = random.randint(0, len(list) - 1)
        ret = list[index]
    return ret
```
{% endcode %}

Na řádku 1 definujeme funkci, která má jeden vstupní parametr - list slov (všimněte si jeho předávání v předchozím výpisu kódu). Nejdříve zkontrolujeme, zda je list prázdný (řádek 2). Pokud ano, budeme vracet prázdnou hodnotu `None` (řádek 3), Pokud je list neprázdný, vygenerujeme náhodný index - celé číslo v rozsahu 0 ... počet\_položek\_list - 1 (řádek 5). Následně budeme vracet toto slovo (řádek 6). Na řádku 7 vrátíme výsledek z funkce.

Tím je aplikace hotova. Výsledný kód:

{% code title="sentence_generator.py" lineNumbers="true" %}
```python
import streamlit as st
import datetime
import random


def get_random_word(list):
    if len(list) == 0:
        ret = None
    else:
        index = random.randint(0, len(list) - 1)
        ret = list[index]
    return ret


st.set_page_config(layout="wide")

# session state
ss = st.session_state
ss["refresh"] = datetime.datetime.now()
if "initialized" not in ss:
    ss["initialized"] = True
    ss["who"] = ["Pračka", "Jezevec"]
    ss["what"] = []
    ss["whom"] = []
st.write(st.session_state)

if "btn_what" in ss and ss["btn_what"]:
    ss["what"].append(ss["txt_what"])
    ss["txt_what"] = ""

if "txt_whom" in ss and len(ss["txt_whom"]) > 0:
    ss["whom"].append(ss["txt_whom"])
    ss["txt_whom"] = ""

# user interface
st.header("Sentence generator")

colA, colB, colC = st.columns(3)

with colA:
    st.subheader("Kdo")

    colAtxt, colAbtn = st.columns(2)
    with colAtxt:
        who = st.text_input("Nové 'kdo'", label_visibility="collapsed")
    with colAbtn:
        btn_who = st.button("Vlož")
        if btn_who:
            ss["who"].append(who)

    for word in ss["who"]:
        st.text(word)

with colB:
    st.subheader("Co")

    colBtxt, colBbtn = st.columns(2)
    with colBtxt:
        st.text_input("Nové 'co'", label_visibility="collapsed", key="txt_what")
    with colBbtn:
        btn_who = st.button("Vlož", key="btn_what")

    for word in ss["what"]:
        st.text(word)

with colC:
    st.subheader("Komu/Čemu")
    st.text_input("Nové 'komu/čemu'", label_visibility="collapsed", key="txt_whom")
    for word in ss["whom"]:
        st.text(word)

btn_generate = st.button("G e n e r u j   v ě t u")
if btn_generate:
    who_word = get_random_word(ss["who"])
    what_word = get_random_word(ss["what"])
    whom_word = get_random_word(ss["whom"])
    if who_word is None or what_word is None or whom_word is None:
        st.text("Nejdříve je třeba zadat slovíčka. Nelze generovat větu.")
    else:
        st.text(who_word + " " + what_word + " " + whom_word + ".")
```
{% endcode %}
