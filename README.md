# Prometheus Lab

A ideia desse projeto é ter um laboratório onde podemos aprender como funciona o Prometheus e nossos sistemas de monitoração.

Based on great job: https://github.com/vegasbrianc/prometheus.

## Um pouco de conceito primeiro

O Prometheus é um sistema de monitoração com um banco de dados "time series" e um sistema poderoso de "query language". Ele ainda possui um sistemas de alerta. Ele é dividido em algumas partes, e possuiu componentes que utilizamos a mais para coletar métricas dos serviços e servidores.

- Prometheus: o banco de dados time series e sistema de coleta de métricas
- Alertmanager: sistema de gerenciamento de monitorações que armazena os alertas de diversos sistemas de monitoração como o Prometheus.
- Node Exporter: sistema que coleta métricas de um nodo e disponibiliza para o Prometheus coletar
- Blackbox: sistema de coleta de métricas externas a um nodo ou serviço.
- mtail: extrair dados de monitoramento de caixa branca dos logs do aplicativo para coleta em um banco de dados de séries temporais
- Grafana: open source analytics que busca métricas em diversas origens como o Prometheus.

## Iniciando o Laboratório

Primeiro de tudo você precisa ter o docker instalado na tua instação de trabalho:

```
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sh get-docker.sh
```

Para iniciar todo o ambiente para começar a brincar:

```
$ docker-compose up
```

Agora você ja pode acessar as principais urls:

- Prometheus: http://127.0.0.1:9090/
- Alerting Manager: http://127.0.0.1:9093/
- Node Exporter: http://127.0.0.1:9100/
- Mtail Exporter: http://127.0.0.1:3903/
- Grafana: http://127.0.0.1:3000/

Usuário e senha de grafana são:

```
username - admin
password - foobar (Password is stored in the `/grafana/config.monitoring` env file)
```

## Coletando as métricas

O Prometheus faz "scraping" das métricas geradas por outros sistemas, nesse exemplo usamos o node exporter para coletar métricas do computador que vc esta utilizando, o prometheus coleta e armazena essas métricas e usamos o grafana para visualizar as mesmas.

O arquivo de configuração que contem as configurações para coletas de dados é `prometheus/prometheus.yaml`.

## Criando alertas

O Prometheus também gera alertas com base nas métricas coletadas. O arquivo que tem as regras para os alertas é: `prometheus/alert.rules`.

Para simular um alerta de "high_load" basta gerar um container em loop com esse comando:

```
$ docker run --rm -it busybox sh -c "while true; do :; done"
```

Para gerar o alarme de down de um nodo basta baixar algum deles como o mtail

```
$ docker-compose stop mtail-nginx
```

Para subir novamente:

```
$ docker-compose up mtail-nginx
```

## Usando o Blackbox

Referencia: https://github.com/prometheus/blackbox_exporter

O Blackbox server para coletar métricas externas, por exemplo quando você quer testar se uma porta esta aberta externamente, ou se um servidor esta respondendo a ping.

O blackbox expoter da comunidade ja tem varios modelos de testes prontos, como http, tcp, icmp, dns, entre outros.

Uma maneira de ver todas as métricas use a query language dessa forma:

```
{job="blackbox"}
```

## Usando o Mtail

Referencia: https://github.com/google/mtail

O mtail coleta os dados a partir de algum log. No caso desse exemplo ele esta coletando dados a partir do log do nginx, e um formato especifico para facilitar a coleta.

A partir dele podemos analisar os dados do nginx.

Para gerar dados para poder olhar nos gráficos previamente gerados no dashboard do Nginx no grafana, ou acompanhar no prometheus.

```
while true; do curl http://127.0.0.1/; done
```

## Principais Querys

O Prometeus tem uma query language poderosa, pode ser usada para extrair dos dados as mais diversos informações sobre os sistemas.

Vou comentar algumas, mas a referencia delas esta em:

https://prometheus.io/docs/prometheus/latest/querying/functions/
https://prometheus.io/docs/prometheus/latest/querying/operators/

#### sum()

Soma uma quantidade de valores

```
sum(up)
```
Ou

```
sum(node_filesystem_size_bytes) by (mountpoint)
```

#### rate()
Calcula a taxa média de aumento por segundo da série temporal no vetor de intervalo.

```
rate(nginx_requests[1m])
```

Combinando com o sum:

```
sum(rate(nginx_requests[1m])) by (instance)
```

#### histogram_quantile()

Treco complicado pra caramba, mas lindamente magnífico. Serve para gerar médias de tempo baseado em quantil.

```
histogram_quantile(0.9, rate(http_request_duration_seconds_bucket[10m]))
```

```
histogram_quantile(0.75, sum(rate(nginx_request_duration_bucket[1m])) by (instance,le))
```

## Recording Rules

As recording rules permitem pré-calcular expressões frequentemente necessárias ou caras em termos de computação e salvar seus resultados como um novo conjunto de séries temporais.

```
groups:
  - name: example
    rules:
    - record: job:http_inprogress_requests:sum
      expr: sum(http_inprogress_requests) by (job)
```
