---
title: Terragrunt란?
author: dnya0
date: 2026-07-01 23:27:00 +0900
categories: [Infra, Terraform]
tag: [Post, Terraform, Terragrunt, Infra]
math: true
mermaid: true
image:
  path: ../assets/img/posts-image/Terragrunt.png
  alt: Terragrunt Image
---

## 🥑 들어가며

최근 회사에서 Terraform으로 관리하던 인프라 구성을 Terragrunt 기반으로 마이그레이션했다.

기존에는 Terraform만으로도 인프라를 코드로 관리할 수 있었지만, 서비스가 확장되면서 테넌시별로 상태를 분리하고 공통 설정을 재사용할 수 있는 구조가 필요해졌다. 특히 여러 테넌시가 같은 형태의 인프라를 사용하면서도 각자의 상태는 독립적으로 관리되어야 했다.

또한 VPC, subnet, route table 같은 기본 네트워크 리소스는 여러 테넌시가 함께 사용하는 기반 리소스에 가까웠다. 그래서 이런 공통 인프라는 `core` 영역에서 먼저 관리하고, 각 테넌시는 `core`에서 만들어진 네트워크 정보를 참조해서 자신의 리소스를 구성하는 방식으로 구조를 잡았다.

이번 글에서는 Terragrunt가 무엇인지, Terraform만 사용할 때 어떤 불편함이 있었는지, 그리고 Terragrunt를 사용하면 어떤 방식으로 테넌시별 상태 관리와 확장 가능한 구조를 만들 수 있는지 정리해보려고 한다.

<br>

## 📌 Terragrunt란?

Terragrunt는 Terraform을 더 편하게 사용하기 위한 얇은 래퍼 도구다.

Terraform을 대체하는 도구가 아니라, Terraform 위에서 동작하면서 반복되는 설정을 줄이고 여러 환경의 인프라 코드를 더 구조적으로 관리할 수 있게 도와준다.

Terraform은 인프라를 코드로 선언하고, `plan`, `apply`를 통해 실제 클라우드 리소스를 생성하거나 변경할 수 있게 해준다. 하지만 실제 운영 환경에서는 단순히 리소스를 만드는 것보다 더 많은 고민이 필요하다.

- dev, stage, prod 같은 환경을 어떻게 나눌지
- 각 환경의 state를 어디에 저장할지
- 공통 module을 어떻게 재사용할지
- 테넌시별 설정을 어떻게 분리할지
- 환경이 늘어날 때 중복 코드를 어떻게 줄일지

Terragrunt는 이런 문제를 해결하기 위해 Terraform 설정 바깥에서 공통 설정, remote state, module source, dependency 등을 관리할 수 있게 해준다.

<br>

## Terraform만 사용할 때의 불편함

Terraform만 사용해도 module을 활용하면 어느 정도 중복을 줄일 수 있다.

하지만 환경이나 테넌시가 늘어나면 비슷한 설정이 여러 곳에 반복되기 쉽다. 예를 들어 같은 module을 사용하지만 tenant-a, tenant-b, tenant-c처럼 테넌시별로 변수만 다른 경우가 있다.

이때 Terraform 코드가 아래처럼 반복될 수 있다.

```text
terraform/
├── prod
│   ├── tenant-a
│   │   ├── main.tf
│   │   ├── backend.tf
│   │   └── variables.tf
│   ├── tenant-b
│   │   ├── main.tf
│   │   ├── backend.tf
│   │   └── variables.tf
│   └── tenant-c
│       ├── main.tf
│       ├── backend.tf
│       └── variables.tf
└── modules
    └── tenant
```

물론 동작은 하지만 테넌시가 추가될수록 관리해야 하는 파일이 많아진다. 특히 backend 설정이나 provider 설정처럼 거의 동일한 내용이 여러 디렉터리에 반복된다.

이런 구조에서는 다음과 같은 문제가 생길 수 있다.

