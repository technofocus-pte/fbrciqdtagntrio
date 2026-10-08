# 사용 사례 02: Fabric data agent를 사용하여 AdventureWorks 데이터 세트로 판매 분석 구축하기

**소개**

소매 인사이트 팀인 Contoso Analytics는 분석가와 비즈니스 매니저의 데이터 접근성을 높이기 위해 보고 워크플로우를 **Microsoft Fabric**으로 전환하고 있습니다. 팀은 비전문가 사용자도 SQL을 작성하거나 대시보드를 탐색하지 않고도 인사이트를 얻을 수 있도록 자연어 기반 데이터 탐색을 가능하게 하고자 합니다.

이를 달성하기 위해 팀은 **Fabric Data Agent**를 기반으로 하는 **intelligent analytics assistant**를 구축하기로 결정합니다. 이 과정의 첫 번째 단계는 **Fabric Lakehouse**에 기반 데이터를 준비하는 것입니다. Fabric Data Agent 튜토리얼에 설명된 대로, 팀은 먼저 **Lakehouse를 생성하고 데이터를 채워 넣는** 작업부터 시작합니다. 이 lakehouse에는 판매 거래 내역, 제품 재고, 매장 프로필 등 선별된 소매 데이터 세트가 저장됩니다. 이 lakehouse는 후속 작업을 위한 관리되고 중앙 집중화된 데이터 소스 역할을 합니다.

lakehouse가 구축되면, 다음 단계는 대화형 시스템과 자동화 도구가 이를 활용할 수 있도록 하는 것입니다. 팀은 **Fabric Data Agent를 생성**하고 **lakehouse를 연결된 데이터 소스로 추가함으로써** 이를 달성하며, 이를 통해 데이터에 대한 안전하고 규정을 준수하는 접근을 가능하게 합니다. 이러한 구성을 통해 Data Agent는 lakehouse의 콘텐츠를 이해하고 쿼리할 수 있게 되며, 이는 조직 전반에 걸쳐 자연어 경험을 구축하기 위한 기반을 마련합니다.

Fabric Data Agent를 통해 Lakehouse가 연결됨에 따라, Contoso는 이제 이 에이전트를 분석 애플리케이션, Copilot 환경 및 내부 도구에 통합할 수 있게 되었습니다. 이를 통해 비즈니스 사용자는 "남부 지역의 오늘 매출을 보여줘" 또는 "모든 매장에서 재고가 가장 적은 상품을 찾아줘"와 같은 질문을 하고, 데이터 기반의 답변을 즉시 받을 수 있게 되었습니다.

**목표**

- **Microsoft Fabric workspace**를 만들고 저장소 및 권한을 설정합니다.

- Notebook을 활용해 **Fabric Lakehouse**를 구축하고 AdventureWorks 데이터 세트를 프로그래밍 방식으로 불러옵니다.

- Lakehouse 테이블에 연결된 **Fabric Data Agent**를 생성하고 구성합니다.

- **지침과 예시 쿼리**를 사용하여 에이전트의 응답을 개선합니다.

- 에이전트를 게시하고 Fabric notebook에서 **API 호출을 통해 프로그래밍 방식으로** 테스트합니다.

- 실습을 마친 후 workspace를 정리하고 삭제합니다.

## 과제 0: 호스트 환경 시간 동기화

1.  VM에서 **Search bar**를 찾아 클릭한 다음, +++Settings+++를 입력하고 **Best match** 아래에 있는 **Settings**를 클릭합니다.

![](./media/image1.png)

2.  Settings 창에서 **Time & language**를 찾아 클릭합니다.

![](./media/image2.png)

3.  **Time & language** 페이지에서 **Date & time**을 찾아 클릭합니다.

![](./media/image3.png)

4.  화면을 아래로 스크롤하여 **Additional settings** 섹션으로 이동한 다음, **Sync now** 버튼을 클릭합니다. 동기화에는 3~5분이 소요됩니다.

![](./media/image4.png)

5.  **Settings** 창을 닫습니다.

![](./media/image5.png)

## 과제 1: Fabric workspace 만들기

이번 과제에서는 Lakehouse, notebook, Data Agent를 호스팅할 Fabric workspace를 생성하여 기본 환경을 설정합니다. 이 workspace는 사용 사례 전반에 걸쳐 사용되는 모든 자산을 위한 중앙 컨테이너 역할을 합니다.

1.  웹 브라우저를 열고 주소 표시줄에 다음 URL을 입력하거나 붙여넣은 다음 **Enter** 키를 누릅니다: +++https://app.fabric.microsoft.com/+++

