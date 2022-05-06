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

### Configuration file
  
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

### `<scrape_config>`  
`scrape_config` 엔 스크랩할 타겟들과, 스크랩할 방법을 묘사하는 파라미터들을 지정한다.      
보통은 스크랩 설정 하나로 job 하나를 지정한다.    
복잡한 설정을 추가할 땐 달라질 수 있다.    

스크랩 타겟은 `static_cofigs` 파라미터를 통해 정적으로 설정할수도 있고,     
지원하는 서비스 디스커버리 매커니즘 중 하나를 이용해 동적으로 잡아낼 수 있다.   
  
이 밖에도, `relabel_configs` 를 사용하면 스크랩하기 전에 모든 타겟과 레이블을 더 수정할 수 있다.  

```
# 스크랩한 메트릭에 기본적으로 할당하는 job 이름
job_name: <job_name>

# 이 job에서 타겟들을 스크랩할 간격.
[ scrape_interval: <duration> | default = <global_config.scrape_interval> ]

# 이 job을 스크랩할 때마다 사용할 타임 아웃.
[ scrape_timeout: <duration> | default = <global_config.scrape_timeout> ]

# 타겟들에서 메트릭을 가져올 때 사용할 HTTP 리소스 경로
[ metrics_path: <path> | default = /metrics ]

# honor_labels는 프로메테우스가 서버 사이드에서 첨부하는 레이블과
# ("job"/"instance" 레이블, 수동으로 설정한 타겟 레이블, 서비스 디스커버리 구현체가 생성한 레이블)
# 스크랩한 데이터에 이미 있는 레이블이 충돌하면 프로메테우스에서 어떻게 처리할지를 결정한다.
#
# honor_labels를 "true"로 설정하면, 스크랩한 데이터에 있는 레이블을 유지하고
# 충돌하는 서버 사이드 레이블을 무시하는 식으로 충돌을 해소한다.
#
# honor_labels를 "false"로 설정하면, 스크랩한 데이터에서 충돌하는 레이블들의 이름을
# "exported_<original-label>"로 변경한 다음 (ex. "exported_instance", "exported_job")
# 서버 사이드 레이블들을 첨부하는 식으로 충돌을 해소한다.
#
# 타겟에 지정한 모든 레이블을 보존해야 하는 federation과 Pushgateway 스크랩 등을 사용할 땐
# honor_labels를 "true"로 설정해주면 된다.
#
# 참고로, 글로벌로 설정한 "external_labels"는 이 설정에 영향 받지 않는다. 
# 외부 시스템과 통신할 때는 항상, 시계열에 주어진 레이블이 이미 없을 때에만 적용하며, 그 외는 무시한다.
[ honor_labels: <boolean> | default = false ]

# honor_timestamps는 프로메테우스가 스크랩한 데이터에 있는 타임스탬프를 그대로 따를지 여부를 결정한다.
#
# honor_timestamps를 "true"로 설정하면, 타겟이 노출한 메트릭의 타임스탬프를 사용한다.
#
# honor_timestamps를 "false"로 설정하면, 타겟이 노출한 메트릭의 타임스탬프는 무시한다.
[ honor_timestamps: <boolean> | default = true ]

# 요청에 사용할 프로토콜 스킴을 설정한다.
[ scheme: <scheme> | default = http ]

# HTTP URL 파라미터들 (Optional).
params:
  [ <string>: [<string>, ...] ]

# 설정한 username과 password로 모든 스크랩 요청에 'Authorization' 헤더를 세팅한다.
# password와 password_file은 함께 사용할 수 없다.
basic_auth:
  [ username: <string> ]
  [ password: <secret> ]
  [ password_file: <string> ]

# 설정한 credential로 모든 스크랩 요청에 'Authorization' 헤더를 세팅한다.
authorization:
  # 요청에서 사용할 인증 타입을 설정한다.
  [ type: <string> | default: Bearer ]
  # 요청에서 사용할 credential을 설정한다.
  # `credentials_file`과는 함께 사용할 수 없다.
  [ credentials: <secret> ]
  # 요청에서 사용할 credential은 설정한 파일에서 읽어온 credential로 설정한다.
  # `credentials`와는 함께 사용할 수 없다.
  [ credentials_file: <filename> ]

# OAuth 2.0 설정 (Optional).
# basic_auth나 authorization과는 동시에 사용할 수 없다.
oauth2:
  [ <oauth2> ]

# 스크랩 요청에서 HTTP 3xx 리다이렉트 지시를 따를지 여부를 설정한다.
[ follow_redirects: <bool> | default = true ]

# 스크랩 요청에서 사용할 TLS 설정을 구성한다.
tls_config:
  [ <tls_config> ]

# 프록시 URL (Optional).
[ proxy_url: <string> ]

# Azure 서비스 디스커버리 설정 목록.
azure_sd_configs:
  [ - <azure_sd_config> ... ]

# Consul 서비스 디스커버리 설정 목록.
consul_sd_configs:
  [ - <consul_sd_config> ... ]

# DigitalOcean 서비스 디스커버리 설정 목록.
digitalocean_sd_configs:
  [ - <digitalocean_sd_config> ... ]

# Docker 서비스 디스커버리 설정 목록.
docker_sd_configs:
  [ - <docker_sd_config> ... ]

# Docker Swarm 서비스 디스커버리 설정 목록.
dockerswarm_sd_configs:
  [ - <dockerswarm_sd_config> ... ]

# DNS 서비스 디스커버리 설정 목록.
dns_sd_configs:
  [ - <dns_sd_config> ... ]

# EC2 서비스 디스커버리 설정 목록.
ec2_sd_configs:
  [ - <ec2_sd_config> ... ]

# Eureka 서비스 디스커버리 설정 목록.
eureka_sd_configs:
  [ - <eureka_sd_config> ... ]

# file 서비스 디스커버리 설정 목록.
file_sd_configs:
  [ - <file_sd_config> ... ]

# GCE 서비스 디스커버리 설정 목록.
gce_sd_configs:
  [ - <gce_sd_config> ... ]

# Hetzner 서비스 디스커버리 설정 목록.
hetzner_sd_configs:
  [ - <hetzner_sd_config> ... ]

# HTTP 서비스 디스커버리 설정 목록.
http_sd_configs:
  [ - <http_sd_config> ... ]

# Kubernetes 서비스 디스커버리 설정 목록.
kubernetes_sd_configs:
  [ - <kubernetes_sd_config> ... ]

# Kuma 서비스 디스커버리 설정 목록.
kuma_sd_configs:
  [ - <kuma_sd_config> ... ]

# Lightsail 서비스 디스커버리 설정 목록.
lightsail_sd_configs:
  [ - <lightsail_sd_config> ... ]

# Linode 서비스 디스커버리 설정 목록.
linode_sd_configs:
  [ - <linode_sd_config> ... ]

# Marathon 서비스 디스커버리 설정 목록.
marathon_sd_configs:
  [ - <marathon_sd_config> ... ]

# AirBnB의 Nerve 서비스 디스커버리 설정 목록.
nerve_sd_configs:
  [ - <nerve_sd_config> ... ]

# OpenStack 서비스 디스커버리 설정 목록.
openstack_sd_configs:
  [ - <openstack_sd_config> ... ]

# PuppetDB 서비스 디스커버리 설정 목록.
puppetdb_sd_configs:
  [ - <puppetdb_sd_config> ... ]

# Scaleway 서비스 디스커버리 설정 목록.
scaleway_sd_configs:
  [ - <scaleway_sd_config> ... ]

# Zookeeper Serverset 서비스 디스커버리 설정 목록.
serverset_sd_configs:
  [ - <serverset_sd_config> ... ]

# Triton 서비스 디스커버리 설정 목록.
triton_sd_configs:
  [ - <triton_sd_config> ... ]

# Uyuni 서비스 디스커버리 설정 목록.
uyuni_sd_configs:
  [ - <uyuni_sd_config> ... ]

# 이 job으로 분류할 타겟 목록을 정적으로 설정한다.
static_configs:
  [ - <static_config> ... ]

# 타겟 relabel 설정 목록.
relabel_configs:
  [ - <relabel_config> ... ]

# 메트릭 relabel 설정 목록.
metric_relabel_configs:
  [ - <relabel_config> ... ]

# 압축하지 않은 응답 바디가 이 사이즈를 넘어가면 스크랩에 실패한다.
# 0은 제한이 없음을 의미한다. ex: 100MB.
# 이 설정은 실험적인 기능으로, 향후 동작이 변경되거나 제거될 수 있다.
[ body_size_limit: <size> | default = 0 ]

# 스크랩 단위로 허용할 샘플 수를 제한한다.
# 메트릭 relabeling을 적용한 후에 샘플 수가 이 숫자보다 많으면
# 스크랩 자체를 실패한 것으로 처리한다. 0은 제한이 없음을 의미한다.
[ sample_limit: <int> | default = 0 ]

# 스크랩 단위로 샘플 하나에서 허용할 레이블 수를 제한한다.
# 메트릭 relabeling을 적용한 후에 레이블 수가 이 숫자보다 많으면
# 스크랩 자체를 실패한 것으로 처리한다. 0은 제한이 없음을 의미한다.
[ label_limit: <int> | default = 0 ]

# 스크랩 단위로 샘플에서 허용할 레이블명의 길이를 제한한다.
# 메트릭 relabeling을 적용한 후에 레이블명이 이 숫자보다 길면
# 스크랩 자체를 실패한 것으로 처리한다. 0은 제한이 없음을 의미한다.
[ label_name_length_limit: <int> | default = 0 ]

# 스크랩 단위로 샘플에서 허용할 레이블 값의 길이를 제한한다.
# 메트릭 relabeling을 적용한 후에 레이블 값이 이 숫자보다 길면
# 스크랩 자체를 실패한 것으로 처리한다. 0은 제한이 없음을 의미한다.
[ label_value_length_limit: <int> | default = 0 ]

# 스크랩 단위로 설정할 수 있는 고유 타겟 수를 제한한다.
# 타겟 relabeling을 적용한 후에 타겟이 이 숫자보다 많으면,
# 프로메테우스는 타겟들을 스크랩하지 않고 실패한 것으로 마킹한다.
# 0은 제한이 없음을 의미한다.
# 이 설정은 실험적인 기능으로, 향후 동작이 변경될 수 있다.
[ target_limit: <int> | default = 0 ]
```

