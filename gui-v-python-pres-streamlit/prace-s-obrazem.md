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

Dále umístíme kód zobrazující nadpis na stránce a jeden prvek `placeholder`, který slouží jako zástupný symbol na stránce. Je to místo, které je aktuálně prázdné, v budoucnu však do něj můžeme vkládat libovolné prvky (my do něj budeme dávat postupně obrazové snímky z kamery - vždy nový snímek nahradí předchozí snímek).

```python
import streamlit as st
import cv2
import numpy as np

st.header("Camera fun")

hld_image = st.eompty()
```



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
