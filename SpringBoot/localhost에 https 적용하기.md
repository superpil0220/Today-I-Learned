```shell  
brew install mkcert  
```  
* mac 기준으로 localhost에 https를 적용  
* brew로 mkcert 설치  
* brew가 없으면 brew 먼저 설치  
* 대부분 keytool을 이용해서 바로 .p12 인증서를 만들어서 사용한다. 이런경우 브라우저에서 “안전하지 않음”이슈가 발생해서 mkcert로 인증서 생성 후 .p12 로 변환시키는 작업이 필요하다.  
  
```shell  
mkcert -version  
```  
* 정상 설치면 버전 정보 출력됨  
  
```shell  
mkcert -install  
```  
* 최초 1회만 명령어 입력  
* 위 명령어는 로컬 개발 환경에 자체 인증기관(CA)을 설치하는 명령어  
* 위 명령어을 무시하고 진행할 경우 인증서 발급은 되지만 브라우저에서 “신뢰하지 않는 인증서” 라고 경고창 뜸  
* 위 명령어 입력하면 mac에서 비밀번호 입력창을 띄움, 본인 mac 비밀번호 입력  
  
```shell  
mkcert localhost  
```  
* localhost 도메인으로 인증서 발급, port는 안적어도 됨  
* 명령어 입력하면 현재 터미널 경로에 인증서 발급 됨, 만약 원하는 경로가 있다면 터미널 경로 변경 필요  
* localhost.pem, localhost-key.pem 발급 됨  
* 테스트 해보지는 않았지만 현재 mac에서 발급된 인증서는 다른 디바이스에서 사용하지 못하는 것 같음  
* 만약 다른 디바이스에서 사용을 원한다면 해당 디바이스에서 위 절차를 이용해서 새로 발급 받아야함, 즉 인증서는 발급 받은 디바이스에 한정됨(테스트 필요)  
  
```shell  
openssl x509 -in localhost.pem -text -noout  
```  
* 인증서 있는 경로에서 위 명령어 입력하면 인증서 내용 볼 수 있음  
* 내용에서 Subject를 보면 어떤 디바이스에서 발급된 인증서가 나옴  
* 그래서 해당 디바이스에만 사용할 수 있는 인증서라고 예상됨  
  
```shell  
openssl pkcs12 -export \
  -in localhost.pem \
  -inkey localhost-key.pem \
  -out keystore.p12 \
  -name local-ssl \
  -passout pass:password  
```  
* 이제 java에서 사용할 수 있게 .p12로 변환작업 필요  
* 위 명령어 입력하면 mkcert 인증서가 .p12로 변환됨  
* 변환된 인증서는 프로젝트에 resources 디렉토리 하위로 옮기기  
  
```yaml  
server:
  ssl:
    key-store: classpath:keystore.p12
    key-store-password: password
    key-store-type: PKCS12  
```  
* application.yml 파일에 위 정보 입력하기  
* .p12로 변환할때 비밀번호랑 인증서 이름 동일해야됨  
  
