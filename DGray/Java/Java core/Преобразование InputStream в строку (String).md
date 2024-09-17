---
title: Преобразование InputStream в строку (String)
updated: 2021-02-23 20:25:05Z
created: 2021-02-23 19:31:50Z
tags:
  - bytearrayoutputstream
  - inputstream
---

# Преобразование InputStream в строку (String)

1.  Используя `IOUtils.toString` из библиотеки `Apache Commons`. Один из самых коротких однострочников.


```java
String result = IOUtils.toString(inputStream, StandardCharsets.UTF_8);

```

2.  Используя `CharStreams` из библиотеки guava. Тоже довольно короткий код.

```java
try(InputStreamReader reader = new InputStreamReader(inputStream, Charsets.UTF_8)) {
String result = CharStreams.toString(reader);
}
```

3.  Используя Scanner (JDK). Решение короткое, хитрое, с помощью чистого JDK, но это скорее хак, который вынесет мозг тем кто о таком фокусе не знает.

```java
try(Scanner s = new Scanner(inputStream).useDelimiter("\\A")) {            
    String result = s.hasNext() ? s.next() : "";
}
```

4.  Используя `Stream Api` с помощью Java 8. Предупреждение: Оно заменяет разные переносы строки (такие как \\r\\n) на \\n, иногда это может быть критично.

```java
try(BufferedReader br = new BufferedReader(new InputStreamReader(inputStream))) {
    String result = br.lines().collect(Collectors.joining("\n"));
}
```

5.  Используя `parallel Stream Api` (Java 8). Предупреждение: Как и 4 решение, оно заменяет разные переносы строки (такие как \\r\\n) на \\n.

```java
try(BufferedReader br = new BufferedReader(new InputStreamReader(inputStream))) {
    String result = br.lines().parallel().collect(Collectors.joining("\n"));
}
```

6.  Используя `InputStreamReader` и `StringBuilder` из обычного JDK

```java
final int bufferSize = 1024;
final char[] buffer = new char[bufferSize];
final StringBuilder out = new StringBuilder();
try(Reader in = new InputStreamReader(inputStream, "UTF-8")) {
    for (; ; ) {
        int rsz = in.read(buffer, 0, buffer.length);
        if (rsz < 0)
             break;
        out.append(buffer, 0, rsz);
    }
    return out.toString();
}
```

7.  Используя `StringWriter` и `IOUtils.copy` из `Apache Commons`

```java
try(StringWriter writer = new StringWriter()) {
    IOUtils.copy(inputStream, writer, "UTF-8");
    return writer.toString();
}
```

8.  Используя `ByteArrayOutputStream` и `inputStream.read` из JDK

```java
try(ByteArrayOutputStream result = new ByteArrayOutputStream()) {
    byte[] buffer = new byte[1024];
    int length;
    while ((length = inputStream.read(buffer)) != -1) {
        result.write(buffer, 0, length);
    }
    return result.toString("UTF-8");
}
```

9.  Используя `BufferedReader` из JDK. Предупреждение: Это решение заменяет разные переносы строк (такие как \\n\\r) на `line.separator` system property (например, в Windows на "\\r\\n").

```java
String newLine = System.getProperty("line.separator");
try(BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream))) {
      StringBuilder result = new StringBuilder();
      String line; boolean flag = false;
      while ((line = reader.readLine()) != null) {
          result.append(flag? newLine: "").append(line);
          flag = true;
      }
     return result.toString();
}
```

10. Используя `BufferedInputStream` и `ByteArrayOutputStream` из JDK

```java
try(BufferedInputStream bis = new BufferedInputStream(inputStream); ByteArrayOutputStream buf = new ByteArrayOutputStream()) {
        int result = bis.read();
        while(result != -1) {
            buf.write((byte) result);
            result = bis.read();
        }
        return buf.toString();
}
```

11. Используя `inputStream.read()` и `StringBuilder` (JDK). Предупреждение: Это решение не работает с Unicode, например с русским текстом

```java
int ch;
StringBuilder sb = new StringBuilder();
while((ch = inputStream.read()) != -1)
    sb.append((char)ch);
reset();
return sb.toString();
```

Итак о использовании:

Решения 4, 5 и 9 преобразую разные переносы строки в одну.

Решения 11 не работает с Unicode текстом

Решение 1, 7 требует использование библиотеки Apache Commons, 2 требует библиотеку Guava, 4 и 5 требуют Java 8 и выше,

Замеры производительности

Предупреждение: замеры производительности всегда сильно зависят от системы, условий замера и т.п. Я измерял на двух разных компьютерах, один Windows 8.1, Intel i7-4790 CPU 3.60GHz2, 16Gb, второй — Linux Mint 17.2, Celeron Dual-Core T3500 2.10Ghz2, 6Gb, однако не могу гарантировать что результаты являются абсолютной истиной, вы всегда можете повторить тесты (test1 и test2) на вашей системе.

