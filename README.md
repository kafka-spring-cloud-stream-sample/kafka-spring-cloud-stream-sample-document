
* Kafka
    * v1.1.0
* Spring Cloud Stream
    * Finchley.BUILD-SNAPSHOT

---


# kafka

構成
* Broker
    * 3 nodes
* Topic
    * name : hoge
    * replica : 2
    * parttition : 2


```bash

### zookeeper 設定変更
vim config/zookeeper.properties
===
dataDir=zookeeper-data/
===

### zookeeper 起動
bin/zookeeper-server-start.sh config/zookeeper.properties


### Muti Broker での Kafka 起動
cp config/server.properties config/server-0.properties
cp config/server.properties config/server-1.properties
cp config/server.properties config/server-2.properties

vim config/server-0.properties
===
broker.id=0
listeners=PLAINTEXT://:9092
log.dirs=kafka-logs-0/
===

vim config/server-1.properties
===
broker.id=1
listeners=PLAINTEXT://:9093
log.dirs=kafka-logs-1/
===

vim config/server-2.properties
===
broker.id=2
listeners=PLAINTEXT://:9094
log.dirs=kafka-logs-2/
===

bin/kafka-server-start.sh config/server-0.properties
bin/kafka-server-start.sh config/server-1.properties
bin/kafka-server-start.sh config/server-2.properties


### 新しい topic 作成
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 2 --partitions 2 --topic hoge

### topic 確認
bin/kafka-topics.sh --list --zookeeper localhost:2181

### topic 状態確認 (最初の行はすべてのパーティションの要約, 追加の各行は1つのパーティションに関する情報)
bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic hoge

### zookeeper-shell
bin/zookeeper-shell.sh localhost:2181
```


# Sinkアプリケーション

```bash

cd kafka-spring-cloud-stream-sample-sink-log/

### ビルド
./gradlew clean bootJar

### デプロイ

cd ../
mkdir demo/
cd demo/
mkdir foo-1 foo-2 bar-1 bar-2

cp ../kafka-spring-cloud-stream-sample-sink-log/build/libs/kafka-spring-cloud-stream-sample-sink-log-0.0.1-SNAPSHOT.jar foo-1/.
cp ../kafka-spring-cloud-stream-sample-sink-log/build/libs/kafka-spring-cloud-stream-sample-sink-log-0.0.1-SNAPSHOT.jar foo-2/.
cp ../kafka-spring-cloud-stream-sample-sink-log/build/libs/kafka-spring-cloud-stream-sample-sink-log-0.0.1-SNAPSHOT.jar bar-1/.
cp ../kafka-spring-cloud-stream-sample-sink-log/build/libs/kafka-spring-cloud-stream-sample-sink-log-0.0.1-SNAPSHOT.jar bar-2/.

cp ../kafka-spring-cloud-stream-sample-sink-log/src/main/resources/application-foo.yml ./foo-1/application.yml
cp ../kafka-spring-cloud-stream-sample-sink-log/src/main/resources/application-foo.yml ./foo-2/application.yml
cp ../kafka-spring-cloud-stream-sample-sink-log/src/main/resources/application-bar.yml ./bar-1/application.yml
cp ../kafka-spring-cloud-stream-sample-sink-log/src/main/resources/application-bar.yml ./bar-2/application.yml

### 実行
./kafka-spring-cloud-stream-sample-sink-log-0.0.1-SNAPSHOT.jar
```


# Sourceアプリケーション

```bash

cd kafka-spring-cloud-stream-sample-source-http/

### ビルド
./gradlew clean bootJar

### 実行
./kafka-spring-cloud-stream-sample-source-http-0.0.1-SNAPSHOT.jar

### リクエスト
curl -XPOST -H 'Content-Type:application/json' -d '{"id":"a"}' localhost:8080/hoge

```


