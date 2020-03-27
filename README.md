# Confluent example of KSQL 

## Prerequisites

 - Clone the project
 - Have Docker and docker-compose installed


## Launch confluent ecosystem
```code
docker-compose up
```

## Connect to the ksqldb server
```code
docker exec -it ksqldb-cli ksql http://ksqldb-server:8088
```

## Create a stream using KSQL
```code
CREATE STREAM raw_movies (ROWKEY INT KEY, id INT, title VARCHAR, genre VARCHAR)
    WITH (kafka_topic='movies', partitions=1, key='id', value_format = 'avro');
```
## Create a stream using KSQL
```code
INSERT INTO raw_movies (id, title, genre) VALUES (294, 'Die Hard::1988', 'action');
INSERT INTO raw_movies (id, title, genre) VALUES (354, 'Tree of Life::2011', 'drama');
INSERT INTO raw_movies (id, title, genre) VALUES (782, 'A Walk in the Clouds::1995', 'romance');
INSERT INTO raw_movies (id, title, genre) VALUES (128, 'The Big Lebowski::1998', 'comedy');
```

## Set offset value
```code
SET 'auto.offset.reset' = 'earliest';
```


## Query the stream
```code
SELECT id, split(title, '::')[1] as title, split(title, '::')[2] AS year, genre FROM raw_movies EMIT CHANGES LIMIT 4;
```

## Create a stream from a queried stream
```code
CREATE STREAM movies WITH (kafka_topic = 'parsed_movies', partitions = 1) AS
    SELECT id,
           split(title, '::')[1] as title,
           CAST(split(title, '::')[2] AS INT) AS year,
           genre
    FROM raw_movies;
```


## Print content of the output stream’s underlying topic
```code
PRINT 'parsed_movies' FROM BEGINNING LIMIT 4;
```

## Test the stream using an input file and an output file
```code
docker exec ksqldb-cli ksql-test-runner -i /opt/app/test/input.json -s opt/app/src/statements.sql -o /opt/app/test/output.json
```

## Go to Production
You will be able to send the sql file to your ksql server running using a POST command
```code
curl -X POST \
-H "Content-Type: application/vnd.ksql.v1+json; charset=utf-8" \
-d @src/statements.json \
http://localhost:8088/ksql
```

## References
https://kafka-tutorials.confluent.io/transform-a-stream-of-events/ksql.html
