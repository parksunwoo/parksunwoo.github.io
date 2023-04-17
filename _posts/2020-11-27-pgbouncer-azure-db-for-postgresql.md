---
title: "[번역] PgBouncer 설정으로 Azure DB for PostgreSQL 성능 높이기"
categories:
  - Dev
tags:
  - postgresql
  - azure
  - pgbouncer
last_modified_at: 2020-11-27T00:53:00-00:00
---

https://techcommunity.microsoft.com/t5/azure-database-for-postgresql/steps-to-install-and-setup-pgbouncer-connection-pooling-proxy/ba-p/730555

위 블로그 내용을 번역해 해당 포스트를 작성했음을  알립니다.

---
이전 Azure 블로그 게시물에서 PostgreSQL에서 연결 풀링 프록시를 활용하는 것이 중요한 이유와 애플리케이션 계층에서 연결 풀링을 사용하지 않는다고 가정할 때, PostgreSQL용 Azure 데이터베이스에 대해 PgBouncer와 같은 연결 풀링 프록시를 사용하면 성능을 크게 향상시킬 수 있는 방법에 대해 설명한 바 있습니다. PgBouncer를 사용하여 Azure Database for PostgreSQL과의 연결 풀링에 사용할 때 성능 향상을 어느 정도 예상하기 위해 pgbench로 간단한 성능 벤치마크 테스트를 실행했습니다. pgbench는 모든 트랜잭션에 대해 새 연결을 생성하는 구성 설정을 제공하므로 이를 활용하여 연결 지연 시간이 애플리케이션의 처리량에 미치는 영향을 측정했습니다. 다음은 PgBouncer를 사용했을 때와 사용하지 않았을 때 처리량을 표준 pgbench 벤치마크 테스트와 비교하는 A/B 테스트에서 관찰된 결과입니다. 테스트에서는 범용 티어(GP_Gen5_2)에서 실행되는 2 vCore(GP_Gen5_2)의 PostgreSQL용 Azure 데이터베이스에 대해 스케일 팩터 5로 pgbench를 실행했습니다. 테스트 중 유일한 변수는 PgBouncer였습니다. PgBouncer를 사용하면 아래 그림과 같이 처리량이 4배 향상되는 동시에 연결 지연 시간이 40% 감소했습니다.

