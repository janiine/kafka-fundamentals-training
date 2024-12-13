# Kafka-fundamentals-training


For this training, you need:
- Kafka
- Python


## Plan

- Preparing the environment
  - Installing kafka and python
  - Running Kafka and Zookeeper
- Create topics
- Create producer
- Create consumer
- Extended Kafka Example: Bank Transaction System 

## Preparing the environment
In codespaces:
- Create a repository, eg. *"kafka-fundamentals-training"*
- Start a codespace there
- Download kafka
  - Create a ***kafka-setup.sh*** script
  

```bash
#!/bin/bash

# Download and Extract Kafka
wget https://downloads.apache.org/kafka/3.9.0/kafka_2.12-3.9.0.tgz
tar -xvf kafka_2.12-3.9.0.tgz

echo "Kafka setup complete!"
```
- Execute the script

```bash
bash kafka-setup.sh
```

- Install Kafka Library in Codespaces
```bash
pip install confluent-kafka
```

- Start Zookeeper
```bash
cd kafka_2.12-3.9.0
```
```bash
./bin/zookeeper-server-start.sh config/zookeeper.properties
```

🚨 **Open a new terminal**
- Start Kafka
```bash
cd kafka_2.12-3.9.0
```
```bash
./bin/kafka-server-start.sh config/server.properties
```

🚨 **Open a new terminal**
- Create a topic called ***test-topic***
```bash
cd kafka_2.12-3.9.0
```
```bash
./bin/kafka-topics.sh --create --topic test-topic --bootstrap-server localhost:9092 --partitions 3 --replication-factor 1
```

- List Kafka topics
```bash
./bin/kafka-topics.sh --list --bootstrap-server localhost:9092
```


- Create a file called ***producer.py***
- Add the following code
```python
from confluent_kafka import Producer

def delivery_report(err, msg):
    if err:
        print(f"Delivery failed: {err}")
    else:
        print(f"Message delivered to {msg.topic()} [{msg.partition()}]")

producer = Producer({'bootstrap.servers': 'localhost:9092'})

for i in range(100):
    producer.produce('test-topic', value=f'Message number {i} !', callback=delivery_report)
    producer.flush()  # Ensures the message is sent
```

- Create a file called ***consumer.py***
- Add the following code
```python
from confluent_kafka import Consumer, KafkaException

consumer = Consumer({
    'bootstrap.servers': 'localhost:9092',
    'group.id': 'mygroup',
    'auto.offset.reset': 'earliest'
})
consumer.subscribe(['test-topic'])

try:
    while True:
        msg = consumer.poll(1.0)
        if msg is None:
            continue
        if msg.error():
            if msg.error().code() == KafkaException._PARTITION_EOF:
                continue
            else:
                print(f"Error: {msg.error()}")
        else:
            print(f"Received: {msg.value().decode('utf-8')}")
except KeyboardInterrupt:
    pass
finally:
    consumer.close()

```

🚨 **Open a new terminal**
- Run the **producer**
```bash
python producer.py
```

- Run the **consumer**
```bash
python consumer.py
```

## Extended Kafka Example: Bank Transaction System

### Features:
- Multiple Accounts and Balances: Track account balances.
- Fraud Detection: Trigger a fraud alert for withdrawals over 500€.
- Account Summary Service: Summarize all account activities.

### Project structure:

  - *producer_bank_operations.py*
  - *consumer_bank_notifications.py*
  - *consumer_fraud_detector.py*
  - *consumer_account_summary.py*

### Kafka Topics:
- ***bank-deposits*** – Handles deposits
- ***bank-withdrawals*** – Handles withdrawals
- ***fraud-alerts*** – Sends fraud notifications

#### Step 1:




🚨 Please make sure kafka and zookeeper is still running in the terminal

- Create Kafka topics


```bash
./bin/kafka-topics.sh --create --topic "________" --bootstrap-server "________" --partitions 1 --replication-factor 1

./bin/kafka-topics.sh --create --topic "________" --bootstrap-server "________" --partitions 1 --replication-factor 1

./bin/kafka-topics.sh --create --topic "________" --bootstrap-server "________" --partitions 1 --replication-factor 1

```
#### Step 2:
- Create the producer

