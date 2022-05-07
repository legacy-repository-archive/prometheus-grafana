# Unit Testing for Rules

정의한 rule 들은 `promtool` 을 통해 테스트가 가능하다.   

```shell 
# 단일 테스트 파일 테스트.
./promtool test rules test.yml

# 테스트 파일이 여러 개라면 test1.yml,test2.yml,test3.yml이라고 알려주면 된다.
./promtool test rules test1.yml test2.yml test3.yml
```

## Test File Format.  

```yml
# 테스트하고 싶은 rule 파일 목록. glob 패턴을 지원한다.
rule_files:
  [ - <file_name> ]

[ evaluation_interval: <duration> | default = 1m ]

# 아래에 그룹명을 나열한 순서대로 rule 그룹들을 평가한다 (정해진 평가 시간에).
# 순서는 아래에서 언급한 그룹들에 대해서만 보장한다.
# 모든 그룹을 전부 다 나열할 필요는 없다.
group_eval_order:
  [ - <group_name> ]

# 여기에 모든 테스트를 나열한다.
tests:
  [ - <test_group> ]
```

### `<test_group>`. 

```yml
# 시계열 데이터
interval: <duration>
input_series:
  [ - <series> ]

# 테스트 그룹 이름
[ name: <string> ]

# 위 데이터로 테스트해볼 내용들.

# alerting rule을 위한 단위 테스트들. 입력 파일에서 지정한 alerting rule을 이용해 테스트한다.
alert_rule_test:
  [ - <alert_test_case> ]

# promql 표현식을 위한 단위 테스트들.
promql_expr_test:
  [ - <promql_test_case> ]

# alert 템플릿에서 사용할 수 있는 외부 레이블들.
external_labels:
  [ <labelname>: <string> ... ]

# alert 템플릿에서 사용할 수 있는 외부 URL.
# 보통 --web.external-url로 설정한다.
  [ external_url: <string> ]
```

### `<series>`

```shell
# 여기서는 평상시 사용하는 시계열 표기법 '<metric name>{<label name>=<label value>, ...}'를 따른다.
# 예시:
#      series_name{label1="value1", label2="value2"}
#      go_goroutines{job="prometheus", instance="localhost:9090"}
series: <string>

# 여기서는 확장 표기법(Expanding notation)을 사용한다.
# 확장 표기법:
#     'a+bxc'는 'a a+b a+(2*b) a+(3*b) … a+(c*b)'와 동일하다
#     'a-bxc'는 'a a-b a-(2*b) a-(3*b) … a-(c*b)'와 동일하다
# 누락된 샘플이나 정체 중인 샘플은 전용 키워드로 표현할 수 있다.
#    '_'은 스크랩 중 누락된 샘플을 나타낸다
#    'stale'은 정체 중인 샘플을 의미한다
# 예시:
#     1. '-2+4x3'은 '-2 2 6 10'과 동일하다
#     2. ' 1-2x4'는 '1 -1 -3 -5 -7'과 동일하다
#     3. ' 1 _x3 stale'은 '1 _ _ _ stale'과 동일하다
values: <string>
```

### `<alert_test_case>`
  
프로메테우스에선 여러 alerting rule이 동일한 alert 이름을 사용할 수 있디.      
따라서 여기에선 특정 alert 이름에서 시행되는 모든 alert를 하나의 `<alert_test_case>` 로 통합해서 나열해야한다.     

```shell
# 0초부터 시작해서 이 시간만큼 경과했을 때의 데이터로 alert를 체크한다.
eval_time: <duration>

# 테스트할 alert 이름
alertname: <string>

# 지정한 평가 시간에 지정한 alert 이름에서 시행할 것으로 예상하는 alert 목록.
# alerting rule이 시행되지 않는 것을 테스트하고 싶다면
# 위 필드들만 지정하고 'exp_alerts'를 비워 두면 된다.
exp_alerts:
  [ - <alert> ]
```

### `<alert>`  

```shell
# 예상 alert에 첨부되는 레이블과 애노테이션들.
# 참고: 레이블은 alert를 활성화한 샘플의 레이블도 포함이다 (`/alerts` 페이지에서 볼 수 있는 레이블과 동일하며, 시계열 `__name__`과 `alertname` 레이블은 제외다).
exp_labels:
  [ <labelname>: <string> ]
exp_annotations:
  [ <labelname>: <string> ]
```
 
### `<promql_test_case>`

```shell
# 평가할 표현식
expr: <string>

# 0초부터 시작해서 이 시간만큼 경과했을 때의 데이터로 표현식을 평가한다.
eval_time: <duration>

# 지정한 평가 시간에 예상하는 샘플들.
exp_samples:
  [ - <sample> ]
```
