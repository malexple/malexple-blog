+++
title = "Пишем скрипт для генерации контекста для AI"
draft = false
date = 2025-11-10
[taxonomies]
categories = ["java"]
tags = ["java", "ai"]
+++

## Описание проблемы
У нейросетей есть ограничение на количество символов в чате или количество запросов. И бывает что лимит символов на чат закончен, а разработка проекта еще не закончена. И чтобы создать новый чат и напомнить контекст, приходится копировать код. Это очень долго и не все хочется копировать.
Проблема еще и в том что DeepSeek не понимает ссылки на репозитории или не смотрит в код который там. Но можно показать ему контекст кода и он включит его в свой анализ.

## Идея
Пишем скрипт, который не надо компилировать, а сразу можно выполнить. Нужна java не ниже 11 версии. Идея в том что мы в один файл собираем весь контекст который нужен для анализа.

## Код
Создаем файл ScanProject.java со следующим содержимым:

```java
import java.io.*;
import java.nio.file.*;
import java.nio.file.attribute.BasicFileAttributes;
import java.util.ArrayList;
import java.util.List;

public class ScanProject {
    private static final List<String> INCLUDED_EXTENSIONS = List.of(
        ".java", ".gradle", ".kt", ".kts", ".xml", ".yml", ".yaml", 
        ".properties", ".json", ".md", ".txt"
    );
    
    private static final List<String> EXCLUDED_DIRS = List.of(
        ".git", ".gradle", "build", "target", "out", "bin", ".idea", "node_modules"
    );
    
    public static void main(String[] args) {
        String projectRoot = args.length > 0 ? args[0] : ".";
        String outputFile = args.length > 1 ? args[1] : "project_code.txt";
        
        try {
            scanProject(projectRoot, outputFile);
            System.out.println("Project scanned successfully! Output: " + outputFile);
        } catch (IOException e) {
            System.err.println("Error scanning project: " + e.getMessage());
            e.printStackTrace();
        }
    }
    
    private static void scanProject(String rootPath, String outputFile) throws IOException {
        Path root = Paths.get(rootPath).toAbsolutePath();
        List<Path> files = new ArrayList<>();
        
        // Собираем все файлы
        Files.walkFileTree(root, new SimpleFileVisitor<Path>() {
            @Override
            public FileVisitResult preVisitDirectory(Path dir, BasicFileAttributes attrs) {
                String dirName = dir.getFileName().toString();
                if (EXCLUDED_DIRS.contains(dirName)) {
                    return FileVisitResult.SKIP_SUBTREE;
                }
                return FileVisitResult.CONTINUE;
            }
            
            @Override
            public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) {
                if (isIncludedFile(file)) {
                    files.add(file);
                }
                return FileVisitResult.CONTINUE;
            }
        });
        
        // Сортируем файлы для удобства чтения
        files.sort((p1, p2) -> {
            int depthCompare = Integer.compare(p1.getNameCount(), p2.getNameCount());
            return depthCompare != 0 ? depthCompare : p1.compareTo(p2);
        });
        
        // Записываем в выходной файл
        try (PrintWriter writer = new PrintWriter(new FileWriter(outputFile))) {
            writer.println("PROJECT STRUCTURE:");
            writer.println("==================");
            for (Path file : files) {
                writer.println(root.relativize(file));
            }
            
            writer.println("\n\nSOURCE CODE:");
            writer.println("============");
            
            for (Path file : files) {
                writer.println("\n" + "=".repeat(80));
                writer.println("FILE: " + root.relativize(file));
                writer.println("=".repeat(80));
                
                try {
                    List<String> lines = Files.readAllLines(file);
                    for (String line : lines) {
                        writer.println(line);
                    }
                } catch (IOException e) {
                    writer.println("ERROR READING FILE: " + e.getMessage());
                }
            }
        }
    }
    
    private static boolean isIncludedFile(Path file) {
        String fileName = file.getFileName().toString();
        return INCLUDED_EXTENSIONS.stream()
                .anyMatch(ext -> fileName.toLowerCase().endsWith(ext));
    }
}
```

Запустить можно командой:

```bash
java ScanProject.java "d:\project\java\udk-pdf-scanner" "result.txt"
```

В файле ScanProject есть два параметра INCLUDED_EXTENSIONS и EXCLUDED_DIRS.

**INCLUDED_EXTENSIONS** расширения файлов которые нужно включать в контекст

**EXCLUDED_DIRS** папки которые нужно исключить из сканирования.

В **result.txt** будет примерно текст такого содержания:
    - структура проекта
    - исходники кода

Весь текст нет смысла показывать там много букв, но контекст таким образом для запроса к AI создается.

````text
PROJECT STRUCTURE:
==================
pom.xml
README.md
src\main\java\ru\mcs\udk\UDKSearcher.java
src\main\java\ru\mcs\udk\factory\DocumentScannerFactory.java
src\main\java\ru\mcs\udk\scanner\DocumentScanner.java
src\main\java\ru\mcs\udk\utils\DocumentUtils.java
src\main\java\ru\mcs\udk\wrapper\DocumentFormat.java
src\main\java\ru\mcs\udk\wrapper\DocumentInfo.java
src\main\java\ru\mcs\udk\scanner\impl\DJVUScanner.java
src\main\java\ru\mcs\udk\scanner\impl\PDFScanner.java


SOURCE CODE:
============

================================================================================
FILE: pom.xml
================================================================================
<source_code>

================================================================================
FILE: README.md
================================================================================
<source_code>


================================================================================
FILE: src\main\java\ru\mcs\udk\UDKSearcher.java
================================================================================
<source_code>


================================================================================
FILE: src\main\java\ru\mcs\udk\factory\DocumentScannerFactory.java
================================================================================
<source_code>


================================================================================
FILE: src\main\java\ru\mcs\udk\utils\DocumentUtils.java
================================================================================
<source_code>


================================================================================
FILE: src\main\java\ru\mcs\udk\wrapper\DocumentFormat.java
================================================================================
<source_code>


================================================================================
FILE: src\main\java\ru\mcs\udk\wrapper\DocumentInfo.java
================================================================================
<source_code>


================================================================================
FILE: src\main\java\ru\mcs\udk\scanner\impl\DJVUScanner.java
================================================================================
<source_code>


================================================================================
FILE: src\main\java\ru\mcs\udk\scanner\impl\PDFScanner.java
================================================================================
<source_code>

}

````