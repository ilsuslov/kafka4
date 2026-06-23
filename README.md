# Запускаю все сервисы:
docker compose up -d
docker compose logs kafka1 | grep "started"

# Создание топиков и настройка ACL

# Создаю topic-1 и topic-2
docker compose exec kafka1 kafka-topics.sh --bootstrap-server localhost:9093 \
  --create --topic topic-1 --replication-factor 3 --partitions 3

docker compose exec kafka1 kafka-topics.sh --bootstrap-server localhost:9093 \
  --create --topic topic-2 --replication-factor 3 --partitions 3

# Для topic-1: разрешаю чтение и запись клиенту
docker compose exec kafka1 kafka-acls.sh --bootstrap-server localhost:9093 \
  --add --allow-principal User:CN=kafka-client \
  --operation Read --operation Write --topic topic-1

# Для topic-2: разрешаю только запись (чтение запрещено)
docker compose exec kafka1 kafka-acls.sh --bootstrap-server localhost:9093 \
  --add --allow-principal User:CN=kafka-client \
  --operation Write --topic topic-2

# Проверяю, что ACL применились
docker compose exec kafka1 kafka-acls.sh --bootstrap-server localhost:9093 --list


# Конфигурация клиента
Создал файл client-ssl.properties для подключения клиента с SSL:

security.protocol=SSL
ssl.truststore.location=./certs/client/client.truststore.jks
ssl.truststore.password=changeit
ssl.keystore.location=./certs/client/client.keystore.jks
ssl.keystore.password=changeit
ssl.key.password=changeit


# Тестирование
Тест 1: topic-1 (чтение и запись разрешены)
Записываю сообщение:
docker compose exec kafka1 kafka-console-producer.sh --broker-list localhost:9093 \
  --topic topic-1 --producer.config /opt/kafka/client-ssl.properties

 Читаю сообщение:
docker compose exec kafka1 kafka-console-consumer.sh --bootstrap-server localhost:9093 \
  --topic topic-1 --from-beginning --consumer.config /opt/kafka/client-ssl.properties --timeout-ms 5000

  Тест 2: topic-2 (только запись, чтение запрещено)
  docker compose exec kafka1 kafka-console-producer.sh --broker-list localhost:9093 \
  --topic topic-2 --producer.config /opt/kafka/client-ssl.properties

  docker compose exec kafka1 kafka-console-consumer.sh --bootstrap-server localhost:9093 \
  --topic topic-2 --from-beginning --consumer.config /opt/kafka/client-ssl.properties --timeout-ms 5000
# Чтение (должно завершиться ошибкой NotAuthorizedException)
  