![](./media/image6.png)

2.  **Microsoft Fabric** 창에서 자격 증명을 입력하고 **Submit** 버튼을 클릭합니다.

| **사용자 이름** | **+++@lab.CloudPortalCredential(User1).Username+++** |
|----|----|
| **비밀번호** | **+++@lab.CloudPortalCredential(User1).Password+++** |

![](./media/image7.png)

3.  그런 다음, **Microsoft** 창에 비밀번호를 입력하고 **Sign in** 버튼을 클릭합니다.

![](./media/image8.png)

4.  **Stay signed in?**(로그인 상태 유지) 창에서 **Yes** 버튼을 클릭합니다.

5.  Power BI 홈 페이지로 이동합니다.

![](./media/image9.png)

6.  Fabric 홈 페이지에서 **+ New workspace** 타일을 선택합니다.

![](./media/image10.png)

7.  오른쪽에 나타나는 **Create a workspace** 창에서 다음 세부 정보를 입력한 다음, **Apply** 버튼을 클릭합니다.

| **설정** | **값** |
|----|----|
| Name | **+++Fabric Data agent-@lab.LabInstance.Id+++** (고유한 이름이어야 함) |
| Advanced | **License mode**에서 **Fabric**을 선택 |
| Default storage format | **Small dataset storage format** |
| Template apps | 템플릿 앱 개발 내용을 확인 |

![](./media/image11.png)

**참고:** Lab Instance ID를 확인하려면 **Help**를 선택하고 Instance ID를 복사합니다.

![](./media/image12.png)

![](./media/image13.png)

8.  배포가 완료될 때까지 기다립니다. 완료까지 1~2분이 소요됩니다.

![](./media/image14.png)

## 과제 2: AdventureWorksLH lakehouse 만들기

이 과제에서는 새 Lakehouse를 만들고 Fabric notebook을 사용하여 AdventureWorks 테이블로 채웁니다. Lakehouse는 Data Agent가 쿼리하는 구조화된 데이터 기반이 됩니다.

1.  workspace에서 **+ New item** 버튼을 클릭하여 새 lakehouse를 만듭니다.

![](./media/image15.png)

2.  **Lakehouse** 타일을 클릭합니다.

![](./media/image16.png)

3.  **New lakehouse** 대화 상자에서 **Name** 필드에 +++AdventureWorksLH+++를 입력하고, **Create** 버튼을 클릭한 다음 새 lakehouse를 엽니다.

**참고:** 이름 앞뒤에 공백이 없는지 확인하십시오.

![](./media/image17.png)

![](./media/image18.png)

4.  **Successfully created SQL endpoint** 알림이 표시됩니다.

![](./media/image19.png)

5.  lakehouse의 **Home** 리본에서 **Open notebook** > **New notebook**을 선택하여 새 notebook을 만듭니다.

![](./media/image20.png)

6.  첫 번째 **cell**의 코드를 다음 코드로 바꾸고, 셀 왼쪽에 나타나는 **▷ Run cell**을 클릭합니다.

```python
import pandas as pd
from tqdm.auto import tqdm
base = "https://synapseaisolutionsa.z13.web.core.windows.net/data/AdventureWorks"

# load list of tables
df_tables = pd.read_csv(f"{base}/adventureworks.csv", names=["table"])

for table in (pbar := tqdm(df_tables['table'].values)):
    pbar.set_description(f"Uploading {table} to lakehouse")

    # download
    df = pd.read_parquet(f"{base}/{table}.parquet")

    # save as lakehouse table
    spark.createDataFrame(df).write.mode('overwrite').saveAsTable(table)
```

![](./media/image21.png)

![](./media/image22.png)

![](./media/image23.png)

![](./media/image24.png)

![](./media/image25.png)

## 과제 3: Data agent 생성하기

이 과제에서는 Fabric Data Agent를 생성하여 Lakehouse에 연결합니다. 또한, 에이전트가 다양한 판매 관련 분석 질문에 답변할 수 있도록 필요한 차원(Dimension) 및 팩트(Fact) 테이블을 선택합니다.

1.  왼쪽 탐색 창에서 **Fabric Data agent-@lab.LabInstance.Id** workspace를 클릭합니다.

![](./media/image26.png)

2.  workspace에서 **+ New item**을 선택합니다.

![](./media/image27.png)

3.  **Filter by item type** 검색 상자에 +++data agent+++를 입력하고 **Data agent**를 선택합니다.

![](./media/image28.png)

