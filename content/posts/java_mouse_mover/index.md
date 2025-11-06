+++
title = "Пишем скрипт на java меремещающий курсор раз в минуту"
draft = false
date = 2025-11-06
[taxonomies]
categories = ["java"]
tags = ["java"]
+++

## Описание проблемы
Когда у тебя нет прав на выполнение bat скриптов и стоит запрет на выполнение всех программ, кроме разрешенных. И тебя раздражает, что через пару минут экран блокируется. Хочется сделать шалость и сделать жизнь немного проще и лучше. 

## Идея
Пишем скрипт, который не надо компилировать, а сразу можно выполнить. Нужна java не ниже 11 версии.

## Код
Создаем файл MouseMover.java со следующим содержимым:

```java
import java.awt.*;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

public class MouseMover {
    public static void main(String[] args) {
        System.out.println("Mouse mover started at " + 
            LocalDateTime.now().format(DateTimeFormatter.ISO_LOCAL_TIME));
        
        try {
            Robot robot = new Robot();
            int moveDirection = 1;
            
            while (true) {
                Point currentLocation = MouseInfo.getPointerInfo().getLocation();
                int x = (int) currentLocation.getX();
                int y = (int) currentLocation.getY();
                
                // Перемещаем курсор на 1 пиксель вправо/влево
                robot.mouseMove(x + moveDirection, y);
                moveDirection *= -1; // Меняем направление для следующего движения
                
                // Логируем действие
                System.out.println("Mouse moved to (" + (x + moveDirection) + ", " + y + ") at " +
                    LocalDateTime.now().format(DateTimeFormatter.ISO_LOCAL_TIME));
                
                // Ожидаем 1 минуту
                Thread.sleep(60000);
            }
        } catch (AWTException e) {
            System.err.println("Robot initialization failed: " + e.getMessage());
        } catch (InterruptedException e) {
            System.err.println("Thread interrupted: " + e.getMessage());
            Thread.currentThread().interrupt();
        }
    }
}
```

Запускаем командой:
```shell
    java MouseMover.java
```

Каждую минуту курсор будет перемещаться на 1 пиксель или в право, или в лево.


## Более продвинутый вариант запускает сборку проекта в Intellij Idea

````java 
import java.awt.*;
import java.awt.event.KeyEvent;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

public class MouseMover {
    private static final String TARGET_PROJECT = "udk-pdf-scanner";
    private static Robot robot;

    public static void main(String[] args) {
        System.out.println("Mouse mover started at " +
                LocalDateTime.now().format(DateTimeFormatter.ISO_LOCAL_TIME));

        try {
            robot = new Robot();
            robot.setAutoDelay(100);

            int moveDirection = 1;

            while (true) {
                // Перемещаем курсор
                moveMouse(moveDirection);
                moveDirection *= -1;

                // Проверяем и взаимодействуем с IntelliJ IDEA
                checkAndInteractWithIDEA();

                // Ждем 2 минуты
                Thread.sleep(120000);
            }
        } catch (AWTException e) {
            System.err.println("Robot initialization failed: " + e.getMessage());
        } catch (InterruptedException e) {
            System.err.println("Thread interrupted: " + e.getMessage());
            Thread.currentThread().interrupt();
        } catch (Exception e) {
            System.err.println("Unexpected error: " + e.getMessage());
            e.printStackTrace();
        }
    }

    private static void moveMouse(int direction) {
        try {
            Point currentLocation = MouseInfo.getPointerInfo().getLocation();
            int x = (int) currentLocation.getX();
            int y = (int) currentLocation.getY();

            robot.mouseMove(x + direction, y);

            System.out.println("Mouse moved to (" + (x + direction) + ", " + y + ") at " +
                    LocalDateTime.now().format(DateTimeFormatter.ISO_LOCAL_TIME));
        } catch (Exception e) {
            System.err.println("Error moving mouse: " + e.getMessage());
        }
    }

    private static void checkAndInteractWithIDEA() {
        try {
            // Получаем список процессов
            ProcessBuilder processBuilder = new ProcessBuilder("tasklist", "/fo", "csv", "/nh");
            Process process = processBuilder.start();
            String output = new String(process.getInputStream().readAllBytes());

            // Проверяем, запущена ли IntelliJ IDEA
            if (output.toLowerCase().contains("idea64.exe") ||
                    output.toLowerCase().contains("idea.exe") ||
                    output.toLowerCase().contains("intellij")) {

                System.out.println("IntelliJ IDEA detected - attempting to interact...");

                // Переключаемся на IDEA
                switchToIDEA();

                // Даем время для переключения
                Thread.sleep(2000);

                // Пытаемся найти и запустить проект "udk-pdf-scanner"
                runSimpleProject();

            } else {
                System.out.println("IntelliJ IDEA not running");
            }
        } catch (Exception e) {
            System.err.println("Error checking processes: " + e.getMessage());
        }
    }

    private static void switchToIDEA() {
        try {
            // Используем Alt+Tab для переключения на IDEA
            robot.keyPress(KeyEvent.VK_ALT);
            robot.keyPress(KeyEvent.VK_TAB);
            robot.keyRelease(KeyEvent.VK_TAB);
            robot.keyRelease(KeyEvent.VK_ALT);

            System.out.println("Attempted to switch to IntelliJ IDEA");
        } catch (Exception e) {
            System.err.println("Error switching to IDEA: " + e.getMessage());
        }
    }

    private static void runSimpleProject() {
        try {
            // Комбинация для запуска проекта: Shift + F10
            robot.keyPress(KeyEvent.VK_SHIFT);
            robot.keyPress(KeyEvent.VK_F10);
            robot.keyRelease(KeyEvent.VK_F10);
            robot.keyRelease(KeyEvent.VK_SHIFT);

            System.out.println("Sent Shift+F10 to run project");
        } catch (Exception e) {
            System.err.println("Error running project: " + e.getMessage());
        }
    }

}
````