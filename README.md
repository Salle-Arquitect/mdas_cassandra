[![CircleCI](https://circleci.com/gh/eloirobe/mdas_cassandra.svg?style=svg)](https://circleci.com/gh/eloirobe/mdas_cassandra)

# MDAS Bases de datos no estructuradas - Cassandra <img src="https://upload.wikimedia.org/wikipedia/commons/5/5e/Cassandra_logo.svg" height="50"  />
Bienvenidos a la imagen de Cassandra para a la asignatura de Bases de datos no estructuradas del master [Máster en Desarrollo y Arquitectura de Software (MDAS)](https://www.salleurl.edu/es/estudios/master-en-desarrollo-y-arquitectura-software)

Para arrancar Cassandra y empezar a utilizar la imagen de docker realizar lo siguiente:

*Requisitos previo tener instalado docker*

1) Clonar el repositorio
```bash
git clone https://github.com/eloirobe/mdas_cassandra.git
```
2) Arrancar la imagen de docker
```bash
cd mdas_cassandra
## En modo stdout
docker-compose up
## En modo silencioso
docker-compose up -d
```
3) Probar que funciona

```
docker-compose exec cassandra cqlsh
```

