---
title: Web-Frontends für Daten-Apps schnell mit Streamlit bauen
author: htrinter
categories: [Quick Tips]
tags: [python, frontend]
shortdesc: Erstelle rasch Datenvisualisierungen mit minimalem Code.
---

Streamlit ist eine hervorragende Wahl für alle, die schnell und unkompliziert Datenvisualisierungen erstellen möchten, selbst wenn sie nur wenig Erfahrung mit der Programmierung von Web-Frontends haben. Es eignet sich ideal für Prototyping und schnelle Visualisierungen, da es den Aufwand reduziert und ermöglicht, sich auf das Wesentliche zu konzentrieren – die Daten.

## Was kann man damit machen

Die [Streamlit App Gallery](https://streamlit.io/gallery), vermittelt einen guten Eindruck, wie vielseitig Streamlit einsetzbar ist. Dort findest du eine Vielzahl von Community-Beispielen samt Source Code, die als Inspiration für eigene Projekte dienen können.

## Components

Streamlit bietet ein modulares System an, mit dem sich Apps mit bestehenden Third-Party Components erweitern lassen. Siehe: <https://streamlit.io/components>.

Diese Komponenten, die unter anderem von namhaften Contributoren wie Facebook stammen, sind gerankt, sodass deren Qualität leicht einzuschätzen ist.

## Einstieg

**Eine erste App erstellen:**

Ein guten und schnellen Start bietet das [Streamlit-Tutorial](https://docs.streamlit.io/get-started/tutorials/create-an-app). In wenigen Zeilen Code kannst du eine Mini-App schreiben, die Daten abruft, cached und auf einer Karte als Chart darstellt. Desweiteren bietet sie UI-Elemente wie Slider zum Filtern der Daten und einen Button zum Ein- und Ausblenden der Rohdaten. Am Ende erfährst du, wie du deine App kostenlos auf <https://share.streamlit.io> veröffentlichen kannst.

Hier der komplette Quellcode:

```python
import streamlit as st
import pandas as pd
import numpy as np

st.title('Uber pickups in NYC')

DATE_COLUMN = 'date/time'
DATA_URL = ('https://s3-us-west-2.amazonaws.com/'
            'streamlit-demo-data/uber-raw-data-sep14.csv.gz')

@st.cache_data
def load_data(nrows):
    data = pd.read_csv(DATA_URL, nrows=nrows)
    lowercase = lambda x: str(x).lower()
    data.rename(lowercase, axis='columns', inplace=True)
    data[DATE_COLUMN] = pd.to_datetime(data[DATE_COLUMN])
    return data

data_load_state = st.text('Loading data...')
data = load_data(10000)
data_load_state.text("Done! (using st.cache_data)")

if st.checkbox('Show raw data'):
    st.subheader('Raw data')
    st.write(data)

st.subheader('Number of pickups by hour')
hist_values = np.histogram(data[DATE_COLUMN].dt.hour, bins=24, range=(0,24))[0]
st.bar_chart(hist_values)

# Some number in the range 0-23
hour_to_filter = st.slider('hour', 0, 23, 17)
filtered_data = data[data[DATE_COLUMN].dt.hour == hour_to_filter]

st.subheader('Map of all pickups at %s:00' % hour_to_filter)
st.map(filtered_data)
```

**30 Days of Streamlit:**

Für einen tieferen Einstieg eignet sich das interaktive Tutorial "[30 Days of Streamlit](https://30days.streamlit.app/)", das in 30 täglichen Häppchen umfassendes Wissen vermittelt.

## Fazit

Streamlit ist ein großartiges Tool mit [umfassender Dokumentation](https://docs.streamlit.io/), und es lohnt sich definitiv, einen Blick darauf zu werfen. Es bietet Entwickelnden eine effiziente Möglichkeit, Daten schnell und ansprechend zu präsentieren.
