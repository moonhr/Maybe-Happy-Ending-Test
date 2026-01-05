# 어쩌면 해피엔딩 테스트

웹 기반 성격 테스트 애플리케이션입니다.

## 배포 설정

이 프로젝트는 GitHub Actions를 통해 자동으로 S3에 배포됩니다.

### GitHub Secrets 설정

GitHub 저장소의 Settings > Secrets and variables > Actions에서 다음 Secrets를 설정해야 합니다:

1. **AWS_ACCESS_KEY_ID**: AWS 액세스 키 ID
2. **AWS_SECRET_ACCESS_KEY**: AWS 시크릿 액세스 키
3. **S3_BUCKET_NAME**: S3 버킷 이름
4. **AWS_REGION**: AWS 리전 (선택사항, 기본값: ap-northeast-2)
5. **CLOUDFRONT_DISTRIBUTION_ID**: CloudFront 배포 ID (선택사항, 캐시 무효화용)

### AWS IAM 권한 설정

S3 업로드를 위한 최소 IAM 정책:

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
        "arn:aws:s3:::YOUR_BUCKET_NAME/*",
        "arn:aws:s3:::YOUR_BUCKET_NAME"
      ]
    }
  ]
}
```

CloudFront 캐시 무효화를 사용하는 경우 추가 권한:

```json
{
  "Effect": "Allow",
  "Action": [
    "cloudfront:CreateInvalidation"
  ],
  "Resource": "*"
}
```

### 배포 트리거

- `main` 또는 `master` 브랜치에 push할 때 자동 배포
- GitHub Actions 탭에서 수동 실행 가능

### 파일 캐시 설정

- HTML 및 JSON 파일: 캐시 없음 (항상 최신 버전)
- CSS, JS 등 정적 파일: 1년 캐시

