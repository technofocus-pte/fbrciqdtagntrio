# 사용 사례 04: 통합적이고 지능적인 데이터 인사이트를 위해 Fabric Data Agent를 Microsoft Foundry에 연결하기

**소개**

현대 조직은 여러 시스템에서 대량의 데이터를 생성하므로 비즈니스 사용자와 분석가가 인사이트에 빠르게 액세스하기가 어렵습니다. 데이터는 사일로에 저장되는 경우가 많으므로 정보를 추출, 분석, 해석하려면 기술 전문 지식이 필요합니다.

Microsoft Fabric 통합 데이터 플랫폼은 분석, 데이터 엔지니어링, 비즈니스 인텔리전스 기능을 단일 환경으로 통합하여 이러한 과제를 해결합니다. Microsoft Foundry의 에이전트 기반 AI 기능을 통합함으로써, 기업은 자연어와 자동화된 워크플로를 활용해 기업 데이터와 상호 작용하는 지능형 애플리케이션을 구축할 수 있습니다.

**Agentic Applications for Unified Data Foundation Solution Accelerator**는 AI 기반 에이전트가 통합된 기업 데이터를 활용하여 질문에 답하고, 데이터 분석을 자동화하며, 기술 및 비기술 사용자 모두에게 인사이트를 제공하는 방법을 보여줍니다. 이러한 Agentic Applications는 작업을 조정하고 관련 데이터를 검색하며 상황에 맞는 응답을 생성함으로써, 더 빠른 의사결정과 운영 효율성 향상을 가능하게 합니다.

이 사용 사례에서 조직은 AI 에이전트를 사용하여 매출 실적, 고객 행동, 제품 트렌드와 같은 비즈니스 데이터를 분석할 수 있습니다. 사용자는 여러 데이터 세트를 수동으로 조회하는 대신, 자연어로 간단히 질문하고 시스템으로부터 즉시 실행 가능한 인사이트를 얻을 수 있습니다.

**목표**

이 사용 사례의 목적은 조직이 **통합 데이터 기반의 에이전트형 AI(agentic AI with a unified data foundation)**를 활용하여 데이터 접근성과 의사결정을 개선하는 방법을 보여주는 것입니다.

주요 목표는 다음과 같습니다:

**1. Microsoft Fabric에서 통합 데이터 기반 구축**

- Lakehouse, Warehouse 및 semantic model을 포함하는, 거버넌스가 적용된 Fabric workspace를 생성합니다.

- 분석을 위해 엔터프라이즈 데이터 세트를 로드하고 검증합니다.

**2. Fabric Data Agent 빌드 및 구성**

- 자연어를 사용하여 데이터 세트를 쿼리할 수 있는 **Fabric Data Agent**를 생성합니다.

- Ontology 리소스를 연결하고 기업별 쿼리를 지원하기 위한 에이전트 지침을 정의합니다.

**3. Azure 및 Foundry 구성 요소 배포**

- Foundry 프로젝트, AI 서비스, 검색, 스토리지, 앱 서비스 등 Azure 리소스를 프로비저닝합니다.

- Azure Developer CLI(azd)를 통해 지원 구성 요소를 배포합니다.

**4. Fabric Data Agent를 Microsoft Foundry에 연결**

- Foundry 내에서 AI 에이전트를 생성하거나 구성합니다.

- Workspace ID와 AI Skills ID를 사용하여 에이전트를 Microsoft Fabric에 연결합니다.

- 에이전트가 Fabric 데이터를 해석하고 분석할 수 있도록 도메인별 지침을 제공합니다.

**5. Conversational Analytics 및 Automated Insights 활성화**

- Foundry Playground에서 실제 비즈니스 쿼리로 에이전트를 테스트합니다.

- Fabric Lakehouse 데이터 세트를 활용한 자연어-데이터 검색 워크플로를 시연합니다.

- 검사 합격/불합격률, 평균, 추세, 그룹별 요약 등의 인사이트를 제공합니다.

**6. 엔드투엔드 Agentic Application Workflow 시연**

- Foundry 에이전트, Fabric 데이터 소스, Azure 인프라를 통합하여 기능적인 웹 애플리케이션을 구축합니다.

- 지능형 데이터 상호작용, 자동화된 추론 및 인사이트 도출을 검증합니다.

**솔루션 아키텍처**

![](./media/image1.png)

