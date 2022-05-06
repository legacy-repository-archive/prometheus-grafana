# Install
## Local

공식 프로메테우스 컴포넌트는 대부분 미리 컴파일해둔 바이너리 파일을 제공한다.     
[다운 받을 수 있는 페이지](https://prometheus.io/download/)를 통해 다운받고 실행만 시키면 된다.     
물론 설정값을 바꾸기 위해서는 prometheus.yml 값을 바꾸면 된다.   

## Docker  

[도커 허브에 있는 이미지](https://hub.docker.com/r/prom/prometheus)를 이용해서 프로메테우스를 실행시킬 수 있다.       

```shell
docker pull prom/prometheus
```
```shell
docker run \
    -p 9090:9090 \
```

## 바인드 마운트  
만약, 로컬에 존재하는 `prometheus.yml` 을 바운드 하고자 한다면 아래와 같이 실행하면 된다.  

```shell
docker run \
    -p 9090:9090 \
    -v /path/to/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
```

`/path/to/prometheus.yml`에 있는 파일을 `:/etc/prometheus/prometheus.yml`에 마운트 한다.    
   
## 커스텀 이미지 

바운드를 하기 싫거나, 커스텀하게 사용하고자 한다면 커스텀 이미지를 만들면 된다.

**Dockerfile**
```
FROM prom/prometheus
ADD prometheus.yml /etc/prometheus/
```

동일 디렉토리에 존재하는 **prometheus.yml** 파일을 도커의 `/etc/prometheus/`로 옮긴다.   

```shell
docker build -t my-prometheus .
docker run -p 9090:9090 my-prometheus
```
이후 커스텀 이미지를 빌드하고 실행하면 된다.   
