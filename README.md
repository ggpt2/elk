docker exec -it elasticsearch bin/elasticsearch-reset-password -u elastic

#유저생성
curl -u elastic:jt+1aIMg0V6uLm8X5QO7 -X POST "localhost:9200/\_security/user/logstash_user" -H "Content-Type: application/json" -d'
{
"password" : "logstash_password",
"roles" : [ "logstash_writer" ],
"full_name" : "Logstash User",
"email" : "logstash@example.com"
}'

curl -u elastic:jt+1aIMg0V6uLm8X5QO7 -X PUT "localhost:9200/\_security/role/logstash_writer" -H "Content-Type: application/json" -d'
{
"cluster": ["all"],
"indices": [
{
"names": [ "logstash-*" ],
"privileges": ["write","create","create_index","manage"]
}
]
}'

curl -u elastic:jt+1aIMg0V6uLm8X5QO7 -X POST "localhost:9200/\_security/user/kibana_user" -H "Content-Type: application/json" -d'
{
"password" : "kibana_password",
"roles" : [ "kibana_system" ],
"full_name" : "Kibana User",
"email" : "kibana@example.com"
}'

curl -u elastic:jt+1aIMg0V6uLm8X5QO7 -X POST "localhost:9200/\_security/user/test_user" -H "Content-Type: application/json" -d'
{
"password" : "test_password",
"roles" : [ "kibana_system" ],
"full_name" : "Kibana User",
"email" : "kibana@example.com"
}'

#유저확인
curl -u elastic:jt+1aIMg0V6uLm8X5QO7 -X GET "localhost:9200/\_security/user?pretty"

#테스트
curl -X POST "http://localhost:5044" -H "Content-Type: application/json" -d'
{
"message": "Hello, ELK Stack! 2222"
}'

docker logs --tail 10 -f logstash

## SSL인증서

docker exec -it elasticsearch /bin/bash

elasticsearch-certutil http

unzip elasticsearch-ssl-http.zip

cd elasticsearch

# 개인 키 추출

openssl pkcs12 -in http.p12 -nocerts -out elasticsearch.key

# 인증서 추출

openssl pkcs12 -in http.p12 -clcerts -nokeys -out elasticsearch.crt

# CA 인증서 추출

openssl pkcs12 -in http.p12 -cacerts -nokeys -out ca.crt

docker cp elasticsearch:/usr/share/elasticsearch/elasticsearch ./certs

#elasticsearch 테스트
curl -k -u elastic:jt+1aIMg0V6uLm8X5QO7 https://localhost:9200

#logstash 테스트
curl -X POST "http://localhost:5044" -H "Content-Type: application/json" -d'
{
"message": "Hello, ELK Stack! 44444"
}'

curl -X GET "https://localhost:9200/logstash-test-\*/\_search" -H "Content-Type: application/json" -u elastic:jt+1aIMg0V6uLm8X5QO7 --cacert ./certs/ca.crt