4.  Data agent 이름으로 +++AI-agent+++를 입력하고 **Create**를 선택합니다.

![](./media/image29.png)

![](./media/image30.png)

5.  AI-agent 페이지에서 **Add a data source**를 선택합니다.

![](./media/image31.png)

6.  **OneLake catalog** 탭에서 **AdventureWorksLH** lakehouse를 선택하고 **Add**를 선택합니다.

![](./media/image32.png)

![](./media/image33.png)

7.  그런 다음, data agent가 접근할 수 있는 테이블을 선택합니다.

    이 실습에서는 다음 테이블을 사용합니다:

    - DimCustomer

    - DimDate

    - DimGeography

    - DimProduct

    - DimProductCategory

    - DimPromotion

    - DimReseller

    - DimSalesTerritory

    - FactInternetSales

    - FactResellerSales

![](./media/image34.png)

## 과제 4: 지침과 예시 쿼리 제공하기

이 과제에서는 자연어 질문과 그에 대응하는 SQL 쿼리를 추가하여 data agent를 보강합니다. 이러한 예시는 에이전트가 도메인별 맥락을 이해하고, 실제 환경의 쿼리에 대해 더 정확한 SQL 응답을 생성하는 데 도움을 줍니다.

1.  나열된 테이블 중 **factinternetsales**를 선택하고 첫 번째 질문을 합니다. data agent가 꽤 잘 답변합니다.

2.  다음 질문을 입력하고 **Submit** 아이콘을 클릭합니다:

    +++What is the most sold product?+++

![](./media/image35.png)

![](./media/image36.png)

3.  질문과 SQL 쿼리(query1)를 복사하여 메모장에 붙여넣은 다음, 메모장을 저장합니다. 이후 단계에서 이 정보를 사용합니다.

![](./media/image37.png)

![](./media/image38.png)

4.  **FactResellerSales**를 선택하고 다음 텍스트를 입력한 다음 **Submit** 아이콘을 클릭합니다.

    +++What is our most sold product?+++

![](./media/image39.png)

![](./media/image40.png)

쿼리로 계속 실험하면서 더 많은 지침을 추가해야 합니다.

5.  **dimcustomer**를 선택하고, 다음 텍스트를 입력한 후 **Submit** 아이콘을 클릭합니다.

    +++how many active customers did we have June 1st, 2013?+++

![](./media/image41.png)

![](./media/image42.png)

6.  모든 질문과 SQL 쿼리를 복사하여 메모장에 붙여넣은 다음, 메모장을 저장합니다. 이후 단계에서 이 정보를 사용합니다.

![](./media/image43.png)

![](./media/image44.png)

7.  **FactInternetSales**와 **dimdate**를 선택하고, 다음 텍스트를 입력한 후 **Submit** 아이콘을 클릭합니다:

    +++what are the monthly sales trends for the last year?+++

![](./media/image45.png)

![](./media/image46.png)

8.  **FactInternetSales**와 **dimproduct**를 선택하고, 다음 텍스트를 입력한 후 **Submit** 아이콘을 클릭합니다:

    +++which product category had the highest average sales price?+++

![](./media/image47.png)

![](./media/image48.png)

문제의 일부는 active customer에 대한 공식적인 정의가 없다는 점입니다. 모델 텍스트 상자에 설명을 더 추가하면 도움이 될 수 있지만, 사용자들이 자주 묻는 질문일 수도 있습니다. AI가 질문을 올바르게 처리하는지 확인해야 합니다.

9.  관련 쿼리는 다소 복잡하므로, **Home** 리본에서 **Example queries** 버튼을 선택하여 예시를 추가합니다.

![](./media/image49.png)

10. **Example queries** 탭에서 **Add example**을 선택합니다.

![](./media/image50.png)

11. 여기에서 생성한 lakehouse 데이터 소스에 대한 예시 쿼리를 추가합니다. 질문 입력란에 다음 질문을 추가합니다:

    +++What is the most sold product?+++

![](./media/image51.png)

12. 메모장에 저장해 둔 query1을 **SQL query** 입력란에 추가합니다:

```sql
SELECT TOP 1 ProductKey, SUM(OrderQuantity) AS TotalQuantitySold
FROM [dbo].[factinternetsales]
GROUP BY ProductKey
ORDER BY TotalQuantitySold DESC
```

![](./media/image52.png)

13. 새 쿼리 필드를 추가하려면 **+ Add**를 클릭합니다.

![](./media/image53.png)

