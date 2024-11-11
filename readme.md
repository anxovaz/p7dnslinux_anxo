# Práctica 7 - Anxo Vázquez Lorenzo
## 1. Crear red docker
Primeiro creo a rede docker:
```shell
docker network create \
  --driver=bridge \
  --subnet=172.28.0.0/16 \
  --ip-range=172.28.5.0/24 \
  --gateway=172.28.5.254 \
  bind9_subnet
```
## 2. docker-compose.yml
No archivo .yml indico ao servidor asir_bind9 os volúmenes "conf" e "zonas" onde vai a estar gardada a confiuración e despois creo o cliente coa imaxe alpine.
Contenido docker-compose.yml:
```shell
services:
  asir_bind9:
    container_name: asir_bind9
    image: ubuntu/bind9
    platform: linux/amd64
    ports:
      - 120:120
    networks:
      bind9_subnet:
        ipv4_address: 172.28.5.1
    volumes:
      - ./conf:/etc/bind
      - ./zonas:/var/lib/bind
  cliente:
    container_name: clientealpine
    image: alpine
    platform: linux/amd64
    tty: true
    stdin_open: true
    dns:
      - 172.28.5.1
    networks:
      bind9_subnet:
        ipv4_address: 172.28.5.2

networks:
  bind9_subnet:
    external: true

```
## 3. Carpeta zonas
Na carpeta "zonas" creo una base de datos para o zoa "anxovaz.com" chamada "db.asircastelao.int" no que configuro a zoa e os seus rexistros:
Contido db.asircastelao.int:
```shell
$TTL    604800
@       IN      SOA     anxovaz.com. root.anxovaz.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL

@       IN      NS      anxovaz.com.
@       IN      A       172.28.5.1      
@       IN      AAAA    ::1             
@       IN      TXT     "anxo"

anxovaz.com. IN  A       172.28.5.1     
ns      IN      A       172.28.5.1      
www     IN      CNAME   anxovaz.com.    

```

## 4. Carpeta conf
Nesta carpeta monto os seguintes archivos:
-named.conf
-named.conf.default-zones
-named.conf.local
-named.conf.options

### Archivo named.conf
Neste archivo simplemente fai referencia a os archivos "named.conf.options" e "named.conf.local"
Contido named.conf:
```shell
include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
```

### Archivo named.conf.options
Neste archivo configuro o forwarder no caso no que non poda resolver a consulta dns de forma local.
Tamen configuro a seguinte liña para que non faga a validación DNSSEC:
```shell
// Disable DNSSEC validation
        dnssec-validation no;

```
Contido named.conf.options:
```shell
options {
	directory "/var/cache/bind";
	// Disable DNSSEC validation
        dnssec-validation no;
	forwarders {
		8.8.8.8;
		8.8.4.4;
	 };
	 forward only;

	listen-on { any; };
	listen-on-v6 { any; };

	allow-query {
		any;
	};
};
```

### Archivo named.conf.local
Neste archivo configuro a zoa anxovaz.com indicandolle o archivo zonas/db.asircastelao.int.
Contido named.conf.local:
```shell
zone "anxovaz.com" {
	type master;
	file "/var/lib/bind/db.asircastelao.int";
	allow-query {
		any;
		};
	};
```

### Archivo named.conf.default-zones
Este archivo contén a información sobre as zoas máis comunes.
Contido named.conf.default-zones:
```shell
// prime the server with knowledge of the root servers
zone "." {
	type hint;
	file "/usr/share/dns/root.hints";
};

// be authoritative for the localhost forward and reverse zones, and for
// broadcast zones as per RFC 1912

zone "localhost" {
	type master;
	file "/etc/bind/db.local";
};

zone "127.in-addr.arpa" {
	type master;
	file "/etc/bind/db.127";
};

zone "0.in-addr.arpa" {
	type master;
	file "/etc/bind/db.0";
};

zone "255.in-addr.arpa" {
	type master;
	file "/etc/bind/db.255";
};

```

## 5.Execución docker compose up
Executo docker compose up e métome na máquina cliente(clientealpine) usando ash xa que e a shell predeterminada de alpine:
```shell
docker exec -it clientealpine /bin/ash

```
Alpine usa o xestor "apk" en vez de "apt" polo cal executo "apk update" e acto seguido fago un ping á miná zoa (anxovaz.com).
PD: Esta imaxe trae por defecto instalados os paquetes necesarios para facer ping, polo cal non preciso de instalalos.
```shell
apk update
ping anxovaz.com
```

Para instalar dig executo o seguinte commando:
```shell
apk add --update bind-tools
```

Executo dig:
```shell
dig anxovaz.com
```