![performance improvement with PgBouncer](https://techcommunity.microsoft.com/t5/image/serverpage/image-id/121737i363E08FEFF67D931/image-dimensions/700x310?v=v2)

이 블로그 게시물에서는 애플리케이션과 PostgreSQL용 Azure DB 간에 PgBouncer 연결 풀링 프록시를 설치 및 설정하여 성능 및 복원력의 이점을 얻는 단계를 공유합니다. 또한, 블로그 게시물의 마지막에 빠른 시작 템플릿을 제공하여 가상 머신에 PgBouncer를 빠르게 배포 및 설정하고 시작할 수 있도록 지원합니다.

아래 이미지와 같이 애플리케이션과 데이터베이스 레이어 사이에 PgBouncer 연결 프록시가 설정됩니다. PostgreSQL용 Azure DB는 완전히 관리되는 플랫폼 서비스이므로 데이터베이스 서버에 외부 구성 요소를 설치할 수 없습니다. 이 경우, Azure VM에서 애플리케이션을 실행하는 경우, 애플리케이션을 실행하는 동일한 VM에 PgBouncer를 설정할 수 있습니다. 애플리케이션이 Azure App Services 또는 Azure Functions와 같은 관리형 서비스에서 실행되는 경우, PgBouncer 프록시 서비스를 실행하기 위해 별도의 Ubuntu VM을 프로비저닝해야 할 수 있습니다. 그러나 Azure Kubernetes 서비스(AKS)에서 애플리케이션을 실행하는 경우, PgBouncer 사이드카를 활용하여 애플리케이션에 대한 Kubernetes 사이드카로 실행할 수 있습니다. 이 블로그 포스팅에서는 우분투 VM에서 PgBouncer를 설치하고 실행하는 단계에 초점을 맞추고, PgBouncer 사이드카를 설정하는 단계는 후속 블로그 포스팅에서 다룰 수 있는 주제입니다.

![Compare Connecting Pooling](https://techcommunity.microsoft.com/t5/image/serverpage/image-id/121738iC727A324F10A610F/image-dimensions/427x277?v=v2)

Ubuntu VM에서 PgBouncer를 설정하는 단계
아래에 언급된 모든 단계는 Azure CLI를 사용하여 리소스를 프로비저닝하고 배포합니다.

1. 우분투 VM을 프로비저닝합니다. 애플리케이션이 이미 Azure VM에서 실행 중이고 VM이 프로비저닝된 경우 이 단계를 건너뛰고 2단계로 이동하면 됩니다.
    
    ```bash
    az group create --name myResourceGroup --location eastus
    
    az vm create \
    
    --resource-group myResourceGroup \
    
    --name PgBouncerPoolVM \
    
    --image UbuntuLTS \
    
    --admin-username pgadminuser \
    
    --generate-ssh-keys
    ```
    

1. 우분투 VM의 NSG(네트워크 보안 그룹)에서 포트 5432를 열어 PgBouncer가 해당 포트에서 애플리케이션에서 들어오는 연결을 수신 대기할 수 있도록 합니다.
    
    ```bash
    az vm open-port --port 5432 --resource-group myResourceGroup -name PgBouncerPoolVM
    ```
    
2. 중요 참고: 위의 명령은 VM에 대한 NSG를 생성하고 VM에서 포트 5432를 모든 소스 IP에 대해 엽니다. 이는 보안 관점에서 권장되지 않습니다. 이상적으로는 VM이 VNet에 구성되어야 하며, 클라이언트 애플리케이션 VM으로만 PgBouncer VM에 대한 액세스를 제한하기 위해 az network nsg를 사용하여 NSG에서 소스 IP 범위를 정의하고 화이트리스트에 추가해야 합니다.

```bash
ssh pgadminuser@<PublicIPEndpoint>

sudo apt-get update
sudo apt-get install -y pgbouncer
```

1. 인증 기관에서 인증서 파일을 다운로드하여 SSL 연결을 사용하여 PostgreSQL용 Azure DB에 연결합니다.

```bash
sudo wget https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt

sudo openssl x509 -inform DER -in BaltimoreCyberTrustRoot.crt -text -out /etc/root.crt
```

1. etc/pgbouncer/pgBouncer.ini에 있는 Pgbouncer.ini 구성 파일을 다음 설정을 사용하여 수정합니다.

```bash
[databases]

 * = host=<servername>.postgres.database.azure.com  port=5432

[pgbouncer]

 # Do not change these settings:

 listen_addr = 0.0.0.0

 auth_file = /etc/pgbouncer/userlist.txt

 auth_type = trust

 server_tls_sslmode = verify-ca

 server_tls_ca_file = /etc/root.crt

 # These are defaults and can be configured

 # please leave them as defaults if you are

 # uncertain.

 listen_port = 5432

 unix_socket_dir =

 user = postgres

 pool_mode = session

 max_client_conn = 100

 ignore_startup_parameters = extra_float_digits

 admin_users = postgres
```

1. etc/pgbouncer/userlist.txt에 있는 인증 파일을 수정하여 클라이언트가 PgBouncer 서비스에 액세스하는 데 사용할 수 있는 사용자 이름/비밀번호 쌍을 지정합니다. 각 줄은 아래 형식이어야 합니다.

```
"username@hostname" "password"

 Each entry is simply new line separated for example:

 "sa@mypgserver" "P@ssword1234"

 "test@mypgserver" "Test@#1234"
```

참고: PostgreSQL용 Azure 데이터베이스 사용자 이름은 항상 사용자 이름@호스트 이름 형식입니다.

1. PgBouncer 서비스를 시작하고 로그에 오류가 없는지 확인합니다.

```bash
sudo service pgbouncer start

more /var/log/postgresql/pgbouncer.log
```

1. 가상 머신에 postgresql-client가 설치되어 있지 않은 경우, postgresql-client를 설치하고 psql을 사용하여 PgBouncer 서비스에 대한 연결의 유효성을 검사한 다음, PostgreSQL 서비스용 백엔드 Azure DB에 연결합니다.

```bash
sudo apt-get update

sudo apt-get install postgresql-client

psql -h 127.0.0.1 -p 5432 -U sa@mypgserver -d postgres
```

1. 외부 애플리케이션 VM 또는 워크스테이션에서 PgBouncer 서비스에 연결하여 연결이 성공했는지 확인합니다.

```
psql -h <PublicIPEndpoint> -p 5432 -U sa@mypgserver -d postgres
```

이제 PgBouncer 연결 풀링 프록시를 활용하여 PostgreSQL용 Azure DB 서비스에 연결하도록 모든 설정이 완료되었습니다.