이 솔루션은 Microsoft Fabric과 Microsoft Foundry를 결합하여 정형 데이터와 비정형 문서를 모두 활용해 질문에 답변할 수 있는 AI 솔루션을 구현합니다:

- **Microsoft Fabric**은 Lakehouse, Warehouse, 그리고 자연어를 SQL로 변환하는 Fabric IQ 시맨틱 계층을 통해 데이터 계층을 제공합니다.

- **Microsoft Foundry**는 문서 검색을 위한 Foundry IQ와 두 가지 기능을 조율하는 Orchestrator Agent를 포함한 AI 에이전트를 호스팅합니다.

- **Azure AI Services**가 언어 모델(GPT-4o-mini)과 임베딩을 구동합니다.

- **Azure AI Search**는 시맨틱 검색을 위해 문서 벡터를 저장합니다.

**전제 조건**

- **GitHub 계정:** 본인의 GitHub 계정이 필요합니다. 계정이 없으면 과제 0의 단계에 따라 +++https://github.com/signup+++에서 계정을 생성합니다.

## 과제 0: GitHub 계정 만들기

이 과제에서는 이번 실습에서 사용하는 것과 동일한 테넌트 자격 증명으로 새로운 **GitHub 계정**을 생성합니다.

1.  +++https://github.com/+++로 이동한 다음, **Sign up**을 클릭합니다.

![](./media/image2.png)

2.  새로운 GitHub 계정을 생성하려면 **Email**, **Password**, 그리고 고유한 **Username**을 입력한 다음 **Continue** 버튼을 클릭합니다.

![](./media/image3.png)

3.  화면의 지시에 따라 **verification puzzle**을 완료하고 **Submit**을 클릭합니다.

4.  이메일로 받은 **verification code**를 입력합니다.

![](./media/image4.png)

5.  계정 정보로 GitHub에 로그인하고 **Sign in**을 클릭합니다.

![](./media/image5.png)

6.  GitHub 계정이 성공적으로 생성되었습니다.

![](./media/image6.png)

## 과제 1: Fabric workspace 만들기

이 과제에서는 솔루션의 Lakehouse, Ontology, Data Agent를 호스팅할 Fabric workspace를 생성합니다.

1.  웹 브라우저를 열고 주소 표시줄에 다음 URL을 입력하거나 붙여넣은 다음 **Enter** 키를 누르고 계정 정보로 로그인합니다: +++https://app.fabric.microsoft.com/+++

| **사용자 이름** | **+++@lab.CloudPortalCredential(User1).Username+++** |
|----|----|
| **TAP** | **+++@lab.CloudPortalCredential(User1).AccessToken+++** |

2.  Fabric 홈 페이지에서 **+ New workspace** 타일을 선택합니다.

![](./media/image7.png)

3.  오른쪽에 나타나는 **Create a workspace** 창에서 다음 세부 정보를 입력한 다음, **Apply** 버튼을 클릭합니다.

| **설정** | **값** |
|----|----|
| Name | **+++FabricAgent-@lab.LabInstance.Id+++** |
| Advanced | **License mode**에서 **Fabric**을 선택 |
| Default storage format | **Small dataset storage format** |
| Template apps | **Develop template apps**를 선택 |

![](./media/image8.png)

**참고:** Lab Instance ID를 확인하려면 **Help**를 선택하고 Instance ID를 복사합니다.

![](./media/image9.png)

![](./media/image10.png)

4.  배포가 완료될 때까지 기다립니다. 완료까지 2~3분이 소요됩니다.

![](./media/image11.png)

## 과제 2: Fabric workspace ID 가져오기

솔루션을 빌드할 때 매개변수로 전달할 workspace ID가 필요합니다.

