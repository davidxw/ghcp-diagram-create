---
name: diagrams-as-code
user-invocable: false
disable-model-invocation: true
description: Use only when creating architecture diagrams with Python's `diagrams` library (mingrammer/diagrams).
---

# Diagrams as Code

## Repository comparison mode (takes precedence)

Adapted from sammcj/agentic-coding (Apache-2.0); see
[source attribution](../SOURCES.md). Read and obey
[the comparison contract](../COMPARISON.md). The upstream installation commands,
`uv` dependency resolution, current-directory output, and custom-icon fetching
are not permitted in comparison mode.

1. Use `run_dir/diagrams-python/`. Require an existing Python interpreter,
   an importable `diagrams` package, and Graphviz's `dot` on PATH. Check with
   `python -c "import diagrams"` and `dot -V` (use `python3` if that is the
   installed interpreter). Missing prerequisites are `blocked`; do not install.
2. Independently author `diagram.py` using the API and bundled
   [providers reference](./references/providers.md). Use only entities and
   relationships explicitly present in the prompt. Never default an unspecified
   provider to AWS/Azure/GCP. Use `diagrams.generic.blank.Blank` with the exact
   supplied labels for provider-neutral nodes.
3. Set `show=False`, `outformat="svg"`, and `filename` to the absolute path
   `<run_dir>/diagrams-python/diagram` without an extension. Set the process
   working directory to that backend directory so incidental files stay there.
   Do not download custom icons or use PEP 723 to auto-install dependencies.
4. Run that interpreter on `diagram.py` once. Require a zero exit code and a
   nonempty, XML-parseable `diagram.svg`. Unsupported types/features are
   `blocked`; execution or serialization errors are `failed`. Never substitute
   Graphviz-only output or another backend, fix imports, or rerun.
5. Record `diagram.py`, `diagram.svg`, and `status.json` under the contract.

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