Замеры производительности для небольших строк (длина = 175), тесты можно найти на github (режим = среднее время выполнения (AverageTime), система = Linux Mint 17.2, Celeron Dual-Core T3500 2.10Ghz2, 6Gb, чем значение ниже тем лучше, 1,343 — наилучшее):

```
              Benchmark                        Mode  Cnt   Score   Error  Units
8. ByteArrayOutputStream and read (JDK)        avgt   10   1,343 ± 0,028  us/op
6. InputStreamReader and StringBuilder (JDK)   avgt   10   6,980 ± 0,404  us/op
10. BufferedInputStream, ByteArrayOutputStream avgt   10   7,437 ± 0,735  us/op
11. InputStream.read() and StringBuilder (JDK) avgt   10   8,977 ± 0,328  us/op
7. StringWriter and IOUtils.copy (Apache)      avgt   10  10,613 ± 0,599  us/op
1. IOUtils.toString (Apache Utils)             avgt   10  10,605 ± 0,527  us/op
3. Scanner (JDK)                               avgt   10  12,083 ± 0,293  us/op
2. CharStreams (guava)                         avgt   10  12,999 ± 0,514  us/op
4. Stream Api (Java 8)                         avgt   10  15,811 ± 0,605  us/op
9. BufferedReader (JDK)                        avgt   10  16,038 ± 0,711  us/op
5. parallel Stream Api (Java 8)                avgt   10  21,544 ± 0,583  us/op
```

Замеры производительности для больших строк (длина = 50100), тесты можно найти на github (режим = среднее время выполнения (AverageTime), система = Linux Mint 17.2, Celeron Dual-Core T3500 2.10Ghz*2, 6Gb, чем значение ниже тем лучше, 200,715 — наилучшее):

```
              Benchmark                        Mode  Cnt   Score        Error  Units
8. ByteArrayOutputStream and read (JDK)        avgt   10   200,715 ±   18,103  us/op
1. IOUtils.toString (Apache Utils)             avgt   10   300,019 ±    8,751  us/op
6. InputStreamReader and StringBuilder (JDK)   avgt   10   347,616 ±  130,348  us/op
7. StringWriter and IOUtils.copy (Apache)      avgt   10   352,791 ±  105,337  us/op
2. CharStreams (guava)                         avgt   10   420,137 ±   59,877  us/op
9. BufferedReader (JDK)                        avgt   10   632,028 ±   17,002  us/op
5. parallel Stream Api (Java 8)                avgt   10   662,999 ±   46,199  us/op
4. Stream Api (Java 8)                         avgt   10   701,269 ±   82,296  us/op
10. BufferedInputStream, ByteArrayOutputStream avgt   10   740,837 ±    5,613  us/op
3. Scanner (JDK)                               avgt   10   751,417 ±   62,026  us/op
11. InputStream.read() and StringBuilder (JDK) avgt   10  2919,350 ± 1101,942  us/op
```

Таблица зависимости среднего времени от длины строки, система Windows 8.1, Intel i7-4790 CPU 3.60GHz 3.60GHz, 16Gb:

```
длинна  182    546     1092    3276    9828    29484   58968

test8  0.38    0.938   1.868   4.448   13.412  36.459  72.708
test4  2.362   3.609   5.573   12.769  40.74   81.415  159.864
test5  3.881   5.075   6.904   14.123  50.258  129.937 166.162
test9  2.237   3.493   5.422   11.977  45.98   89.336  177.39
test6  1.261   2.12    4.38    10.698  31.821  86.106  186.636
test7  1.601   2.391   3.646   8.367   38.196  110.221 211.016
test1  1.529   2.381   3.527   8.411   40.551  105.16  212.573
test3  3.035   3.934   8.606   20.858  61.571  118.744 235.428
test2  3.136   6.238   10.508  33.48   43.532  118.044 239.481
test10 1.593   4.736   7.527   20.557  59.856  162.907 323.147
test11 3.913   11.506  23.26   68.644  207.591 600.444 1211.545
```

Выводы

Самым быстрым решением во всех случаях и всех системах оказался 8 тест: Используя ByteArrayOutputStream и inputStream.read из JDK

```java
ByteArrayOutputStream result = new ByteArrayOutputStream();
byte[] buffer = new byte[1024];
int length;
while ((length = inputStream.read(buffer)) != -1) {
    result.write(buffer, 0, length);
}
return result.toString("UTF-8");
```

Коротким и весьма быстрым решением будет использование IOUtils.toString из Apache Commons

```java
String result = IOUtils.toString(inputStream, StandardCharsets.UTF_8);
```

Stream Api из Java 8 показывает среднее время, а использование параллельных стримов имеет смысл только при довольно большой строки, иначе он работает очень долго (что в общем-то было ожидаемо)

Решение 11 лучше не использовать в принципе, так как он работает медленнее всех и не работает с Unicode


[[InputStream]] [[ByteArrayOutputStream]]




