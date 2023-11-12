---
description: >-
  Cílem tohoto úkolu je vytvořit aplikaci, která bude snímat obraz z kamery
  (nebo z MP4 souboru) a zobrazovat jej. Následně lze obraz zesvětlit/ztmavit,
  případně zapnout detekci hran.
---

# Práce s obrazem



## Vytvoření projektu

Aplikaci budeme vytvářet do souboru `camera.py`. Vytvoříme ve naší složce nový soubor `camera.py`.&#x20;

Do souboru vložíme opět základní text:

{% code title="camera.py" lineNumbers="true" %}
```python
import streamlit as st

st.text("Ahoj")
```
{% endcode %}

Dále ve složce projektu vytvoříme spouštěcí soubor `start_camera.bat`  a do něj vložíme kód:

{% code title="start_camera.bat" %}
```batch
.\venv\scripts\activate.bat & streamlit run camera.py
```
{% endcode %}

Nyní soubor `start_camera.bat` spustíme. Mělo by se opět otevřít okno prohlížeče s nápisem "Ahoj".

## Příprava obsahu stránky

Nyní už budeme pracovat pouze se souborem `camera.py`, kam budeme vkládat kód naší aplikace.

Do projektu připojíme knihovny, které budeme používat:

* cv2 pro práci s kamerou/videem a operace s obrazem,
* numpy (jako  zkratku `np`) pro práci s maticemi

Dále umístíme kód zobrazující nadpis na stránce a jeden prvek `empty`, který slouží jako zástupný symbol na stránce. Je to místo, které je aktuálně prázdné, v budoucnu však do něj můžeme vkládat libovolné prvky (my do něj budeme dávat postupně obrazové snímky z kamery - vždy nový snímek nahradí předchozí snímek). Abychom mohli obsah prvku nahrazovat, potřebujeme si na něj odkaz uložit do proměné `hld_image`.

```python
import streamlit as st
import cv2
import numpy as np

st.header("Camera demo")

hld_image = st.empty()
```

Po obnovení stránky se její obsah nezmění (nepřibyl žádný vizuálně reprezentovatelný prvek).&#x20;

## Vyčítání snímků z kamery/videa

Do stránky nyní vložíme vyčítání dat z kamery:

<pre class="language-python" data-title="camera.py - doplnění" data-line-numbers><code class="lang-python"># ... předchozí kód

<strong># cap = cv2.VideoCapture(0) # vyčítání z kamery PC/NB
</strong>cap = cv2.VideoCapture("C:\\Users\\Vajgl\\Downloads\\Mlha1.mp4") # vyčítání z videa
<strong>
</strong><strong>while True:
</strong><strong>    ret, frame = cap.read()
</strong><strong>    if ret is False:
</strong><strong>        cap.release()
</strong>        st.text("Chyba nebo konec videa.")
        break

<strong>    hld_image.image(frame)
</strong></code></pre>

&#x20;Vytvoříme proměnnou `cap` (řádek 3/4), která načítá video zdroj  - řádek 3 ukazuje, jak otevří t připojení ke kameře na NB/PC, řádek 4 definuje jako zdroj video MP4

Následně v "nekonečné smyčce" (řádek 6) provádíme vyčtení snímku z kamery (řádek 7). Pokud se snímek nepodařilo načíst (proměnná `ret` je `False`, řádek 8), uvolníme připojení ke kameře, vypíšeme na stránku chybu a smyčku ukončíme. Pokud se snímek načíst povedlo, pokračujeme řádkem 13, kde vložíme snímek `frame` jako obrázek do dříve vytvořeného zástupného prvku.

Pokud nyní kód spustíme, uvidíme postupně zobrazované video.

Video však bude mít zvláštní barvy. Je to proto, že knihovna "cv2" vyčítá snímky ve formátu BGR (modrá-zelená-červená), ale zobrazování dat pracuje v klasickém modelu RGB (červená-zelená-modrá).  Proto musíme ještě před zobrazení vložit řádek provádějící převod z jednoho barevného modelu na druhý:

{% code title="camera.py - doplnění" lineNumbers="true" %}
```python
# ... předchozí kód

    frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
    hld_image.image(frame)
```
{% endcode %}

Výsledný kód souboru tedy bude:

{% code title="camera.py - úplný kód" lineNumbers="true" %}
```python
import streamlit as st
import cv2
import numpy as np

st.header("Camera demo")

hld_image = st.empty()


#cap = cv2.VideoCapture(0)
cap = cv2.VideoCapture("C:\\Users\\Vajgl\\Downloads\\Mlha1.mp4")

while True:
    ret, frame = cap.read()
    if ret is False:
        cap.release()
        st.text("Chyba nebo konec videa.")
        break

    frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
    hld_image.image(frame)
```
{% endcode %}

## Zesvětlení/ztmavení videa

Pro úpravu světlosti videa nejdříve vložíme do kódu vhodné ovládací prvky - do řádku 6 v předchozím výpisu. Následně, na základě volby uživatele provedeme požadované zesvětlení/ztmavení videa - do řádku 19 předchozího výpisu.

Ovládací prvky budou 2 - zaškrtávací pole a posunovátko:

<pre class="language-python" data-title="camera.py - vložený kód" data-line-numbers><code class="lang-python"><strong># ...
</strong>
chk_bright = st.checkbox("Úprava světlosti")
val_bright = st.slider("",
                       min_value=0, max_value=200, value=100,
                       label_visibility="collapsed")
