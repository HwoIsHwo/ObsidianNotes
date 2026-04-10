При интеграции Apache Ant и Apache Ivy с JFrog Artifactory основная сложность связана не с самой загрузкой зависимостей, а с настройкой доступа: креды, токены, SSL и корректный resolver.

Важно понимать, что Ivy работает поверх HTTP(S) и воспринимает Artifactory как Maven-совместимый репозиторий. Поэтому вся интеграция строится вокруг конфигурации `ivysettings.xml`.


---
## Базовая схема подключения
Обычно используется ibiblio-resolver с Maven-совместимым layout:
```xml
<ivysettings>
    <resolvers>
        <ibiblio name="artifactory"
                 root="https://artifactory.example.com/artifactory/libs-release"
                 m2compatible="true"/>
    </resolvers>
</ivysettings>
```
Без `m2compatible="true"` Ivy будет использовать свой layout и не сможет корректно находить артефакты в Artifactory, который почти всегда настроен под Maven-структуру.


---
## Аутентификация (Basic Auth и токены)
Самый простой способ — через `<credentials>`. Ivy автоматически сопоставляет host и применяет их к HTTP-запросам.
```xml
<ivysettings>
    <credentials host="artifactory.example.com"
                 realm="Artifactory Realm"
                 username="user"
                 passwd="password"/>

    <resolvers>
        <ibiblio name="artifactory"
                 root="https://artifactory.example.com/artifactory/libs-release"
                 m2compatible="true"/>
    </resolvers>
</ivysettings>
```
На практике пароль часто заменяется API token’ом. Для Artifactory он обычно используется как password в Basic Auth:
```xml
<credentials host="artifactory.example.com"
             username="user"
             passwd="API_TOKEN"/>
```
Ключевой момент: `host` должен совпадать с доменом в URL один в один. Даже различие в поддомене приведёт к игнорированию credentials.


---
## Подключение ivysettings в Ant
Ivy не использует настройки автоматически, их нужно явно включить в Ant build:
```xml
<target name="resolve">
    <ivy:settings file="ivysettings.xml"/>
    <ivy:resolve/>
</target>
```
Без этого Ivy уйдёт в дефолтные репозитории (обычно Maven Central).


---
## Альтернативный способ: URL resolver
Иногда используют явное описание URL-шаблонов:
```xml
<ivysettings>
    <resolvers>
        <url name="artifactory">
            <artifact pattern="https://artifactory.example.com/artifactory/libs-release/[organisation]/[module]/[revision]/[artifact]-[revision].[ext]"/>
            <ivy pattern="https://artifactory.example.com/artifactory/libs-release/[organisation]/[module]/[revision]/ivy-[revision].xml"/>
        </url>
    </resolvers>
</ivysettings>
```
Этот вариант менее удобен, но даёт полный контроль над структурой репозитория.


---
## Передача токенов через JVM параметры
В некоторых конфигурациях используется HTTP header:
```bash
ANT_OPTS="-Divy.http.header.Authorization=Bearer <TOKEN>"
```
Этот подход зависит от версии Ivy и инфраструктуры Artifactory. Он менее стандартный, чем Basic Auth.

---
## SSL и корпоративные сертификаты
Если Artifactory использует самоподписанный сертификат, Java может блокировать соединение. Тогда сертификат добавляется в truststore:
```bash
keytool -import -trustcacerts -alias artifactory \
    -file cert.pem \
    -keystore $JAVA_HOME/lib/security/cacerts
```
Без этого Ivy будет падать с SSLHandshakeException.


---
## Proxy-конфигурация
В корпоративных сетях часто требуется proxy:
```bash
ANT_OPTS="-Dhttp.proxyHost=proxy.company.local -Dhttp.proxyPort=8080"
```
При HTTPS аналогично используется `https.proxyHost`.


---
## Кэш Ivy и проблемы доступа
Ivy активно кэширует зависимости в `~/.ivy2/cache`. Это может приводить к ложным ошибкам при смене credentials или URL.
Типичная мера диагностики:
```bash
rm -rf ~/.ivy2/cache
```
Также стоит учитывать `~/.ivy2/local`, если используется локальная публикация.


---
## Публикация в Artifactory
При публикации используется `ivy:publish`, и credentials должны быть доступны так же, как при resolve:
```xml
<target name="publish">
    <ivy:publish resolver="artifactory"
                 publishivy="true"/>
</target>
```
Ошибки 401/403 почти всегда связаны с отсутствием или неверной настройкой credentials.


