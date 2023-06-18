---
title: tensorflow
date: 2021-04-08 16:26:20
tags:
- tensorflow
categories:
- 源码阅读
---

## Graph.h

```c++
class Node {
public:
    int id() const { return id_; }
    int const_id() const { return cost_id_; }
    const string& name() const;
    void set_name(string name);
    const string& type_string() const;

    const NodeDef& def() const;
    const OpDef& op_def() const;
private:
    friend class Graph;
    Node();

    int id_;
    int const_id_;

    NodeClass class_;

    EdgeSet in_edges_;
    EdgeSet out_edges_;

    Graph* graph_;
};
```

```c++
class Edge {
public:
    Node* src() const { return src_; }
    Node* dst() const { return dst_; }
    int id() const  { return id_; }

    int src_output() const { return src_output_; }
    int dst_input() const { return dst_input_; }
    bool IsControlEdge() const;

private:
    Edge() {}
    friend class Graph;
    Node* src_;
    Node* dst_;
    int id_;
    int src_output_;//前驱节点的第src_output_条输出边
    int dst_input_;//后继节点的第dst_output_条输入边
};
```

控制依赖边，其`src_output/dst_output`均为`Graph::kControlSlot`(-1)，意味着控制依赖边不承载任何数据。

计算图的普通边承载Tensor，并使用`TensorId`标识，TensorId由二元组`node_name:src_output`唯一标识，其中`node_name`为边的前驱节点。`src_output`缺省为0，即`node_name`与`node_name:0`等价，

```c++
class Graph {
public:
    explicit Graph(const OpRegistryInterface* ops);
    explicit Graph(const FunctionLibraryDefinition& flib_def);
    ~Graph();

    static const int kControlSlot; // -1 控制边，不承载任何数据
    void set_versions(const VersionDef& versions);

    Node* AddNode(NodeDef node_def, Status* status);
    Node* CopyNode(const Node* node);
    void RemoveNode(Node* node);
    const Edge* AddEdge(Node* source, int x, Node* dest, int y);
    const Edge* AddControlEdge(Node* source, Node* dest, bool allow_deuplicates=false);
    void RemoveEdge(const Edge* edge);
    void RemoveControlEdge(const Edge* edge);

    enum {kSourceId = 0, kSinkId = 1};
    Node* FindNodeId(int id) const { return nodes_[id]; }
    Node* source_node() const { return FindNodeId(kSourceId); }
    Node* sink_node() const { return FindNodeId(kSinkId); }
    
private:
    FunctionLibraryDefinition ops_;
    const std::unique_ptr<VersionDef> versions_;
    
    core::Arena arena_;
    vector<Node*> nodes_;
    int64 num_nodes_ = 0;
    vector<Edge*> edges_;
    int num_edges_ = 0;    
};


Graph::Graph(const OpRegistryInterface* ops) 
      : ops_(ops, FunctionDefLibrary()), 
        versions_(new VersionDef),
        arena_(8 << 10) {
    // versions_->set_procuder(...);
    // versions_->set_min_consumer(...);

    device_names.push_back("");

    NodeDef def;
    def.set_name("_SOURCE");
    def.set_op("NoOp");
    Node* source = AddNode(def, &status);
    def.set_name("_SINK");
    Node* sink = AddNode(def, &status);

    AddControlEdge(source, sink);
}
```

Graph是一个DAG，按照拓扑排序运行，若存在多个入度为0的节点，则并行运行。初始状态，有一个起始节点Source和终止节点Sink，普通节点的id必大于1。

Source和Sink之间有一个控制依赖边，保证计算图的执行始于Source，止于Sink。