hld_image = st.empty() # původní kód
</code></pre>

Zaškrtáváto pojmenujeme `chk_bright` - řádek 3 - a dáme mu odpovídající text k zobrazení. Posunovátko pojmenujeme `val_bright` - řádek 4, nastavíme mu minimum, maximum, výchozí hodnotu a nakonec stanovíme, že jeho text nechceme zobrazit (text je jako první parametr; vzhledem k tomu, že nad ním je přímo zaškrtávátko, zobrazení druhého nápisu by působilo duplicitně).

Dalším krokem bude přidání odpovídajícího kódu do videa. Pokud bude zaškrtávátko zaškrtnuté, budeme chtít provést zesvětlení videa:

{% code title="camera.py - vložený kód" lineNumbers="true" %}
```python
if chk_bright:
    multiplier = val_bright / 100.0
    frame = frame * multiplier
    frame[frame > 255] = 255
    frame = frame.astype(np.uint8)
```
{% endcode %}

Kód je trošku složitější. Pokud je  zaškrtávátko zaškrtnuté (řádek 1), převedeme hodnotu % z `val_bright`do násobící konstanty (rozsah 0.0-2.0). Konstantou vynásobíme obraz (čádek 3). Obraz očekává hodnoty v rozsahu 0-255; pokud bychom při násobení přešli tuto hranici, řádek 4 zajistí, že všechny hodnoty v matici `frame` přesahující hodnotu 255 se nastaví na hodnotu 255. Nakonec převedeme obraz zpátky do formátu celých čísel (unsigned 8bit integer - uint8).

{% hint style="info" %}
Zesvětlení obrazu se správným způsobem provádí úpravou světlosti v barevném modelu HSV/HSL. My však provádíme pro názornost jen triviální pronásobení v modelu RGB. Pro zájemce o správné řešení například viz [https://stackoverflow.com/questions/32609098/how-to-fast-change-image-brightness-with-python-opencv](https://stackoverflow.com/questions/32609098/how-to-fast-change-image-brightness-with-python-opencv).
{% endhint %}

{% hint style="info" %}
Proč provádíme převod typu na `uint8`? Na začátku (řádek 1 např) je matice `frame` ve formátu celých čísel - uint8. Protože ale na řádku 3 provádíme násobení mezi typy uint8 a float, výsledkem operace je datový typ float. Proto na řádku 5 musíme převést hodnoty v `frame` zpátky do `uint8`.

Pokud bychom to neprovedli, při zobrazování by si streamlit myslel, že se snažíme zobrazit maitic desetinných čísel a předpokládal bych rozsah jako 0.0-1.0 namísto 0-255.
{% endhint %}

Výsledný kód po úpravách:

{% code title="camera.py" lineNumbers="true" %}
```python
import streamlit as st
import cv2
import numpy as np

st.header("Camera demo")

chk_bright = st.checkbox("Úprava světlosti")
val_bright = st.slider("",
                       min_value=0, max_value=200, value=100,
                       label_visibility="collapsed")
hld_image = st.empty()

# cap = cv2.VideoCapture(0)
cap = cv2.VideoCapture("C:\\Users\\Vajgl\\Downloads\\Mlha1.mp4")

while True:
    ret, frame = cap.read()
    if ret is False:
        cap.release()
        st.text("Chyba nebo konec videa.")
        break

    if chk_bright:
        multiplier = val_bright / 100.0
        frame = frame * multiplier
        frame[frame > 255] = 255
        frame = frame.astype(np.uint8)

    frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
    hld_image.image(frame)
```
{% endcode %}



Následně v nekonečné smyčce vyčítáme jednotlivé snímky videa a zobrazujeme je jako obrázky v prvku `hld_image`:

```python
import datetime
import random

import streamlit as st
import datetime
import random
import cv2
import numpy as np

st.set_page_config(layout="wide")

st.header("Camera fun")

ss = st.session_state
ss["refresh"] = datetime.datetime.now()
if "initialized" not in ss:
    ss["initialized"] = True

# st.write(st.session_state)
st.write(ss["refresh"])

_, col_image, _ = st.columns((1, 2, 1))

# Simple Image Show From Camera
# with col_image:
#     st.text("du")
#     image_buffer = st.camera_input("Take a picture")
# if image_buffer is not None:
#     st.image(image_buffer, use_column_width="always")
# image_bytes = image_buffer.getvalue()
# image = cv2.imdecode(np.frombuffer(image_bytes, np.uint8))


#cap = cv2.VideoCapture(0)
cap = cv2.VideoCapture("C:\\Users\\Vajgl\\Downloads\\Mlha1.mp4")

with col_image:
    st.checkbox("Brightness enabled", value=False, key="chkBrightness")
    st.slider("Brightness %", min_value=0, max_value=200, value=100, key="sldBrightness")
    st.checkbox("Detect edges", value=False, key="chkEdges")

    hld_image = st.empty()

while True:
    ret, frame = cap.read()
    if not ret:
        cap.release()
        st.text("Camera released")
    frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)

    if ss["chkBrightness"]:
        multiplier = ss["sldBrightness"] / 100.0
        frame = frame * multiplier
        frame[frame > 255] = 255
        frame = frame.astype(np.uint8)

    if ss["chkEdges"]:
        frame = cv2.Canny(image=frame, threshold1=10, threshold2=50)
        #frame = cv2.Sobel(src=frame, ddepth=cv2.CV_64F, dx=1, dy=1, ksize=15)

    hld_image.image(frame, clamp=True)

```
