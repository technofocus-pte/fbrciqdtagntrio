# 사용 사례 01: 의미 체계에서 인사이트까지: Fabric 데이터 에이전트를 사용하여 Fabric IQ 온톨로지 활용하기

**시나리오**

**Lakeshore Retail**은 지역 물류 센터에서 공급받는 매장을 통해 일반 상품, 신선 식품, 냉동 제품(아이스크림 등)을 판매하는 가상의 다지역 소매업체입니다.

Lakeshore Retail의 데이터는 여러 시스템에 분산되어 있습니다:

- 위치(매장 및 물류 센터), 제품, 공급업체, 재고 현황, 배송, 냉장 장치에 대한 **레이크하우스 테이블**

- Eventhouse에 저장되는 각 매장 냉장 장치의 **스트리밍 텔레메트리**(온도, 습도, 문 열림 상태)

- 판매 거래 및 판매 측정값이 포함된 **Power BI 의미 체계 모델**

운영 팀은 다음과 같은 도메인 간 질문에 빠르게 답할 수 있어야 합니다:

*Which high-priority stores in the West have frozen products below safety stock, a recent refrigeration-temperature exception, and on-shelf availability below target? For each store, show the next inbound shipment and its expected arrival.*

(서부 지역의 우선순위가 높은 매장 중 냉동 제품이 안전 재고 미만이고, 최근 냉장 온도 예외가 발생했으며, 진열 가용률이 목표 미만인 매장은 어디입니까? 각 매장의 다음 입고 배송과 예상 도착 시간을 보여 주세요.)

현재 이 질문에 답하려면 재고, 텔레메트리, 위치, 배송 데이터를 수동으로 조인해야 합니다. 이 실습에서 여러분은 Lakeshore Retail의 데이터 엔지니어입니다. 이러한 모든 소스를 비즈니스 용어로 연결하는 **RetailSalesOntology** 온톨로지를 구축하고, 설명과 비즈니스 규칙으로 보강하고, 그래프로 탐색한 다음, **Ontology agent**를 사용하여 시나리오 질문에 자연어로 답합니다.

**소개**

현대적인 데이터 플랫폼에는 **비즈니스 중심 의미 체계 계층**, 즉 비즈니스를 고유한 용어로 설명하고 여러 데이터 소스에서 동일하게 작동하는 공유 모델이 필요합니다. **Microsoft Fabric IQ**의 **Ontology (preview)** 항목이 이 계층을 제공합니다. 온톨로지에서는 **엔터티 타입**(매장, 제품, 배송과 같은 비즈니스 개념), 해당 **속성**, 엔터티 간의 **관계**, 비즈니스 **규칙**을 정의한 다음, 이러한 정의를 OneLake의 실제 데이터(레이크하우스 테이블, Eventhouse(스트리밍) 테이블, Power BI 의미 체계 모델)에 바인딩합니다.

온톨로지가 준비되면 사용자와 AI 에이전트는 기본 테이블이나 조인을 알지 못해도 온톨로지를 **그래프**로 탐색하고 **자연어**로 질문할 수 있습니다.