14. 질문 입력란에 두 번째 질문을 추가합니다:

    +++What are the monthly sales trends for the last year?+++

![](./media/image54.png)

15. 메모장에 저장해 둔 query3을 추가합니다:

```sql
SELECT
    d.CalendarYear,
    d.MonthNumberOfYear,
    d.EnglishMonthName,
    SUM(f.SalesAmount) AS TotalSales
FROM
    dbo.factinternetsales f
    INNER JOIN dbo.dimdate d ON f.OrderDateKey = d.DateKey
WHERE
    d.CalendarYear = (
        SELECT MAX(CalendarYear)
        FROM dbo.dimdate
        WHERE DateKey IN (SELECT DISTINCT OrderDateKey FROM dbo.factinternetsales)
    )
GROUP BY
    d.CalendarYear,
    d.MonthNumberOfYear,
    d.EnglishMonthName
ORDER BY
    d.MonthNumberOfYear
```

![](./media/image55.png)

16. 새 쿼리 필드를 추가하려면 **+ Add**를 클릭합니다.

![](./media/image56.png)

17. 질문 입력란에 세 번째 질문을 추가합니다:

    +++Which product category has the highest average sales price?+++

![](./media/image57.png)

18. 메모장에 저장해 둔 query4를 추가합니다:

```sql
SELECT TOP 1
    dp.ProductSubcategoryKey AS ProductCategory,
    AVG(fis.UnitPrice) AS AverageSalesPrice
FROM
    dbo.factinternetsales fis
INNER JOIN
    dbo.dimproduct dp ON fis.ProductKey = dp.ProductKey
GROUP BY
    dp.ProductSubcategoryKey
ORDER BY
    AverageSalesPrice DESC
```

![](./media/image58.png)

19. 모든 예시 질문과 SQL 쿼리를 추가한 다음, **Export all**을 클릭합니다. 예시가 **examples.json** 파일로 저장됩니다.

![](./media/image59.png)

![](./media/image60.png)

## 과제 5: 프로그래밍 방식으로 Data agent 사용하기

Data agent에 지침과 예시가 모두 추가되었습니다. 테스트를 진행하면서 예시와 지침을 보강하면 AI의 역량을 더욱 향상시킬 수 있습니다. 동료들이 주로 묻는 질문 유형을 포괄하는 예시와 지침이 제공되었는지 함께 확인해 보십시오.

Fabric notebook에서 프로그래밍 방식으로 data agent를 사용할 수 있습니다. 먼저 data agent에 게시된 URL 값이 있는지 확인합니다.

1.  Data agent 페이지의 **Home** 리본에서 **Settings**를 선택합니다.

![](./media/image61.png)

2.  data agent를 게시하기 전에는 이 스크린샷에서 볼 수 있듯이 게시된 URL 값이 없습니다.

3.  data agent 설정을 닫습니다.

![](./media/image62.png)

4.  **Home** 리본에서 **Publish**를 선택합니다.

![](./media/image63.png)

![](./media/image64.png)

## 과제 6: 리소스 삭제하기

1.  왼쪽 탐색 메뉴에서 **Fabric Data agent-@lab.LabInstance.Id** workspace를 선택합니다. 그러면 workspace 항목 보기가 열립니다.

![](./media/image73.png)

2.  workspace 이름 아래에 있는 **...** 옵션을 선택한 다음 **Workspace settings**를 선택합니다.

![](./media/image74.png)

3.  **General** 탭에서 아래로 스크롤하여 **Remove this workspace**를 선택합니다.

![](./media/image75.png)

4.  팝업으로 나타나는 경고 창에서 **Delete**를 클릭합니다.

![](./media/image76.png)

![](./media/image77.png)

**요약**

이 실습에서는 Microsoft Fabric의 Data Agent를 사용하여 대화형 분석의 잠재력을 활용하는 방법을 배웠습니다. Fabric workspace를 구성하고, 구조화된 데이터를 lakehouse에 수집했으며, 자연어 질문을 SQL 쿼리로 변환하는 data agent를 설정했습니다. 또한 쿼리 생성을 개선하기 위한 지침과 예시를 제공하여 AI 에이전트의 기능을 강화했습니다. 마지막으로, Fabric notebook에서 프로그래밍 방식으로 에이전트를 호출하여 종단 간 AI 통합을 시연했습니다. 이 실습은 자연어 및 생성형 AI 기술을 활용하여 비즈니스 사용자가 기업 데이터에 더 쉽게 접근하고, 유용하고 지능적으로 활용할 수 있도록 지원하는 방법을 보여줍니다.
