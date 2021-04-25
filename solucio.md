
??? como interfiere la creación de un mensaje (?) TODO (?)
> usuario ver cantidad de mensajes no leidos por chat y directos
BATCH ???


# 1) Crea una base de datos llamada “messaging\_system” que utilice la estrategia de replicación simple, con un factor de 1.
<!---
oo arquitectura cassandra, pagina 7 (18)
Simple Strategy
- particionador determina donde colocar la primera replica de los datos
- las siguientes replicas se ubican, si miramos el anillo, en los siguientes nodos sentido del reloj
-->
```cassandra
CREATE KEYSPACE IF NOT EXISTS messaging_system
WITH replication = {
	'class': 'SimpleStrategy',
	'replication_factor': 1
};
USE messaging_system;
```

# 2)
```cassandra
CREATE TABLE IF NOT EXISTS user (
	alias text,
	full_name text,
	email text,
	age Int,
	PRIMARY KEY (alias)
);

CREATE TABLE IF NOT EXISTS room (
	name text,
	participants Set<text>,
	PRIMARY KEY (name)
);

CREATE TYPE IF NOT EXISTS content (
	message text,
	attachments Map<text, text>
);

CREATE TABLE IF NOT EXISTS message (
	date timestamp,
	author text,
	content FROZEN<content>,
	destinatary text,
	PRIMARY KEY ((destinatary), date)
)
WITH CLUSTERING ORDER BY (date DESC);

CREATE TABLE IF NOT EXISTS  messages_not_read (
	user text,
	input text,
	count counter,
	PRIMARY KEY (user, input)
);
```


# y 3 TODO
TODO

# 4) Inserta los siguientes datos en tus tablas, estos datos servirán para responder las próximas preguntas. Como respuesta a esta pregunta muestra las queries que hayas utilizado para insertar/modificar los datos que se piden a continuación, en el mismo orden en el que aparecen en el enunciado.

## 4.1) Crea los usuarios alice, el usuario bob y el usuario james
```cassandra
INSERT INTO user (alias, full_name, email, age)
VALUES ('alice', 'Alice Levine', 'lonca@gulugulu.com', 34);

INSERT INTO user (alias, full_name, email, age)
VALUES ('bob', 'Bob Newhart', 'bob@actor.act', 91);

INSERT INTO user (alias, full_name, email, age)
VALUES ('james', 'James Charles Dickinson', 'jcd@youtube.yt', 21);
```

## 4.2) Crea las salas de chat **general** y **summer\_party**
```cassandra
INSERT INTO room (name, participants)
VALUES ('general', {});

INSERT INTO room (name, participants)
VALUES ('summer_party', {});
```

# 4.3) Los usuarios Alice y Bob participan en la sala **general**, y los usuarios Alice, Bob y James participan en la sala **summer\_party**
```cassandra
UPDATE room
SET participants = participants + {'alice', 'bob'}
WHERE name = 'general';

UPDATE room
SET participants = participants + {'alice', 'bob', 'james'}
WHERE name = 'summer_party';
```

```cassandra
(?) TODO (?) SELECT alias FROM user ;
UPDATE room
SET participants = participants + {'tutu'}
WHERE name = 'general'
IF 'tutu' IN (SELECT alias FROM user);
```

# 4.4) Alice ha escrito 2 mensajes en **summer\_party**
```cassandra
BEGIN BATCH
	INSERT INTO message (date, author, content, destinatary)
	VALUES ('2017-04-01T11:21:59.001+0000', 'alice', {message: 'holis', attachments : []}, 'summer_party');

	INSERT INTO


CREATE TABLE IF NOT EXISTS  messages_not_read (
	user text,
	input text,
	count counter,
	PRIMARY KEY (user, input)
);
TODO, com combino cerques?
	SELECT participants
	FROM room
	WHERE name = 'summer_party'

	INSERT INTO messages_not_read (user, input)
	VALUES ();
APPLY BATCH
```