## Recording rules

프로메테우스에선 `recording rule` 과 `alerting rule` 두 타입의 rule 을 설정한 뒤 일정한 간격으로 평가할 수 있다.    
프로메테우스에 rule을 추가할 땐,   
필요한 rule 구문을 담고 있는 파일(YAML)을 만들어 프로메테우스 설정에서 `rule_files` 필드를 통해 로드하면 된다.     

rule 파일은 프로메테우스 프로세스에 SIGHUP 을 전송하면 런타임에 다시 로드할 수 있다.   
변경사항은 모든 rule 파일 포맷에 문제가 없을 때만 반영된다.   

### Syntax-checking rule

프로메테우스 커맨드라인 유틸리티 툴 `promtool` 을 사용하면  
프로메테우스 서버를 시작해보지 않아도 rule 파일 구문을 바로 검증해볼 수 있다.    

```shell
promtool check rules /path/to/example.rules.yml
```
`promtool` 바이너리 파일은 prometheus 아카이브에 포함되 있다.   
파일이 구문상 문제가 없으면 표준 출력에 파싱한 rule 들을 텍스트로 찍은 다음 종료 상태 `0` 을 리턴하고 종료한다.    
구문 오류가 있거나 입력 인자가 올바르지 않으면 stderr에 에러 메시지를 출력하고 종료 상태 1과 함께 종료한다.   

