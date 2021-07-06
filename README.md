# Docker Multi-Stage Builds範例說明

## 說明

* Multi-Stage Builds是Docker在17.05版本引入的功能，可以解決在複雜部署環境需要同時維護多個Dockerfile的麻煩。
* Multi-Stage Builds是一個Dockerfile可以有多個建置階段，每一個Stage可以使用前面建置步驟產生的中繼 (Intermediate)檔案 (artifacts)，
  丟棄過程中不必要的檔案，使整個建置過程更簡潔，也可使產生的Image變得更小。

## 範例

* 為了可以將Spring Boot應用程式打包進Image，需要在pom.xml加上以下設定：
    ```xml
    <project ...>
        <!-- other settiungs -->
        <build>
            <plugins>
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                    <executions>
                        <execution>
                            <goals>
                                <goal>repackage</goal>
                            </goals>
                        </execution>
                    </executions>
                </plugin>
            </plugins>
        </build>
        <!-- other settings -->
    </project>
    ```
  若不加上該設定，會導致Container無法執行並在log中顯示錯誤訊息： `no main manifest attribute, in target/<jar filename>.jar`
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
  * 使用Tag名稱為`app`的Image建立Container，並將Container的port 8080對應到本機的port
    8080，如此就可以直接在瀏覽器輸入[http://localhost:8080/hello](http://localhost:8080/hello)
    訪問服務並取得回傳資料。
* 採用Multi-stage builds建置出來的Image大小，比不使用Multi-stage builds建置出來的Image要小

  | 使用Multi-stage builds所建出來的Image大小 | 不使用Multi-stage builds所建出來的Image大小 |
    | ----------------------------------------- | ------------------------------------------- |
  | 106.78 MB                                 | 125.62 MB                                   |

## 參考資料

### Docker

* [Use multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/)

### Others

* [透過Multi-Stage Builds改善持續交付流程](https://tachingchen.com/tw/blog/docker-multi-stage-builds/)