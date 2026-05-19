---
trigger: always_on
description: The widget is a Streamlit component.
---

# yFiles Graphs for Streamlit (Python-first coding guide)

The widget is a Streamlit component.

## 1) Installation
```bash
pip install yfiles_graphs_for_streamlit
```

## 2) Minimal working example
```python
import streamlit as st
from yfiles_graphs_for_streamlit import StreamlitGraphWidget, Layout

st.set_page_config(page_title="yFiles Graphs for Streamlit", layout="wide")

nodes = [
    {"id": 0, "properties": {"firstName": "Alpha", "label": "Person A"}},
    {"id": 1, "properties": {"firstName": "Bravo", "label": "Person B"}},
    {"id": 2, "properties": {"firstName": "Charlie", "label": "Person C", "has_hat": False}},
    {"id": 3, "properties": {"firstName": "Delta", "label": "Person D", "likes_pizza": True}}
]
edges = [
    {"id": 0, "start": 0, "end": 1, "properties": {"since": "1992", "label": "knows"}},
    {"id": 1, "start": 1, "end": 3, "properties": {"label": "knows", "since": "1992"}},
    {"id": 2, "start": 2, "end": 3, "properties": {"label": "knows", "since": "1992"}},
    {"id": 3, "start": 0, "end": 2, "properties": {"label": "knows", "since": 234}}
]

graph = StreamlitGraphWidget(nodes, edges)

# Shows the interactive graph component
graph.show()

# Return tuple (selected_nodes, selected_edges) only when sync_selection=True
selected_nodes, selected_edges = graph.show(
    sync_selection=True,
    graph_layout=Layout.HIERARCHIC
)
st.write("Second widget (edges):", ", ".join(str(e["id"]) for e in selected_edges))
st.write("Second widget (nodes):", ", ".join(str(n["id"]) for n in selected_nodes))
```

## 3) Data model you pass in

- **Nodes:** list of dicts. Each node **must** have `id`. Optional `properties` dict for arbitrary data.
- **Edges:** list of dicts. Each edge **must** have `id`, `start`, `end` referencing node `id`s. Optional `properties` dict.

> The library uses your original dicts; ids may be numbers or strings. Properties can be nested.

## 4) Constructors

```python
from yfiles_graphs_for_streamlit import StreamlitGraphWidget

# Provide nodes/edges directly
widget = StreamlitGraphWidget(nodes, edges)

# Import from other graph formats
widget = StreamlitGraphWidget.from_graph(g)   # supports neo4j, graph_tool, networkx, pygraphviz, pandas
```

**NetworkX example**
```python
from yfiles_graphs_for_streamlit import StreamlitGraphWidget
from networkx import erdos_renyi_graph

g = erdos_renyi_graph(10, 0.3, seed=2)
widget = StreamlitGraphWidget.from_graph(g)
widget.show()
```

## 5) Rendering the component

```python
from yfiles_graphs_for_streamlit import Layout

nodes_sel, edges_sel = widget.show(
    directed=True,                              # default True
    graph_layout=Layout.ORGANIC,                # default Layout.ORGANIC
    sync_selection=False,                       # default False
    sidebar={"enabled": False},                 # or {"enabled": True, "start_with": "Neighborhood"|"Data"|"Search"|"About"}
    neighborhood={"max_distance": 1, "selected_nodes": []},
    overview=True,                              # default True
    key="graph-component"                       # default None
)
```

### Return value of `show()`
- `sync_selection=False` → returns `None`.
- `sync_selection=True`  → returns a **tuple** `(selected_nodes, selected_edges)`; each item is a `List[Dict]` from your original data.

### Preserving state across reruns
It is recommended to define a specific `key` to preserve the widget's state across reruns. For example, when the passed 
data is changed interactively, then the component only updates its graph without rebuilding the entire component.

## 6) Data‑driven visualization mappings

Each setter takes a **callable** that receives your original item and returns the specified type.

Each of the following mapping function can also be set as property on the widget. However, due to better type hints, keyword arguments are preferred for defining data mappings.

### Property mappings (what downstream mappings “see”)
```python
widget = StreamlitGraphWidget(
    nodes,
    edges,
    node_property_mapping=lambda node: node.get("properties", {}),
    edge_property_mapping=lambda edge: edge.get("properties", {}),
)
# By default, the original dict is returned.
```

### Label mappings
```python
from yfiles_graphs_for_streamlit import LabelStyle, FontWeight, LabelPosition, TextWrapping, TextAlignment

widget = StreamlitGraphWidget(
    nodes,
    edges,
    # Option A: specify a string (resolved first against properties, otherwise used verbatim)
    node_label_mapping="label",
    # Option B: set a lambda, return a LabelStyle
    edge_label_mapping=lambda edge: LabelStyle(
        text=edge["properties"]["label"],
        font_size=12,
        font_weight=FontWeight.BOLD,
        color="#222",
        background_color="#eef",
        position=LabelPosition.NORTH,
        maximum_width=160,
        wrapping=TextWrapping.WORD,
        text_alignment=TextAlignment.CENTER,
    ),
)
```

### Color mappings (CSS color strings)
```python
widget = StreamlitGraphWidget(
    nodes,
    edges,
    node_color_mapping=lambda node: "#4CAF50",
    edge_color_mapping=lambda edge: "rgb(120,120,120)",
)
```

### Item visualization mappings
```python
from yfiles_graphs_for_streamlit import NodeStyle, EdgeStyle, NodeShape, DashStyle

widget = StreamlitGraphWidget(
    nodes,
    edges,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yWorks/yfiles-graphs-for-streamlit](https://github.com/yWorks/yfiles-graphs-for-streamlit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
