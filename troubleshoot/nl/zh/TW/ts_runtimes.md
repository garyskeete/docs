---

copyright:
  years: 2015, 2016
  
lastupdated: "2016-08-18"

---

{:tsSymptoms: .tsSymptoms} 
{:tsCauses: .tsCauses} 
{:tsResolve: .tsResolve} 
{:new_window: target="_blank"}  
{:shortdesc: .shortdesc}
{:codeblock: .codeblock} 





# 運行環境疑難排解
{: #runtimes}



您在使用 IBM® Bluemix™ 運行環境時可能會遇到問題。然而，在許多情況下，您可以依照下列一些簡單的步驟，從這些問題中回復。
{:shortdesc}


## 推送應用程式時，使用已作廢的建置套件
{: #ts_loading_bp}


您推送應用程式時可能無法使用最新的建置套件元件。您可以使用具有內建機制的建置套件，以避免載入已作廢的元件，或者您可以刪除應用程式快取目錄中的內容，然後才推送或重新編譯打包應用程式。 

 

當您在建置套件更新之後推送或重新編譯打包應用程式時，不會自動載入最新的建置套件元件。因此，您的應用程式會使用快取中已作廢的建置套件元件。將不會實作自從您前次推送應用程式後已套用至建置套件的更新。
{: tsSymptoms}



部分建置套件未配置為自動從網際網路下載所有更新的元件，以確保您隨時使用最新的版本。
{: tsCauses} 

 

您可以使用具有內建機制的建置套件來避免載入已作廢的元件。下列建置套件是其中兩個範例：
{: tsResolve}

  * [Cloud Foundry Java 建置套件](https://github.com/cloudfoundry/java-buildpack){: new_window}。這個建置套件具有內建的機制，可以確保使用最新版本的建置套件。如需此機制運作方式的相關資訊，請參閱 [extending-caches.md](https://github.com/cloudfoundry/java-buildpack/blob/master/docs/extending-caches.md){: new_window}。 
  * [Cloud Foundry Node.js 建置套件](https://github.com/cloudfoundry/nodejs-buildpack){: new_window}。這個建置套件功能與使用環境變數類似。為了讓 Node.js 建置套件能每次從網際網路下載 node 模組，請在 cf 指令行介面中，鍵入下列指令： 	
  ```
set NODE_MODULES_CACHE=false
```
如果您使用的建置套件未提供自動載入最新元件的機制，可以手動刪除快取目錄中的內容，然後採取下列步驟來重新推送應用程式：
  1. 移出空值建置套件的分支，例如 https://github.com/ryandotsmith/null-buildpack。如需如何移出分支的相關資訊，請參閱 [Git Basics - Getting a Git Repository](http://www.git-scm.com/book/en/v2/Git-Basics-Getting-a-Git-Repository){: new_window}。  
  2. 將下行新增到 `null-buildpack/bin/compile` 檔案並確定變更。如需如何確定變更的相關資訊，請參閱 [Git Basics - Recording Changes to the Repository](http://www.git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository){: new_window}。
  ```
rm -rfv $2/*
```
  3. 使用下列指令，用已修改的空值建置套件推送應用程式，以刪除快取。完成此步驟之後，應用程式快取目錄中的所有內容都會刪除。
  ```
cf push appname -p app_path -b <modified_null_buildpack>
```
  4. 使用下列指令，用您想要使用的最新建置套件來推送應用程式： 
  ```
cf push appname -p app_path -b <latest_buildpack>
```
  
	


## PHP 建置套件產生的 NOTICE 訊息
{: #ts_phplog}

您可能會在日誌中看到包含 NOTICE 的訊息。您可以變更記載層次，以停止記載這些訊息。	
	
 

當您使用 PHP 建置套件將應用程式推送至 Bluemix 時，可能會看到包含 `NOTICE` 的訊息：
{: tsSymptoms}

```
• 2015-01-26T15:00:59.70+0100 [App/0] ERR [26-Jan-2015 14:00:59] NOTICE: [pool www] 'user' directive is ignored when FPM is not running as root
• 2015-01-26T15:01:00.63+0100 [App/0] ERR [26-Jan-2015 14:00:59] NOTICE: [pool www] 'user' directive is ignored when FPM is not running as root
• 2015-01-26T15:01:00.63+0100 [App/0] ERR [26-Jan-2015 14:00:59] NOTICE: fpm is running, pid 93
• 2015-01-26T15:01:00.63+0100 [App/0] ERR [26-Jan-2015 14:00:59] NOTICE: ready to handle connections
```



在 PHP 建置套件中，error_log 參數可用來定義記載層次。依預設，`error_log` 參數的值為 **stderr notice**。下列範例顯示 Cloud Foundry 所提供之 PHP 建置套件的 `nginx-defaults.conf` 檔案中的預設記載層次配置。如需相關資訊，請參閱 [cloudfoundry/php-buildpack](https://github.com/cloudfoundry/php-buildpack/blob/ff71ea41d00c1226d339e83cf2c7d6dda6c590ef/defaults/config/nginx/1.5.x/nginx-defaults.conf){: new_window}。
{: tsCauses} 

```
daemon off;
error_log stderr notice;
pid @{HOME}/nginx/logs/nginx.pid;
```

 	
	
`NOTICE` 訊息是參考資訊，不一定表示有發生問題。若要停止記載這些訊息，您可以在建置套件的 nginx-defaults.conf 檔案中，將記載層次從 stderr notice 變更為 stderr error。例如：
{: tsResolve}

```
daemon off;
error_log stderr error;
pid @{HOME}/nginx/logs/nginx.pid;
```
如需如何變更預設記載配置的相關資訊，請參閱 [error_log](http://nginx.org/en/docs/ngx_core_module.html#error_log){: new_window}。
	

## 無法將協力廠商的 Python 檔案庫匯入 {{site.data.keyword.Bluemix_notm}}
{: #ts_importpylib}

您可能無法將協力廠商的 Python 檔案庫匯入 {{site.data.keyword.Bluemix_notm}}。將配置檔新增至 Python 應用程式的根目錄中，即可解決這個問題。


當您嘗試匯入協力廠商的 Python 檔案庫（例如 `web.py` 檔案庫）時，`cf push` 指令會失敗。
{: tsSymptoms}


 

當 Python 應用程式的配置資訊遺失時，就會發生這個問題。
{: tsCauses}


 

若要解決此問題，請在 Python 應用程式的根目錄中，新增 `requirements.txt` 檔案及 `Procfile` 檔案。下列資訊是假設您要匯入 web.py 檔案庫：
{: tsResolve}

  1. 在 Python 應用程式的根目錄中，新增 `requirements.txt` 檔案。`requirements.txt` 檔案可指定 Python 應用程式所需的檔案庫套件，以及套件的版本。下列範例顯示 `requirements.txt` 檔案的內容，其中 `web.py==0.37` 表示將要下載的 `web.py` 檔案庫版本為 0.37，而 `wsgiref==0.1.2` 表示 web.py 檔案庫所需的 Web 伺服器閘道介面版本為 0.1.2。
	 ```
	 web.py==0.37
     wsgiref==0.1.2
```
	如需如何配置 `requirements.txt` 檔案的相關資訊，請參閱[需求檔案](https://pip.readthedocs.org/en/1.1/requirements.html)。
  2. 在 Python 應用程式的根目錄中，新增 `Procfile` 檔案。`Procfile` 檔案必須包含 Python 應用程式的啟動指令。在下列指令中，*yourappname* 是 Python 應用程式的名稱，而 *PORT* 是 Python 應用程式必須用來接收應用程式使用者要求的埠號。*$PORT* 是選用項目。如果您未於啟動指令中指定 PORT，則會改用應用程式內部的 `VCAP_APP_PORT` 環境變數下的埠號。 
	```
	web: python <yourappname>.py $PORT
```
您現在可以將協力廠商的 Python 檔案庫匯入 {{site.data.keyword.Bluemix_notm}} 了。



## 已停用「實例詳細資料」頁面上的「動作」按鈕
{: #ts_actionsbutton}



已停用「實例詳細資料」頁面上的「動作」按鈕。
{: tsSymptoms} 

 

此問題的發生原因如下：
{: tsCauses}

  * 應用程式不是 Java™ Web 應用程式。Runtime Management Utilities (RMU) 只支援使用 Liberty 建置套件所部署的 Web 應用程式。
  * 應用程式不是使用內嵌的 Liberty 建置套件所部署。
  * 應用程式是使用舊版 Liberty 建置套件所部署。



如果問題是舊版 Liberty 建置套件所造成，請在 {{site.data.keyword.Bluemix_notm}} 中重新部署應用程式。否則，您可以將下列用戶端應用程式日誌檔提供給支援團隊：
{: tsResolve} 

  * logs/messages.log
  * logs/stdout.log
  * logs/stderr.log
  

  
  
## 開啟追蹤或傾出視窗時需要認證
{: #ts_username}


 

開啟追蹤或傾出視窗時，需要使用者名稱及密碼。
{: tsSymptoms}

 

此問題的發生原因是登入階段作業逾時。
{: tsCauses}

 

解決方案是重新輸入使用者名稱及密碼。
{: tsResolve}




## 追蹤或傾出作業執行時發生錯誤
{: #ts_target}

 

當追蹤或傾出作業在執行時顯示錯誤訊息。訊息指出應用程式的目標實例不在執行中狀態：
{: tsSymptoms}

```
實例 0：已順利設定追蹤規格
實例 2：已順利設定追蹤規格
實例 1：應用程式 abcd 的目標實例 1 未處於執行中狀態。
實例 3：已順利設定追蹤規格
實例 4：已順利設定追蹤規格
```



此問題的發生原因如下：
{: tsCauses} 

  * 只有執行中應用程式實例才有追蹤或傾出管理功能。無法對已停止、啟動中或已損毀的應用程式實例執行追蹤或傾出作業。
  * 開啟追蹤或傾出對話框時，應用程式實例的狀態為變更中。 
  


解決方案是關閉視窗，然後再重新開啟。
{: tsResolve} 



## 實例具有不同的 traceSpecification 配置
{: #ts_different_config}

 

對於某個應用程式的不同實例，您可能會看到不同的 traceSpecification 配置。
{: tsSymptoms}

 

此問題的發生原因如下：
{: tsCauses}

  * 您先前可能已變更一個以上實例的配置。如果您變更某個實例的 traceSpecification 配置，則它不會套用至相同應用程式的其他實例。例如，您的應用程式使用 log4j，而且您具有這個應用程式的兩個實例。您可以將實例 0 的記載層次從 info 變更為 debug，但實例 1 的記載層次仍為 info。 
  * 應用程式橫向擴充，並具有新的實例。RMU 不會將現有實例的 traceSpecification 配置套用至新的橫向擴充實例。新的實例會使用預設配置。例如，您的應用程式使用 log4j，而且它有一個實例。您可以將此實例的記載層次從 info 變更為 debug。進行這項變更之後，如果您將應用程式橫向擴充至兩個實例，則新實例的記載層次是 info，而非 debug。
  


這是預期的行為。
{: tsResolve} 





## 已超出磁碟限額
{: #ts_diskquota}

您可能會在應用程式日誌中看到已超出磁碟限額。

 

您在應用程式的日誌中看到`已超出磁碟限額`訊息。
{: tsSymptoms}



此問題是下列其中一個原因所導致：
{: tsCauses} 

  * 傾出檔案是與執行中應用程式實例一起產生，而且檔案會耗盡配置的磁碟限額。一個應用程式實例的磁碟限額預設為 1 GB。您可以按一下**儀表板 > 應用程式 > 應用程式運行環境**，來檢查您的磁碟用量。下列範例顯示兩個應用程式實例的運行環境資訊（包括磁碟用量）：

    ```
Instance	State	CPU	Memory Usage	Disk Usage
0		Running	1.0%	344.8MB/512MB	236.8MB/1GB
	2		Running	2.3%	361.2MB/512MB	235.7MB/1GB
    ```
  * 磁碟限額受限於現行組織配額。
  
  


您可以使用下列其中一種方法來解決這個問題：
{: tsResolve} 

  * 在下載傾出檔案之後刪除它們。
  * 在部署資訊清單中包括下列項目，以較大的磁碟限額來重新部署應用程式：
    ```
	disk_quota: 2048
```
	
	
<!-- begin STAGING ONLY --> 

	
## Log4js 日誌程式物件未顯示在「Node.js 追蹤」蹦現視窗中
{: #ts_logger}

在應用程式中同時使用 log4js 及 ibmbluemix 模組時，log4js 日誌程式物件未顯示在「Node.js 追蹤」蹦現視窗中。 	

 
在應用程式中同時使用 log4js、winston 及 ibmbluemix 模組時，log4js 日誌程式物件未顯示在「Node.js 追蹤」蹦現視窗中。
{: tsSymptoms}


因為 ibmbluemix 模組針對使用 log4js 及 winston 模組的日誌作業提供統一的 API，所以只有 ibmbluemix 日誌程式物件會顯示在「Node.js 追蹤」蹦現視窗中。這是要停止 ibmbluemix、log4js 及 winston 日誌程式物件互相改寫的記載層次設定。
{: tsCauses}


這是預期的行為。
{: tsResolve}

<!-- end STAGING ONLY -->




<!-- begin STAGING ONLY -->


## 已停用「將追蹤設定套用至應用程式的所有實例」勾選框
{: #ts_bunyan}

修改 Bunyan 日誌程式層次時，已在「Node.js 追蹤」蹦現視窗中取消勾選及停用**將追蹤設定套用至應用程式的所有實例**勾選框。



當您變更 Bunyan 日誌程式物件的層次時，已在「Node.js 追蹤」蹦現視窗中取消勾選及停用**將追蹤設定套用至應用程式的所有實例**勾選框。
{: tsSymptoms} 

 

修改 Bunyan 記載層次時，追蹤設定無法套用至應用程式的所有實例。原因是 Bunyan 程式庫不需要 Bunyan 日誌程式物件的名稱或 ID 是唯一的。應用程式的日誌訊息中用來指定層次的多個 Bunyan 日誌程式物件可能會有相同的名稱或 ID。因此，如果啟用應用程式的追蹤設定，則應用程式的日誌訊息中所指定的記載層次可能不正確。
{: tsCauses}




這是預期的行為。
{: tsResolve} 

<!-- end STAGING ONLY -->

