# 사용 사례 03: Microsoft Fabric에서 Mirrored Azure SQL Database를 사용하여 Fabric Data Agent 구축하기

**소개**

현대 조직은 복잡한 데이터 이동 없이도 운영 데이터를 신속히 분석하고 의미 있는 인사이트를 제공할 수 있는 지능형 시스템을 필요로 합니다. 이 사용 사례에서는 Microsoft Fabric을 사용해 Azure SQL Database의 데이터를 Fabric 환경으로 미러링하고, 미러링된 데이터를 쿼리하고 분석할 수 있는 Fabric Data Agent를 생성합니다.

이 과정은 샘플 비즈니스 데이터를 포함하는 Azure SQL Database를 생성하는 것으로 시작됩니다. 이 데이터베이스는 Azure SQL Mirroring을 통해 Microsoft Fabric에 미러링되어, Fabric workspace 내에서 운영 데이터에 거의 실시간으로 접근할 수 있게 됩니다. Mirrored Database가 준비되면, 데이터 소스에 연결하고 자연어 질의에 응답하는 Fabric Data Agent를 구성합니다.

이 방식은 사용자가 복잡한 SQL 쿼리를 작성하지 않고도 지능형 에이전트를 통해 기업 데이터와 상호작용하며 제품 성과, 고객 분포, 판매 추세에 대한 빠른 인사이트를 얻을 수 있게 합니다.

**목표**

이 실습의 목적은 Azure SQL Database에서 미러링된 운영 데이터를 분석할 수 있는 Fabric Data Agent를 구축하고 구성하는 방법을 보여주는 것입니다.

이 실습을 완료하면 다음을 수행할 수 있습니다:

- 샘플 데이터를 사용해 **Azure SQL Database**를 생성합니다.

- 데이터와 분석 리소스를 호스팅할 **Microsoft Fabric workspace**를 생성합니다.

- Azure SQL Mirroring을 사용해 **Azure SQL Database를 Microsoft Fabric에** 미러링합니다.

- **Fabric Data Agent**를 구성하고 mirrored database에 연결합니다.

- **자연어 프롬프트**를 사용해 데이터를 질의하고 인사이트를 도출합니다.

- 샘플 분석 질문을 사용해 에이전트의 응답을 검증합니다.

## 과제 0: 호스트 환경 시간 동기화

1.  VM에서 **Search bar**를 찾아 클릭한 다음, +++Settings+++를 입력하고 **Best match** 아래에 있는 **Settings**를 클릭합니다.

![](./media/image1.png)

2.  Settings 창에서 **Time & language**를 찾아 클릭합니다.

![](./media/image2.png)

3.  **Time & language** 페이지에서 **Date & time**을 찾아 클릭합니다.

![](./media/image3.png)

4.  아래로 스크롤하여 **Additional settings** 섹션으로 이동한 다음, **Sync now** 버튼을 클릭합니다. 동기화에는 3~5분이 소요됩니다.

![](./media/image4.png)

5.  **Settings** 창을 닫습니다.

![](./media/image5.png)

## 과제 1: 단일 데이터베이스 만들기 - Azure SQL Database

이 과제에서는 샘플 데이터가 포함된 Azure SQL Database를 생성합니다. AdventureWorksLT 샘플 스키마를 배포하고 테이블을 확인한 다음, 이후 Fabric에서 미러링을 수행하는 데 필요한 서버 연결 정보를 준비합니다.

1.  브라우저를 열고 +++https://portal.azure.com+++에 접속한 다음, 아래 자격 증명으로 로그인합니다.

| **사용자 이름** | **+++@lab.CloudPortalCredential(User1).Username+++** |
|----|----|
| **비밀번호** | **+++@lab.CloudPortalCredential(User1).Password+++** |

2.  Azure Portal 홈 페이지에서 Microsoft Azure 명령 모음 왼쪽에 있는 가로줄 3개 모양의 **Azure portal menu**를 클릭하고 **SQL databases**를 선택합니다.

![](./media/image6.png)

3.  **+ Create**를 클릭합니다.