---
## Типичные проблемы
На практике чаще всего встречаются следующие ошибки:
- неверный `host` в `<credentials>`
- отсутствие `ivy:settings` в build-файле
- несоответствие Maven layout (`m2compatible`)
- SSL ошибки из-за сертификатов
- proxy, который не передан JVM
- использование токена не в том формате (Bearer vs Basic)
- устаревший кэш Ivy


---
## Настройка Proxy  и SSL
В связке Ant/Ivy сетевое взаимодействие полностью опирается на JVM, поэтому настройки proxy и SSL не являются частью Ant или Ivy-конфигураций. Они задаются через системные параметры Java, либо через модификацию truststore.

Proxy настраивается через стандартные system properties. Эти параметры подхватываются автоматически всеми HTTP-клиентами внутри JVM, включая Ivy:
```bash
export ANT_OPTS="-Dhttp.proxyHost=proxy.company.local \
-Dhttp.proxyPort=8080 \
-Dhttps.proxyHost=proxy.company.local \
-Dhttps.proxyPort=8080"
```

Если proxy требует авторизацию, добавляются дополнительные параметры:
```bash
export ANT_OPTS="-Dhttp.proxyHost=proxy.company.local \
-Dhttp.proxyPort=8080 \
-Dhttp.proxyUser=user \
-Dhttp.proxyPassword=pass \
-Dhttps.proxyHost=proxy.company.local \
-Dhttps.proxyPort=8080"
```

Альтернативный вариант — `JAVA_TOOL_OPTIONS`. Он менее привязан к Ant и влияет на все Java-приложения в системе:
```bash
export JAVA_TOOL_OPTIONS="$JAVA_TOOL_OPTIONS \
-Dhttp.proxyHost=proxy.company.local \
-Dhttp.proxyPort=8080 \
-Dhttps.proxyHost=proxy.company.local \
-Dhttps.proxyPort=8080"
```
С точки зрения диагностики proxy-слой обычно проверяется отдельно через `curl`. Если `curl` с `-x` работает, а Ivy нет — проблема почти всегда в том, что JVM-параметры не были подхвачены.


---
SSL/TLS устроен иначе. Здесь проблема находится не в сетевом уровне, а в доверии к сертификатам внутри Java. JVM использует truststore, расположенный в:
```text
$JAVA_HOME/lib/security/cacerts
```
Если сервер (например Artifactory) использует self-signed или корпоративный сертификат, возникает ошибка вида:
```text
javax.net.ssl.SSLHandshakeException: PKIX path building failed
```
Это означает, что цепочка сертификатов не доверена JVM.


---
Самый прямой способ решения — импорт сертификата в системный truststore:
```bash
keytool -import \
  -alias artifactory \
  -file cert.pem \
  -keystore $JAVA_HOME/lib/security/cacerts
```
Пароль truststore по умолчанию — `changeit`.

Этот подход глобальный и влияет на всю Java-среду, что может быть нежелательно в CI или многопользовательских системах.


---
Более изолированный вариант — отдельный truststore:
```bash
keytool -import \
  -alias artifactory \
  -file cert.pem \
  -keystore ./truststore.jks
```
И подключение через JVM параметры:
```bash
export ANT_OPTS="-Djavax.net.ssl.trustStore=/path/to/truststore.jks \
-Djavax.net.ssl.trustStorePassword=pass"
```
Этот вариант предпочтительнее для контейнеров и CI/CD, так как не модифицирует системную Java.


---
Типовые проблемы SSL обычно связаны не с отсутствием сертификата как такового, а с:
- неполной цепочкой (нет intermediate CA) 
- несовпадением hostname сертификата и URL
- использованием IP вместо доменного имени
- устаревшей версией JDK с ограниченной поддержкой TLS

---
Для диагностики TLS полезен встроенный debug режим Java:
```bash
export ANT_OPTS="-Djavax.net.debug=ssl,handshake"
```
Он выводит процесс TLS handshake и позволяет определить, на каком этапе соединение разрывается.

---
proxy решается исключительно JVM system properties, SSL — через truststore. Ant и Ivy здесь не участвуют напрямую, они лишь используют уже настроенный сетевой слой Java. Основные ошибки возникают не на уровне конфигурации Ant/Ivy, а на уровне несоответствия сетевого окружения и доверенных сертификатов JVM.

Теги: #Прога 
Ссылки:
[[Ant и Ivy]]