1.  브라우저의 URL을 확인합니다. workspace ID는 **/groups/** 뒤에 표시되는 GUID입니다.

2.  URL(예: https://app.fabric.microsoft.com/groups/\{workspace-id\}/...)에서 **Workspace ID**를 복사하여 나중에 사용할 수 있도록 **메모장**에 저장합니다.

![](./media/image12.png)

## 과제 3: 개발 환경 열기

1.  브라우저를 열고 주소창에 다음 URL을 입력하거나 붙여넣습니다: +++https://github.com/technofocus-pte/agnticapp-for-unified-data/tree/main+++

![](./media/image13.png)

2.  **Fork**를 클릭하여 repo를 fork합니다. repo에 고유한 이름을 지정하고 **Create fork** 버튼을 클릭합니다.

![](./media/image14.png)

![](./media/image15.png)

3.  **Code** > **Codespaces** > **Create codespace on main**을 클릭합니다.

![](./media/image16.png)

4.  Codespaces 환경이 설정될 때까지 기다립니다. 설정이 완료되기까지 몇 분 정도 소요됩니다.

![](./media/image17.png)

![](./media/image18.png)

![](./media/image19.png)

## 과제 4: 서비스를 프로비저닝하고 Azure 및 Fabric에 애플리케이션 배포하기

1.  터미널에서 다음 명령을 실행합니다. 복사할 코드가 생성됩니다. 해당 코드를 복사한 후 **Enter** 키를 누릅니다.

    +++azd auth login+++

![](./media/image20.png)

2.  인증을 위해 기본 브라우저가 열립니다. 코드를 입력하고 **Next**를 클릭한 다음, 위의 자격 증명으로 로그인합니다.

![](./media/image21.png)

![](./media/image22.png)

![](./media/image23.png)

![](./media/image24.png)

3.  Azure에 로그인합니다:

    +++az login+++

![](./media/image25.png)

4.  인증을 위해 기본 브라우저가 열립니다. 생성된 코드를 입력하고 **Next**를 클릭합니다.

![](./media/image26.png)

5.  터미널에서 Azure 구독을 선택합니다.

![](./media/image27.png)

6.  Microsoft Cognitive Services 리소스 공급자를 등록합니다(구독에 아직 등록되지 않은 경우 필수).

    +++az provider register --namespace Microsoft.CognitiveServices+++

![](./media/image28.png)

**알림:** 왼쪽 탐색기에서 **infra** 폴더로 이동한 다음, **main.bicep** 파일의 **122번째 줄**에서 *Lab Instance ID* 문자열을 +++@lab.LabInstance.Id+++로 변경하고 파일을 저장합니다.

7.  모든 리소스를 프로비저닝하고 배포합니다:

    +++azd up+++

![](./media/image29.png)

8.  메시지가 표시되면 아래 값을 입력하거나 선택합니다.

    - **Enter a unique environment name**: +++env@lab.LabInstance.Id+++

    - **Select an Azure Subscription to use**: **@lab.CloudSubscription.Name**

    - **'aiDeploymentsLocation' infrastructure parameter**: **ResourceGroup1**과 같은 위치를 선택

    - **Resource group**: **@lab.CloudResourceGroup(ResourceGroup1).Name**

![](./media/image30.png)

![](./media/image31.png)

![](./media/image32.png)

**참고:** 선택한 Azure 지역에서 배포가 실패하면 다음 명령으로 배포 지역을 변경한 후 **azd up**을 다시 실행합니다.

```bash
azd env set AZURE_RESOURCE_LOCATION <region>
```

예:

```bash
azd env set AZURE_RESOURCE_LOCATION westus2
```

지원 지역: **westus2**, **japaneast**, **swedencentral**, **northeurope**

9.  이 배포 과정에서 리소스를 프로비저닝하고 샘플 데이터로 솔루션을 설정하는 데 **7~10분**이 소요됩니다.

10. 배포가 완료되었습니다.

![](./media/image33.png)

11. 가상 환경을 만듭니다:

    +++python -m venv .venv+++

![](./media/image34.png)

12. **Visual Studio Code** 왼쪽 상단의 **menu** 아이콘을 클릭한 다음, **Terminal** > **New Terminal**로 이동하여 새 터미널을 엽니다. 새 터미널에서 다음 명령을 실행하여 가상 환경을 활성화합니다.

    +++source .venv/bin/activate+++

![](./media/image35.png)

![](./media/image36.png)

13. 다음 명령을 실행하여 필요한 Python 종속성을 설치합니다.

    +++pip install uv && uv pip install -r scripts/requirements.txt+++

![](./media/image37.png)

![](./media/image38.png)

14. 다음 명령을 실행합니다. 복사할 코드가 생성됩니다. 해당 코드를 복사한 후 **Enter** 키를 누르고 브라우저에서 인증을 완료합니다.

    +++az login+++

![](./media/image39.png)

![](./media/image40.png)

![](./media/image41.png)

15. 목록에서 **Azure subscription**을 선택합니다.

![](./media/image42.png)

16. azd 배포 결과에 출력된 스크립트를 실행합니다. **\<your-workspace-id\>**를 과제 2에서 저장한 Fabric workspace ID로 바꿉니다. 스크립트는 다음과 같습니다:

    +++python scripts/00_build_solution.py --from 02 --fabric-workspace-id <your-workspace-id>+++

![](./media/image43.png)

17. **Enter**를 눌러 리소스 생성을 시작합니다.

![](./media/image44.png)

![](./media/image45.png)

![](./media/image46.png)

## 과제 5: Fabric Lakehouse 및 데이터 검토

1.  +++https://app.fabric.microsoft.com/+++에서 **FabricAgent-@lab.LabInstance.Id** workspace로 이동합니다.

2.  리소스가 성공적으로 배포되었는지 확인합니다.

![](./media/image47.png)

3.  **Lakehouse**를 클릭하여 데이터가 성공적으로 로드되었는지 확인합니다.

![](./media/image48.png)

![](./media/image49.png)

4.  **Codespace**로 돌아가 에이전트를 테스트합니다.

## 과제 6: 에이전트 테스트하기

1.  에이전트를 테스트하려면 터미널에서 다음 명령을 실행합니다.

    +++python scripts/08_test_agent.py+++

![](./media/image50.png)

2.  다음 예시 질문을 입력합니다:

    +++What is the average score from inspections?+++

![](./media/image51.png)

3.  다음 질문을 입력합니다:

    +++What constitutes a failed inspection?+++

![](./media/image52.png)

![](./media/image53.png)

4.  다음 질문을 입력합니다:

    +++Do any inspections violate quality control standards in our Inspection Procedures?+++

![](./media/image54.png)

![](./media/image55.png)

5.  **Ctrl+C**를 눌러 프로세스를 종료합니다.

![](./media/image56.png)

## 과제 7: Fabric data agent 만들기

1.  +++https://app.fabric.microsoft.com/+++에서 **FabricAgent-@lab.LabInstance.Id** workspace로 이동합니다.

2.  **+ New item**을 선택하고 +++Data agent+++를 검색한 다음 **Data agent**를 선택합니다.

![](./media/image57.png)

3.  이름으로 +++FabricDataAgent@lab.LabInstance.Id+++를 입력하고 **Create**를 클릭합니다.

![](./media/image58.png)

4.  새 데이터 소스를 구성하려면 **Add data source**를 선택합니다.

![](./media/image59.png)

5.  솔루션 배포로 생성된 **Ontology** 리소스(이름이 **_ontology_1**로 끝남)를 선택하고 **Add**를 클릭합니다.

![](./media/image60.png)

![](./media/image61.png)

6.  상단 메뉴에서 **Agent instructions**를 클릭합니다.

![](./media/image62.png)

7.  다음 에이전트 지침을 추가합니다:

    +++You are a helpful assistant that can answer user questions using data. Support group by in GQL+++

![](./media/image63.png)

![](./media/image64.png)

8.  상단 메뉴에서 **Publish**를 클릭하고, **Publish data agent** 창에서 **Publish**를 선택합니다.

![](./media/image65.png)

![](./media/image66.png)

![](./media/image67.png)

**참고:** Ontology 설정에 최대 15분이 소요될 수 있으므로, 적절한 응답이 표시되지 않으면 잠시 후 다시 시도합니다.

9.  에이전트 채팅 창에 다음 샘플 질문을 입력하여 응답을 확인합니다.

    +++How many tickets are high priority+++

![](./media/image68.png)

![](./media/image69.png)

    +++What is the average score from inspections?+++

![](./media/image70.png)

![](./media/image71.png)

![](./media/image72.png)

    +++Show tickets grouped by status+++

![](./media/image73.png)

![](./media/image74.png)

10. 브라우저 주소 표시줄의 URL에서 **Workspace ID**(**/groups/** 뒤의 GUID)와 **AISkills ID**(**/aiskills/** 뒤의 GUID)를 복사하여 나중에 사용할 수 있도록 **메모장**에 저장합니다.

![](./media/image75.png)

11. 애플리케이션을 배포하고 실행하기 위해 **Codespace**로 돌아갑니다.

## 과제 8: 애플리케이션 배포 및 실행하기

1.  배포 전에 다음 명령을 실행하여 **AZURE_ENV_DEPLOY_APP** 환경 변수를 **true**로 설정합니다.

    +++azd env set AZURE_ENV_DEPLOY_APP true+++

![](./media/image76.png)

2.  **azd up**을 실행하여 Azure 리소스와 앱을 배포합니다.

    +++azd up+++

![](./media/image77.png)

3.  배포가 성공적으로 완료되면 웹 앱 URL을 복사합니다.

![](./media/image78.png)

4.  다음 명령을 실행하여 앱 권한을 설정합니다.

    +++python scripts/00_build_solution.py --from 09+++

![](./media/image79.png)

5.  **Enter** 키를 눌러 구성을 시작합니다.

![](./media/image80.png)

![](./media/image81.png)

6.  앱 URL을 클릭합니다.

![](./media/image82.png)

![](./media/image83.png)

![](./media/image84.png)

**샘플 질문**

앱에서 다음 **샘플 질문**을 물어볼 수 있습니다.

소매 판매 분석 사용 사례:

+++Show total revenue by year for last 5 years+++

![](./media/image85.png)

![](./media/image86.png)

![](./media/image87.png)

![](./media/image88.png)

**알림:** 데이터의 날짜 범위로 인해 응답이 표시되지 않을 수 있습니다. 이 경우 남은 실습을 계속 진행합니다.

## 과제 9: Azure 리소스 확인

1.  브라우저를 열고 +++https://portal.azure.com+++에 접속한 다음, 아래 자격 증명으로 로그인합니다.

| **사용자 이름** | **+++@lab.CloudPortalCredential(User1).Username+++** |
|----|----|
| **TAP** | **+++@lab.CloudPortalCredential(User1).AccessToken+++** |

2.  **Resource groups**를 선택합니다.

![](./media/image89.png)

3.  할당된 **ResourceGroup1** 리소스 그룹을 클릭합니다.

![](./media/image90.png)

4.  아래 리소스가 성공적으로 배포되었는지 확인합니다.

    - Foundry

    - Foundry project

    - Application Insights

    - Search service

    - Storage account

    - App Service

    - Azure Cosmos DB account

![](./media/image91.png)

## 과제 10: Microsoft Foundry에서 Fabric data agent 사용하기

1.  리소스 그룹에서 **Foundry** 리소스를 선택합니다.

![](./media/image92.png)

2.  **Overview** 창에서 **Go to Foundry portal**을 클릭합니다. Microsoft Foundry 포털로 이동합니다.

![](./media/image93.png)

![](./media/image94.png)

3.  Foundry 포털의 왼쪽 메뉴에서 **Agents**를 선택하면 **미리 생성된** 에이전트를 확인할 수 있습니다. 에이전트가 없으면 **+ New agent**를 클릭하여 에이전트를 생성합니다.

![](./media/image95.png)

![](./media/image96.png)

4.  에이전트를 선택하면 오른쪽에 구성 창이 열립니다. 에이전트 이름으로 +++Fabric Agent+++를 입력합니다.

![](./media/image97.png)

5.  같은 구성 창에서 아래로 스크롤하여 **Knowledge** 항목의 **+ Add**를 클릭합니다.

![](./media/image98.png)

6.  **Add knowledge** 창에서 **Microsoft Fabric**을 선택합니다.

![](./media/image99.png)

7.  **+ Create connection**을 클릭합니다.

![](./media/image100.png)

8.  **Custom keys**에 과제 7의 10단계에서 저장한 값을 입력하고 각 키의 **is secret**을 선택합니다. **Connection name**에 +++Fabric-aiskills+++를 입력하고 **Connect**를 클릭합니다.

| **키** | **값** |
|----|----|
| +++workspace-id+++ | 저장한 **Workspace ID** |
| +++artifact-id+++ | 저장한 **AISkills ID** |

![](./media/image101.png)

9.  **Instructions** 상자에 다음 지침을 입력합니다.

```
당신은 Microsoft Fabric에 저장된 검사 데이터를 분석하는 데이터 보조 도구입니다.

Fabric Lakehouse 데이터 세트를 사용하여 검사 결과 및 점수에 관한 질문에 답하십시오. 이 데이터 세트에는 다음과 같은 열이 포함되어 있습니다.

- inspection_id: 각 검사에 대한 고유 식별자
- ticket_id: 검사 티켓과 관련된 식별자
- result: 검사 결과 (합격 또는 불합격)
- score: 검사에 부여된 수치 점수

데이터를 분석하고 요약하여 다음과 같은 인사이트를 제공할 수 있습니다:

- 검사 총 횟수
- 합격한 검사와 불합격한 검사의 수
- 평균, 최고 및 최저 검사 점수
- 검사 결과의 분포
- 검사 또는 티켓별 점수 추세

응답할 때:

- 정확한 정보를 가져오려면 Fabric 데이터 소스를 사용하십시오.
- 검사 결과를 바탕으로 명확한 요약과 인사이트를 제공하십시오.
- 적절한 경우, 합격과 불합격의 분포나 점수 비교를 보여주기 위해 막대 그래프나 원형 그래프와 같은 시각화 자료를 제안하십시오.
- 답변은 간결하고 정확하며, 제공된 데이터 세트만을 바탕으로 작성되어야 합니다.
```

![](./media/image102.png)

10. 왼쪽 메뉴에서 **Agents**를 선택한 다음, **Fabric Agent**를 선택하고 **Try in playground**를 클릭합니다.

![](./media/image103.png)

11. 프롬프트를 입력할 수 있는 채팅 패널이 열립니다. 에이전트는 연결된 문서와 데이터 세트를 사용하여 응답합니다. 다음 샘플 프롬프트를 입력합니다.

    +++What constitutes a failed inspection?+++

![](./media/image104.png)

![](./media/image105.png)

    +++What is the total number of tickets in the system?+++

![](./media/image106.png)

![](./media/image107.png)

    +++Do any inspections violate quality control standards in our Inspection Procedures?+++

![](./media/image108.png)

![](./media/image109.png)

## 과제 11: 리소스 삭제하기

1.  Azure portal의 검색 창에 +++Resource groups+++를 입력하고, **Services** 아래의 **Resource groups**를 클릭합니다.

![](./media/image110.png)

2.  **ResourceGroup1** 리소스 그룹을 선택합니다.

3.  리소스 그룹 홈 페이지에서 **Fabric Capacity**를 제외한 모든 리소스를 선택한 다음, **Delete**를 클릭합니다.

![](./media/image111.png)

4.  오른쪽에 나타나는 **Delete Resources** 창의 **Enter "delete" to confirm deletion** 입력란에 +++delete+++를 입력한 다음, **Delete** 버튼을 클릭합니다.

![](./media/image112.png)

![](./media/image113.png)

5.  +++https://app.fabric.microsoft.com/+++에서 **FabricAgent-@lab.LabInstance.Id** workspace로 이동합니다.

6.  workspace 이름 아래에 있는 **...** 옵션을 선택하고 **Workspace settings**를 선택합니다.

![](./media/image114.png)

7.  **General** 탭에서 **Remove this workspace**를 선택합니다.

![](./media/image115.png)

8.  팝업으로 나타나는 경고 창에서 **Delete**를 클릭합니다.

![](./media/image116.png)

9.  workspace가 삭제되었다는 알림이 표시될 때까지 기다립니다.

![](./media/image117.png)

**요약**

이 사용 사례는 조직이 **Microsoft Fabric**과 **Microsoft Foundry**를 통합하여 **지능형 에이전트 기반 데이터 애플리케이션**을 구축하는 방법을 보여줍니다. 이 솔루션은 Fabric Lakehouse 및 Warehouse에 저장된 기업 데이터에 AI 기반 에이전트를 통해 접근하고 이를 분석할 수 있는 **통합 데이터 기반**을 마련합니다.

사용자는 **Fabric Data Agent**를 Foundry에 연결함으로써, 복잡한 SQL을 작성하거나 여러 데이터 소스를 수동으로 분석하는 대신 **자연어 쿼리**를 사용하여 기업 데이터 세트를 활용할 수 있습니다. 이 AI 에이전트는 관련 데이터를 검색 및 분석하고, 평균, 추세, 요약, 그룹화된 결과 등의 인사이트를 생성합니다.

또한 이 솔루션은 AI 서비스, 검색, 스토리지, 웹 애플리케이션 등 지원 Azure 서비스를 프로비저닝하여 완전한 **엔드투엔드 에이전트 기반 애플리케이션 아키텍처**를 구현합니다. 이를 통해 조직은 **구조화된 기업 데이터를 AI 기능**과 결합하여 대화형 분석 및 자동화된 인사이트를 제공할 수 있습니다.

전반적으로 이 사용 사례는 통합 데이터 플랫폼을 기반으로 구축된 **에이전트형 AI 애플리케이션**이 기술 및 비기술 사용자 모두를 위해 데이터 접근을 간소화하고, 분석을 가속화하며, 더 신속한 데이터 기반 의사결정을 지원하는 방식을 보여줍니다.

