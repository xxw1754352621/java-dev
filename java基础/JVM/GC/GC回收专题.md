# GC回收专题

## GC Roots 集合（**引用类型**）-根节点

- 虚拟机栈中的引用对象

- 方法区中的类静态属性引用的对象

- 方法区中的常量引用的对象

- 本地方法栈中JNI（native方法）引用的对象

  ```properties
  Local variables 本地变量
  Static variables 静态变量
  JNI References JNI引用等
  ```

  

## （**多cpu**）[垃圾回收器选择](https://hllvm-group.iteye.com/group/wiki/2865-JVM)

| 参数                                                   | 新生代            | 算法                   | 老年代                           | 算法     |              |
| ------------------------------------------------------ | ----------------- | ---------------------- | -------------------------------- | -------- | ------------ |
| -XX:+UseParallelGC/-XXUserParallelOldGC (**JDK8默认**) | parallel Scavenge | 复制                   | parallel old                     | 标记整理 | 吞吐量优先   |
| -XX:+UseConcMarSweepGc                                 | ParNew            | 复制                   | CMS+SerialOld（SerialOld是后备） | 标-清    | 响应时间优先 |
| -XX:+UseG1GC                                           | G1                | 局部复制，整体标记整理 |                                  |          | 大堆等考虑   |

## JVM参数查看

- java -XX:+PrintFlagsFinal 默认参数

- -XX:+PrintCommandLineFlags  （JVM动态适配或者人）修改的参数

- jps 和jinfo工具的使用
  -XX:+UseAdaptiveSizePolicy：设置此选项后，并行收集器会自动选择年轻代区大小和相应的Survivor区比例，以达到目标系统规定的最低相应时间或者收集频率等，此值建议使用并行收集器时，一直打开。
  -XX:MaxGCPauseMillis=100:设置每次年轻代垃圾回收的最长时间，如果无法满足此时间，JVM会自动调整年轻代大小，以满足此值
  -XX:+UseParallelOldGC：配置年老代垃圾收集方式为并行收集。JDK6.0支持对年老代并行收集。
  -XX:ParallelGCThreads=20：配置并行收集器的线程数，即：同时多少个线程一起进行垃圾回收。此值最好配置与处理器数目相等。

