# CORS


- CORS (Cross Origin Resource Sharing)


|             |                    |
| ----------- | ------------------ |
| 동일 도메인 | 항상 허용          |
| 다른 도메인 | CORS에 의해 제어됨 |

``` txt
https://aaa.com -> https://bbb.com 

req
Origin: https://aaa.com

res
Access-Control-Allow-Origin: *



req
Access-Control-Request-Method: POST
Origin: https://aaa.com

res
Access-Control-Allow-Methods: POST
Access-Control-Allow-Origin: *
```