![](./media/image7.png)

4.  **Create SQL Database** 창의 **Basics** 탭에서 아래 세부 정보를 입력합니다.

| **설정** | **값 / 작업** |
|----|----|
| Subscription | 구독을 선택 |
| Resource group | **ResourceGroup1** 선택 |
| Database name | **+++sqldatabaseXXXX+++** (XXXX = 실습 Instance ID의 마지막 4자리) |
| Server | **Create new**를 선택 |
| Server name | **+++sqlserverXXXX+++** (XXXX = 실습 Instance ID의 마지막 4자리) |
| Location | **(Asia Pacific) Southeast Asia** |
| Authentication method | **Use SQL authentication** |
| Server admin login | **+++sqladmin+++** |
| Password | **+++password321!+++** |
| Confirm password | **+++password321!+++** |
| 작업 | **OK**를 클릭 |

![](./media/image8.png)

![](./media/image9.png)

5.  **Compute + storage** 섹션에서 **Configure database**를 클릭합니다.

![](./media/image10.png)

6.  **Service tier** 드롭다운에서 **Standard (Budget friendly)**를 선택하고, **DTUs**에 **100**을 입력한 뒤 **Apply**를 클릭합니다. 그런 다음 **Next : Networking**을 클릭합니다.

![](./media/image11.png)

![](./media/image12.png)

7.  **Networking** 탭에서 **Public endpoint**를 선택하고, **Allow Azure services and resources to access this server**와 **Add current client IP address**를 **Yes**로 설정한 다음 **Next : Security**를 클릭합니다.

![](./media/image13.png)

8.  **Security** 페이지에서 내용을 검토한 후, **Next : Additional settings**를 선택합니다.

![](./media/image14.png)

9.  **Additional settings** 탭의 **Use existing data**에서 **Sample**을 선택하고, 메시지가 표시되면 **AdventureWorksLT**를 선택한 다음 **OK**를 클릭하고 **Review + create**를 선택합니다.

![](./media/image15.png)

10. **Review + create** 페이지에서 내용을 검토한 후 **Create**를 선택합니다.

![](./media/image16.png)

![](./media/image17.png)

11. 배포가 완료되면 **Go to resource** 버튼을 클릭합니다.

![](./media/image18.png)

12. SQL 데이터베이스 페이지에서 **Query editor**를 선택합니다.

![](./media/image19.png)

13. **Query editor (preview)**에서 **Login**에 +++sqladmin+++, **Password**에 +++password321!+++를 입력한 다음, **OK**를 클릭하여 데이터베이스에 연결합니다.

![](./media/image20.png)

14. 모든 샘플 테이블이 성공적으로 배포되었는지 확인합니다.

![](./media/image21.png)

15. SQL 데이터베이스 **Overview** 페이지로 돌아갑니다. **Server name** (1)과 **SQL Database name** (2)을 복사하여 메모장에 붙여넣은 뒤, 다음 과제에서 사용할 수 있도록 메모장을 저장합니다.

![](./media/image22.png)

16. 메인 페이지로 돌아가려면 **Home**을 클릭합니다.

![](./media/image23.png)

17. **Resource groups**를 클릭합니다.

![](./media/image24.png)

18. **ResourceGroup1** 리소스 그룹을 클릭합니다.

![](./media/image25.png)

19. **sqlserverXXXX** SQL server를 선택합니다.

![](./media/image26.png)

20. **Identity**로 이동하여 **System assigned managed identity**의 **Status**를 **On**으로 변경한 다음, **Save**를 클릭합니다.

![](./media/image27.png)

![](./media/image28.png)

## 과제 2: Fabric workspace 만들기

이 과제에서는 mirrored database와 Fabric Data Agent를 호스팅할 Fabric workspace를 생성합니다.

1.  브라우저를 열고 주소창에 다음 URL을 입력하거나 붙여넣은 다음 **Enter** 키를 누르고 로그인합니다: +++https://app.fabric.microsoft.com/+++