- 테넌시별 backend 설정이 반복된다.
- 공통 설정이 변경될 때 여러 파일을 수정해야 한다.
- 설정 누락이나 불일치가 발생하기 쉽다.
- 신규 테넌시 추가 절차가 번거로워진다.
- state를 어떤 단위로 분리했는지 한눈에 파악하기 어렵다.

<br>

## 왜 테넌시별 state 분리가 필요할까?

Terraform은 state 파일을 통해 현재 인프라 상태를 추적한다.

state에는 Terraform이 관리하는 리소스 정보가 저장된다. 따라서 state를 어떻게 나눌지는 인프라 운영에서 중요한 설계 포인트다.

멀티 테넌시 환경에서는 테넌시별로 state를 분리하는 것이 유리한 경우가 많다.

예를 들어 tenant-a의 리소스를 수정하는 작업이 tenant-b의 state와 섞여 있다면 변경 영향 범위가 커진다. 반대로 테넌시별로 state가 분리되어 있다면 특정 테넌시의 변경은 해당 테넌시의 state 안에서만 관리된다.

테넌시별 state 분리는 다음과 같은 장점이 있다.

- 특정 테넌시 변경이 다른 테넌시에 영향을 주는 위험을 줄일 수 있다.
- 장애가 발생했을 때 영향 범위를 좁혀서 확인할 수 있다.
- 테넌시별 plan, apply 작업이 명확해진다.
- 신규 테넌시를 추가할 때 기존 테넌시의 state를 건드리지 않아도 된다.
- 운영 중 롤백이나 재적용 범위를 제한하기 쉽다.

회사에서도 이번 마이그레이션의 핵심은 Terraform 코드를 단순히 Terragrunt로 바꾸는 것이 아니라, 테넌시별로 state를 독립적으로 관리할 수 있는 구조를 만드는 것이었다.

<br>

## Terragrunt를 사용한 구조

Terragrunt를 사용하면 상위 디렉터리에 공통 설정을 두고, 하위 디렉터리에서는 각 환경이나 테넌시에 필요한 값만 정의할 수 있다.

이때 자주 사용하는 방식이 `live`와 `modules`를 나누는 구조다.

- `live`: 실제 환경별 Terragrunt 설정
- `modules`: 재사용 가능한 Terraform module

`modules`에는 VPC, tenant 리소스처럼 재사용할 Terraform 코드가 들어가고, `live`에는 prod, stage 같은 실제 환경에서 어떤 module을 어떤 값으로 실행할지 정의한다.

예를 들어 아래와 같은 구조를 만들 수 있다.

```text
infra/
├── live
│   ├── terragrunt.hcl
│   ├── prod
│   │   ├── core
│   │   │   └── network
│   │   │       └── terragrunt.hcl
│   │   └── tenants
│   │       ├── tenant-a
│   │       │   └── terragrunt.hcl
│   │       └── tenant-b
│   │           └── terragrunt.hcl
│   └── stage
│       ├── core
│       │   └── network
│       │       └── terragrunt.hcl
│       └── tenants
│           └── tenant-a
│               └── terragrunt.hcl
└── modules
    ├── network
    └── tenant
```

`live/terragrunt.hcl`에는 공통 remote state 설정이나 공통 input을 둘 수 있다.

```hcl
remote_state {
  backend = "s3"

  config = {
    bucket         = "my-terraform-state"
    key            = "${path_relative_to_include()}/terraform.tfstate"
    region         = "ap-northeast-2"
    encrypt        = true
    dynamodb_table = "terraform-lock"
  }
}
```

여기서 `path_relative_to_include()`를 사용하면 현재 Terragrunt 설정 파일의 상대 경로를 기준으로 state key를 만들 수 있다.

예를 들어 `live/prod/tenants/tenant-a`에서 실행하면 아래와 같은 key를 사용할 수 있다.

```text
prod/tenants/tenant-a/terraform.tfstate
```

`live/prod/tenants/tenant-b`에서 실행하면 state key는 달라진다.

