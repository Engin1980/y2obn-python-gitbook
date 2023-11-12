# Práce s obrazem



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