| **사용자 이름** | **+++@lab.CloudPortalCredential(User1).Username+++** |
|----|----|
| **비밀번호** | **+++@lab.CloudPortalCredential(User1).Password+++** |

2.  Fabric 홈 페이지에서 **+ New workspace** 타일을 선택합니다.

![](./media/image29.png)

3.  오른쪽에 나타나는 **Create a workspace** 패널에서 다음 세부 정보를 입력하고 **Apply** 버튼을 클릭합니다.

| **설정** | **값** |
|----|----|
| Name | **+++FabricAgent-mirroringdatabase@lab.LabInstance.Id+++** |
| Advanced | **License mode**에서 **Fabric**을 선택 |
| Default storage format | **Small dataset storage format** |
| Template apps | **Develop template apps**를 선택 |

![](./media/image30.png)

**참고:** Lab Instance ID를 확인하려면 **Help**를 선택하고 Instance ID를 복사합니다.

![](./media/image31.png)

![](./media/image32.png)

4.  배포가 완료될 때까지 기다립니다. 완료하는 데 2~3분 정도 걸립니다.

![](./media/image33.png)

## 과제 3: Azure SQL Mirroring을 사용하여 데이터 미러링하기

이 과제에서는 Azure SQL Mirroring을 사용하여 Azure SQL Database를 Microsoft Fabric에 연결합니다. 테이블을 선택하고 mirrored database를 생성한 뒤, 데이터가 성공적으로 동기화되었는지 확인합니다.

1.  workspace에서 **+ New item** 버튼을 클릭합니다.

![](./media/image34.png)

2.  **Filter by keyword** 검색 상자에 +++Mirrored Azure SQL Database+++를 입력하고 **Mirrored Azure SQL Database** 항목을 선택합니다.

![](./media/image35.png)

3.  **Choose a database connection to get started** 창에서 **Azure SQL Database**를 선택합니다.

![](./media/image36.png)

4.  **Connection settings**에서 아래 세부 정보를 입력한 후 **Connect** 버튼을 클릭합니다.

| **필드** | **값** |
|----|----|
| Server | 과제 1의 15단계에서 저장한 **Server name** (예: sqlserverXXXX.database.windows.net) |
| Database | 과제 1의 15단계에서 저장한 **SQL Database name** |
| Authentication kind | **Basic** |
| Username | **+++sqladmin+++** |
| Password | **+++password321!+++** |

![](./media/image37.png)

5.  **Choose data** 창에서 **Select all**을 선택하고 **Connect** 버튼을 클릭합니다.

![](./media/image38.png)

6.  **Destination** 탭에서 **Create mirrored database**를 클릭합니다.

![](./media/image39.png)

7.  **Refresh**를 클릭하여 최신 상태를 확인합니다.

![](./media/image40.png)

![](./media/image41.png)

8.  왼쪽 탐색 메뉴에서 **FabricAgent-mirroringdatabase@lab.LabInstance.Id** workspace를 클릭합니다.

![](./media/image42.png)

## 과제 4: Data agent를 생성하고 Mirrored Database 연결하기

이 과제에서는 새로운 Fabric Data Agent를 생성하고, 미러링된 Azure SQL Database를 데이터 소스로 사용하도록 구성합니다. 이 에이전트는 미러링된 데이터를 활용하여 자연어 프롬프트에 응답합니다.

1.  workspace에서 **+ New item**을 선택합니다.

![](./media/image43.png)

2.  **Filter by item type** 검색 상자에 +++data agent+++를 입력한 후 **Data agent**를 선택합니다.

![](./media/image44.png)

3.  data agent 이름으로 +++FabricDataAgent@lab.LabInstance.Id+++를 입력하고 **Create**를 선택합니다.

![](./media/image45.png)

4.  새 데이터 소스를 구성하려면 **Add data source**를 선택합니다.

![](./media/image46.png)

5.  과제 3에서 만든 mirrored database(**sqldatabaseXXXX**)를 선택하고 **Add**를 선택합니다.

![](./media/image47.png)

![](./media/image48.png)

## 과제 5: 에이전트 테스트하기

