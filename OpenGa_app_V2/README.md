# OpenGa app V2

The complete V1 app with a **Flowsheet** tab connected to the live model.
Use this folder as your app project. No manual flowsheet integration is needed.

## Run locally

Open a terminal in this folder:

```sh
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
python -m streamlit run app/streamlit_app.py
```

Python 3.11+ is required by the dependencies. V2 was tested with Python 3.13
and the exact versions in `requirements.txt`. On Windows, activate the virtual
environment with `.venv/Scripts/activate` instead.

Open **Flowsheet**, next to **Mass & energy**. **Image** displays a scalable SVG.
**Interactive** retains card tooltips and horizontal scrolling. Both views
update from current model results and support total mass, Ga, Al and V,
light/dark appearance and SVG download.

Select **User** in the sidebar to edit the Inputs tab. Select the **Custom**
resin preset before changing its ion-exchange performance parameters; named
presets intentionally override those parameters. Changes update the model and
image in the same rerun.

## Use V2 for your existing deployment

Replace your repository's app files with the **contents of this folder**,
including the new `openga_flowsheet/` directory and
`openga_v1/stream_metadata.json`. Preserve `.streamlit/config.toml`.
The main file path remains **`app/streamlit_app.py`**.
See [DEPLOY.md](DEPLOY.md) for the steps.

The backend is still named `openga_v1` for import compatibility; this is expected
inside V2. The MEB, equipment, energy, cost, finance and carbon equations are
unchanged. The flowsheet is a reporting extension, not a replacement MEB.

## New code and fixes

| File | Purpose |
|---|---|
| `openga_v1/streams.py` | Builds 37 live stream records from V1's MEB |
| `openga_v1/stream_metadata.json` | Descriptions and routing; no snapshot quantities |
| `openga_v1/engine.py` | Exposes the register as `Results.streams` |
| `openga_flowsheet/flowsheet.py` | Generates SVG and interactive HTML |
| `openga_flowsheet/app_flowsheet_tab.py` | Image display, selectors and download |
| `app/streamlit_app.py` | Includes the new Flowsheet tab |

Custom resin inputs are now populated from V1’s existing baseline preset, so Custom mode can run and accept edits. Input edits now apply before the model runs. Repeated IX recovery controls
have distinct keys, stay synchronised, and reset correctly. Workbook regression
is compared only at its applicable baseline; modified scenarios are no longer
incorrectly marked as baseline regression failures.

The integration already included in the app is:

```python
from openga_v1 import engine
from openga_flowsheet.app_flowsheet_tab import flowsheet_tab

results = engine.run(model_inputs)
flowsheet_tab(results.streams, display_mode="image")
```

For just the image without selectors or a table:

```python
from openga_flowsheet import render_svg
st.image(render_svg(results.streams, interactive=False), width=1440)
```

## Checks

```sh
python selftest.py
python -m unittest discover -s tests -v
```

The self-test includes the original 14 internal and 10 workbook checks plus
52 stream balance checks. Integration tests cover scenario changes, stream
conservation, live image updates, reset, repeated controls and display modes.

Read [FLOW_MODEL_NOTES.md](FLOW_MODEL_NOTES.md) for the reporting assumptions
and [MODEL_NOTES_V1.md](MODEL_NOTES_V1.md) for the original model documentation.
