
# S15: The "Manual Graph" Benchmark

## Why This Exists (Educational Goal)
Before we let `NetworkX` automate our graph operations in S16, we must understand **what it actually takes** to manage a plan graph manually. This "V15 Manual" codebase serves as the "Control Group" or "Life Before Libraries" benchmark.

It demonstrates the **complexity** of state management that libraries usually hide from us.

## Architectural Changes in V15 (Manual)

We took the standard Agentic Framework and **removed all graph libraries**.

| Component | V15 Manual Implementation | S16 (NetworkX) Equivalent |
| :--- | :--- | :--- |
| **Graph Storage** | `self.steps: Dict[str, StepNode]` <br> `self.edges: List[Dict]` | `self.graph = nx.DiGraph()` |
| **Finding Children** | **Recursive DFS:** <br> `_get_descendants(node_id)` manually traverses edges. | `nx.descendants(G, node_id)` |
| **Subtree Operations** | **Manual Rename:** <br> Iterating keys, creating new nodes, updating edge targets, deleting old nodes. | `nx.relabel_nodes(G, mapping)` |
| **Serialization** | **Custom JSON Builder:** <br> Manually constructing `nodes` and `links` arrays. | `nx.node_link_data(G)` |

### The "Manual" Code Reality
In `S15_Manual/agent/contextManager.py`, simple operations become verbose algorithms.

**Example: Finding Dependent Steps (Descendants)**
* **NetworkX**: `dependents = list(nx.descendants(G, 'step_1'))`
* **Manual V15**:
```python
def _get_descendants(self, node_id, visited=None):
    if visited is None: visited = set()
    children = [e['target'] for e in self.edges if e['source'] == node_id]
    for child in children:
        if child not in visited:
            visited.add(child)
            self._get_descendants(child, visited)
    return visited
```

**Example: Renaming a Subtree (e.g. for Retry `1F1`)**
* **NetworkX**: `nx.relabel_nodes(G, mapping)` handles all edges automatically.
* **Manual V15**: We must:
    1. Collect all nodes in the subtree.
    2. Create new nodes with new IDs.
    3. Copy data from old to new.
    4. **Find and redirect** all incoming edges to the new root.
    5. **Find and redirect** all internal edges to new targets.
    6. Delete old nodes.

## The Logic Flow (V15 Manual)

1.  **Perception**: Analyzes the request, sees `ROOT`.
2.  **Decision**:
    *   Generates a PROPOSED plan (JSON).
    *   Passes it to `AgentLoop`.
3.  **AgentLoop -> ContextManager**:
    *   `ctx.add_step()`: Manually creates a `StepNode` object and stores it in `self.steps`.
    *   `ctx.edges.append()`: Manually records the connection.
4.  **Execution**:
    *   Iterates `self.steps` dict to find `status="pending"`.
    *   Executes the code.
    *   Updates `self.steps[id].status`.
5.  **Refinement/Retry**:
    *   If a step fails, `Decision` requests a retry.
    *   `ContextManager` must manually clone the failed branch (using the verbose logic above) to create `Step 1F1`.

## Why This Matters
By running `S15_Manual`, you verify that **Agentic Intelligence is not about the library**. The logic (DAG, parents, children, state) is fundamental. `NetworkX` is just an optimization.

S15 proves you can build a Super Agent with pure Python dictionaries—it's just harder to maintain. **S16 introduces NetworkX precisely to solve the maintenance headache you see here.**
