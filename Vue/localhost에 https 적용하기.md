```shell  
brew install mkcert  
```  
* mac 기준으로 localhost에 https를 적용  
* brew로 mkcert 설치  
* brew가 없으면 brew 먼저 설치  
  
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
* 명령어 입력하면 현재 터미널 경로에 인증서 발급이 됨, 만약 원하는 경로가 있다면 터미널 경로 변경 필요  
* localhost.pem, localhost-key.pem 발급 됨  
* 발급된 인증서를 vue 프로젝트 root 아래에 두기  
* 테스트 해보지는 않았지만 현재 mac에서 발급된 인증서는 다른 디바이스에서 사용하지 못하는 것 같음  
* 만약 다른 디바이스에서 사용을 원한다면 해당 디바이스에서 위 절차를 이용해서 발급 받아야함, 즉 인증서는 발급 받은 디바이스에 한정됨(테스트 필요)  
  
```shell  
openssl x509 -in localhost.pem -text -noout  
```  
* 인증서 있는 경로에서 위 명령어 입력하면 인증서 내용 볼 수 있음  
* 내용에서 Subject를 보면 어떤 디바이스에서 발급된 인증서가 나옴  
* 그래서 해당 디바이스에만 사용할 수 있는 인증서라고 예상됨  
  
```javascript  
// vite.config.js

import { fileURLToPath, URL } from 'node:url';
import { defineConfig } from 'vite';
import fs from 'fs';
import vue from '@vitejs/plugin-vue';

// 현재 실행 모드 확인 (default: "development")
const isLocal = process.env.npm_lifecycle_event === 'local';

export default defineConfig(({ mode }) => ({
  server: {
    port: 5173,
    https: isLocal
      ? {
          key: fs.readFileSync('localhost-key.pem'),
          cert: fs.readFileSync('localhost.pem')
        }
      : false // local이 아닐 때는 HTTPS 비활성화
  },
  
  ... 생략
  
}));  
```  
* vite 기준으로 설정  
* vite.config.js 에 위 같이 설정하기  
* local환경에서만 mkcert로 발급한 인증서를 사용하겠다는 설정  
  
const isLocal = process.env.npm_lifecycle_event === 'local'   
* local 환경인지 체크  
* local 환경 아니면 https를 적용하지 않기 위함  
  
key: fs.readFileSync('localhost-key.pem'), cert: fs.readFileSync('localhost.pem')  
* 예제에서 인증서를 vue 프로젝트 root에 위치했기 때문에 경로를 설정함  
* 만약 인증서를 다른 위치에 뒀다면 해당 경로를 찾아가게 작성하면 됨   
  