다음과 같은 분석 질문으로 data agent를 테스트합니다:

- *어떤 제품 카테고리가 가장 높은 매출을 올리나요?*

- *정가는 높지만 판매량은 낮은 제품 목록을 작성하십시오.*

- *고객 수가 가장 많은 도시는 어디입니까?*

이를 통해 에이전트가 비즈니스 질문을 이해하고 응답하는 능력을 검증합니다.

1.  **Explorer**에서 **SalesLT** 스키마의 모든 테이블을 선택합니다.

2.  쿼리 패널에 다음 질문을 입력하고 전송 아이콘을 클릭하여 에이전트의 응답을 확인합니다.

    +++Which product categories generate the highest sales?+++

![](./media/image49.png)

![](./media/image50.png)

3.  다음 질문을 입력하고 응답을 확인합니다.

    +++List products with high list price but low sales volume.+++

![](./media/image51.png)

![](./media/image52.png)

4.  다음 질문을 입력하고 응답을 확인합니다.

    +++List the cities with the highest number of customers+++

![](./media/image53.png)

![](./media/image54.png)

5.  상단 메뉴에서 **Agent instructions**를 클릭하고 에이전트 지침을 검토합니다.

![](./media/image55.png)

6.  상단 메뉴에서 **Publish**를 클릭하고, **Publish data agent** 창에서 **Publish**를 선택합니다.

![](./media/image56.png)

![](./media/image57.png)

![](./media/image58.png)

## 과제 6: 리소스 삭제하기

1.  왼쪽 탐색 창에서 **FabricAgent-mirroringdatabase@lab.LabInstance.Id** workspace를 클릭합니다.

![](./media/image59.png)

2.  workspace 이름 아래에 있는 **...** 옵션을 선택하고 **Workspace settings**를 선택합니다.

![](./media/image60.png)

3.  **General** 탭에서 **Remove this workspace**를 선택합니다.

![](./media/image61.png)

4.  팝업으로 표시되는 경고 창에서 **Delete**를 클릭합니다.

![](./media/image62.png)

5.  workspace가 삭제되었다는 알림이 표시될 때까지 기다립니다.

![](./media/image63.png)

6.  브라우저에서 +++https://portal.azure.com+++로 이동하고, 필요하면 과제 1과 동일한 자격 증명으로 로그인합니다.

7.  Azure portal의 검색 창에 +++Resource groups+++를 입력하고, **Services** 아래의 **Resource groups**를 클릭합니다.

![](./media/image64.png)

8.  **ResourceGroup1** 리소스 그룹을 선택합니다.

9.  리소스 그룹 홈 페이지에서 **Fabric Capacity**를 제외한 모든 리소스를 선택한 다음, **Delete**를 클릭합니다.

![](./media/image65.png)

10. 오른쪽에 나타나는 **Delete Resources** 창의 **Enter "delete" to confirm deletion** 필드에 +++delete+++를 입력한 다음, **Delete** 버튼을 클릭합니다.

![](./media/image66.png)

![](./media/image67.png)

**요약**

이번 실습에서는 Azure SQL Database를 생성하고, Azure SQL Mirroring을 사용하여 해당 데이터를 Microsoft Fabric으로 미러링했습니다. 그런 다음 Fabric Data Agent를 구성하여 미러링된 데이터베이스에 연결하고, 자연어 쿼리를 통해 데이터를 분석했습니다.

에이전트는 판매 실적이 높은 제품군, 가격은 높지만 판매량은 저조한 제품, 고객 수가 가장 많은 도시를 파악하는 등의 분석 질문에 답변했습니다. 이는 Microsoft Fabric이 운영 데이터 소스와 지능형 에이전트를 통합하여 데이터 탐색을 간소화하고, 더 신속하게 비즈니스 인사이트를 도출할 수 있게 해준다는 점을 보여줍니다.

이 사용 사례는 Microsoft Fabric 생태계 내에서 **데이터 미러링과 AI 기반 data agent**를 결합하여 상호작용형의 지능적인 데이터 경험을 구현하는 역량을 보여줍니다.
