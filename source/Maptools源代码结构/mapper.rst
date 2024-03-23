mapper
^^^^^^^^^^^^^^
'mapper'为 |name| 核心代码，包含'graph_shaper' 'operator_parser' 'onnx_converter' 'tile_mapper' 'noc_mapper'五个子文件


graph_shaper
--------------

operator_parser
----------------

onnx_converter
---------------


tile_mapper
--------------
'tile_mapper'将'device_graph'转换为'CTG'，将算子规则化映射到'tile'中，并根据需要将'tile'配置中的'operator quant config (OQC)'对象转换到'tile quant config (TQC)'对象。


数据结构
>>>>>>>>>>>>>>

.. py:data:: map_list
   :value: List[np.ndarray]

   包含每一层的映射信息，列表中的每一个元素为一个'numpy array'代表一层的映射信息，'numpy array'中的每一个元素又代表当前层映射'tile'组成的'block'，'numpy array'中的元素的数值就是这个'block'中的'tile'数量

.. py:data:: match_dict
   :value: Dict[str, int]

   匹配算子图'operator graph'中的每一个计算节点和'map_list'中的每一层

.. py:data:: map_dict
   :value: Dict[Tuple[int, int, int, int], Dict[str, Any]]

   可以获取每个映射得到的'tile'相关配置信息的查找表。四元元组'Tuple'组成的'key'含义为'(layer_idx, cluster_idx, block_idx, idx_in_block)'，这个'key'可以唯一确定一个'tile'

