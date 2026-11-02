---
name: diagrams-as-code
user-invocable: false
disable-model-invocation: true
description: Use only when creating architecture diagrams with Python's `diagrams` library (mingrammer/diagrams).
---

# Diagrams as Code

## Repository workflow

Adapted from sammcj/agentic-coding (Apache-2.0); see
[source attribution](../SOURCES.md). Follow the normal technical workflow below
within [the shared scope and bookkeeping contract](../COMPARISON.md).

1. Work in `run_dir/diagrams-python/`. Select an existing Python environment
   containing `diagrams` and a usable Graphviz `dot`. Check that environment with
   `python -c "import diagrams"` and `dot -V` (or its interpreter's actual name).
   An existing Graphviz installation may be added to this process's PATH;
   do not persist machine changes. Missing prerequisites are `blocked`.
2. Independently author `diagram.py` using the normal API, provider libraries,
   and bundled [providers reference](./references/providers.md). Use suitable
   provider icons only when the input identifies that provider/product; otherwise
   use generic nodes/shapes without implying AWS/Azure/GCP or another platform.
   Clusters, Graphviz layout attributes, edge styling, and custom local/downloaded
   icons are allowed when they improve the requested diagram.
3. Set `show=False` and `filename` to the absolute path
   `<run_dir>/diagrams-python/diagram` without an extension. Choose the requested
   `outformat`; otherwise use the normal PNG default, or SVG when better suited
   to the diagram. Keep downloaded assets inside this backend directory.
4. Run the script, inspect its rendered output with an available viewer, and fix
   concrete import, layout, labeling, or routing problems before rerunning.
   Verify successful execution, nonempty output, and format validity (parse SVG
   as XML; inspect raster output with a suitable image viewer).
5. Preserve the Python source and successful final image/export files and record
   `status.json`. PEP 723 metadata is allowed for reproducibility, but automatic
   `uv`/pip installation from upstream examples is not: use preinstalled
   dependencies under the shared setup policy. Report unresolved unsupported
   diagram types or missing tools instead of using a different backend.

## Upstream technical guidance

Generate architecture and infrastructure diagrams using the Python `diagrams` library. Diagrams are Python scripts that render to image files via Graphviz.

## Workflow

1. **Check prerequisites** - verify graphviz and diagrams are installed
2. **Write the diagram script** to a `.py` file in the user's working directory
3. **Run it** with `python3 <file>.py`
4. **Report the output path** to the user (image appears in working directory)

### Prerequisites check

```bash
command -v dot >/dev/null || brew install graphviz
uv pip install --system diagrams 2>/dev/null || pip install diagrams
```

Without graphviz, the script fails with a cryptic error. Always verify first.

## Core API

### Diagram context manager

```python
from diagrams import Diagram

with Diagram(
    name="Title",           # used as default filename (slugified)
    filename="output",      # override filename (no extension)
    direction="LR",         # LR | RL | TB | BT
    curvestyle="spline",    # spline | ortho | curved | polyline
    outformat="png",        # png | jpg | svg | pdf | dot (or list)
    show=False,             # ALWAYS False - prevents auto-opening
    strict=False,           # merge duplicate edges
    autolabel=False,        # prefix labels with provider/type
    graph_attr={},          # graphviz graph attributes
    node_attr={},           # graphviz node attributes
    edge_attr={},           # graphviz edge attributes
):
    ...
```

Always set `show=False` so the script runs non-interactively.

### Nodes

Import from `diagrams.<provider>.<category>`:

```python
from diagrams.aws.compute import EC2, Lambda, ECS
from diagrams.aws.database import RDS, Aurora
from diagrams.aws.network import ELB, Route53, CloudFront
from diagrams.k8s.compute import Pod, Deployment
from diagrams.onprem.database import PostgreSQL, MySQL
from diagrams.onprem.network import Nginx
```

See `references/providers.md` for the full provider and category listing with common node names.

### Data flow operators

```python
ELB("lb") >> EC2("web") >> RDS("db")    # left to right
RDS("db") << EC2("web")                  # right to left
RDS("primary") - RDS("replica")           # undirected
ELB("lb") >> [EC2("w1"), EC2("w2")]      # fan-out to list
```

Operator precedence matters: `-` (subtraction) binds more tightly than `>>` / `<<` (bitshift) in Python, so `A >> B - C` parses as `A >> (B - C)`. Wrap in parentheses when mixing: `(A >> B) - C`.

### Clusters

```python
from diagrams import Cluster

with Cluster("VPC"):
    with Cluster("Private Subnet"):
        svc = [ECS("svc1"), ECS("svc2")]
```

Unlimited nesting depth. Clusters accept `graph_attr` for styling (e.g. background colour).

### Edges

```python
from diagrams import Edge

node_a >> Edge(label="HTTPS", color="darkgreen", style="dashed") >> node_b
```

Attributes: `label` (text on the edge), `color` (X11 name or hex), `style` (solid | dashed | dotted | bold).

### Custom nodes

```python
from diagrams.custom import Custom

svc = Custom("My Service", "./icon.png")  # local PNG, ideally 256x256
```

## Key Techniques

- **Direction**: use `TB` for layered/hierarchical diagrams, `LR` for pipelines and flows
- **Transparent background**: `graph_attr={"bgcolor": "transparent"}`
- **Multiple outputs**: `outformat=["png", "svg"]`
- **Reduce edge noise**: import `Node` from `diagrams` and create blank junctions with `Node("", shape="plaintext", width="0", height="0")`, or merge overlapping edges with `graph_attr={"concentrate": "true", "splines": "spline"}`
- **Assign nodes to variables** when they connect to multiple other nodes
- **PEP 723** for standalone scripts so `uv run` handles the dependency automatically:
  ```python
  # /// script
  # dependencies = ["diagrams"]
  # ///
  ```
