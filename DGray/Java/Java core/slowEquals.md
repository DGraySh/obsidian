---
title: slowEquals
updated: 2021-03-13 05:30:06Z
created: 2021-03-13 05:27:45Z
tags:
  - slowequals
---

[[slowEquals]]

```java
private static boolean slowEquals(byte[] a, byte[] b)
     {
         int diff = a.length ^ b.length;
         for(int i = 0; i < a.length && i < b.length; i++)
             diff |= a[i] ^ b[i];
         return diff == 0;
     }
```

     