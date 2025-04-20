
# JanusGraph Fraud Detection Project

This project demonstrates how to build and visualize a fraud detection graph using **JanusGraph**, **Gremlin**, and **Python**.

---

## 1. Java & JanusGraph Setup

**1.1 Install Java JDK (Version 8 or 11)**  
👉 https://www.oracle.com/java/technologies/javase-downloads.html

**1.2 Download JanusGraph**  
👉 https://github.com/JanusGraph/janusgraph/releases  
Download and extract `janusgraph-full-1.1.0`

---

## 2. Start JanusGraph Server

```bash
cd C:\janusgraph-full-1.1.0\janusgraph-full-1.1.0
bin\gremlin-server.bat conf\gremlin-server\gremlin-server.yaml
```

If you see `Channel started at port 8182`, JanusGraph is running.

---

## 3. Python Environment Setup

**3.1 Create and Activate Virtual Environment**

```bash
cd C:\Users\82108\Desktop
mkdir janus_env
cd janus_env
python -m venv venv
venv\Scripts\activate
```

**3.2 Install Required Libraries**

```bash
pip install gremlinpython ipykernel networkx matplotlib nest_asyncio
python -m ipykernel install --user --name=janus_env --display-name "Python (janus_env)"
```

---

## 4. Launch Jupyter Notebook

```bash
jupyter notebook
```

Go to `http://localhost:8888/tree`, then click:  
`New > Notebook > Python (janus_env)`

---

## 5. Basic Gremlin Connection in Python

```python
from gremlin_python.driver import client, serializer

gremlin_client = client.Client(
    'ws://localhost:8182/gremlin',
    'g',
    message_serializer=serializer.GraphSONSerializersV2d0()
)
```

---

## 6. Insert Nodes and Edges (Fraud Example)

```python
queries = [
    "g.addV('user').property('user_id', 'u001').property('name', 'Alice')",
    "g.addV('device').property('device_id', 'd001').property('type', 'mobile')",
    "g.addV('ip').property('ip_addr', '192.168.1.10')",
    "g.addV('transaction').property('tx_id', 't001').property('amount', 1200)",
    "g.addV('merchant').property('merchant_id', 'm001').property('name', 'LuxuryStore')",

    "g.V().has('user', 'user_id', 'u001').as('u').V().has('device', 'device_id', 'd001').addE('uses').from('u')",
    "g.V().has('user', 'user_id', 'u001').as('u').V().has('ip', 'ip_addr', '192.168.1.10').addE('logs_in_from').from('u')",
    "g.V().has('user', 'user_id', 'u001').as('u').V().has('transaction', 'tx_id', 't001').addE('initiates').from('u')",
    "g.V().has('transaction', 'tx_id', 't001').as('t').V().has('merchant', 'merchant_id', 'm001').addE('pays_to').from('t')"
]

for q in queries:
    gremlin_client.submit(q).all().result()
```

---

## 7. Graph Visualization in Python

```python
import matplotlib.pyplot as plt
import networkx as nx

# 노드
nodes = gremlin_client.submit("g.V().elementMap()").all().result()
edges = gremlin_client.submit(
    "g.E().as('e').project('id','label','out','in')"
    ".by(id).by(label).by(outV().id).by(inV().id)"
).all().result()

G = nx.DiGraph()

for node in nodes:
    G.add_node(node['id'], label=node['label'], **{k: v[0] for k, v in node.items() if k not in ['id', 'label']})

for edge in edges:
    G.add_edge(edge['out'], edge['in'], label=edge['label'])

plt.figure(figsize=(10, 6))
pos = nx.spring_layout(G)
nx.draw(G, pos, with_labels=True, node_color='lightblue', edge_color='gray', node_size=2000, font_size=10)
edge_labels = nx.get_edge_attributes(G, 'label')
nx.draw_networkx_edge_labels(G, pos, edge_labels=edge_labels)
plt.title("Fraud Graph Visualization")
plt.show()
```

---

## 8. GitHub Upload Instructions

```bash
cd C:\janusgraph-fraud-demo
git init
git remote add origin https://github.com/yourusername/janusgraph-fraud-demo.git
git add .
git commit -m "Initial commit with JanusGraph fraud detection project"
git branch -M main
git push -u origin main
```

---

## 9. Optional: LLM Explanation Code (Later Use)

```python
def generate_fraud_explanation(log):
    prompt = f"""
    사용자 {log['user']}는 {log['device']} 장치를 통해 IP {log['ip']}에서 {log['merchant']}에서 {log['tx_amount']}원 결제를 시도했습니다.
    이 장치는 등록된 장치가 아니며, 금액이 비정상적으로 높아 이상 거래로 판단되었습니다.
    조치: {log['action']}이 취해졌습니다.
    """
    return prompt

log_sample = {
    'user': 'u001',
    'device': 'd001',
    'ip': '192.168.1.10',
    'merchant': 'LuxuryStore',
    'tx_amount': 1200,
    'action': '거래 차단 및 사용자 알림'
}

print(generate_fraud_explanation(log_sample))
```

---

📌 This guide is designed for first-time users to reproduce the full setup and visualization.

