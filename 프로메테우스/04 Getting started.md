# Getting started 

## downloading and running prometheus 

사용중인 플랫폼(OS)에 맞는 프로메테우스 [최신 릴리즈](https://prometheus.io/download/)를 다운받고 압축을 풀자.     
  
```shell 
tar xvfz prometheus-*.tar.gz
cd prometheus-*
```

## Configuring Prometheus to monitor itself

프로메테우스 메트릭 HTTP 엔드포인트를 스크랩하는 식으로 타겟들의 메트릭을 수집한다.       
프로메테우스 자체 데이터도 같은 방식으로 노출시키기 때문에, 자체 상태도 스크랩하고 모니터링 할 수 있다.    

```yml 
global:
  scrape_interval:     15s # 기본적으로 타겟들을 15초마다 스크랩한다.

  # 외부 시스템(federation, 리모트 스토리지, Alertmanager)과 통신할 때는
  # 모든 시계열, alert에 이 레이블들을 덧붙인다.
  external_labels:
    monitor: 'codelab-monitor'

# 스크랩할 엔드포인트를 딱 하나 가지고 있는 스크랩 설정:
# 여기선 프로메테우스 자체를 스크랩한다.
scrape_configs:
  # 이 설정으로 스크랩한 모든 시계열엔 여기 있는 job 이름이 `job=<job_name>` 레이블로 추가된다.
  - job_name: 'prometheus'

    # 글로벌로 설정해둔 기본값을 재정의하며, 이 job에선 타겟을 5초 간격으로 스크랩한다.
    scrape_interval: 5s

    static_configs:
      - targets: ['localhost:9090']
```

프로메테우스 서버로 자체 데이터만 수집하는 건 따힉 효용은 없지만 처음 이해하기에 좋은 예시다.  
전체 옵션 스펙은 [설정 문서](#https://godekdls.github.io/Prometheus/configuration/)를 참고하자    

## Staring Prometheus  

새로 만든 설정 파일로 프로메테우스를 시작하려면,     
프로메테우스 바이너리 파일이 들어있는 디렉토리로 이동해서 아래 명령어를 실행해라    

```
# 프로메테우스를 시작한다.
# 프로메테우스는 기본적으로 ./data 경로 밑에 데이터베이스를 저장한다 (--storage.tsdb.path 플래그).
./prometheus --config.file=prometheus.yml
```

프로메테우스가 시작되면 `http://localhost:9090`에서 프로메테우스 자체의 status 페이지를 둘러볼 수 있다.   
자체 HTTP 메트릭 엔드포인트에서 자신에 대한 데이터를 수집할 수 있도록 몇 초 기다려야 한다.   

자체 메트릭 엔드 포인트 `http://localhost:9090/metrics`로 접근하면    
프로메테우스가 자체 메트릭을 서빙하고 있는지도 확인해볼 수 있다.   

## Using the expression browser 
  
프로메테우스가 자체적으로 수집한 데이터를 살펴보자     
프로메테우스의 내장 expression 브라우저를 사용하려면      
http://localhost:9090/graph 로 이동해서 "Graph" 탭의 "Table" 뷰를 클릭하자    

http://localhost:9090/metrics 에서도 확인할 수 있듯이   
프로메테우스가 자체적으로 내보내는 메트릭 중에는 `prometheus_target_interval_length_seconds` 라는 메트릭이 있다.  

```
prometheus_target_interval_length_seconds
```
표현식 콘솔을 위와 같이 입력하고 "Execute"를 클릭해보자.   
이렇게 하면 `prometheus_target_interval_length_seconds`란 이름을 가진 시계열이 반환 될건데, 레이블은 모두 다를거다.   
(시계열 데이터마다 최신 기록 값을 가지고 있다.)      
이 레이블들은 각자 다른 지연 시간 백분위수와 타겟 그룹의 interval을 나타낸다.     

```
prometheus_target_interval_length_secondes{quantile="0.99"}.   
``` 
99번째 백분위수에만 관심이 있다면 위 쿼리로 검색하면 된다.   

```
count(promethes_target_interval_length_seconds)
```  
반환된 시계열의 갯수는 위와 같이 조회할 수 있다.    
표현식 언어에 대한 자세한 내용은 표현식 언어 문서를 참고하자(PromQL).   


## Using the graphing interface 

표현식을 그래프로 보려면 http://localhost:9090/graph 로 이동해서 "Graph" 탭의 "Graph" 뷰를 클릭하자      

```
rate(prometheus_tsdb_head_chunks_created_total[1m])
```
예시로 프로메테우스 지표에서, 생성하는 청크 수를 초 단위로 그려보면 위와 같은 표현식을 입력하면 된다.    
그래프 range 파라미터나 다른 설정으로 여러가지 실험을 해봐도 좋다.    

## Starting up some sample targets 

프로메테우스가 스크랩할 타겟을 더 주가해보자.    
타겟 예시로 노드 익스포터를 사용할 예정이다.(자세한 사용법은 [가이드 참고](https://godekdls.github.io/Prometheus/guides.node-exporter/)) 

```
tar -xzvf node_exporter-*.*.tar.gz
cd node_exporter-*.*

# 별도 터미널에서 샘플 타겟을 3벌 시작한다
./node_exporter --web.listen-address 127.0.0.1:8080
./node_exporter --web.listen-address 127.0.0.1:8081
./node_exporter --web.listen-address 127.0.0.1:8082
```

이제 `127.0.0.1:8080`, `127.0.0.1:8081`, `127.0.0.1:8082` 에서 메트릭을 수집할 수 있게 되었다.  
즉, `/metrics`로 수신 가능한 샘플 타겟이 더 생겼다.  

## Cofiguration Prometheus to monitor the sample targets 

타겟이 생겼지만 아직 프로메테우스에서 수집한다는 설정을 하지 않았다.     

```yml
scrape_configs:
  - job_name:       'node'

    # 글로벌로 설정해둔 기본값을 재정의하며, 이 job에선 타겟을 5초 간격으로 스크랩한다.
    scrape_interval: 5s

    static_configs:
      - targets: ['localhost:8080', 'localhost:8081']
        labels:
          group: 'production'

      - targets: ['localhost:8082']
        labels:
          group: 'canary'
```
3개의 익스포터를 기준으로 하나의 'node' 라는 잡을 만들었다.    
처음 두 엔드포인트는 프로덕션 타겟, 세번째 엔드포인트는 카나리 인스턴스라고 가정한다.     

프로메테우스에서 이 구조를 모델링할 땐,   
단일 job 엔드포인트 그룹을 여러개 추가하고 타겟 그룹마다 별도 레이블을 추가해주면 된다.    

설정을 완료하고, expression 브라우저로 돌아가보자.   
이제 프로메테우스에 위 예제 엔드포인트들에서 노출하는 시계열 정보가 있는지 확인하자(node_cpu_seconds_total 등)  







