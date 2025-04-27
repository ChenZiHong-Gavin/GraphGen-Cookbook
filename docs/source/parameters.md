# Parameters

## 1. `qa_form`
- **Purpose**: Defines the form of generated QAs.
- **Options**: 
    - `"atomic"`: Simple, single-node questions.  
    - `"multi_hop"`: Multi-step reasoning questions.  
    - `"open"`: Open-ended questions with no fixed answer.  
- **Default**: `"multi_hop"`  

## 2. `expand_method`
- **Purpose**: Controls traversal limits.  
- **Options**:  
  - `"max_width"`: Limits by edge count (`max_extra_edges`).  
  - `"max_tokens"`: Limits by token count (`max_tokens`).  
- **Default**: `"max_tokens"`

## 3. `bidirectional`
- **Purpose**: Enables traversal in both directions.  
- **Type**: `bool`  
- **Default**: `True`  

## 4. `max_extra_edges`
- **Purpose**: Max edges per direction (if `expand_method="max_width"`).  
- **Type**: `int`  
- **Default**: `5`  

## 5. `max_tokens`
- **Purpose**: Restricts input length (if `expand_method="max_tokens"`).  
- **Type**: `int`  
- **Default**: `256`

## 6. `max_depth`
- **Purpose**: Limits traversal depth per direction.  
- **Type**: `int`  
- **Default**: `2`  
- **Example**: `A → B → C` for `max_depth=2`.  

## 7. `edge_sampling`
- **Purpose**: Selects edges at the same traversal level.  
- **Options**:  
  - `"max_loss"`: Prioritizes high-loss edges.  
  - `"min_loss"`: Prioritizes low-loss edges.  
  - `"random"`: Random selection.  
- **Default**: `"max_loss"`  

## 8. `isolated_node_strategy`
- **Purpose**: Manages nodes with no connections.  
- **Options**:  
  - `"add"`: Includes isolated nodes.  
  - `"ignore"`: Skips them.  
- **Default**: `"add"`  

## 9. `loss_strategy`
- **Purpose**: Defines loss computation focus.  
- **Options**:  
  - `"only_edge"`: Loss for edges only.  
  - `"both"`: Loss for nodes and edges.  
- **Default**: `"only_edge"`  
