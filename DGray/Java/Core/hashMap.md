---
title: hashMap
updated: 2021-07-21 08:53:46Z
created: 2021-07-21 08:53:36Z
---

[[hashMap]]

```
// Получаем набор элементов
Set<Map.Entry<String, Integer>> set = hashMap.entrySet();

// Отобразим набор
for (Map.Entry<String, Integer> me : set) {
    System.out.print(me.getKey() + ": ");
    System.out.println(me.getValue());
}
```