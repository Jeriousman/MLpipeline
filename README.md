## 머신러닝 파이프라인

[머신러닝 파이프라인](https://www.notion.so/1fd334f193144ac9a5d23213c6d4e356)

---

머신러닝을 자동화 시키고 쉽게 관리하기 위해 파이프라인을 구성하는 것이 중요합니다.

파이프라인을 구성하자면 대략적으로 아래와 같이 표현 할 수 있습니다.

데이터 수집/저장 → 데이터 유효성 검사 → 데이터 불러오기 → 데이터 전처리 → 머신러닝모델 트레이닝/하이퍼파라메터튜닝 → 모델배포  → 머신러닝 결과값 추론  

이 순서를 리서치 하면서 어떤 서비스를 사용해서 구축해야 하는지 생각해 보았을 때 아래와 같이 생각하게 되었습니다.

데이터 수집 [몽고DB] → 데이터 저장 [AWS S3] → 데이터 유효성 검사 [AWS EMR] → 데이터 불러오기 [AWS EMR] → 데이터 전처리 [AWS EMR] → 머신러닝모델 트레이닝/하이퍼파라메터튜닝 [AWS SageMaker] → 모델배포 [AWS SageMaker/TorchServe] → 머신러닝 결과값 추론 [AWS SageMaker]  

**SageMaker과정을 관리/디버깅/스케줄링 등, 통합적 관리를 위해 Airflow를 사용 할 수 있습니다.*

---

### 여기서 각 서비스들에 대한 간략한 설명을 보실 수 있습니다.

몽고DB: 크로스 플랫폼 도큐먼트 지향 데이터베이스 시스템입니다. NoSQL 데이터베이스로 분류되는 몽고DB는 JSON과 같은 동적 스키마형 도큐먼트들(몽고DB는 이러한 포맷을 BSON이라 부름)을 선호함에 따라 전통적인 테이블 기반 관계형 데이터베이스 구조의 사용을 삼가합니다.

[https://www.mongodb.com/ko/cloud](https://www.mongodb.com/ko/cloud)

AWS S3: Amazon S3는 업계 최고의 확장성, 데이터 가용성 및 보안과 성능을 제공하는 객체 스토리지 서비스입니다. 

[https://aws.amazon.com/ko/s3/](https://aws.amazon.com/ko/s3/)

AWS EMR: Amazon EMR은 Apache Spark, Apache Hive, Apache HBase, Apache Flink, Apache Hudi 및 Presto와 같은 오픈 소스 도구를 사용하여 방대한 양의 데이터를 처리하기 위한 업계 최고의 클라우드 빅 데이터 플랫폼입니다.

[https://aws.amazon.com/ko/emr/?whats-new-cards.sort-by=item.additionalFields.postDateTime&whats-new-cards.sort-order=desc](https://aws.amazon.com/ko/emr/?whats-new-cards.sort-by=item.additionalFields.postDateTime&whats-new-cards.sort-order=desc)

AWS SageMaker: Amazon SageMaker를 통해 데이터 사이언티스트와 개발자는 ML을 위해 특별히 구축된 다양한 기능 세트를 함께 활용하여 고품질 기계 학습(ML) 모델을 빠르게 준비, 구축, 훈련 및 배포할 수 있습니다.

[https://aws.amazon.com/ko/sagemaker/](https://aws.amazon.com/ko/sagemaker/)

TorchServe: 파이토치 모델을 배포하는데 쓰는 라이브러리입니다. AWS와 연동해서 쓸 수 있다고 하지만 아직 미성숙한 단계라는 말이 있습니다.

[https://github.com/pytorch/serve](https://github.com/pytorch/serve)

Airflow: SageMaker 단계들을 관리하는 Airflow는 아래의 사진처럼 머신러닝 파이프라인 중 어느 부분에서 문제가 있는지 등을 쉽게 확인할 수 있습니다

<Airflow 웹 UI>

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1b99b107-06ae-4b9c-8d3c-9c0268f5aff9/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1b99b107-06ae-4b9c-8d3c-9c0268f5aff9/Untitled.png)

*출처: [https://aws.amazon.com/ko/blogs/machine-learning/build-end-to-end-machine-learning-workflows-with-amazon-sagemaker-and-apache-airflow/](https://aws.amazon.com/ko/blogs/machine-learning/build-end-to-end-machine-learning-workflows-with-amazon-sagemaker-and-apache-airflow/)*

---

### 여기서 각 사용할 툴을 왜 우리 앱에 사용하려고 하는지 간략하게 얘기해 보겠습니다.

데이터 수집 [몽고DB]

몽고DB로 이미 진행중인 것으로 알고 있습니다. 

데이터 저장 [AWS S3]

몽고 DB를 현재 앱서비스 서버로도 사용하고 있기 때문에, 머신러닝을 위한 쿼리처리를 하다가 DB에 부하가 가서 서비스에 차질이 생기면 안되므로 데이터들을 S3로 보내서 저장을 하려고 합니다. 즉, 머신러닝을 위해 데이터를 사용해야 할 때, DB에 접근하기 않고 S3로 접근하여 서버문제를 원천차단하고자 함이 큰 목적입니다. S3는 가장 저렴한 AWS의 서비스 중 하나이며, 용량을 언제나 늘릴 수 있으므로 우리가 하드웨어를 매번 새로 사는 것보다 더욱 유연하게 데이터를 저장 할 수 있을 것으로 보입니다.

데이터 유효성 검사, 데이터 불러오기, 데이터 전처리 [AWS EMR]

실제로 컴퓨팅을 해 보아야 알겠지만, 캐치잇플레이가 가지고 있는 데이터의 양이 빅데이터의 양만큼 크지는 않을 것이기 때문에 EMR을 사용하는 것이 현재로써 큰 강점이 있다고 생각하지 않습니다. 왜냐하면 EMR은 빅데이터 대용량 데이터를 처리하기 위한 서비스이기 때문입니다. 그러나 캐치잇플레이가 커질 것을 예상하고 EMR을 미리 써보는 것도 나쁘지 않을 것으로 보입니다 (물론 서비스가 너무 비싸면 계획 변경). EMR은 제가 사용하려고 생각한 다음단계 서비스인 SageMaker와 EMR Spark를 모두 EMR Notebook에서 코딩 할 수 있어 부드러운 연동이 될 것으로 보입니다.

머신러닝 트레이닝/튜닝 [AWS SageMaker or Deep Learning AMI] 

전처리 된 데이터로 AWS EC2의 컴퓨팅 파워를 빌려와 SageMaker에서 모델트레이닝을 하려고 합니다. SageMaker에는 머신러닝 프로세스를 전체적으로 빠르게 이어 줄 수 있는 기능들을 가지고 있고, 머신러닝을 모르더라도 기본적인 이미지분류등 작업들을 쉽게 실행할 수 있는 디폴트 모델들을 가지고 있습니다 (물론 저희가 쓸 모델들은 별로 없는 것으로 보입니다만)

모델배포 [AWS SageMaker]

SageMaker에서 지원하는TorchServe 등을 사용하여 모델을 프로덕션에 배포하려고합니다. SageMaker에서 Kubernetes나 TorchServe를 지원하는 것으로 보여, 모델을 실제 프로덕션을 위한 배포를 하기 용이할 것으로 생각됩니다. 

머신러닝 결과값 추론 [AWS SageMaker]

SageMaker를 사용하여 모델을 완성하였으면 모델을 통해 원하는 예측값을 출력하고 그 값들을 S3에 저장하여 실제 서비스에 사용할 수 있도록 하려고 합니다.
