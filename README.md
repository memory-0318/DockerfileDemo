# Dockerfile範例說明

## Dockerfile說明
* 將Dockerfile放在根目錄下，內容與說明如下：
```dockerfile
# 第一個Stage，Stage命名為build，使用maven:3-openjdk-8這個Image
# 這個Stage目標是使用maven編譯並產生jar檔
FROM maven:3-openjdk-8 AS build
# 在Image中產生projects目錄
RUN mkdir /projects
# 將Dockerfile所在目錄下的檔案與目錄 (含子目錄)，複製到/projects目錄中
COPY . /projects
# 將當前工作目錄設定為./projects
WORKDIR /projects
# 執行mvn指令產生jar檔
RUN mvn clean package

# 第二個Stage
# 這個Stage目的是將jar放進Image
FROM openjdk:8-jre-alpine
# 在image中創建/app目錄
RUN mkdir /app
# 將前一個Stage產生的jar檔，複製到當前Image的/app目錄，並將jar檔更名為application.jar
COPY --from=build /projects/target/*.jar /app/application.jar
# 將當前工作目錄設定為/app目錄
WORKDIR /app
# 設定jar檔執行執行指令，當Container建立後會執行以下指令
CMD "java" "-jar" "application.jar"
```
* 執行`docker build -t app .`產生Image，並將產生的Image加上Tag：`app`
  > 建置的過程會比較久，大約3~5分鐘不等，或甚至更長
* 執行`docker run -p 8080:8080 -t app`
  * 使用Tag名稱為`app`的Image建立Container，並將Container的port 8080對應到本機的port 8080，如此就可以直接在瀏覽器輸入[http://localhost:8080/hello](http://localhost:8080/hello) 
  訪問服務並取得回傳資料。