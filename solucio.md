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

# 2) Crea el modelo de datos necesario para cumplir con las especificaciones.  A medida que vayáis avanzando en el ejercicio y vayáis respondiendo las preguntas, es posible que necesitéis ir adaptando vuestro modelo de datos, si es así, en esta pregunta se piden las queries necesarias para crear el modelo de datos final.
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

CREATE TYPE IF NOT EXISTS attachment (
   name text,
   path text
);

CREATE TYPE IF NOT EXISTS content (
	message text,
	attachments SET<FROZEN <attachment>>
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


# 3) Justifica para cada tabla el motivo que te ha llevado a escoger las claves de partición y las claves de clustering.

## Tenemos la tabla `user` con:
- alias
- Nombre real completo
- Email
- Edad
Primary key alias, ya que solo queremos encontar la información en base su alias.

## Tenemos la tabl `room` con:
- name
- participants `Set<text>`
Con primary key name, ya que solo queremos buscar por su nombre de sala.

## Tenemos la tabla `message` con:
- Fecha timestamp
- Autor
- Contenido
```json
{
  message text,
  attachments {
    name
    path
  }
}
```
- Sala de chat o usuario quien lo envia
  - Para mostrar siempre los mensajes más recientes primero, hemos puesto como llave de clustering la fecha y el destinatary la primary key.
- Los mensajes pueden incluir archivos adjuntos como imágenes, documentos, etc. De cada archivo adjunto almacenaremos un nombre y su ubicación.

## El usuario ha de poder ver la cantidad de mensajes no leídos de cada sala de chat, y de cada mensaje directo
Tenemos la tabla `messages_not_read` con:
- user text
- input text
- count counter
con primary key user y input ya que lo requiere el count.


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

## 4.3) Los usuarios Alice y Bob participan en la sala **general**, y los usuarios Alice, Bob y James participan en la sala **summer\_party**
```cassandra
UPDATE room
SET participants = participants + {'alice', 'bob'}
WHERE name = 'general';

UPDATE room
SET participants = participants + {'alice', 'bob', 'james'}
WHERE name = 'summer_party';
```

## 4.4) Alice ha escrito 2 mensajes en **summer\_party**
```cassandra
-- Primer mensaje
INSERT INTO message (date, author, content, destinatary)
VALUES ('2018-04-01T11:21:59.001+0000', 'alice', {message: 'holis', attachments : {}}, 'summer_party');

UPDATE messages_not_read
SET count = count +1
WHERE user = 'alice'
 AND input = 'summer_party';

UPDATE messages_not_read
SET count = count +1
WHERE user = 'bob'
 AND input = 'summer_party';

UPDATE messages_not_read
SET count = count +1
WHERE user = 'james'
 AND input = 'summer_party';

-- Segundo mensaje
INSERT INTO message (date, author, content, destinatary)
VALUES ('2018-04-01T11:23:42.001+0000', 'alice', {message: 'donde estais todos', attachments : {}}, 'summer_party');

UPDATE messages_not_read
SET count = count +1
WHERE user = 'alice'
 AND input = 'summer_party';

UPDATE messages_not_read
SET count = count +1
WHERE user = 'bob'
 AND input = 'summer_party';

UPDATE messages_not_read
SET count = count +1
WHERE user = 'james'
 AND input = 'summer_party';
```

## 4.5) Bob ha leído los mensajes que hay en `summer_party`
```cassandra
DELETE
FROM messages_not_read
WHERE user = 'bob'
 AND input = 'summer_party';
```

## 4.6) Alice ha enviado a James un mensaje directo que incluye un archivo adjunto.
```cassandra
INSERT INTO message (date, author, content, destinatary)
VALUES (
  '2018-04-01T11:23:42.001+0000',
  'alice',
  {message: 'hoy destacaste mucho', attachments : {{name: 'james.txt', path: '/var/lib/idea/withLove/james.txt'}}},
  'james'
);

UPDATE messages_not_read
SET count = count +1
WHERE user = 'james'
 AND input = 'alice';
```

# 5) Muestra la cantidad de mensajes que tienen Bob y James no leídos en la sala `summer_party`
<!-- No funciona amb bach!!
```cassandra
BEGIN BATCH
	SELECT * FROM messages_not_read WHERE user = 'bob' AND input = 'summer_party';
	SELECT * FROM messages_not_read WHERE user = 'james' AND input = 'summer_party';
APPLY BATCH;
```
-->
```cassandra
SELECT * FROM messages_not_read WHERE user = 'bob' AND input = 'summer_party';
SELECT * FROM messages_not_read WHERE user = 'james' AND input = 'summer_party';
```

# 6) Muestra los mensajes de la sala `summer_party` ordenados por fecha descendiente.
```cassandra
SELECT * FROM message WHERE destinatary = 'summer_party' ;
```
Con output:
```
 destinatary  | date                            | author | content
--------------+---------------------------------+--------+------------------------------------------------
 summer_party | 2018-04-01 11:23:42.001000+0000 |  alice | {message: 'donde estais todos', attachments: {}}
 summer_party | 2018-04-01 11:21:59.001000+0000 |  alice |              {message: 'holis', attachments: {}}
```

Muestra la tabla ordenada por como hemos creado la tabla:
```
WITH CLUSTERING ORDER BY (date DESC);
```

# 7) Muestra la sala y nombre de los de participantes de cada sala de chat
```cassandra
SELECT * FROM room ;
```

Con output:
```
 name         | participants
--------------+---------------------------
 summer_party | {'alice', 'bob', 'james'}
      general |          {'alice', 'bob'}

(2 rows)
```