```text
prod/tenants/tenant-b/terraform.tfstate
```

즉, 공통 remote state 설정은 재사용하면서도 실제 state 파일은 테넌시별로 분리할 수 있다.

<br>

## core와 tenant 분리

이번 구조에서 중요한 부분은 `core`와 `tenant`를 분리한 것이다.

`core`는 여러 테넌시가 공통으로 사용하는 기반 인프라를 관리하는 영역이다. 예를 들면 VPC, subnet, route table, security group처럼 테넌시별로 매번 새로 만들기보다 공통 기반으로 두고 함께 사용하는 리소스가 여기에 들어갈 수 있다.

반면 `tenant` 영역은 각 테넌시에 필요한 리소스를 관리한다. 이때 테넌시 리소스는 네트워크를 직접 새로 만드는 것이 아니라, `core`에서 만들어둔 네트워크 정보를 참조해서 사용한다.

이렇게 나누면 책임이 명확해진다.

- `core`: 공통 네트워크와 기반 리소스 관리
- `tenant`: 테넌시별 애플리케이션 리소스 관리
- `modules`: 실제 Terraform module 정의

예를 들어 `core/network`에서는 VPC와 subnet을 만들고 output으로 필요한 값을 내보낼 수 있다.

```hcl
output "vpc_id" {
  value = aws_vpc.main.id
}

output "private_subnet_ids" {
  value = aws_subnet.private[*].id
}
```

그리고 테넌시 쪽 Terragrunt 설정에서는 `dependency`를 사용해 `core/network`의 output을 가져올 수 있다.

```hcl
dependency "network" {
  config_path = "../../core/network"
}

inputs = {
  environment        = "prod"
  tenant_name        = "tenant-a"
  vpc_id             = dependency.network.outputs.vpc_id
  private_subnet_ids = dependency.network.outputs.private_subnet_ids
}
```

이 구조에서는 테넌시가 늘어나도 VPC나 subnet 같은 기본 네트워크를 계속 복사해서 만들 필요가 없다. 각 테넌시는 공통 네트워크 위에서 필요한 리소스만 추가하면 된다.

또한 `core`와 `tenant`의 state가 분리되기 때문에 공통 네트워크를 변경하는 작업과 특정 테넌시 리소스를 변경하는 작업을 구분할 수 있다. 공통 리소스를 수정할 때는 더 신중하게 리뷰하고, 테넌시별 변경은 상대적으로 좁은 범위에서 적용할 수 있다.

<br>

## 테넌시별 terragrunt.hcl 예시

각 테넌시 디렉터리의 `terragrunt.hcl`에서는 공통 설정을 include하고, 사용할 Terraform module과 입력값만 정의한다.

```hcl
include {
  path = find_in_parent_folders()
}

terraform {
  source = "../../../../modules/tenant"
}

dependency "network" {
  config_path = "../../core/network"
}

inputs = {
  environment        = "prod"
  tenant_name        = "tenant-a"
  vpc_id             = dependency.network.outputs.vpc_id
  private_subnet_ids = dependency.network.outputs.private_subnet_ids
}
```

`include`는 상위 디렉터리의 `terragrunt.hcl`을 가져오는 설정이다.

`find_in_parent_folders()`를 사용하면 현재 디렉터리에서부터 상위 디렉터리로 올라가며 `terragrunt.hcl`을 찾는다. 그래서 하위 테넌시 설정에서는 remote state 같은 공통 설정을 반복해서 작성하지 않아도 된다.

`terraform.source`에는 실제 사용할 Terraform module 경로를 지정한다.

`dependency`는 다른 Terragrunt module의 output을 가져올 때 사용한다. 위 예시에서는 `core/network`에서 생성한 VPC ID와 private subnet ID를 가져와 테넌시 module의 input으로 넘기고 있다.

결과적으로 테넌시별 디렉터리에서는 아래 내용만 관리하면 된다.

