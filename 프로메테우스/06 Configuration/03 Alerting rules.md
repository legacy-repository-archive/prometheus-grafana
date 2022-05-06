# Alerting rules  

Altering rule 을 사용하면 프로메테우스 표현식 언어를 기반으로     
alert 조건을 정의하고 alert가 시행되면 외부 서비스에 통지할 수 있다.    
   
특정 시점에서 alert 표현식이 벡터 요소를 하나 이상 생성하게 되면,        
이 요소들이 가지고 있는 레이블 셋에서 alert가 활성 상태에 들어갔다고 판단한다.     

## Defining alerting rules   

프로메테우스에 altering rule 을 설정하는 방법은 recording rule 을 설정할 때와 동일하다.   
alert를 사용하는 rule 파일 예시는 다음과 같다.   

```
groups:
- name: example
  rules:
  - alert: HighRequestLatency
    expr: job:request_latency_seconds:mean5m{job="myjob"} > 0.5
    for: 10m
    labels:
      severity: page
    annotations:
      summary: High request latency
```

옵션으로 `for` 절을 지정하면,    
프로메테우스는 이 표현식으로 출력 벡터 요소를 처음 발견하고 나서부터    
지정 기간이 지난뒤에야 해당 alert를 시행된 것으로 계산한다.      

이 예제에선 매 평가마다 10분동안, alert 가 활성 상태로 지속되는지 확인해본 후 alert를 시행시킨다.    
활성 상태이지만 아직 시행되지 않은 요소들은 보류 상태로 본다.   
  
`labels` 절로는 alert에 첨부할 별도 레이블 셋을 지정할 수 있다.      
기존 레이블과 충돌되는게 있다면 모두 덮어쓴다.        
레이블 값은 템플릿을 사용해 지정할 수 있다.    

`annotations` 절에 alert 설명이나 런북 링크같이, 좀 더 긴 부가 정보를 저장할 수 있는 정보성 레이블 셋을 지정한다.   
애노테이션 값은 탬플릿을 사용해 지정할 수 있다.   