이 실습은 Microsoft Learn 자습서 시리즈 [Ontology (preview) tutorial](https://learn.microsoft.com/fabric/iq/ontology/tutorial-0-introduction)을 기반으로 합니다.

**목표**

이 실습을 완료하면 다음을 수행할 수 있습니다:

- 레이크하우스, Eventhouse, 의미 체계 모델을 데이터 소스로 사용하는 Microsoft Fabric 작업 공간 준비하기

- **Ontology (preview)** 항목을 생성하고 상속된 엔터티 타입을 포함한 엔터티 타입 정의하기

- 엔터티 타입을 OneLake 테이블의 정적 데이터, Eventhouse의 시계열 데이터, Power BI 의미 체계 모델에 바인딩하기

- 실제 비즈니스 프로세스를 나타내는 관계 타입 생성하기 (예: Store *operates* Refrigeration Unit)

- 엔터티, 속성, 관계 메타데이터와 비즈니스 규칙으로 온톨로지 보강하기

- 캔버스 뷰, 엔터티 인스턴스, 구체화된 그래프, 경로 쿼리로 온톨로지를 탐색하고 검증하기

- **Ontology agent**를 사용하여 자연어로 질문하기

**아키텍처**

| **구성 요소** | **실습에서의 역할** |
|----|----|
| Fabric 작업 공간 Fabric IQ Ontology\<number\> | 실습의 모든 항목 포함 |
| 레이크하우스 IQ_Lakehouse | 정적 데이터: 위치, 제품, 공급업체, 재고 현황, 배송, 냉장 장치 |
| Eventhouse TelemetryDataEH (KQL 데이터베이스) | 스트리밍 데이터: RefrigerationTelemetry 테이블 |
| 의미 체계 모델 SalesReport | 판매 거래 및 측정값 (Gross Margin %, Net Sales) |
| 온톨로지 RetailSalesOntology | 데이터에 바인딩된 비즈니스 엔터티 타입, 관계, 메타데이터, 규칙 |
| 그래프 모델 | 탐색 및 경로 쿼리를 위한 온톨로지의 구체화된 그래프 |
| Ontology agent (preview) | 온톨로지에 대한 자연어 질문에 답하는 Copilot |

**대상 온톨로지**

| **엔터티 타입** | **상속 대상** | **데이터 소스** |
|----|----|----|
| Location | | IQ_Lakehouse \> dimlocations |
| Store | Location | IQ_Lakehouse \> dimlocations |
| Distribution Center | Location | IQ_Lakehouse \> dimlocations |
| Product | | IQ_Lakehouse \> dimproducts |
| Frozen Product | Product | IQ_Lakehouse \> dimproducts |
| Perishable Product | Product | IQ_Lakehouse \> dimproducts |
| Inventory | | IQ_Lakehouse \> fact_inventory_positions |
| Supplier | | IQ_Lakehouse \> dimsuppliers |
| Shipment | | IQ_Lakehouse \> factshipments |
| Refrigeration Unit | | IQ_Lakehouse \> dim_refrigeration_units |
| Refrigeration Telemetry | | TelemetryDataEH \> RefrigerationTelemetry |
| Sale | | SalesReport 의미 체계 모델 \> Sales (Ontology agent로 생성) |

**사전 요구 사항**

- **Microsoft Fabric 지원 용량**의 작업 공간

- Fabric 관리자가 **Ontology (preview)** 항목, Fabric 항목, Copilot / Azure OpenAI 기능(Ontology agent에 필요)에 대한 테넌트 설정을 활성화했어야 합니다.

- 실습 VM의 **C:\LabFiles\Lab1**에 있는 실습 파일: 정적 데이터가 포함된 CSV 파일 6개, **refrigeration_telemetry.csv**, **SalesReport.pbix**

**참고:** Microsoft Fabric의 온톨로지, Ontology agent, 그래프는 **미리 보기** 상태입니다. 화면과 레이블이 약간 다를 수 있습니다. AI가 생성한 답변은 부정확할 수 있으므로 중요한 답변은 항상 원본 데이터와 대조하여 확인하세요.

## 연습 1: 환경 설정하기

이 연습에서는 온톨로지에 사용할 세 가지 데이터 소스, 즉 정적 데이터가 포함된 레이크하우스, 스트리밍 데이터가 포함된 Eventhouse, 판매 데이터가 포함된 Power BI 의미 체계 모델을 준비합니다.

### 작업 1: Fabric 작업 공간 생성하기

1.  브라우저를 열고 +++https://app.fabric.microsoft.com/+++로 이동한 다음 자격 증명으로 로그인하세요.

| **사용자 이름** | **+++@lab.CloudPortalCredential(User1).Username+++** |
|----|----|
| **비밀번호** | **+++@lab.CloudPortalCredential(User1).Password+++** |

2.  Fabric 홈 페이지에서 **+ New workspace**를 선택하세요.

![](./media/image1.png)

3.  **Create a workspace** 창에서 다음 세부 정보를 입력하세요:

    - **Name**: +++Fabric IQ Ontology@lab.LabInstance.Id+++ (고유한 이름이어야 함)

    - **Advanced**를 확장하세요.

![](./media/image2.png)

4.  **License mode**에서 **Fabric**을 선택한 다음 **Apply**를 선택하세요.

![](./media/image3.png)

5.  새 작업 공간이 열립니다.

![](./media/image4.png)

### 작업 2: 레이크하우스 생성하기

1.  작업 공간에서 **+ New item**을 선택하세요.

![](./media/image5.png)

2.  필터에서 **Lakehouse**를 찾아 선택하세요.

![](./media/image6.png)

3.  **New Lakehouse** 대화 상자의 **Name** 상자에 +++IQ_Lakehouse+++를 입력하고, **Lakehouse schemas** 확인란을 선택 해제한 다음 **Create**를 선택하세요.

![](./media/image7.png)

**참고:** Microsoft Learn 자습서에서는 **Lakehouse schemas**를 선택한 상태로 두며, 이 경우 테이블은 **dbo** 스키마에 생성됩니다. 이 실습의 나머지 부분은 어느 쪽이든 동일하게 진행됩니다.

4.  레이크하우스가 열립니다. **Successfully created SQL analytics endpoint** 알림이 표시될 때까지 기다리세요.

![](./media/image8.png)

![](./media/image9.png)

### 작업 3: 정적 데이터를 레이크하우스 테이블로 로드하기

1.  **IQ_Lakehouse** 페이지의 **Get data in your lakehouse**에서 **Upload files**를 선택하세요.

![](./media/image10.png)

2.  **Upload files** 창에서 폴더 아이콘을 선택하세요.

![](./media/image11.png)

3.  **C:\LabFiles\Lab1**로 이동하여 CSV 파일 6개 **dim_refrigeration_units**, **dimlocations**, **dimproducts**, **dimsuppliers**, **fact_inventory_positions**, **factshipments**를 선택한 다음 **Open**을 선택하세요.

![](./media/image12.png)

**중요:** **refrigeration_telemetry.csv** 또는 **SalesReport.pbix**는 업로드하지 마세요. 이 두 파일은 이 연습의 뒷부분에서 Eventhouse와 작업 공간에 로드합니다.

4.  **Upload**를 선택하세요.

![](./media/image13.png)

5.  6개 파일이 모두 업로드되었는지 확인한 다음 **X**를 선택하여 **Upload files** 창을 닫으세요.

![](./media/image14.png)

6.  **Explorer**에서 **Files**를 선택하고 6개 파일이 표시되는지 확인하세요. 필요하면 **Refresh**를 선택하세요.

![](./media/image15.png)

7.  **dim_refrigeration_units.csv** 위에 마우스를 올리고 **...**를 선택한 다음 **Load to Tables \> New table**을 선택하세요.

![](./media/image16.png)

![](./media/image17.png)

8.  **Load file to new table** 대화 상자에서 기본 테이블 이름과 설정을 유지하고 **Load**를 선택하세요.

![](./media/image18.png)

9.  **Tables** 아래에 **dim_refrigeration_units** 테이블이 표시되는지 확인하세요.

![](./media/image19.png)

**참고:** 테이블을 확인하고 데이터를 미리 보려면 **Refresh**를 여러 번 선택해야 할 수도 있습니다.

10. 나머지 5개 파일 각각에 대해 7~9단계를 반복하세요. 예를 들어 **dimlocations.csv**의 경우 **... \> Load to Tables \> New table**을 선택한 다음 **Load**를 선택하세요.

![](./media/image20.png)

![](./media/image21.png)

11. 완료되면 레이크하우스에 **dim_refrigeration_units**, **dimlocations**, **dimproducts**, **dimsuppliers**, **fact_inventory_positions**, **factshipments** 테이블 6개가 생성됩니다. 기본 테이블 이름은 파일 이름을 소문자로 바꾼 것입니다.

![](./media/image22.png)

**참고**: 실습 중 Fabric 용량 관련 문제가 발생하면 Azure Portal에서 Fabric 용량으로 이동하여 용량을 약 5분간 일시 중지한 후 다시 시작하십시오. 용량이 정상적으로 실행되면 나머지 실습 단계를 계속 진행하십시오.

12. 왼쪽 탐색 모음에서 작업 공간 **Fabric IQ Ontology\<number\>**를 선택하세요.

![](./media/image23.png)

### 작업 4: 스트리밍 데이터를 Eventhouse로 로드하기

1.  작업 공간에서 **+ New item**을 선택한 다음 **Eventhouse**를 선택하세요.

![](./media/image24.png)

2.  **Eventhouse name**에 +++TelemetryDataEH+++를 입력한 다음 **Create**를 선택하세요.

![](./media/image25.png)

3.  준비가 완료되면 Eventhouse가 열립니다. 같은 이름의 KQL 데이터베이스가 자동으로 생성됩니다.

![](./media/image26.png)

4.  **KQL databases** 아래에서 **TelemetryDataEH**를 선택하여 데이터베이스를 여세요.

![](./media/image27.png)

![](./media/image28.png)

5.  리본에서 **Get data \> Local file**을 선택하세요.

![](./media/image29.png)

6.  **Select or create a destination table**에서 **+ New table**을 선택하세요.

![](./media/image30.png)

7.  테이블 이름으로 +++RefrigerationTelemetry+++를 입력하고 확인 표시를 선택한 다음 **Browse for files**를 선택하세요.

![](./media/image31.png)

![](./media/image32.png)

8.  **C:\LabFiles\Lab1**로 이동하여 **refrigeration_telemetry.csv**를 선택한 다음 **Open**을 선택하세요.

![](./media/image33.png)

9.  **Next**를 선택하세요.

![](./media/image34.png)

10. **Inspect the data** 페이지에서 열(**TelemetryId**, **UnitId**, **Timestamp**, **TemperatureC** 등)을 검토한 다음 **Finish**를 선택하세요.

![](./media/image35.png)

11. 수집 결과에 **1 blobs: 1 succeeded, 0 failed**가 표시될 때까지 기다린 다음 **Close**를 선택하세요.

![](./media/image36.png)

12. KQL 데이터베이스에 **RefrigerationTelemetry** 테이블이 표시되는지 확인하세요.

![](./media/image37.png)

13. 왼쪽 탐색 모음에서 작업 공간을 선택하세요.

![](./media/image38.png)

### 작업 5: 의미 체계 모델 업로드하기

1.  작업 공간에서 **Import \> Report, Paginated Report or Workbook \> From this computer**를 선택하세요.

![](./media/image39.png)

2.  **C:\LabFiles\Lab1**로 이동하여 **SalesReport.pbix**를 선택한 다음 **Open**을 선택하세요.

![](./media/image40.png)

3.  작업 공간에 **SalesReport** 보고서와 같은 이름의 **SalesReport** 의미 체계 모델이 표시되는지 확인하세요.

![](./media/image41.png)

## 연습 2: 온톨로지 구축하기

이 연습에서는 온톨로지 항목과 **엔터티 타입**을 생성합니다. 엔터티 타입은 비즈니스 객체의 유형을 나타냅니다. 데이터 소스의 열에 바인딩하여 엔터티 타입의 **속성**을 생성합니다. 또한 **상속**을 사용합니다. 자식 엔터티 타입(예: *Store*)은 부모(*Location*)의 모든 속성을 상속합니다.

### 작업 1: 온톨로지 항목 생성하기

1.  작업 공간에서 **+ New item**을 선택하세요. **Ontology (preview)**를 검색하여 선택하세요.

![](./media/image42.png)

2.  **New Ontology** 대화 상자에서 **Name**에 +++RetailSalesOntology+++를 입력하고, **Location**은 작업 공간으로 유지한 다음 **Create**를 선택하세요.

![](./media/image43.png)

**팁:** 온톨로지 이름은 문자로 시작해야 하며 문자, 숫자, 밑줄만 포함할 수 있습니다. 공백이나 대시는 사용하지 마세요.

**중요:** Fabric에서 온톨로지 항목을 생성할 수 없으면 관리자에게 필요한 테넌트 설정이 활성화되어 있는지 확인을 요청하세요.

3.  온톨로지가 구성 캔버스에 열립니다.

![](./media/image44.png)

### 작업 2: Location 엔터티 타입 생성하기

1.  리본에서 **+ Add entity type**을 선택하세요.

![](./media/image45.png)

2.  **Entity type name**에 +++Location+++을 입력한 다음 **Add Entity Type**을 선택하세요.

![](./media/image46.png)

3.  **Location** 엔터티 타입이 캔버스와 **Explorer**에 추가됩니다.

![](./media/image47.png)

4.  캔버스에서 **Location** 옆의 **...**를 선택한 다음 **Bind data**를 선택하세요.

![](./media/image48.png)

5.  **Add a data source**에서 **Add**를 선택하세요.

![](./media/image49.png)

6.  **OneLake catalog**에서 **IQ_Lakehouse**를 확장하고 **dimlocations** 테이블을 선택한 다음 **Select table**을 선택하세요.

![](./media/image50.png)

7.  소스를 검토하세요. **dimlocations**가 **Primary source**로 설정되어 있습니다.

![](./media/image51.png)

8.  **Entity type properties**를 선택하세요.

![](./media/image52.png)

9.  **dimlocations** 테이블의 열이 제안된 속성(**LocationId**, **LocationType**, **Name**, **Region**, **City** 등)으로 채워집니다. 변경하지 말고 **Create**를 선택하세요.

![](./media/image53.png)

10. **Entity type updated successfully**가 표시되면 **Cancel**을 선택하여 바인딩 페이지를 닫으세요.

![](./media/image54.png)

11. 엔터티 타입의 **Configure** 페이지가 열리고 속성과 데이터 소스가 표시됩니다.

![](./media/image55.png)

### 작업 3: Store 엔터티 타입 생성하기 (Location에서 상속)

1.  **Home**을 선택하여 구성 캔버스로 돌아가세요.

![](./media/image56.png)

2.  리본에서 **+ Add entity type**을 선택하세요.

![](./media/image57.png)

3.  **Entity type name**에 +++Store+++를 입력하세요. **Additional configuration**을 확장하고 **Choose entity to inherit from (optional)**을 **Location**으로 설정한 다음 **Add Entity Type**을 선택하세요.

![](./media/image58.png)

4.  **Store** 엔터티 타입이 캔버스에 표시됩니다.

![](./media/image59.png)

5.  **Explorer**에서 **Store**를 선택한 상태로 **View Entity Type details**를 선택하세요.

![](./media/image60.png)

6.  **Configure** 탭에서 **Store**에 이미 속성이 있는 것을 확인하세요. 이 속성은 **Location**에서 상속되었지만 **Data source**는 **Unbound** 상태입니다.

![](./media/image61.png)

7.  **Manage property bindings \> Add binding and properties**를 선택하세요.

![](./media/image62.png)

8.  **Add**를 선택하세요.

![](./media/image63.png)

9.  **IQ_Lakehouse**를 확장하고 **dimlocations**를 선택한 다음 **Select table**을 선택하세요.

![](./media/image64.png)

10. **Entity type properties**를 선택하고 각 속성에 소스 열이 있는지 확인한 다음 **Create**를 선택하세요.

![](./media/image65.png)

11. **Entity type updated successfully**가 표시되면 **Cancel**을 선택하세요.

![](./media/image66.png)

**참고:** **Store**와 **Distribution Center**는 모두 **dimlocations** 테이블을 사용합니다. **LocationType** 열(STORE 또는 DC)로 둘을 구분합니다. 이 내용은 연습 4에서 엔터티 설명으로 문서화합니다.

### 작업 4: 나머지 엔터티 타입 생성하기

같은 단계로 다음 엔터티 타입을 생성하세요. 각 엔터티 타입에 대해 표에 나열된 테이블을 바인딩하고 모든 소스 열을 속성으로 유지하세요.

| **엔터티 타입 이름** | **상속 대상** | **데이터 소스 테이블** |
|----|----|----|
| Distribution Center | Location | IQ_Lakehouse \> dimlocations |
| Product | | IQ_Lakehouse \> dimproducts |
| Frozen Product | Product | IQ_Lakehouse \> dimproducts |
| Perishable Product | Product | IQ_Lakehouse \> dimproducts |
| Inventory | | IQ_Lakehouse \> fact_inventory_positions |
| Supplier | | IQ_Lakehouse \> dimsuppliers |
| Shipment | | IQ_Lakehouse \> factshipments |
| Refrigeration Unit | | IQ_Lakehouse \> dim_refrigeration_units |
| Refrigeration Telemetry | | TelemetryDataEH \> RefrigerationTelemetry |

다음 단계에서는 처음 몇 개를 예로 보여 줍니다.

1.  **Distribution Center:** **Home \> + Add entity type**을 선택하세요. +++Distribution Center+++를 입력하고 **Additional configuration**을 확장한 다음 부모로 **Location**을 선택하고 **Add Entity Type**을 선택하세요.

![](./media/image67.png)

![](./media/image68.png)

2.  **Distribution Center** 옆의 **... \> Bind data**를 선택하고 **Add**를 선택한 다음 **IQ_Lakehouse \> dimlocations**를 선택하고 **Select table**을 선택하세요.

![](./media/image69.png)

![](./media/image70.png)

![](./media/image71.png)

3.  **Entity type properties \> Create**를 선택하고 **Entity type updated successfully**가 표시되면 **Cancel**을 선택하세요. **Configure** 페이지에 바인딩된 속성이 표시됩니다.

![](./media/image72.png)

![](./media/image73.png)

![](./media/image74.png)

4.  **Product:** **Home \> + Add entity type**을 선택하고 +++Product+++를 입력한 다음 **Add Entity Type**을 선택하세요.

![](./media/image75.png)

**참고:** 여기에 표시된 대로 **Product**(또는 복수형 *Products*)를 사용하세요. 그래프 쿼리 언어인 GQL의 예약어(예: **ORDER**)와 같은 엔터티 타입 이름은 피하세요.

5.  **... \> Bind data \> Add**를 선택하고 **IQ_Lakehouse \> dimproducts**를 선택한 다음 **Select table**, **Entity type properties \> Create**, **Cancel**을 차례로 선택하세요.

![](./media/image76.png)

![](./media/image77.png)

![](./media/image78.png)

6.  **Frozen Product:** **Home \> + Add entity type**을 선택하고 +++Frozen Product+++를 입력한 다음 부모로 **Product**를 설정하고 **Add Entity Type**을 선택하세요. 같은 방법으로 **IQ_Lakehouse \> dimproducts**에 바인딩하세요.

![](./media/image79.png)

![](./media/image80.png)

7.  같은 방법으로 위 표의 테이블을 사용하여 **Perishable Product**(부모: **Product**), **Inventory**, **Supplier**, **Shipment**, **Refrigeration Unit**을 생성하세요.

8.  **Refrigeration Telemetry:** **Home \> + Add entity type**을 선택하고 +++Refrigeration Telemetry+++를 입력한 다음 **Add Entity Type**을 선택하세요.

![](./media/image81.png)

9.  **... \> Bind data \> Add**를 선택하세요. **OneLake catalog**에서 **TelemetryDataEH \> TelemetryDataEH**(KQL 데이터베이스)를 확장하고 **RefrigerationTelemetry** 테이블을 선택한 다음 **Select table**을 선택하세요.

![](./media/image82.png)

**참고:** 이 엔터티 타입은 레이크하우스 테이블이 아니라 **Eventhouse** 테이블을 데이터 소스로 사용합니다.

10. **Entity type properties**를 선택하고 속성(**TelemetryId**, **UnitId**, **Timestamp**, **TemperatureC**, **HumidityPct**, **DoorOpen** 등)을 검토한 다음 **Create**와 **Cancel**을 선택하세요.

![](./media/image83.png)

11. **확인:** **Home**을 선택하세요. **Explorer**에 엔터티 타입 11개(**Location**, **Store**, **Distribution Center**, **Product**, **Frozen Product**, **Perishable Product**, **Inventory**, **Supplier**, **Shipment**, **Refrigeration Unit**, **Refrigeration Telemetry**)가 표시됩니다.

![](./media/image84.png)

### 작업 5: Ontology agent로 Sale 엔터티 타입 추가하기

**Ontology agent**로 온톨로지를 *구축*할 수도 있습니다. 이 작업에서는 에이전트를 사용하여 **SalesReport** 의미 체계 모델에 바인딩된 **Sale** 엔터티 타입을 생성합니다.

1.  리본에서 **Ontology agent**를 선택하세요. 에이전트가 오른쪽 창에서 **Plan** 모드로 열립니다.

![](./media/image85.png)

**참고:** **Plan** 모드에서 에이전트는 온톨로지를 쿼리 및 탐색하고 변경 사항을 제안할 수 있지만 실제로 변경하지는 않습니다. **Act** 모드에서는 변경 사항을 적용합니다.

2.  **Say something** 상자에 다음 프롬프트를 입력하고 **Send**를 선택하세요:

    +++Create a new entity type called Sale, bound to data in the Sales table from the SalesReport semantic model inside this workspace.+++

![](./media/image86.png)

3.  에이전트가 새 엔터티 타입에 대한 계획을 생성합니다. **Enrichment added** 요약을 검토하세요.

![](./media/image87.png)

4.  토글을 **Act**로 전환하고 +++Apply the entity type plan+++을 입력한 다음 **Send**를 선택하세요.

![](./media/image88.png)

5.  에이전트가 온톨로지 변경 사항을 생성하는 동안 기다리세요.

![](./media/image89.png)

6.  에이전트가 캔버스에 **Sale** 엔터티 타입을 추가하고 결과를 보고합니다.

![](./media/image90.png)

**참고:** 다른 초안 정의와의 충돌로 인해 에이전트가 적용이 *부분적*이라고 보고할 수 있습니다. **Sale exists now**가 표시되면 계속 진행할 수 있습니다.

7.  **Explorer**에서 **Sale**을 선택한 다음 **View Entity Type details**를 선택하세요.

![](./media/image91.png)

8.  속성(**Channel**, **CostAmount**, **NetSalesAmount**, **ProductId**, **SaleId** 등)이 의미 체계 모델의 **Sales** 테이블에 바인딩되어 있는지 확인하세요.

![](./media/image92.png)

9.  **Metrics**까지 아래로 스크롤하여 의미 체계 모델의 DAX 측정값에서 **Gross Margin %**와 **Net Sales** 두 메트릭이 추가되었는지 확인하세요.

![](./media/image93.png)

## 연습 3: 관계 타입 생성하기

**관계 타입**은 원본 엔터티 타입을 대상 엔터티 타입에 연결합니다. 이 실습에서는 속성으로 연결합니다. **원본 속성**과 **대상 속성**은 같은 값을 가져야 합니다.

### 작업 1: Store operates Refrigeration Unit 관계 생성하기

1.  캔버스에서 엔터티 타입 옆의 **...**를 선택하고 **Add relationship type**을 선택하세요(또는 리본에서 **Add relationship**을 선택).

![](./media/image94.png)

2.  다음 세부 정보를 입력한 다음 **Create**를 선택하세요:

    - **Relationship type name**: +++operates+++

    - **Origin entity type**: **Store**

    - **Target entity type**: **Refrigeration Unit**

![](./media/image95.png)

3.  캔버스의 **Store**와 **Refrigeration Unit** 사이에 관계가 표시됩니다.

![](./media/image96.png)

4.  **operates** 관계를 선택하여 구성을 여세요.

![](./media/image97.png)

5.  페이지에는 **Origin entity type**(Store), **Relationship**(operates), **Target entity type**(Refrigeration Unit)의 세 섹션이 있습니다. **Use mapping table?**은 **Off**로 두세요.

![](./media/image98.png)

6.  원본 엔터티 타입의 **Property**에서 **LocationId**를 선택하세요. 대상 엔터티 타입의 **Property**에서 **StoreId**를 선택하세요. **Save**를 선택하세요.

![](./media/image99.png)

**참고:** 이 설정은 Store의 **LocationId**가 Refrigeration Unit의 **StoreId**와 일치한다는 의미입니다.

7.  **Successfully updated the relationship type**이 표시되면 **Cancel**을 선택하세요.

![](./media/image100.png)

8.  **Store**의 **Configure** 페이지에서 **Relationships** 섹션에 새 관계가 표시됩니다.

![](./media/image101.png)

### 작업 2: 나머지 관계 타입 생성하기

1.  **Home**을 선택한 다음 **Add relationship**을 선택하세요.

![](./media/image102.png)

2.  원본 **Shipment**, 대상 **Store**로 **deliversTo**를 생성하고 **Create**를 선택하세요.

![](./media/image103.png)

![](./media/image104.png)

3.  **deliversTo** 관계를 선택하고 원본 **Property**를 **ToStoreId**로, 대상 **Property**를 **LocationId**로 설정한 다음 **Save**와 **Cancel**을 선택하세요.

![](./media/image105.png)

4.  같은 방법으로 나머지 관계 타입을 생성하세요. 표에는 이미 생성한 **operates**와 **deliversTo**를 포함한 모든 관계 타입이 나와 있습니다:

| **관계 타입 이름** | **원본 엔터티 타입 (속성)** | **대상 엔터티 타입 (속성)** |
|----|----|----|
| operates | Store (LocationId) | Refrigeration Unit (StoreId) |
| deliversTo | Shipment (ToStoreId) | Store (LocationId) |
| occursAt | Sale (StoreId) | Store (LocationId) |
| stockedAt | Inventory (StoreId) | Store (LocationId) |
| originatesAt | Shipment (FromLocationId) | Distribution Center (LocationId) |
| forProduct | Sale (ProductId) | Product (ProductId) |
| stockedAt | Product (ProductId) | Inventory (ProductId) |
| suppliedBy | Product (SupplierId) | Supplier (SupplierId) |
| contains | Shipment (ProductId) | Product (ProductId) |
| hasTelemetryReading | Refrigeration Unit (UnitId) | Refrigeration Telemetry (UnitId) |

5.  **확인:** **Home**을 선택하세요. 캔버스에 엔터티 타입을 연결하는 관계가 표시됩니다.

![](./media/image106.png)

**중요:** 올바른 속성을 선택하세요. 원본 속성과 대상 속성의 값이 다르면 관계는 생성되지만 일치하는 항목을 찾지 못합니다.

## 연습 4: 온톨로지 보강하기

메타데이터와 규칙은 AI 에이전트가 데이터를 올바르게 해석하는 데 필요한 비즈니스 맥락을 제공합니다. 이 연습에서는 엔터티 타입, 속성, 관계 타입에 설명을 추가하고 비즈니스 규칙을 정의합니다.

### 작업 1: 엔터티 타입 메타데이터 추가하기

1.  **Explorer**에서 **Store**를 선택한 다음 **View Entity Type details**를 선택하세요.

![](./media/image107.png)

2.  **Entity metadata**까지 스크롤하세요. **Description**에 다음 텍스트를 입력한 다음 **Update**를 선택하세요:

    +++Filtered locations binding where LocationType = STORE; priorityDefinition=Tier 1 stores require same-day response.+++

![](./media/image108.png)

3.  설명을 확인한 다음 **Home**을 선택하세요.

![](./media/image109.png)

4.  같은 방법으로 다음 엔터티 타입에 설명을 추가하세요(엔터티 타입 선택, **View Entity Type details** 선택, **Description** 입력, **Update** 선택):

| **엔터티 타입** | **설명** |
|----|----|
| Distribution Center | +++Filtered locations binding where LocationType = DC.+++ |
| Frozen Product | +++Filtered items binding where StorageClass = FROZEN.+++ |
| Perishable Product | +++Filtered items binding where StorageClass = PERISHABLE.+++ |

![](./media/image110.png)

![](./media/image111.png)

![](./media/image112.png)

![](./media/image113.png)

![](./media/image114.png)

5.  **Inventory**를 여세요. **Additional metadata**에서 **+**를 선택하여 다음 키-값 쌍을 추가한 다음 **Update**를 선택하세요:

    - **On-Shelf Availability %**: +++On-shelf availability percentage is calculated by dividing the total ShelfAvailableUnits by the total ShelfCapacityUnits. If total ShelfCapacityUnits is zero, the result is blank to avoid division by zero.+++

    - **Low On-Shelf Availability**: +++On-Shelf Availability % less than 95+++

![](./media/image115.png)

6.  **Sale**을 여세요. 의미 체계 모델에서 추가된 **Description**과 **Synonyms**를 검토하세요. 변경할 필요는 없습니다.

![](./media/image116.png)

### 작업 2: 속성 메타데이터 추가하기

1.  **Explorer**에서 **Product** 옆의 **...**를 선택한 다음 **Bind data**를 선택하세요.

![](./media/image117.png)

2.  **Entity type properties**를 선택하세요.

![](./media/image118.png)

3.  **ProductId** 행에서 **Metadata**를 선택하세요.

![](./media/image119.png)

4.  **Description**에 +++Enterprise product identifier; not a supplier SKU or UPC.+++를 입력하고 **Update**를 선택하세요.

![](./media/image120.png)

5.  **Save**를 선택하고 **Entity type updated successfully**가 표시되면 **Cancel**을 선택하세요.

![](./media/image121.png)

![](./media/image122.png)

6.  **Refrigeration Telemetry**에 대해 반복하세요: **TemperatureC** 행에서 **Metadata**를 선택하고 +++Temperature in Celsius+++를 입력한 다음 **Update**와 **Save**를 선택하세요.

![](./media/image123.png)

![](./media/image124.png)

![](./media/image125.png)

7.  **Inventory**에 대해 반복하세요: **InventoryStatus** 행에서 **Metadata**를 선택하고 +++0 is AT_RISK and 1 is HEALTHY.+++를 입력한 다음 **Update**와 **Save**를 선택하세요.

![](./media/image126.png)

![](./media/image127.png)

### 작업 3: 관계 메타데이터 추가하기

1.  **Home**을 선택하고 **Store**를 선택한 다음 캔버스에서 **operates** 관계를 선택하세요.

![](./media/image128.png)

![](./media/image129.png)

2.  **Metadata** 섹션에서 **Edit**를 선택하세요.

![](./media/image130.png)

3.  **Description**에 +++Identifies the refrigeration equipment operating in a store.+++를 입력하고 **Update**를 선택하세요.

![](./media/image131.png)

4.  **Save**를 선택하고 **Successfully updated the relationship type**이 표시되면 **Cancel**을 선택하세요.

![](./media/image132.png)

![](./media/image133.png)

5.  같은 방법으로 다른 관계 타입에도 설명을 추가하세요:

| **관계** | **설명** |
|----|----|
| deliversTo (Shipment \> Store) | +++Identifies the store receiving the shipment.+++ |
| occursAt (Sale \> Store) | +++Identifies where the sale occurred.+++ |
| stockedAt (Inventory \> Store) | +++Represents an active store assortment and its current inventory position, not merely a historical sale.+++ |
| originatesAt (Shipment \> Distribution Center) | +++Identifies the distribution center sending the shipment.+++ |
| forProduct (Sale \> Product) | +++Identifies the item sold.+++ |
| stockedAt (Product \> Inventory) | +++The item is part of the store's active assortment and has a current inventory position there.+++ |
| suppliedBy (Product \> Supplier) | +++Identifies the supplier responsible for the items.+++ |
| contains (Shipment \> Product) | +++Identifies the item being replenished.+++ |
| hasTelemetryReading (Refrigeration Unit \> Refrigeration Telemetry) | +++Identifies the sensor telemetry readings for refrigeration units.+++ |

### 작업 4: 비즈니스 규칙 추가하기

**규칙**은 비즈니스에서 반드시 참이어야 하거나, 참이면 안 되거나, 참이어야 하는 내용을 자연어로 설명합니다. 엔터티 타입과 일치하는 단어는 온톨로지에 연결됩니다.

1.  **Home**을 선택하고 **Explorer**에서 **Rules**를 선택하세요.

![](./media/image134.png)

2.  **+ Create rule**을 선택하세요.

![](./media/image135.png)

3.  **Rule name**에 +++Cold-chain exception+++을 입력하고 **Create**를 선택하세요.

![](./media/image136.png)

4.  **Rule definition**에 다음 텍스트를 입력하세요. **Linked ontology concepts**에서 **Add concept**를 선택하여 **Frozen Product**와 **Refrigeration Unit**을 추가하세요. **Save**를 선택하세요.

    +++A frozen product has a cold-chain exception when the temperature of the refrigeration unit storing it remains above the product's maximum storage temperature for more than 20 minutes.+++

![](./media/image137.png)

5.  **Rule saved**가 표시되면 **Cancel**을 선택하세요.

![](./media/image138.png)

6.  **New rule**을 선택하고 +++Inventory at risk+++를 입력한 다음 **Create**를 선택하세요.

![](./media/image139.png)

![](./media/image140.png)

7.  정의에 +++A store inventory position is at risk when projected on-hand inventory falls below safety stock before the next scheduled delivery+++를 입력하고, **Store**와 **Inventory**를 연결한 다음 **Save**와 **Cancel**을 선택하세요.

![](./media/image141.png)

8.  **New rule**을 선택하고 +++Late replenishment+++를 입력한 다음 **Create**를 선택하세요.

![](./media/image142.png)

9.  정의에 +++A replenishment shipment is late when its estimated arrival is more than four hours after its scheduled arrival.+++를 입력하고, **Shipment**를 연결한 다음 **Save**와 **Cancel**을 선택하세요.

![](./media/image143.png)

10. **Business rules** 목록에 규칙 3개가 표시되는지 확인하세요.

![](./media/image144.png)

## 연습 5: 온톨로지 탐색하기

### 작업 1: 캔버스 뷰와 엔터티 인스턴스 탐색하기

1.  **Home**을 선택하고 **Product**를 선택하세요. 캔버스 오른쪽 위에서 **Full ontology**를 선택하여 모든 엔터티 타입과 관계를 확인하세요.

![](./media/image145.png)

2.  **Lineage**를 선택하여 상속 구조를 확인하세요. **Product**에는 파생 타입 **Frozen Product**와 **Perishable Product**가 있습니다.

![](./media/image146.png)

3.  **Relationship**을 선택하면 선택한 엔터티 타입의 관계만 표시됩니다.

![](./media/image147.png)

4.  **Product**를 선택한 상태로 **View Entity Type details**를 선택하세요.

![](./media/image148.png)

5.  **Instances** 탭을 선택하세요. **dimproducts** 테이블에서 채워진 제품 레코드가 목록에 표시됩니다.

![](./media/image149.png)

### 작업 2: 엔터티 타입 키 정의하기

**엔터티 타입 키**는 엔터티 타입의 각 레코드를 고유하게 식별합니다. 그래프에 포함되는 모든 엔터티 타입에는 키가 필요합니다.

1.  **Location**을 선택하고 **View Entity Type details**를 선택하세요.

![](./media/image150.png)

2.  **Entity type key** 옆에서 **Define entity type key**를 선택하세요.

![](./media/image151.png)

3.  **LocationId**를 선택한 다음 **Save**를 선택하세요.

![](./media/image152.png)

4.  **Entity type key**에 **LocationId**가 표시되는지 확인하세요.

![](./media/image153.png)

5.  다른 엔터티 타입에 대해서도 반복하세요. 예를 들어 **Store**의 경우 **LocationId**를 선택하세요.

![](./media/image154.png)

| **엔터티 타입** | **키** |
|----|----|
| Location, Store, Distribution Center | LocationId |
| Product, Frozen Product, Perishable Product | ProductId |
| Inventory | **fact_inventory_positions**의 고유 ID 열 |
| Supplier | SupplierId |
| Shipment | **factshipments**의 고유 ID 열 |
| Refrigeration Unit | UnitId |

**참고:** **Sale**과 **Refrigeration Telemetry**는 건너뛰세요. 이 실습에서는 그래프에 포함되지 않습니다.

### 작업 3: 그래프 구체화하기(Optional)

[!중요] 전체 온톨로지를 구체화하는 데 2시간 이상 걸릴 수 있습니다. 이는 일시적인 문제입니다. 처리 시간이 길어지는 것을 방지하려면 필요한 구성 요소만 구체화하세요.

1.  **Home**을 선택한 다음 리본에서 **Manage graph**를 선택하세요.

![](./media/image155.png)

2.  **Configure Graph** 페이지가 열리고 **Use the entire Ontology**가 **Yes**로 설정되어 있습니다. **Sale**과 **Refrigeration_Telemetry**(미리 보기에서 회색으로 표시)를 제외한 모든 엔터티 타입이 선택되어 있습니다.

![](./media/image156.png)

![](./media/image157.png)

![](./media/image158.png)

**중요:** **전체** 온톨로지를 구체화하는 데는 **1시간 이상** 걸릴 수 있습니다. 시간 내에 실습을 마치려면 다음 단계에 설명된 대로 일부 엔터티 타입만으로 그래프를 구축하세요. 작은 그래프는 몇 분 안에 생성되며 이 실습의 탐색에 충분합니다.

3.  **Use the entire Ontology**를 **No**로 설정하세요. **Entities** 목록에서 **Store**, **Product**, **Shipment**, **Supplier**만 선택하고 나머지는 모두 선택 해제하세요. **Preview**에서 이 엔터티 타입만 파란색으로 강조 표시되는지 확인한 다음 **Continue**를 선택하세요.

![](./media/image159.png)

**참고:** 스크린샷은 선택하는 과정을 보여 줍니다. 다음 작업에서 **Shipment**, **Store**, **Product** 간의 관계를 사용하고 **Store**에서 **Supplier**로 경로 쿼리를 실행하므로 **Shipment**와 **Supplier**도 선택되어 있는지 확인하세요. 관계는 양쪽 엔터티 타입이 모두 선택된 경우에만 포함됩니다.

**팁:** 시간이 더 있으면 **Use the entire Ontology**를 **Yes**로 유지하여 **Sale**과 **Refrigeration_Telemetry**를 제외한 모든 엔터티 타입을 구체화할 수 있습니다. 1시간 이상 걸립니다.

4.  **Projection summary** 페이지에서 선택한 엔터티를 검토한 다음 **Materialize**를 선택하세요.

![](./media/image160.png)

5.  **Creating graph model** 알림이 표시됩니다. 그래프 모델이 생성될 때까지 기다리세요.

![](./media/image161.png)

![](./media/image162.png)

### 작업 4: 그래프 탐색하기

1.  리본에서 **Explore graph**를 선택하세요. 이 버튼은 그래프가 구체화된 후에 사용할 수 있습니다.

![](./media/image163.png)

2.  그래프 쿼리셋이 열립니다. 오른쪽에서 퍼즐 조각 아이콘을 선택하여 **Components** 창을 펼치세요.

![](./media/image164.png)

![](./media/image165.png)

![](./media/image166.png)

3.  **Components** 창에 그래프에서 사용 가능한 **Nodes**와 **Edges**가 표시됩니다.

![](./media/image167.png)

4.  **Nodes**에서 **Store**, **Product**, **Shipment**를 선택하세요. **Edges**에서 **Shipment_Store**와 **Shipment_Product**를 선택하세요. 노드와 간선이 캔버스에 추가됩니다.

![](./media/image168.png)

5.  리본에서 **Path query**를 선택하고 메시지가 표시되면 **Switch**를 확인하세요. 시애틀 매장과 공급업체 Cascade Fresh Products 사이의 2홉 이내 경로를 찾도록 다음 세부 정보를 입력한 다음 **Run**을 선택하세요:

    - **Start node**: **Store**

    - **End node**: **Supplier**

    - **Filter start node**: **City** = +++Seattle+++

    - **Filter end node**: **SupplierName** = +++Cascade Fresh Products+++

    - **Max hops**: **2**

![](./media/image169.png)

6.  쿼리 캔버스와 **Results** 창에서 결과를 검토하세요.

![](./media/image170.png)

## 연습 6: Ontology agent로 질문하기

**Ontology agent (preview)**는 자연어로 온톨로지를 구축하고 사용할 수 있도록 지원하는 AI 기반 Copilot입니다. 메타데이터와 규칙을 포함한 온톨로지를 바탕으로 추론하고, 바인딩된 데이터 소스를 쿼리하여 질문에 답합니다.

### 작업 1: 자연어로 질문하기

1.  **Home**을 선택하고 리본에서 **Ontology agent**를 선택하세요.

![](./media/image171.png)

2.  에이전트가 오른쪽 창에서 **Plan** 모드로 열립니다.

![](./media/image172.png)

3.  +++Which frozen products are stocked at Tier 1 stores in the West?+++를 입력하고 **Send**를 선택하세요.

![](./media/image173.png)

4.  에이전트가 잠시 추론한 후 온톨로지를 바탕으로 답변합니다. **Reasoning**을 확장하여 답에 도달한 과정을 확인하세요.

![](./media/image174.png)

![](./media/image175.png)

5.  +++Which of those inventory positions are below safety stock?+++를 입력하고 **Send**를 선택하세요.

![](./media/image176.png)

6.  재고 현황별 보유 수량, 안전 재고, 부족 수량이 나열된 답변을 검토하세요.

![](./media/image177.png)

7.  (선택 사항) 다음 질문으로 온톨로지를 더 탐색하거나 직접 질문해 보세요:

    - +++Which affected stores had a qualifying refrigeration-temperature exception?+++

    - +++What inbound shipments are expected for the affected products and stores?+++

    - +++What is the on-shelf availability of frozen products for each store?+++

    - +++Which of these stores have low on-shelf availability?+++

    - +++Which of these stores have declining gross margin?+++

### 작업 2: 시나리오 질문에 답하기

1.  Lakeshore Retail 시나리오의 주요 질문을 입력하고 **Send**를 선택하세요:

    +++Which high-priority stores in the West have frozen products below safety stock, a recent refrigeration-temperature exception, and on-shelf availability below target? For each store, show the next inbound shipment and its expected arrival.+++

![](./media/image178.png)

2.  답변을 검토하세요. 에이전트는 온톨로지를 통해 재고, 냉장 텔레메트리, 매장, 배송 데이터를 결합하고 비즈니스 규칙과 메타데이터(예: 진열 가용률 목표)를 적용하여 조건에 맞는 매장과 다음 입고 배송을 나열합니다.

![](./media/image179.png)

![](./media/image180.png)

**참고:** 에이전트가 *recent*(최근), *target*(목표)과 같은 용어를 해석하는 방식에 따라 실행할 때마다 답변이 다를 수 있습니다. 에이전트는 답변에 가정을 명시합니다. 중요한 답변은 원본 데이터와 대조하여 확인하세요.

## 연습 7: 리소스 정리하기

1.  왼쪽 탐색 모음에서 작업 공간 **Fabric IQ Ontology\<number\>**를 선택한 다음 **Workspace settings**를 선택하세요.

![](./media/image181.png)

2.  **General** 탭에서 **Delete workspace**까지 스크롤한 다음 **Remove this workspace**를 선택하세요. 삭제를 확인하세요.

![](./media/image182.png)

3.  **Workspace deleted** 알림을 확인하세요.

![](./media/image183.png)

**요약**

이 실습에서는 **Microsoft Fabric IQ Ontology (preview)**를 사용하여 Lakeshore Retail 운영에 대한 연결된 비즈니스 친화적 모델을 구축했습니다. 정적 데이터가 포함된 레이크하우스, 냉장 텔레메트리가 포함된 Eventhouse, 판매 데이터가 포함된 Power BI 의미 체계 모델을 준비했습니다. 엔터티 타입과 상속(Location \> Store 및 Distribution Center, Product \> Frozen 및 Perishable Product)을 사용하여 **RetailSalesOntology**를 생성하고 세 가지 데이터 소스에 모두 바인딩했으며, Ontology agent를 사용하여 의미 체계 모델에서 **Sale** 엔터티 타입을 추가했습니다. 관계 타입으로 엔터티 타입을 연결하고, 엔터티, 속성, 관계 메타데이터와 비즈니스 규칙으로 모델을 보강했으며, 캔버스 뷰, 인스턴스, 구체화된 그래프, 경로 쿼리로 탐색했습니다. 마지막으로 **Ontology agent**를 사용하여 조인을 하나도 작성하지 않고 도메인 간 시나리오 질문에 자연어로 답했습니다. 이를 통해 온톨로지가 운영 데이터와 분석 데이터를 연결하고, 사용자와 AI 에이전트가 비즈니스를 공통적으로 이해할 수 있게 해 준다는 것을 알 수 있습니다.
