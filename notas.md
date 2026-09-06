# Workshop de Docker - IEEE IDP

## Aula 1

### fundamentos de SO: kernel, processo, 
interface de usuário <- SO -> máquina em modo privilegiado

#### kernel

executa em modo privilegiado \
pede para a máquina prover os recursos pedidos pelo programa em modo usuário

#### processos

árvore de processos

fork do processo (duplica) 

SO já tem isolamento de processos

##### deamon

processo em segundo plano

convenção UNIX: terminação em 'd'

### Docker

originalmente, se fazia o isolamento de processos por máquinas virtuais. porém, é um método custoso
- SOs simulados isoladamente

- monta as imagens
- usa a infraestrutura do linux para isolar e gerenciar processos
- comunicação entre processos (linux IPC inter process com)

OCI : ---- container intervention


### imagens

template imutável
- filesystem
- metadados

apenas as instâncias de uma imagem são mutáveis, dentro de um docker

imagem -> ... -> servidor ... CICD -> deploy

uma imagem é baseada num programa e um SO. se o programa é grande, melhor escolher um SO mais leve

CI: build da imagem
CD: 


### comandos

run: vai acessar uma imagem e rodar seu processo
stop: para a execução de um processo
exec: vai executar um programa da 
ps: 


## Aula 2

redes, images, persistência em imagens?

### redes

alguns argumentos
-d detach
-p XXXX:XX mapeamento de porta, com biding entre as portas XXXX e XX

cada container tem sua própria rede
interface de pilha de rede, conjunto de sockets

assim, quando alocamos uma porta no container a porta não é ocupada para o resto do sistema host

podemos fazer o biding da porta 8080 para duas portas, 80 e 81, de forma que ambos os processos escutam a porta 8080 mas não conflitam

no caso de -p, estamos criando um IP interno na máquina para o processo que estamos iniciando. no caso, o sistema usa NAT para gerenciar a comunicação dos processos, de forma que criamos uma camada intermediária entre a interface de rede do sistema e o 

-P mapeia uma porta livre qualquer para o biding, modo EXPOSE (puramente documental, não é uma publicação pois o docker não tem autorização pra isso)

nome    | descrição
--      | --
brigde  | barramento do docker comum para os containers
host    | 
none    | 

-it : comando iterativo, acessando pelo shell \
-r  : recursivo

```sh
docker network ls

# criação de uma rede
docker network create app-net   # nome da rede

# iniciando um processo conectado à sub-rede, com nome do processo, nome da rede, variáveis e imagem
docker run -d --name database_docker --network app-net \
        -e POSTGRES_PASSWORD=senha postgres:16-alpine

# podemos checar o estado (data de criação) do processo com
docker ps

docker run -it --network app-net alpine sh
```

### volumes

por padrão, containers não têm persistência após serem fechados. para termos persistência, usamos volumes

bind mount: diretório do host montado no container (dev)
named volume: gerenciado pelo Docker em /var/lib/docker/volumes (produção, db)
tmpfs: filesystem em RAM

```sh
# lista dos volumes do docker
docker volume ls

# JSON com dados de redes e volumes (e etc.)
docker inspect

# bind mount
docker run -v $(pwd)

# named volume
docker volume create dados
docker run -v dados:/var/lib/postgresql/data --name daatabase_docker \
        --network app-net -e POSTGRES_PASSWORD=senhaseguda postgres:16-alpine

# tmpfs
docker run --tempfs /tmp alpine

```

exemplo de criação de um banco de dados PostgreSQl, execução de um comando no banco e remoção do processo
```sh
docker run -d --name pg1 -e POSTGRES_PASSWORD=dev postgres:16-alpine

docker exec -it pg1 psql -U postgres -c "CREATE TABLE t (id INT); INSERT INTO t VALUES (1);"

docker exec -it pg1 psql -U postgres -c "SELECT * FROM t;"

docker rm -f pf1

#
docker run -v pgdata:/var/lib/postgresql/data -d --name pg1 -e POSTGRES_PASSWORD=dev postgres:16-alpine
```

### build e contexto

builds têm cache, então a segunda vez que se builda um container é mais rápida

```sh
docker build -t minha-api .
docker images
docker run -d -p 3000:3000 minha-api
curl http://localhost:3000
```


### boas práticas

dockerignore filtra arquivos na instanciação de uma imagem

multi-stage building: primeiro estágio de build, depois de runtime (imagem enxuta)


# Aula 3

como trabalhar com uma imagem e vários containers?

## Docker Compose

arquivo YAML que define toda a staack: serviços, redes, volumes, variáveis de ambiente \
útil pra vários estágios de dev

### Imperativo vs Declarativo

imperativo: `docker run`
declarativo: `docker compose`

IaC (Infrastructure as Code)

## docker compose

```yml
image: nginx:alpine
build:
ports: ["", ""]
networks: []
restart: 
```

ao rodar um docker compose, o Docker cria uma rede automaticamente, com nome <nome-do-projeto>_default

sobe todos os serviços de imagem

```sh
docker compose
  - up                    foreground, logs ao vivo
    - -d                  detached (background)
    - --build             força rebuild antes
  - down                  derruba containers e rede
    - -v    derruba os    derruba tbm os volumes
    - --rmi all           remove as imagens
```

```sh
docker compose ps                 # 
docker compose logs               # logs de todos
docker compose logs -f api        # follow só de 1 serviço (continua ativo)
docker compose logs --tail=50 db  # últimas 50 linhas

docker compose top                # 
docker compose images             # 
```

```sh
docker compose exec api bash            # 
docker compose exec db psql -U user app
docker compose run --rm api npm test
docker compose run ____________ # 

```


SIGTERM


## comunicação entre serviços

`healthcheck`: pinga o serviço e confere se ele responde com saída healthy (não basta apenas o docker dizer que subiu, pode ter falha silenciosa)

`service_started`
`service_healty`
...

`depends_on`: vai estabelecer dependências para serviços que vamos subir


no compose, não podemos passar as variáveis de ambiente por um `.env`, ele apenas copia as imagens. mas devemos definir quais variáveis o serviço deve receber em outra interface

orquestração de containers: Kubernetes

## Política de reinício



## Prática

1. otimizar Dockerfile com multi-stage-build
2. docker-compose.yml orquestrando os 3 serviços
3. garantir que `docker compose up` sobe tudo na ordem 
4. testar o endpoint que escreve no Postgres e cacheia no Redis


### docker-compose.yml

- container_name
- depends_on
- passar .env
- health_checks

### Dockerfile

bota a inicialização do python pra ir pra cache antes de subir total \
inicializa o serviço como usuário não-privilegiado 


```sh
cp .env.example .env
```
- service: definição de como subir um sistema
