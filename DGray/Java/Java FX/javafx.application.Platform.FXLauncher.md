---
title: javafx.application.Platform.FXLauncher
updated: 2021-02-23 20:45:09Z
created: 2021-02-23 20:44:19Z
tags:
  - fxlauncher
  - javafx
---

## javafx.application.Platform.FXLauncher
```java
import javafx.application.Platform;

public class FXLauncher {
    public static void fxThreadRun(Runnable runnable) {
        if (Platform.isFxApplicationThread()) {
            runnable.run();
        } else {
            Platform.runLater(() -> {
                runnable.run();
            });
        }
    }
}
```