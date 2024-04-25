mapper
^^^^^^^^^^^^^^
'mapper'为 |name| 核心代码，包含'graph_shaper' 'operator_parser' 'onnx_converter' 'tile_mapper' 'noc_mapper'五个子文件


graph_shaper
--------------

operator_parser
----------------

onnx_converter
---------------
模型转换的全部流程都被集成到 :py:data:`OnnxConverter` 这个 `class` 中, 用户只需要创建 :py:data:`OnnxConverter` 对象并调用其 :py:data:`OnnxConverter.run_conversion` 方法便可实现整个模型转换流程。

:py:data:`run_conversion`内依次调用了:py:data:`construct_origin_graph`，:py:data:`construct_host_graph`，:py:data:`construct_device_graph` 三个方法，其中：

:py:data:`construct_origin_graph`
"""""""""""""""
首先调用了:py:data:`_construct_raw_graph`方法，在该方法中，遍历了输入ONNX模型的index（i）和node（n）。在每个node上，如果该node没有名字，那么将用index的值作为它的名字。接下来，用:py:data:`_assert_node`验证该node的操作类型是否支持。然后，如果node的操作类型属于Merge类型(多个输入)，那么将遍历它的所有前继节点，将它们和它们对应的edge加入graph中。否则（单个输入），仅将该节点的前继节点和对应的edge加入graph中。接下来，检查节点的第一个输出（`succ_node`），将和对应的edge其加入到graph中。最后，使用parser对graph再次整理。

接下来调用了:py:data:`raw_graph.connect_concats`方法，对concat类型的node进行配置。首先，将遍历所有graph中的所有节点，寻找concat类型的node。找到concat类型的node后，将搜索这些node所对应的目标节点，并调用:py:data:`_config_concat`方法进行配置。配置过程中,将遍历所有Conv烈性节点,配置其concat_slot和block_boxes。完成所有目标节点的配置后，将去搜索下一个节点。最后，调用了:py:data:`_format_block_boxes`，遍历查找所有Conv类型节点，为其添加block_boxes。



:py:data:`construct_host_graph`
"""""""""""""""

:py:data:`construct_device_graph` 
"""""""""""""""

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

