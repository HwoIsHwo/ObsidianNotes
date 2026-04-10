Рассматривается установка Apache Ant и Apache Ivy без использования пакетного менеджера. Такой подход даёт контроль над версиями и путями, что важно в изолированных или воспроизводимых окружениях.

---
## Установка Apache Ant
Сначала требуется установленная Java (JDK). Проверка:
```bash
java -version
javac -version
```
Если JDK отсутствует, его необходимо установить заранее.

Далее скачивается архив Ant с официального сайта и распаковывается:
```bash
wget https://downloads.apache.org/ant/binaries/apache-ant-1.10.14-bin.tar.gz
tar -xzf apache-ant-1.10.14-bin.tar.gz
sudo mv apache-ant-1.10.14 /opt/ant
```
После этого настраиваются переменные окружения. Удобнее всего сделать это через `~/.bashrc` или `~/.profile`:
```bash
export ANT_HOME=/opt/ant
export PATH=$ANT_HOME/bin:$PATH
```
Применение изменений:
```bash
source ~/.bashrc
```
Проверка установки:
```bash
ant -version
```
Ожидается вывод версии Ant. Если команда не найдена — проблема с PATH. Если ошибка JVM — проблема с Java.

---
## Минимальная проверка Ant
Создаётся тестовый проект:
```bash
mkdir ant-test && cd ant-test
```
Файл `build.xml`:
```xml
<project name="TestAnt" default="hello">

    <target name="hello">
        <echo message="Ant работает"/>
    </target>

</project>
```
Запуск:
```bash
ant
```
Если выводится сообщение `Ant работает`, сборка функционирует корректно.

---
## Установка Apache Ivy
Ivy распространяется как отдельный JAR-файл. Его необходимо скачать и подключить к Ant.
```bash
wget https://downloads.apache.org/ant/ivy/2.5.2/apache-ivy-2.5.2-bin.tar.gz
tar -xzf apache-ivy-2.5.2-bin.tar.gz
```
Основной файл: `ivy-2.5.2.jar`.
Для интеграции с Ant его нужно поместить в директорию библиотек Ant:
```bash
sudo cp apache-ivy-2.5.2/ivy-2.5.2.jar /opt/ant/lib/
```
После этого Ant автоматически подхватит Ivy как набор дополнительных задач.
Проверка доступности Ivy:
```bash
ant -diagnostics | grep ivy
```
Если Ivy подключён, он будет отображаться в списке доступных библиотек.

---
## Минимальная проверка Ivy
Создаётся новый проект:
```bash
mkdir ivy-test && cd ivy-test
```
Файл `ivy.xml` (описание зависимостей):
```xml
<ivy-module version="2.0">
    <info organisation="test" module="demo"/>

    <dependencies>
        <dependency org="junit" name="junit" rev="4.13.2"/>
    </dependencies>
</ivy-module>
```
Файл `build.xml`:
```xml
<project name="TestIvy" xmlns:ivy="antlib:org.apache.ivy.ant">

    <!-- Загрузка зависимостей -->
    <target name="resolve">
        <ivy:resolve/>
        <ivy:retrieve pattern="lib/[artifact]-[revision].[ext]"/>
    </target>

</project>
```
Запуск:
```bash
ant resolve
```
Если установка корректна, в директории `lib` появится JAR-файл `junit-4.13.2.jar`.

Это означает:
* Ivy корректно подключён
* доступ к репозиториям работает
* зависимости успешно разрешаются

## Типичные проблемы
Если `ivy:resolve` не распознаётся, значит:
* Ivy не скопирован в `/opt/ant/lib`
* или используется другой экземпляр Ant

Если ошибка сети при загрузке:
* проверить доступ к Maven Central
* проверить настройки proxy

Если Ant не видит Java:
* проверить переменную `JAVA_HOME`
```bash
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
```

## Итог
Ручная установка Ant и Ivy сводится к:
* размещению Ant в системе и настройке PATH
* копированию Ivy JAR в каталог `lib` Ant
Работоспособность проверяется через:
* `ant -version`
* выполнение простого `build.xml`
* загрузку зависимости через Ivy
Такая конфигурация даёт минимально необходимую среду для сборки Java-проектов с управлением зависимостями без использования Maven или Gradle.

Теги: #Прога 
Ссылки:
[[Ant и Ivy]]