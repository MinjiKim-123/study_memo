- <strong>문제: </strong> AWS IAM에 MFA를 인증해야 서비스 접근이 가능하도록 정책을 설정한 이후
로컬에서 기존의 사용자 액세스키로 접근이 되던 서비스들이 접근이 되지 않음.

- <strong>사유: </strong> 설정한 MFA정책으로 인해 서비스 접근시 MFA 인증된 토큰이 필요한데 
access key는 mfa인증을 포함하지 않음.

- <strong>해결 방안: </strong> 
'aws sts get-session-token --serial-number "mfa디바이스 시리얼" --token-code "인증코드"' <br/>
명령어를 호출해서 MFA인증을 한 세션 토큰을 발급받아서 임시 보안 자격을 생성. 
필요시 --duration-seconds "초" 옵션을 붙여서 세션 토큰의 생명 시간을 설정.
기본 값과 최대 값은 43200(12시간)
