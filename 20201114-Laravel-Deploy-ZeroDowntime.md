# Laravel - Deploy ZeroDowntime

# 前行提要
這是 `PHP7 (opcache)` + `Laravel` + `Nginx` + `PHP-FPM` ZeroDowntime 部署經驗

# 問題點
## PHP7 (opcache)
在 `PHP7` 開啟 opcache 提升效能已成了必須之一  
但是 opcache 開啟其中一個問題就是會把 code cache 起來，導致部屬之後線上的版本還是舊的  

然而天真的我一直都是使用 `sudo service php-fpm reload/restart`  

但是這樣會有什麼問題?  重啟的瞬間正在執行的長事務就會斷掉  
就會被客戶壓在地上磨擦了...

這時候我們就需要 `opcache_reset()` 來讓 `PHP-FPM` 刷新 cache  

要注意的是 `PHP-FPM` 有分兩個
1. `cli` 為 terminal 使用的是沒有 opcache 的
2. `fpm` 為 web 使用的才有 opcache

所以從 terminal 去下 `php -r 'opcache_reset();'`  
只會刷新 `cli` 但是瑞凡他沒 cache，想當然 `fpm` 的不會被刷新到  

這時候有兩種解決方法
1. 創建一個 api 裡面寫 ```<?php opcache_reset()``` 部署完戳一下
2. 使用 [cachetool](https://gordalina.github.io/cachetool/) 人家寫好的工具，如果要看細節請點進去查看  
基本上是跟第一個方法一樣，差別是你可以在部署的腳本加上 `cachetool.phar opcache:reset` 就可以清除 opcache cache 了

這樣部署之前的長事務並不會被中斷，並且之後的都會是跑新的程式  
這樣清除 opcache cache downtime 問題就解決了

## Laravel (Queue)
Job 排程在 Laravel 也是常用的服務之一  

那部署的時候怎麼更新 Supervisor worker 的舊 code ?  

之前我一直都是直接 `sudo supervisorctl restart all`  
就會跟上面一樣正在執行中的事務就斷了，如果其中有什麼業務邏輯是無法 rollback 的...

如果你有好好看官方文件那就是 `php artisan queue:restart`

那你知道他是怎麼運作嗎? 是直接一樣 restart supervisorctl 還是奇怪的做法? 
 
追了原始碼 Laravel7.x 實作方式
下了 `php artisan queue:restart` 指令會在 cache 放一個 `illuminate:queue:restart` 標記時間  

然後 `php artisan queue:work` 後會有一個 while 迴圈在跑  
`檢查是否為 laravel Maintenance` -> `執行 Job` -> `檢查 cache 中的 'illuminate:queue:restart'` 標記如果大於 worker 的紀錄時間就重啟 worker` 

所以部署完 `php artisan queue:restart` 可以讓你之前正在執行中的 Job 執行完  
下一個 Job 開始才是跑新的程式碼