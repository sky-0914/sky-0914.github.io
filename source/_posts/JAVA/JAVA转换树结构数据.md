---
title: JAVA转换树结构数据
tags:
  - JAVA
  - fastjson
categories:
  - JAVA
---
# JAVA 转换 树结构数据

**第一步：引入fastjson**

```pom
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>${fastjson.version}</version>
</dependency>
```

**第二步：用到了工具内的JSONPath**

[JSONPath使用教程](https://blog.csdn.net/lwg_1540652358/article/details/84111339)

```java
    /**
     * 树转换
     *
     * @param obj                  需要转换的对象
     * @param parentCodeFieldName  父标识字段名
     * @param parentCode           父标识值
     * @param currentCodeFieldName 当前标识字段名
     * @param childrenFiledName    子树的字段名
     * @param c                    需要转换的Class类型
     * @param <T>                  泛型
     * @return 返回List<T>
     */
    public static <T> List<T> tree(Object obj, String parentCodeFieldName, String parentCode, String currentCodeFieldName, String childrenFiledName, Class<T> c) {
        long t1 = System.currentTimeMillis();
        String jsonStr = JSON.toJSONString(obj);
        log.debug("树转换开始 >>>>>>>>>>>>>>>> {}", JSON.toJSONString(obj));
        //获取第一层级的数据
        JSONArray jsonArray = (JSONArray) JSONPath.read(jsonStr, "$[" + parentCodeFieldName + "=" + parentCode + "]");
        if (CollectionUtils.isEmpty(jsonArray)) {
            //为空的话直接返回空集合
            return Lists.newArrayList();
        }
        for (int i = 0; i < jsonArray.size(); i++) {
            JSONObject jsonObject = jsonArray.getJSONObject(i);
            String code = jsonObject.getString(currentCodeFieldName);
            treeChildren(jsonStr, jsonObject, parentCodeFieldName, code, currentCodeFieldName, childrenFiledName);
        }
        List<T> list = JSONArray.parseArray(jsonArray.toString(), c);
        log.debug("树转换结束, 转换时间: {} ms . >>>>>>>>>>>>>>>> {}", (System.currentTimeMillis() - t1), JSON.toJSONString(list));
        return list;
    }

    private static void treeChildren(String jsonStr, JSONObject currentJsonObj, String parentCodeFieldName, String parentCode, String currentCodeFieldName, String childrenFiledName) {
        JSONArray jsonArray = (JSONArray) JSONPath.read(jsonStr, "$[" + parentCodeFieldName + "=" + parentCode + "]");
        if (CollectionUtils.isEmpty(jsonArray)) {
            return;
        }
        currentJsonObj.put(childrenFiledName, jsonArray);
        for (int i = 0; i < jsonArray.size(); i++) {
            JSONObject jsonObject = jsonArray.getJSONObject(i);
            String code = jsonObject.getString(currentCodeFieldName);
            treeChildren(jsonStr, jsonObject, parentCodeFieldName, code, currentCodeFieldName, childrenFiledName);
        }
    }
```


