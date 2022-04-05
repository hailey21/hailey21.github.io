# AWS SDK for Java 1.v -> 2.x 마이그레이션 
## 레퍼런스 https://sdk.amazonaws.com/java/api/latest/
## 새 기능
1. HTTP 클라이언트 구성 가능
2. 비동기 프로그래밍 지원
3. 자동 페이지된 응답(*****)
4. SDK 시작 시간 성능 향상 - AWS Lambda 함수 개선 
5. 새로운 약식 메서드 지원 

## 사전 조건 
2.14.13 이상 AWS SDK 버전 사용 
1.x 2.x 동시사용 pom 구성
~~~
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.amazonaws</groupId>
            <artifactId>aws-java-sdk-bom</artifactId>
            <version>1.12.1</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <dependency>
          <groupId>software.amazon.awssdk</groupId>
          <artifactId>bom</artifactId>
          <version>2.16.1</version>
          <type>pom</type>
          <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <dependency>
      <groupId>com.amazonaws</groupId>
      <artifactId>aws-java-sdk-s3</artifactId>
    </dependency>
    <dependency>
      <groupId>software.amazon.awssdk</groupId>
      <artifactId>dynamodb</artifactId>
    </dependency>
</dependencies>
~~~
## 버전 1.x와 버전 2.x 간에 매핑된 자격 증명 공급자 변경
### 메서드 이름 변경
1.x|2.x|
---|---|
AWSCredentialsProvider.getCredentials|AwsCredentialsProvider.resolveCredentials|
DefaultAWSCredentialsProviderChain.getInstance|지원되지 않음|
AWSCredentialsProvider.getInstance|지원되지 않음|
AWSCredentialsProvider.refresh|지원되지 않음|

### 환경 변수 이름 변경 
1.x|2.x|
---|---|
AWS_ACCESS_KEY|AWS_ACCESS_KEY_ID|
AWS_SECRET_KEY|AWS_SECRET_ACCESS_KEY|
AWS_CREDENTIAL_PROFILES_FILE|AWS_SHARED_CREDENTIALS_FILE|

### 시스템 속성 이름 변경
1.x|2.x|
---|---|
aws.secretKey|aws.secretAccessKey|
com.amazonaws.sdk.disableEc2Metadata|aws.disableEc2Metadata|
com.amazonaws.sdk.ec2MetadataServiceEndpointOverride|aws.ec2MetadataServiceEndpoint|

### 리전 클래스 이름 변경 
com.amazonaws.regions.Regions/com.amazonaws.regions.Region -> software.amazon.awssdk.regions.Region 통합 

### 예외 클래스 이름 변경
1.x|2.x|
---|---|
|com.amazonaws.SdkBaseException/com.amazonaws.AmazonClientException|software.amazon.awssdk.core.exception.SdkException|
|com.amazonaws.SdkClientException|software.amazon.awssdk.core.exception.SdkClientException|
|com.amazonaws.AmazonServiceException|software.amazon.awssdk.awscore.exception.AwsServiceException|

예외 클래스 매핑 표 

1.x|2.x|
---|---|
|AmazonServiceException.getRequestId|SdkServiceException.requestId|
|AmazonServiceException.getServiceName|AwsServiceException.awsErrorDetails().serviceName|
|AmazonServiceException.getErrorCode|AwsServiceException.awsErrorDetails().errorCode|
|AmazonServiceException.getErrorMessage|AwsServiceException.awsErrorDetails().errorMessage|
|AmazonServiceException.getStatusCode|AwsServiceException.awsErrorDetails().sdkHttpResponse().statusCode|
|AmazonServiceException.getHttpHeaders|AwsServiceException.awsErrorDetails().sdkHttpResponse().headers|
|AmazonServiceException.rawResponse|AwsServiceException.awsErrorDetails().rawResponse|

## Next Step 
- sdk 2.x에 맞는 자격증명과 api를 사용하도록 변경 
- 예외처리 및 retry 관련 로직 확인 (https://docs.aws.amazon.com/ko_kr/sdk-for-java/latest/developer-guide/http-configuration.html)
