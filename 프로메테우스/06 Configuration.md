# Configuration 
## Scrape

프로메테우스 설정은 **`커맨드라인 플래그`와 `설정 파일`을 이용한다.**    

* 커맨드 라인 플래그 : 변하지 않는 시스템 파라미터 설정(스토리지 위치, 디스크나 메모리에 보관할 데이터 양)   
* 설정 파일 : 스크랩할 job, 인스턴스 와 관련된 모든 정보들과 로드할 rule 파일들을 정의    
  
**`./prometheus -h` 를 실행하면 사용할 수 있는 전체 커맨드라인 플래그들을 조회해볼 수 있다.**   
       
프로메테우스는 **런타임에도 설정을 다시 로드할 수 있다.**            
더불어, **새 설정 파일을 적용하려 했는데 형식이 잘못되었다면 변경사항을 적용하지 않는다.**           
   
프로메테우스 프로세스에 `SIGHUP`을 전송하거나,       
`/~/reload` 엔드포인트에 HTTP POST 요청을 보내면        
(`--web.enable-lifecycle` 플래그를 활성화한 경우) 설정을 다시 로드할 수 있다.    
이렇게하면 설정해둔 rule 파일도 전부 다시 로드한다.     

## Configuration file
  
**로드할 설정 파일을 지정할 땐 `--config.file` 플래그를 사용한다.**     
설정 파일은 YAML 형식으로 저장하며 아래에서 설명하는 스키마로 정의한다.   
 
* 대괄호는 파라미터를 생략할 수 있음을 나타낸다.    
* 리스트가 아닌 일반 파라미터들엔 디폴트 값을 명시해놨다.  

|스키마 이름|설명|
|----------|----|
|`<boolean>`|true/false 를 사용하는 boolean|
|`<duration>`|정규 표현식|
|`<filename>`|현재 작업 디렉토리 안에 있는 유효한 경로|
|`<host>`|호스트명이나 IP로 구성된 유효한 문자열(뒤에 포트 번호가 있을 수도 있음)|
|`<int>`|정수 값|
|`<lablename>`|정규 표현식 `[a-zA-Z_][a-zA-Z0-9_]*`에 매칭되는 문자열|
|`<labelvalue>`|유니코드 문자로 이루어진 문자열|
|`<path>`|유효한 URL path|
|`<scheme>`|http나 https 를 사용하는 문자열|
|`<secret>`|비밀번호 같이 평소 시크릿에 사용되는 문자열|
|`<string>`|평범한 문자열|
|`<size>`|바이트로 표현하는 사이즈(ex: 512MB). 단위를 명시해야한다.<br>지원하는 단위: B, KB, MB, GB, TB, PB, EB|   
|`<tmpl_string>`|사용전에 템플릿을 이용해 확장되는 문자열|
  
그 외 다른 플레이스 홀더는 별도로 명시했다.   
예시는 [다음 사이트](https://github.com/prometheus/prometheus/blob/release-2.32/config/testdata/conf.good.yml)에서 확인하면 된다.  

글로벌 설정에서 명시한 파라미터들은 모든 설정 컨텍스트에서 유효하다.     
나머지 설정의 디폴트 값 역할도 수행한다.      
  
```yml
global:
  # 기본적으로 타겟을 스크랩할 간격.
  [ scrape_interval: <duration> | default = 1m ]

  # 스크랩 요청을 언제 타임 아웃시킬지.
  [ scrape_timeout: <duration> | default = 10s ]

  # rule들을 평가하는 주기.
  [ evaluation_interval: <duration> | default = 1m ]

  # 외부 시스템(federation, 리모트 스토리지, Alertmanager)과 통신할 때
  # 모든 시계열, alert에 추가할 레이블들.
  external_labels:
    [ <labelname>: <labelvalue> ... ]

  # PromQL 쿼리 로그를 남길 파일.
  # 설정을 다시 로드하면 이 파일도 다시 연다.
  [ query_log_file: <string> ]

# rule_files에선 glob 패턴 목록을 지정한다.
# 매칭되는 모든 파일에서 rule과 alert를 읽어들인다.
rule_files:
  [ - <filepath_glob> ... ]

# 스크랩 설정 목록.
scrape_configs:
  [ - <scrape_config> ... ]

# alerting에선 Alertmanager와 관련된 설정들을 지정한다.
alerting:
  alert_relabel_configs:
    [ - <relabel_config> ... ]
  alertmanagers:
    [ - <alertmanager_config> ... ]

# remote write 기능과 관련된 설정들.
remote_write:
  [ - <remote_write> ... ]

# remote read 기능과 관련된 설정들.
remote_read:
  [ - <remote_read> ... ]
  
# 런타임에 다시 로드할 수 있는 스토리지 관련 설정들.
storage:
  [ - <exemplars> ... ]
```