- ```properties
  JVM仅指定新生代垃圾收集器的情况下，默认老年代采用Serial Old垃圾收集器（带压缩）：
  -XX:+UseSerialGC
  Serial (DefNew)  + Serial Old（Serial Mark Sweep Compact）
  -XX:+UseParNewGC
  Parallel (ParNew)  +  Serial Old（Serial Mark Sweep Compact）
  
  jdk5出现CMS
  jdk6出现ParallelOldGC

  jdk8默认以下：
  -XX:+UseParallelGC
  Parallel Scavenge (PSYoungGen) + ParallelOldGC 
  
  所以以下配置说明：
  
  -XX:+UseParNewGC -XX:+UseConcMarkSweepGC
  Parallel (ParNew) + CMS（Concurrent Mark Sweep） + Serial Old（Serial Mark Sweep Compact）
  
  作者：二进制之路
  链接：https://juejin.im/post/5c27ab426fb9a049be5d9318
  
  ```

  
  
  {
    "steps": [
      {
        "join_preparation": {
          "select#": 1,
          "steps": [
            {
              "expanded_query": "/* select#1 */ select `parcel`.`fpx_tracking_no` AS `fpx_tracking_no`,`parcel`.`create_time` AS `create_time` from `parcel` where (`parcel`.`create_time` > '2019-09-09') order by `parcel`.`pickup_time`"
            }
          ]
        }
      },
      {
        "join_optimization": {
          "select#": 1,
          "steps": [
            {
              "condition_processing": {
                "condition": "WHERE",
                "original_condition": "(`parcel`.`create_time` > '2019-09-09')",
                "steps": [
                  {
                    "transformation": "equality_propagation",
                    "resulting_condition": "(`parcel`.`create_time` > '2019-09-09')"
                  },
                  {
                    "transformation": "constant_propagation",
                    "resulting_condition": "(`parcel`.`create_time` > '2019-09-09')"
                  },
                  {
                    "transformation": "trivial_condition_removal",
                    "resulting_condition": "(`parcel`.`create_time` > '2019-09-09')"
                  }
                ]
              }
            },
            {
              "substitute_generated_columns": {
              }
            },
            {
              "table_dependencies": [
                {
                  "table": "`parcel`",
                  "row_may_be_null": false,
                  "map_bit": 0,
                  "depends_on_map_bits": [
                  ]
                }
              ]
            },
            {
              "ref_optimizer_key_uses": [
              ]
            },
            {
              "rows_estimation": [
                {
                  "table": "`parcel`",
                  "range_analysis": {
                    "table_scan": {
                      "rows": 1277,
                      "cost": 284.5
                    },
                    "potential_range_indexes": [
                      {
                        "index": "PRIMARY",
                        "usable": false,
                        "cause": "not_applicable"
                      },
                      {
                        "index": "uniq_fpx_tracking_no_1",
                        "usable": false,
                        "cause": "not_applicable"
                      },
                      {
                        "index": "idx_create_time",
                        "usable": true,
                        "key_parts": [
                          "create_time",
                          "id"
                        ]
                      },
                      {
                        "index": "idx_parcel_status",
                        "usable": false,
                        "cause": "not_applicable"
                      },
                      {
                        "index": "idx_courier_station_code",
                        "usable": false,
                        "cause": "not_applicable"
                      },
                      {
                        "index": "idx_pickup_time",
                        "usable": false,
                        "cause": "not_applicable"
                      }
                    ],
                    "setup_range_conditions": [
                    ],
                    "group_index_range": {
                      "chosen": false,
                      "cause": "not_group_by_or_distinct"
                    },
                    "analyzing_range_alternatives": {
                      "range_scan_alternatives": [
                        {
                          "index": "idx_create_time",
                          "ranges": [
                            "0x5d752580 < create_time"
                          ],
                          "index_dives_for_eq_ranges": true,
                          "rowid_ordered": false,
                          "using_mrr": false,
                          "index_only": false,
                          "rows": 156,
                          "cost": 188.21,
                          "chosen": true
                        }
                      ],
                      "analyzing_roworder_intersect": {
                        "usable": false,
                        "cause": "too_few_roworder_scans"
                      }
                    },
                    "chosen_range_access_summary": {
                      "range_access_plan": {
                        "type": "range_scan",
                        "index": "idx_create_time",
                        "rows": 156,
                        "ranges": [
                          "0x5d752580 < create_time"
                        ]
                      },
                      "rows_for_plan": 156,
                      "cost_for_plan": 188.21,
                      "chosen": true
                    }
                  }
                }
              ]
            },
            {
              "considered_execution_plans": [
                {
                  "plan_prefix": [
                  ],
                  "table": "`parcel`",
                  "best_access_path": {
                    "considered_access_paths": [
                      {
                        "rows_to_scan": 156,
                        "access_type": "range",
                        "range_details": {
                          "used_index": "idx_create_time"
                        },
                        "resulting_rows": 156,
                        "cost": 219.41,
                        "chosen": true,
                        "use_tmp_table": true
                      }
                    ]
                  },
                  "condition_filtering_pct": 100,
                  "rows_for_plan": 156,
                  "cost_for_plan": 219.41,
                  "sort_cost": 156,
                  "new_cost_for_plan": 375.41,
                  "chosen": true
                }
              ]
            },
            {
              "attaching_conditions_to_tables": {
                "original_condition": "(`parcel`.`create_time` > '2019-09-09')",
                "attached_conditions_computation": [
                ],
                "attached_conditions_summary": [
                  {
                    "table": "`parcel`",
                    "attached": "(`parcel`.`create_time` > '2019-09-09')"
                  }
                ]
              }
            },
            {
              "clause_processing": {
                "clause": "ORDER BY",
                "original_clause": "`parcel`.`pickup_time`",
                "items": [
                  {
                    "item": "`parcel`.`pickup_time`"
                  }
                ],
                "resulting_clause_is_simple": true,
                "resulting_clause": "`parcel`.`pickup_time`"
              }
            },
            {
              "reconsidering_access_paths_for_index_ordering": {
                "clause": "ORDER BY",
                "index_order_summary": {
                  "table": "`parcel`",
                  "index_provides_order": false,
                  "order_direction": "undefined",
                  "index": "idx_create_time",
                  "plan_changed": false
                }
              }
            },
            {
              "refine_plan": [
                {
                  "table": "`parcel`",
                  "pushed_index_condition": "(`parcel`.`create_time` > '2019-09-09')",
                  "table_condition_attached": null
                }
              ]
            }
          ]
        }
      },
      {
        "join_execution": {
          "select#": 1,
          "steps": [
            {
              "filesort_information": [
                {
                  "direction": "asc",
                  "table": "`parcel`",
                  "field": "pickup_time"
                }
              ],
              "filesort_priority_queue_optimization": {
                "usable": false,
                "cause": "not applicable (no LIMIT)"
              },
              "filesort_execution": [
              ],
              "filesort_summary": {
                "rows": 156,
                "examined_rows": 156,
                "number_of_tmp_files": 0,
                "sort_buffer_size": 262128,
                "sort_mode": "<sort_key, packed_additional_fields>"
              }
            }
          ]
        }
      }
    ]
  }