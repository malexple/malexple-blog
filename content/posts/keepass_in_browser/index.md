+++
title = "Настройка KeePass для браузера"
draft = false
date = 2024-08-21
[taxonomies]
categories = ["KeePass"]
tags = ["keepass", "automation"]
+++


1. Скачать KeePass на сайте разработчика [KeePass](https://keepass.info/download.html)

   ![image-20200806100532013](image-20200806100532013.png)

2. Скачать plugin **KeePassHttp** [тут](https://keepass.info/plugins.html)

   ![image-20200806101459512](image-20200806101459512.png) Данный плагин необходимо поместить в папку Plugins в папке с программой

3. Скачать расширение для браузера [ChromeKeePass](https://chrome.google.com/webstore/detail/chromekeepass/dphoaaiomekdhacmfoblfblmncpnbahm)

   ![image-20200806100732809](image-20200806100732809.png)

4. Если у вас иной браузер и он не поддерживает расширение ChromeKeePass можно поискать плагин позволяющий запускать chrome плагин.

   Для Opera:  [Install Chrome Extention](https://addons.opera.com/ru/extensions/details/install-chrome-extensions/)

5. Запускаем KeePass, запускаем браузер. Нажимаем на значек плагина. Откроется окно с настройками:

![image-20200807233215796](image-20200807233215796.png)

Нажимаем **Connect**

![image-20200807233317973](image-20200807233317973.png)

Вводим название ключа (даем любое имя).

![image-20200807234038663](image-20200807234038663.png)

В KeePass будет создан ключ ![image-20200807234531974](image-20200807234531974.png)

Нажмите правой кнопкой на ключе  и скопируйте AES Key. Его необходимо добавить в пароли используемые для web страниц.

![image-20200807234724410](image-20200807234724410.png)

Создайте пароль или откройте меню изменения пароля, во вкладке **Дополнительно** добавьте строкое поле, вставив в поле скопированный AES Key. Важно иметь ввиду, что KeePass в буфере обмена хранит значения не больше 12 сек. Это нужно повторить для всех web паролей.

![image-20200807235403665](image-20200807235403665.png)