- 어떤 module을 사용할지
- 어떤 환경인지
- 어떤 테넌시인지
- 어떤 core 리소스를 참조할지
- 테넌시별로 달라지는 입력값은 무엇인지

<br>

## Terragrunt로 개선된 점

이번 마이그레이션을 통해 가장 크게 개선된 부분은 중복 제거와 확장성이다.

기존에는 새로운 테넌시를 추가할 때 비슷한 Terraform 설정을 복사하고, backend 설정이나 변수 파일을 직접 수정해야 했다. 이 과정에서 설정이 누락되거나 기존 테넌시와 다르게 작성될 가능성이 있었다.

Terragrunt를 사용하면 공통 설정은 상위 파일에서 관리하고, 테넌시별 설정은 필요한 입력값만 작성하면 된다.

신규 테넌시를 추가할 때도 전체 Terraform 코드를 복사할 필요가 없다. 새로운 디렉터리를 만들고 `terragrunt.hcl`에 테넌시별 input만 정의하면 된다.

```text
live/
└── prod/
    ├── core/
    │   └── network/
    │       └── terragrunt.hcl
    └── tenants/
        ├── tenant-a/
        │   └── terragrunt.hcl
        ├── tenant-b/
        │   └── terragrunt.hcl
        └── tenant-c/
            └── terragrunt.hcl
```

이 구조에서는 테넌시가 늘어나도 기존 module과 공통 설정을 재사용할 수 있다. 공통 네트워크는 `core`에서 한 번 관리하고, 각 테넌시는 그 output을 참조해서 필요한 리소스만 생성한다. 동시에 state는 `core`와 테넌시별로 분리되기 때문에 변경 영향 범위도 명확해진다.

<br>

## Terragrunt를 사용할 때 주의할 점

Terragrunt가 모든 문제를 자동으로 해결해주는 것은 아니다.

Terraform 위에 한 계층이 더 생기는 만큼 디렉터리 구조와 설정 규칙을 명확히 정해야 한다. 구조가 애매하면 Terraform만 사용할 때보다 오히려 복잡해질 수 있다.

특히 아래 내용은 초기에 잘 정하는 것이 좋다.

- state를 어떤 단위로 분리할지
- 환경과 테넌시 디렉터리 구조를 어떻게 가져갈지
- 공통 설정과 개별 설정의 경계를 어디에 둘지
- core 리소스와 tenant 리소스의 책임을 어디까지 나눌지
- module 버전 관리를 어떻게 할지
- 리소스 간 dependency를 Terragrunt로 관리할지
- 신규 테넌시 추가 절차를 어떻게 표준화할지

Terragrunt 도입의 핵심은 도구를 바꾸는 것이 아니라, 인프라 코드의 경계와 책임을 다시 설계하는 것에 가깝다.

<br>

## 마무리

Terragrunt는 Terraform을 대체하는 도구가 아니라 Terraform을 더 구조적으로 사용하기 위한 도구다.

Terraform만으로도 인프라를 코드로 관리할 수 있지만, 여러 환경과 여러 테넌시를 운영해야 하는 상황에서는 공통 설정과 state 관리가 점점 복잡해진다.

이번 마이그레이션에서는 Terragrunt를 사용해 공통 설정을 재사용하고, 테넌시별로 state를 분리할 수 있는 구조를 만들었다. 특히 기본 네트워크 같은 공통 리소스는 `core`에서 관리하고, 각 테넌시는 `core`의 output을 참조하는 방식으로 구성했다. 이를 통해 신규 테넌시를 추가할 때의 작업량을 줄이고, 변경 영향 범위를 더 명확하게 관리할 수 있었다.

결국 중요한 것은 Terragrunt 자체보다도 state를 어떤 단위로 나누고, 어떤 설정을 공통화하며, 어떤 값을 테넌시별로 분리할지 결정하는 설계다.

Terragrunt는 그 설계를 코드로 유지하기 쉽게 도와주는 도구라고 볼 수 있다.
