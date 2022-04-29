# Glossary(용어 사전) 

## Alert
Alert 은 프로메테우스에서 alerting rule 이 활성상태(fire)로 시행될 때 만들어진다.      
Alert 은 프로메테우스에서 Altermanager로 전송된다.   

## AlertManager
 
AlertManager 는 alert 를 받아 그룹으로 집계하고, 중복을 제거하고,     
slience와 throttle 을 적용한 다음 이메일이나 Pagerduty, Slack 등에 통보를 보낸다.    
(진짜 알림용 메니저라고 생각하면 된다)    

## Bridge

Bridge 는 클라이언트 라이브러리에서 샘플을 가져와 프로메테웃그 외의 다른 모니터링 시스템에 노출해주는 컴포넌트다.   

## Client Library 

클라이언트 라이브러리는 프로메테우스를 여러 가지 언어로 제공하는 라이브러리다.  
 
1. 어플리케이션 코드에서 직접 지표를 측정하고     
2. 커스텀 컬렉터를 작성해 다른 시스템에서 메트릭을 가져오고       
3. 이 메트릭을 프로메테우스에 노출하는 것을 도와준다.        

## Collector   

컬렉터는 익스포터에 사용하는 인터페이스로, 메트릭 셋을 나타낸다.     
직접 계측에서 사용한다면 단일 메트릭을 표현한걸 나타낼수도 있고,     
다른 시스템에서 메트릭을 가져온다면 여러 개의 메트릭을 나타낼 수도 있디.  

## Direct instrumentation.  

직접 계측은 클라이언트 라이브러리를 통해 프로그램의 소스 코드에 인라인으로 계측 코드를 추가하는 것을 말한다.   

## EndPoint 

스크랩할 메트릭을 가져올 소스로, 보통 단일 프로세스의 엔드포인트를 의미한다.   

## Exporter 

익스포터는 메트릭을 추출할 어플리케이션과 함께 실행하는 바이너리다.       
익스포터가 프로메테우스 메트릭을 노출할 때는 보통,     
프로메테우스 포맷을 따르지 않는 메트릭을 프로메테우스가 지원하는 형식으로 변환시킨다.      
(JMX)

## Instance 

instnace 는 특정 Job에 있는 타겟을 고유하게 식별해주는 레이블이다.  

## Job

같은 목적을 가진 타겟 모음을 하나의 Job 이라고 부른다.    
예를 들어, 확장성이나 안정성을 위해 복제한 유사한 프로세스 그룹을 모니터링 할 수 있다.   

## Notification 

notification 은 하나 이상의 alert 그룹을 나타내며,   
Alertmanager 가 notification 을 이메일, Pagerduty, Slack 등으로 전송한다.   

## Promdash. 

Promdash는 과거 프로메테우스의 네이티브 대시보드 빌더였다.     
현재는 deprecated 되고 그라파나로 대체됐다.    

## Prometheus 

보통 프로메테우스라고 하면 **프로메테우스 시스템의 코어 바이너리를 뜻한다.**.   
프로메테우스 모니터링 시스템 전체를 지칭하기도 한다.   

## PromQL  
 
PromQL 은 프로메테우스 쿼리 언어다.      
aggregation, slicing/dicing, predication, join 을 포함한 다양한 연산을 지원한다.     

## Pushgateway 

Pushgateway 는 배치 job에서 가장 최근에 push한 메트릭을 지정한다.      
덕분에 프로메테우스는 배치 job이 종료된 후에도 관련 메트릭을 스크랩할 수 있다.   

## Remote Read

Remote Read는 달느 시스템(장기 스토리지)에서 시계열 데이터를 알아서 읽어와 질의해주는 프로메테우스 기능이다.   

## 







