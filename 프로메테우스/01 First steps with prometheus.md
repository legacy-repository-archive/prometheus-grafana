# First steps with prometheus

프로메테우스는 모니터링 대상에서 메트릭 HTTP 엔드포인트를 스크랩해 **메트릭을 수집하는 모니터링 플랫폼이다.**   
이 단원은 프로메테우스를 설치하고 설정하고 모니터링하는 방법을 안내할 예정이다.      
또한 호스트와 서비스에 있는 시계열 데이터를 노출해주는 익스포터도 다운받아 설치할 예정이다.    
 
첫번째로 다뤄볼 익스포터는 프로메테우스 자체로,     
메모리 사용량이나 가비지 컬렉션 등에 방대한 호스트 레벨 메트릭을 제공한다.  

## Downloading Pronetheus   

최신 버전을 다운 받아서 사용하면 된다.  
  
* [다운로드 사이트](https://prometheus.io/download/)   

```yml
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

scrape_configs:
  - job_name: "prometheus"

    static_configs:
      - targets: ["localhost:9090"]
```
기본으로 `global`, `alerting`, `rule_files`, `scrape_configs` 세가지 설정 블록이 있다.    

### Global 

```yml
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
```
global 블록에는 프로메테우스 서버의 글로벌 설정을 지정한다.    

* scrape_interval : 타겟을 스크랩할 간격 설정(기본값은 1분, 기본 파일에서는 15초)   
* evaluation_interval : rule 평가 주기 지정, rule을 사용해 새 시계열 만들고 alert 생성 (기본값은 1분, 기본 파일에서는 15초)   

### rule_files 

```yml 
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"
```

프로메테우스 서버가 로드할 rule 위치를 지정(기본 파일에는 rule 없다.)  

### scrape_configs
프로메테우스가 모니터링할 리소스를 지정한다.   
프로메테우스는 자기 자신에 대한 데이터도   
HTTP 엔드포인트로 노출하기 때문에, 자체 상태도 스크랩하고 모니터링할 수 있다.    

```yml
scrape_configs:
  - job_name: "prometheus"

    static_configs:
      - targets: ["localhost:9090"]
```

기본 파일 설정에는 프로메테우스 서버가 노출하는 시계열 데이터를 스크랩하는 **`prometheus` 라는 job** 이 하나가 있다.    
이 job 은 정적으로 설정해둔 localhost 9090 포트를 타겟으로 하나 가지고 있다.    
프로메테우스는 기본적으로 타겟의 /metrics 경로에서 메트릭을 조회한다.   
즉, 디폴트 job 은 http://localhost:9090/metrics URL을 통해 스크랩을 진행한다.    
  
여기서 반환하는 시계열 데이터엔 프로메테우스 서버의 상태와 성능에 관한 세부정보가 담겨 있다.  
자체 설정 옵션 스펙은 [설정 문서](https://godekdls.github.io/Prometheus/configuration)를 참고하자.     

## Starting Prometheus. 

새로 만든 설정 파일로 프로메테우스를 시작하려면,   
프로메테우스 바이너리가 들어 있는 디렉토리로 이동해서 아래 명령어를 실행해야한다.  

```shell
./prometheus --config.file=prometheus.yml
```
   
기본 설정 파일로 프로메테우스를 실행했기에,       
http://localhost:9090 에서 프로메테우스 자체의 status 페이지를 돌려볼 수도 있다.       
자체 HTTP 메트릭 엔드포인트에선 자신에 대한 데이터를 수집할 수 있도록 30초 정도를 기다리는 것이 좋다.   
http://localhost:9090/metrics 로 접근해보면 프로메테우스가 자체 메트릭을 서빙하고 있는지 확인 가능하다.    