### Recording rules

Recording rule 을 사용하면 자주 필요한 표현식이나 계산 비용이 큰 표현식을   
미리 계산해서 그 결과를 별도 시계열 셋으로 저장해둘 수 있다.    

Recording rule, altering rule 은 rule 그룹 내에 존재한다.   
같은 그룹내에 있는 rule은 모두 함께 평가되며, 일정한 간격을 두고 순차적으로 실행된다.    
Recording rule 의 이름엔 [유효한 메트릭명](https://godekdls.github.io/Prometheus/data-model/#metric-names-and-labels)을 사용해야하며,    
altering rule의 이름엔 [유효한 레이블값](https://godekdls.github.io/Prometheus/data-model/#metric-names-and-labels)이 사용되어야 한다.   


**구문(틀)**
```yml
groups:
  [ - <rule_group> ]
```
룰 파일의 구문은 위와 같다.  

**예시**
```yml
groups:
  - name: example
    rules:
    - record: job:http_inprogress_requests:sum
      expr: sum by (job) (http_inprogress_requests)
```

### `<rule_group>`

```yml
<rule_group>
# 그룹명. 이 파일 내에서 유니크해야 한다.
name: <string>

# 이 그룹에 있는 rule들을 평가할 간격.
[ interval: <duration> | default = global.evaluation_interval ]

# alerting rule이 생성할 수 있는 alert 갯수와 recording rule이 생성할 수 있는 시계열 갯수를 제한한다.
# 0은 제한이 없음을 의미한다.
[ limit: <int> | default = 0 ]

rules:
  [ - <rule> ... ]
```

### `<rule>` 
#### Recording rule  

```yml
# 출력할 시계열 데이터 이름. 유효한 메트릭명을 지정해야 한다.
record: <string>

# 평가할 PromQL 표현식.
# 매 평가 주기마다 현재 시간을 기준으로 평가하고 새 시계열 셋에 기록한다.
# 메트릭명은 'record'에 지정한 이름을 사용한다.
expr: <string>

# 결과를 저장하기 전 추가하거나 덮어쓸 레이블.
labels:
  [ <labelname>: <labelvalue> ]
```
#### altering_rul  

```yml
# alert 이름. 유효한 레이블 값을 지정해야 한다.
alert: <string>

# 평가할 PromQL 표현식.
# 매 평가 주기마다 현재 시간을 기준으로 평가하며,
# 평가 결과는 alert 시계열에 보류 (pending) 또는 시행 상태(firing)로 저장한다.
expr: <string>

# 이 기간 동안 alert가 지속해서 반환된다면 시행(firing)되는 것으로 간주한다.
# 이 기간이 지났음에도 시행되지 않은 alert는 보류(pending) 중인 것으로 간주한다.
[ for: <duration> | default = 0s ]

# 각 alert에 추가하거나 덮어쓸 레이블.
labels:
  [ <labelname>: <tmpl_string> ]

# 각 alert에 추가할 애노테이션.
annotations:
  [ <labelname>: <tmpl_string> ]
```







