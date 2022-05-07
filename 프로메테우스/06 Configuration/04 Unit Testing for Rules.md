# Unit Testing for Rules

정의한 rule 들은 `promtool` 을 통해 테스트가 가능하다.   

```shell 
# 단일 테스트 파일 테스트.
./promtool test rules test.yml

# 테스트 파일이 여러 개라면 test1.yml,test2.yml,test3.yml이라고 알려주면 된다.
./promtool test rules test1.yml test2.yml test3.yml
```
