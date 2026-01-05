# S3 버킷 생성 및 설정 가이드

## 추천 버킷 이름

S3 버킷 이름은 전 세계적으로 고유해야 하므로, 다음 중 하나를 선택하거나 변형하여 사용하세요:

### 추천 목록 (우선순위 순)

1. **`maybe-happy-ending-test`** ⭐ (가장 추천)
   - 프로젝트 이름과 일치
   - 명확하고 간결함

2. **`maybe-happy-ending-web`**
   - 웹 애플리케이션임을 명시

3. **`happy-ending-quiz-app`**
   - 퀴즈 앱임을 강조

4. **`maybe-happy-ending-prod`**
   - 프로덕션 환경임을 명시

5. **`mhe-test-app`** (약어 버전)
   - 짧고 간결함

### 버킷 이름 규칙
- 3-63자 길이
- 소문자, 숫자, 하이픈(-)만 사용 가능
- 점(.)은 사용 가능하지만 권장하지 않음 (HTTPS 인증서 이슈)
- IP 주소 형식 불가
- 전 세계적으로 고유해야 함

## S3 버킷 생성 단계

### 1. AWS 콘솔에서 버킷 생성

1. AWS 콘솔 로그인 → S3 서비스 이동
2. "버킷 만들기" 클릭
3. 버킷 설정:
   - **버킷 이름**: 위 추천 이름 중 선택 (예: `maybe-happy-ending-test`)
   - **AWS 리전**: `ap-northeast-2` (서울) 또는 원하는 리전
   - **객체 소유권**: ACL 비활성화 (권장) 또는 버킷 소유자 선호
   - **퍼블릭 액세스 차단 설정**: 
     - ✅ "모든 퍼블릭 액세스 차단" 해제 (웹사이트 호스팅을 위해 필요)
     - 또는 "새 퍼블릭 버킷 정책을 통해 부여된 퍼블릭 액세스 허용" 체크
   - **버킷 버전 관리**: 필요시 활성화
   - **기본 암호화**: AES-256 또는 AWS-KMS 선택

### 2. 버킷 정책 설정 (퍼블릭 읽기 허용)

버킷 → 권한 탭 → 버킷 정책 편집:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME/*"
    }
  ]
}
```

⚠️ **주의**: `YOUR_BUCKET_NAME`을 실제 버킷 이름으로 변경하세요.

### 3. 정적 웹사이트 호스팅 활성화

버킷 → 속성 탭 → 정적 웹사이트 호스팅:

1. "정적 웹사이트 호스팅" 편집
2. "사용" 선택
3. 인덱스 문서: `index.html`
4. 오류 문서: (선택사항) `index.html` 또는 `error.html`
5. 저장

**웹사이트 엔드포인트 URL**이 생성됩니다:
```
http://YOUR_BUCKET_NAME.s3-website-ap-northeast-2.amazonaws.com
```

### 4. CORS 설정 (필요시)

외부 도메인에서 리소스 접근이 필요한 경우:

버킷 → 권한 탭 → CORS (교차 출처 리소스 공유) 편집:

```json
[
  {
    "AllowedHeaders": ["*"],
    "AllowedMethods": ["GET", "HEAD"],
    "AllowedOrigins": ["*"],
    "ExposeHeaders": [],
    "MaxAgeSeconds": 3000
  }
]
```

## IAM 사용자 및 권한 설정

### 1. IAM 사용자 생성

1. AWS 콘솔 → IAM 서비스
2. 사용자 → 사용자 추가
3. 사용자 이름: `github-actions-s3-deploy` (또는 원하는 이름)
4. 액세스 유형: "액세스 키 - 프로그래밍 방식 액세스" 선택
5. 다음 단계

### 2. 권한 정책 연결

**옵션 A: 기존 정책 직접 연결 (권장)**

"기존 정책 직접 연결" 선택 후 다음 정책 검색 및 선택:
- `AmazonS3FullAccess` (간단하지만 과도한 권한)
- 또는 아래 커스텀 정책 사용 (최소 권한 원칙)

**옵션 B: 커스텀 정책 생성 (최소 권한 원칙 - 권장)**

1. "정책 생성" 클릭
2. JSON 탭에서 다음 정책 붙여넣기:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:DeleteObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::YOUR_BUCKET_NAME",
        "arn:aws:s3:::YOUR_BUCKET_NAME/*"
      ]
    }
  ]
}
```

⚠️ **주의**: `YOUR_BUCKET_NAME`을 실제 버킷 이름으로 변경하세요.

3. 정책 이름: `S3DeployPolicy-YOUR_BUCKET_NAME`
4. 정책 생성 후 사용자에게 연결

### 3. 액세스 키 저장

1. 사용자 생성 완료 후 **액세스 키 ID**와 **시크릿 액세스 키** 복사
2. ⚠️ **중요**: 시크릿 액세스 키는 한 번만 표시되므로 안전하게 저장하세요.

## GitHub Secrets 설정

1. GitHub 저장소 → Settings → Secrets and variables → Actions
2. "New repository secret" 클릭하여 다음 Secrets 추가:

| Secret 이름 | 값 | 예시 |
|------------|-----|------|
| `AWS_ACCESS_KEY_ID` | IAM 사용자의 액세스 키 ID | `AKIAIOSFODNN7EXAMPLE` |
| `AWS_SECRET_ACCESS_KEY` | IAM 사용자의 시크릿 액세스 키 | `wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY` |
| `S3_BUCKET_NAME` | 생성한 S3 버킷 이름 | `maybe-happy-ending-test` |
| `AWS_REGION` | 버킷이 생성된 리전 | `ap-northeast-2` |
| `CLOUDFRONT_DISTRIBUTION_ID` | (선택사항) CloudFront 배포 ID | `E1234567890ABC` |

## CloudFront 설정 (선택사항)

더 빠른 속도와 HTTPS를 원하는 경우:

1. CloudFront → 배포 생성
2. 원본 도메인: 생성한 S3 버킷 선택
3. 뷰어 프로토콜 정책: "HTTPS만" 또는 "리디렉션 HTTP를 HTTPS로"
4. 배포 생성 후 배포 ID를 GitHub Secret에 추가

## 테스트

1. 코드를 `main` 또는 `master` 브랜치에 push
2. GitHub Actions 탭에서 workflow 실행 확인
3. S3 버킷에 파일이 업로드되었는지 확인
4. 웹사이트 엔드포인트 URL로 접속하여 정상 작동 확인

## 보안 체크리스트

- [ ] 버킷 정책에서 필요한 권한만 허용
- [ ] IAM 사용자에게 최소 권한만 부여
- [ ] GitHub Secrets에 민감 정보 저장 (코드에 하드코딩 금지)
- [ ] 버킷 버전 관리 활성화 (필요시)
- [ ] CloudFront를 통한 HTTPS 사용 (권장)
- [ ] 정기적인 액세스 키 로테이션

## 문제 해결

### 업로드 실패 시
- IAM 사용자 권한 확인
- 버킷 이름이 정확한지 확인
- AWS 리전이 일치하는지 확인

### 웹사이트 접속 불가 시
- 버킷 정책에서 퍼블릭 읽기 권한 확인
- 정적 웹사이트 호스팅 활성화 확인
- 인덱스 문서 이름이 `index.html`인지 확인