```python
from confluent_kafka import Producer
import random
import time

# Kafka Producer Config
producer = Producer({'bootstrap.servers': 'localhost:9092'})

# Account balances dictionary
account_balances = {
    "ACC1234": 1000,
    "ACC5678": 1500,
    "ACC9101": 2000
}

# Function to produce transactions
def send_transaction():
    account = random.choice(list(account_balances.keys()))
    amount = random.randint(50, 1000)
    transaction_type = random.choice(['deposit', 'withdrawal'])

    # Initialize topic and message variables
    topic = None
    message = None

    if transaction_type == 'deposit':
        account_balances[account] += amount
        message = f"Deposit: {amount}€ to {account}, New Balance: {account_balances[account]}€"
        topic = '________'
    elif transaction_type == 'withdrawal':
        if amount <= account_balances[account]:  # Ensure sufficient balance
            account_balances[account] -= amount
            message = f"Withdrawal: {amount}€ from {account}, New Balance: {account_balances[account]}€"
            topic = '________'

            # Trigger fraud alert if amount > 500€
            if ________ > 500:
                fraud_message = f"⚠️ Fraud Alert! Suspicious withdrawal of {amount}€ from {account}"
                producer.produce("fraud-alerts", value=fraud_message.encode('utf-8'))
                print(f"Sent Fraud Alert: {fraud_message}\n")
        else:
            message = f"🚨 Failed Withdrawal: {amount}€ from {account}, Insufficient funds"
            topic = 'bank-withdrawals'

    if topic and message:
        producer.produce(topic, value=message.encode('utf-8'))
        print(f"Sent Transaction: {message}\n")
        producer.flush()

# Continuously send transactions
while True:
    send_transaction()
    time.sleep(5)


```
#### Step 3:
- Create consumers:

#### Transaction Notification Consumer (*consumer_bank_notifications.py*)

```python
from confluent_kafka import Consumer

consumer = Consumer({
    'bootstrap.servers': 'localhost:9092',
    'group.id': 'bank-group',
    'auto.offset.reset': 'earliest'
})

consumer.subscribe(["________", "________"])

print("Listening for bank transactions...")

try:
    while True:
        msg = consumer.poll(1.0)
        if msg is None:
            continue
        
        if msg.error():
            print(f"Error: {msg.error()}")
            continue

        # Display notifications
        topic = msg.topic()
        transaction = msg.value().decode('utf-8')

        if topic == "________":
            print(f"💰 Deposit Notification: {transaction}\n")
        elif topic == "________":
            print(f"⚠️ Withdrawal Alert: {transaction}\n")

finally:
    consumer.close()

```
#### Fraud Detector Consumer (*consumer_fraud_detector.py*)

```python
from confluent_kafka import Consumer

consumer = Consumer({
    'bootstrap.servers': 'localhost:9092',
    'group.id': 'fraud-group',
    'auto.offset.reset': 'earliest'
})

consumer.subscribe(["________"])

print("Monitoring for Fraud Alerts...")

try:
    while True:
        msg = consumer.poll(1.0)
        if msg is None:
            continue
        
        if msg.error():
            print(f"Error: {msg.error()}")
            continue

        fraud_alert = msg.value().decode('utf-8')
        print(f"🚨 FRAUD ALERT: {fraud_alert}\n")

finally:
    consumer.close()

```
#### Account Summary Consumer (*consumer_account_summary.py*)

```python
from confluent_kafka import Consumer

consumer = Consumer({
    'bootstrap.servers': 'localhost:9092',
    'group.id': 'summary-group',
    'auto.offset.reset': 'earliest'
})

consumer.subscribe(["________", "________"])

account_summary = {}

print("Generating Account Summary...")

try:
    while True:
        msg = consumer.poll(1.0)
        if msg is None:
            continue
        
        if msg.error():
            print(f"Error: {msg.error()}")
            continue

        transaction = msg.value().decode('utf-8')
        print(f"📊 Transaction Recorded: {transaction}\n")

finally:
    consumer.close()

```

#### Step 4:

#### Run the Bank System

##### Run each script in separate terminals:

```bash
python producer_bank_operations.py
python consumer_bank_notifications.py
python consumer_fraud_detector.py
python consumer_account_summary.py
```

### Expected Output Results:

#### Producer Output:

```bash
Sent Transaction: Deposit: 850€ to ACC5678, New Balance: 2350€
Sent Transaction: Withdrawal: 750€ from ACC9101, New Balance: 1250€
Sent Fraud Alert: ⚠️ Fraud Alert! Suspicious withdrawal of 750€ from ACC9101
````
#### Consumer Notifications Output:

```bash
💰 Deposit Notification: Deposit: 850€ to ACC5678, New Balance: 2350€
⚠️ Withdrawal Alert: Withdrawal: 750€ from ACC9101, New Balance: 1250€
```
#### Fraud Detector Output:

```bash
🚨 FRAUD ALERT: ⚠️ Fraud Alert! Suspicious withdrawal of 750€ from ACC9101
```
#### Account Summary Output:
```bash
📊 Transaction Recorded: Deposit: 850€ to ACC5678
📊 Transaction Recorded: Withdrawal: 750€ from ACC9101
```
