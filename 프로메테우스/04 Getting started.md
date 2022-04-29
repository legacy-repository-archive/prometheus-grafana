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
http://localhost:9090/graph 로 이동해서 "Graph" 탭의 "Console"뷰를 클릭하자    

http://localhost:9090/metrics 에서도 확인할 수 있듯이   
프로메테우스가 자체적으로 내보내는 메트릭 중에는 `prometheus_target_interval_length_seconds` 라는 메트릭이 있다.  

```
prometheus_target_interval_length_seconds
```
표현식 콘솔을 위와 같이 입력하고 "Execute"를 클릭해보자.   
이렇게 하면 `prometheus_target_interval_length_seconds`란 이름을 가진 시계열이 반환 될건데, 레이블은 모두 다를거다.
(시계열 데이터마다 최신 기록 값을 가지고 있다.)    
이 레이블들은 각자 다른 지연 시간 백분위수와 타겟 그룹의 interval을 나타낸다.   










