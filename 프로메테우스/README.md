# 설치 

https://prometheus.io/download/

# 도커 

프로메테우스 서비스는 [Quay.io](https://quay.io/repository/prometheus/prometheus) 나 [도커 허브](https://hub.docker.com/r/prom/prometheus) 에 있는 도커 이미지로도 이용할 수 있다.     

```shell
docker run -p 9090:9090 prom/prometheus
```

기본 설정으로 시행해도 되지만   
프로메테우스 이미지는 실제 메트릭을 저장할 땐 볼륨을 사용한다.   
프로덕션 배포에선 named volume 을 사용하는게 프로메테우스를 업그레이드하더라도 데이터를 쉽게 관리할 수 있다.  

## Volumes & bind-mount 

호스트에 있는 `prometheus.yml` 을 바인드 마운트 시킨다.    
  
```shell
docker run \
    -p 9090:9090 \
    -v /path/to/prometheus.yml:/etc/prometheus/prometheus.yml \
    prom/prometheus
```

아니면 다음 명령어로 `promethues.yml` 이 들어있는 디렉토리를 `/etc/prometheus` 에 바인드 마운트한다.  

```shell 
docker run \
    -p 9090:9090 \
    -v /path/to/config:/etc/prometheus \
    prom/prometheus
```

## Custom Image

파일을 호스트에서 관리하고 바인드 마운트를 사용하는게 싫으면 설정 자체를 이미지에 실어버릴 수 있다.   
설정 자체가 고정돼있고 모든 환경에서 동일하다면 이 방법도 괜찮다.  

디렉토리 하나 새로 만들어서, 프로메테우스 설정 파일과 다음과 같은 Dockerfile 을 추가하자.  

```Dockerfile
FROM prom/prometheus
ADD prometheus.yml /etc/prometheus/
```
```shell
docker build -t my-prometheus .
docker run -p 9090:9090 my-prometheus
```
더 나아가면 몇 가지 툴을 활용해 기동시 설정을 동적으로 렌더링하거나, 데몬을 이용해 설정을 주기적으로 업데이트할 수도 있다.  
