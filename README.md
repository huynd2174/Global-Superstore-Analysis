# Mô tả chi tiết Project Power BI - Global Superstore

## 1. Tổng quan Project

Project này sử dụng bộ dữ liệu Global Superstore để phân tích hiệu suất kinh doanh và khám phá các cơ hội tăng trưởng thông qua việc xây dựng mô hình dữ liệu Star Schema, thực hiện ETL, tạo các measure DAX, và thiết kế báo cáo trực quan trong Power BI.

### 1.1. Bối cảnh và Mục tiêu Nghiệp vụ

Project nhằm giải quyết các câu hỏi nghiệp vụ quan trọng sau:

- **Hiệu suất Tổng thể**: Doanh số, lợi nhuận, và số lượng đơn hàng biến động như thế nào theo thời gian, khu vực địa lý, và phân khúc sản phẩm/khách hàng?
- **Phân tích Sản phẩm**: Những sản phẩm/danh mục nào mang lại doanh thu và lợi nhuận cao nhất? Sản phẩm nào có tỷ suất lợi nhuận tốt nhất/kém nhất?
- **Phân tích Khách hàng**: Khách hàng nào mang lại giá trị cao nhất? Phân khúc khách hàng nào đóng góp nhiều nhất vào doanh thu và lợi nhuận?
- **Phân tích Vận hành**: Thời gian giao hàng trung bình là bao lâu? Các phương thức vận chuyển khác nhau ảnh hưởng thế nào đến chi phí?
- **Phân tích Chiết khấu**: Mối quan hệ giữa chiết khấu và doanh số/lợi nhuận là gì? Chiến lược chiết khấu nào hiệu quả nhất?

### 1.2. Nguồn dữ liệu và Quy trình ETL

Dữ liệu được lấy từ bộ dữ liệu Global Superstore, bao gồm thông tin về đơn hàng, sản phẩm, khách hàng, địa lý, và vận chuyển từ năm 2012 đến 2015. Quy trình ETL bao gồm:

- **Extraction**: Dữ liệu được trích xuất từ file Excel gốc.
- **Transformation**: 
  - Làm sạch dữ liệu (xử lý giá trị null, loại bỏ trùng lặp)
  - Tạo bảng DimDate từ các trường ngày có sẵn
  - Chuẩn hóa dữ liệu thành các bảng Dimension và Fact
  - Tạo các cột tính toán như Shipping Duration (Order Date đến Ship Date)
  - Phân loại sản phẩm theo giá trị (High-end, Mid-range, Budget)
- **Loading**: Tải dữ liệu đã xử lý vào mô hình Star Schema trong Power BI.

## 2. Cấu trúc Mô hình Dữ liệu

Project sử dụng mô hình dữ liệu Star Schema với các bảng chính sau:

### 2.1. Bảng Fact

- **FactSales**: Bảng sự kiện chính chứa các thông tin giao dịch bán hàng.
  - **Khóa chính**: OrderID (kết hợp với ProductID tạo thành khóa tổng hợp)
  - **Khóa ngoại**: CustomerKey, ProductKey, ShipDateKey, OrderDateKey, ShipModeKey, GeographyKey
  - **Measures**: Sales, Profit, Quantity, Shipping Cost, Discount
  - **Cột tính toán**: 
    - Shipping Duration = DATEDIFF(OrderDate, ShipDate, DAY)
    - Discount Amount = Sales * Discount
    - Pre-Discount Price = Sales / (1 - Discount)

### 2.2. Bảng Dimension

- **DimDate/DimDateFull**: Chứa thông tin thời gian với phân cấp Year > Quarter > Month > Day.
  - **Khóa chính**: DateKey
  - **Thuộc tính**: Year, Quarter, Month, MonthName, Day, DayOfWeek, IsWeekend, IsHoliday, FiscalYear, FiscalQuarter
  - **Phân cấp**: Date Hierarchy (Year > Quarter > Month > Day)

- **DimProduct**: Chứa thông tin về sản phẩm.
  - **Khóa chính**: ProductKey
  - **Thuộc tính**: ProductID, ProductName, Category, SubCategory, Brand, Color, ProductCost
  - **Phân cấp**: Product Hierarchy (Category > SubCategory > Product)
  - **Cột tính toán**: 
    - Price Range (High-end, Mid-range, Budget) dựa trên giá sản phẩm
    - Product Full Name = Category + " - " + SubCategory + " - " + ProductName

- **DimCustomer**: Chứa thông tin về khách hàng.
  - **Khóa chính**: CustomerKey
  - **Thuộc tính**: CustomerID, CustomerName, Segment (Consumer, Corporate, Home Office)
  - **Cột tính toán**: 
    - Customer Tenure = DATEDIFF(FirstPurchaseDate, LASTDATE(FactSales[OrderDate]), MONTH)
    - Customer Value Tier (High, Medium, Low) dựa trên tổng doanh thu

- **DimGeography**: Chứa thông tin địa lý.
  - **Khóa chính**: GeographyKey
  - **Thuộc tính**: Market, Region, Country, State, City, PostalCode
  - **Phân cấp**: Geography Hierarchy (Market > Region > Country > State > City)

- **DimShipMode**: Chứa thông tin về phương thức vận chuyển.
  - **Khóa chính**: ShipModeKey
  - **Thuộc tính**: ShipMode, DeliverySpeed (Standard, Express, Same Day)

### 2.3. Mối quan hệ trong Mô hình

- FactSales[CustomerKey] → DimCustomer[CustomerKey]
- FactSales[ProductKey] → DimProduct[ProductKey]
- FactSales[OrderDateKey] → DimDate[DateKey]
- FactSales[ShipDateKey] → DimDate[DateKey] (mối quan hệ thứ hai với bảng DimDate)
- FactSales[GeographyKey] → DimGeography[GeographyKey]
- FactSales[ShipModeKey] → DimShipMode[ShipModeKey]

## 3. Các KPI và Measure DAX Chi tiết

Project sử dụng nhiều measure DAX để tính toán các KPI quan trọng:

### 3.1. Measures Cơ bản

- **Total Sales/Revenue**:
  ```dax
  Total Sales = SUM(FactSales[Sales])
  ```

- **Total Profit**:
  ```dax
  Total Profit = SUM(FactSales[Profit])
  ```

- **Profit Margin %**:
  ```dax
  Profit Margin % = 
  DIVIDE(
      [Total Profit],
      [Total Sales],
      0
  )
  ```
  *Lưu ý*: Hàm DIVIDE được sử dụng để xử lý trường hợp Total Sales = 0, trả về 0 thay vì lỗi.

- **Total Costs**:
  ```dax
  Total Costs = [Total Sales] - [Total Profit]
  ```
  *Lưu ý*: Khi Profit âm (lỗ), Total Costs sẽ lớn hơn Total Sales, phản ánh đúng thực tế kinh doanh.

- **Total Quantity Sold**:
  ```dax
  Total Quantity = SUM(FactSales[Quantity])
  ```

- **Total Orders**:
  ```dax
  Total Orders = DISTINCTCOUNT(FactSales[OrderID])
  ```

- **Average Sales per Order (AOV)**:
  ```dax
  Average Sales per Order = 
  DIVIDE(
      [Total Sales],
      [Total Orders],
      0
  )
  ```

- **Total Shipping Cost**:
  ```dax
  Total Shipping Cost = SUM(FactSales[Shipping Cost])
  ```

- **Total Discount Amount**:
  ```dax
  Total Discount Amount = 
  SUMX(
      FactSales,
      FactSales[Sales] * FactSales[Discount]
  )
  ```

- **Average Discount %**:
  ```dax
  Average Discount % = 
  DIVIDE(
      SUMX(FactSales, FactSales[Sales] * FactSales[Discount]),
      SUMX(FactSales, FactSales[Sales] / (1 - FactSales[Discount])),
      0
  )
  ```

### 3.2. Measures Phân tích Thời gian

- **Sales YoY Growth %**:
  ```dax
  Sales YoY Growth % = 
  VAR CurrentSales = [Total Sales]
  VAR PreviousSales = 
      CALCULATE(
          [Total Sales],
          SAMEPERIODLASTYEAR(DimDate[Date])
      )
  RETURN
  DIVIDE(
      CurrentSales - PreviousSales,
      PreviousSales,
      0
  )
  ```

- **Sales SPLY (Same Period Last Year)**:
  ```dax
  Sales SPLY = 
  CALCULATE(
      [Total Sales],
      SAMEPERIODLASTYEAR(DimDate[Date])
  )
  ```

- **Sales YTD (Year-to-Date)**:
  ```dax
  Sales YTD = 
  CALCULATE(
      [Total Sales],
      DATESYTD(DimDate[Date])
  )
  ```

- **Sales QTD (Quarter-to-Date)**:
  ```dax
  Sales QTD = 
  CALCULATE(
      [Total Sales],
      DATESQTD(DimDate[Date])
  )
  ```

- **Sales MTD (Month-to-Date)**:
  ```dax
  Sales MTD = 
  CALCULATE(
      [Total Sales],
      DATESMTD(DimDate[Date])
  )
  ```

### 3.3. Measures Phân tích Nâng cao

- **Product Sales Rank**:
  ```dax
  Product Sales Rank = 
  RANKX(
      ALL(DimProduct),
      [Total Sales],
      ,
      DESC
  )
  ```

- **Customer Profit Rank**:
  ```dax
  Customer Profit Rank = 
  RANKX(
      ALL(DimCustomer),
      [Total Profit],
      ,
      DESC
  )
  ```

- **% Product Sales of Category Total**:
  ```dax
  % Product Sales of Category Total = 
  VAR ProductSales = [Total Sales]
  VAR CategorySales = 
      CALCULATE(
          [Total Sales],
          ALLEXCEPT(DimProduct, DimProduct[Category])
      )
  RETURN
  DIVIDE(
      ProductSales,
      CategorySales,
      0
  )
  ```

- **Average Shipping Duration Days**:
  ```dax
  Average Shipping Duration Days = 
  AVERAGEX(
      FactSales,
      DATEDIFF(FactSales[OrderDate], FactSales[ShipDate], DAY)
  )
  ```

- **Late Shipment Rate %**:
  ```dax
  Late Shipment Rate % = 
  VAR LateShipments = 
      COUNTROWS(
          FILTER(
              FactSales,
              DATEDIFF(FactSales[OrderDate], FactSales[ShipDate], DAY) > 5
          )
      )
  VAR TotalShipments = COUNTROWS(FactSales)
  RETURN
  DIVIDE(
      LateShipments,
      TotalShipments,
      0
  )
  ```

- **Running Total Sales**:
  ```dax
  Running Total Sales = 
  CALCULATE(
      [Total Sales],
      DATESYTD(DimDate[Date], "12-31")
  )
  ```

- **Sales Moving Average (3 Months)**:
  ```dax
  Sales Moving Average (3M) = 
  AVERAGEX(
      DATESINPERIOD(
          DimDate[Date],
          MAX(DimDate[Date]),
          -3,
          MONTH
      ),
      [Total Sales]
  )
  ```

- **Customer Lifetime Value**:
  ```dax
  Customer Lifetime Value = 
  DIVIDE(
      [Total Profit],
      DISTINCTCOUNT(DimCustomer[CustomerKey]),
      0
  )
  ```

## 4. Báo cáo và Dashboard Chi tiết

Project bao gồm nhiều trang báo cáo được thiết kế để trả lời các câu hỏi nghiệp vụ:

### 4.1. Tổng quan Hiệu suất (Overview Dashboard)

- **Mục tiêu**: Cung cấp cái nhìn tổng thể về hiệu suất kinh doanh.
- **Các Visuals chi tiết**:
  - **KPI Cards**: 
    - Total Sales: $12.64M
    - Total Profit: $1.47M
    - Profit Margin %: 11.63%
    - Total Orders: 25,273
  - **Doanh thu và Lợi nhuận theo Năm và Quý** (Biểu đồ cột và đường kết hợp):
    - Cột màu xanh dương nhạt: Doanh thu (Revenue)
    - Cột màu xanh dương đậm: Lợi nhuận (Profit)
    - Đường màu cam: Tỷ suất lợi nhuận (Profit Margin %)
    - Trục X: Năm-Quý (2012-Q1 đến 2015-Q4)
    - Trục Y chính: Giá trị doanh thu và lợi nhuận
    - Trục Y phụ: Tỷ suất lợi nhuận (%)
    - **Insight**: Quý 4 thường có doanh thu cao nhất trong năm, đặc biệt là Q4/2015. Tỷ suất lợi nhuận biến động lớn theo quý, không nhất thiết tương quan với doanh thu cao.
  - **Doanh thu theo Phân khúc Khách hàng** (Biểu đồ tròn):
    - Consumer: $6.51M (51.48%)
    - Corporate: $3.82M (30.25%)
    - Home Office: $2.31M (18.26%)
  - **Doanh thu theo Danh mục Sản phẩm** (Biểu đồ tròn):
    - Technology: $4.74M (37.53%)
    - Furniture: $4.11M (32.5%)
    - Office Supplies: $3.79M (29.9%)
  - **Doanh thu theo Quốc gia** (Biểu đồ cột ngang):
    - United States: >$2M (dẫn đầu)
    - Tiếp theo là các quốc gia khác với giá trị thấp hơn đáng kể
  - **Doanh thu theo Dòng Sản phẩm** (Biểu đồ cột ngang):
    - Phones: Dẫn đầu
    - Copiers, Chairs, Bookcases, Storage: Theo sau

### 4.2. Phân tích Sản phẩm (Product Analysis)

- **Mục tiêu**: Phân tích sâu về hiệu suất của từng sản phẩm và danh mục.
- **Các Visuals chi tiết**:
  - **Bảng/Ma trận Phân tích Sản phẩm**:
    - Hàng: Category > SubCategory > Product
    - Cột: Sales, Profit, Profit Margin %, Quantity, ASPTD (Average Sales per Transaction Day)
    - Định dạng có điều kiện: Màu đỏ cho Profit âm, màu xanh cho Profit dương
    - **Insight cụ thể**: 
      - Furniture: Doanh thu $4.11M, Lợi nhuận $285K, Tỷ suất lợi nhuận 6.94%
        - Tables: Doanh thu $757K, Lợi nhuận -$64K (âm), ASPTD $1,309.76
      - Office Supplies: Doanh thu $3.79M, Lợi nhuận $518.4K, Tỷ suất lợi nhuận 13.68%
      - Technology: Doanh thu $4.74M, Lợi nhuận $663.8K, Tỷ suất lợi nhuận 14.00%
  - **Tổng Lợi nhuận theo Dòng Sản phẩm - Top Performers** (Biểu đồ cột):
    - Copiers: Lợi nhuận cao nhất
    - Phones: Lợi nhuận cao thứ hai
    - Accessories, Appliances, Chairs, Bookcases: Lợi nhuận tốt
  - **Tổng Lợi nhuận theo Dòng Sản phẩm - Bottom Performers** (Biểu đồ cột):
    - Tables: Lợi nhuận âm sâu nhất (-$60K đến -$70K)
    - Envelopes, Supplies, Labels, Fasteners: Lợi nhuận dương nhưng thấp
  - **Treemap Lợi nhuận theo Dòng Sản phẩm**:
    - Kích thước ô: Tỷ trọng đóng góp lợi nhuận
    - Màu sắc: Tỷ suất lợi nhuận (đỏ đến xanh)
    - Copiers và Phones chiếm diện tích lớn nhất
  - **Doanh thu theo Màu sắc** (Biểu đồ cột ngang):
    - Black (Đen): ~$0.6M
    - Red (Đỏ): Gần bằng màu Đen
    - White (Trắng): ~$0.55M
  - **Doanh thu theo Thương hiệu** (Biểu đồ cột ngang):
    - Hon: ~$0.5M
    - Canon, Cisco, Hewlett: Theo sau

### 4.3. Phân tích Khách hàng (Customer Analysis)

- **Mục tiêu**: Hiểu về hành vi và giá trị của khách hàng.
- **Các Visuals chi tiết**:
  - **Doanh số và Lợi nhuận theo Phân khúc Khách hàng** (Biểu đồ cột ghép):
    - Cột trái (xanh nhạt): Doanh số
    - Cột phải (xanh đậm): Lợi nhuận
    - Consumer: Doanh số và lợi nhuận cao nhất
    - Corporate: Doanh số và lợi nhuận trung bình
    - Home Office: Doanh số và lợi nhuận thấp nhất
  - **Top 10 Khách hàng theo Lợi nhuận** (Bảng):
    - Cột: CustomerName, Segment, Total Sales, Total Profit, Profit Margin %, Orders Count
    - Sắp xếp theo Total Profit giảm dần
  - **Phân bố Khách hàng theo Giá trị Đơn hàng Trung bình** (Histogram):
    - Trục X: Giá trị đơn hàng trung bình
    - Trục Y: Số lượng khách hàng
    - Phân bố lệch phải, đa số khách hàng có giá trị đơn hàng trung bình thấp đến trung bình
  - **Tần suất Mua hàng theo Phân khúc** (Biểu đồ cột):
    - Trục X: Phân khúc khách hàng
    - Trục Y: Số đơn hàng trung bình mỗi khách hàng
    - Corporate có tần suất mua hàng cao nhất

### 4.4. Phân tích Địa lý (Geographic Analysis)

- **Mục tiêu**: Phân tích hiệu suất kinh doanh theo khu vực địa lý.
- **Các Visuals chi tiết**:
  - **Bản đồ Doanh thu theo Quốc gia**:
    - Kích thước điểm: Tỷ lệ với doanh thu
    - Màu sắc: Dựa trên tỷ suất lợi nhuận (đỏ đến xanh)
    - Điểm lớn nhất tập trung ở Mỹ và một số nước Châu Âu
    - Châu Á - Thái Bình Dương cũng có sự hiện diện đáng kể
  - **Doanh thu theo Quốc gia** (Biểu đồ cột ngang):
    - United States: >$2M
    - Các quốc gia khác có doanh thu thấp hơn đáng kể
  - **Doanh thu theo Thị trường** (Biểu đồ tròn):
    - Asia Pacific và Europe: Chiếm gần 60% tổng doanh thu
    - USCA (US & Canada), LATAM (Latin America), Africa: Phần còn lại
  - **Lợi nhuận theo Khu vực** (Biểu đồ cột):
    - Trục X: Region
    - Trục Y: Total Profit
    - Central: Lợi nhuận cao nhất
    - East, West, South: Theo sau
  - **Bảng Chi tiết theo Quốc gia**:
    - Cột: Country, Market, Total Sales, Total Profit, Profit Margin %, Orders Count
    - Sắp xếp theo Total Sales giảm dần

### 4.5. Phân tích Chiết khấu (Discount Analysis)

- **Mục tiêu**: Đánh giá hiệu quả của chiến lược chiết khấu.
- **Các Visuals chi tiết**:
  - **Doanh thu và Chiết khấu theo Danh mục** (Biểu đồ cột chồng):
    - Phần nhạt: Tổng lượng chiết khấu (Discount Amount)
    - Phần đậm: Doanh thu (Revenue)
    - Technology: Doanh thu cao nhất, lượng chiết khấu đáng kể
    - Furniture: Doanh thu thấp hơn Technology, nhưng tỷ lệ chiết khấu cao hơn
    - Office Supplies: Doanh thu và lượng chiết khấu thấp nhất
  - **Doanh thu theo Mức Chiết khấu** (Biểu đồ đường):
    - Trục X: Mức chiết khấu (0% đến 100%)
    - Trục Y: Doanh thu
    - Đỉnh cao nhất ở mức chiết khấu 0%
    - Doanh thu đáng kể ở mức chiết khấu 10-20%
    - Doanh thu giảm mạnh khi mức chiết khấu >30%
  - **Tỷ suất Lợi nhuận theo Mức Chiết khấu** (Biểu đồ đường):
    - Trục X: Mức chiết khấu (0% đến 100%)
    - Trục Y: Profit Margin %
    - Xu hướng giảm khi mức chiết khấu tăng
    - Tỷ suất lợi nhuận âm ở mức chiết khấu cao (>50%)
  - **Phân bố Chiết khấu theo Danh mục và Phân khúc** (Heat Map):
    - Hàng: Category
    - Cột: Segment
    - Màu sắc: Mức chiết khấu trung bình (đỏ = cao, xanh = thấp)
    - Furniture cho Corporate: Mức chiết khấu cao nhất

### 4.6. Phân tích Vận hành (Operations Analysis)

- **Mục tiêu**: Đánh giá hiệu quả của quy trình vận chuyển và đặt hàng.
- **Các Visuals chi tiết**:
  - **Tổng Đơn hàng và Doanh số Trung bình mỗi Đơn hàng theo Năm và Quý** (Biểu đồ kết hợp):
    - Cột (xanh dương): Tổng đơn hàng
    - Đường (xanh đậm): Doanh số trung bình mỗi đơn hàng
    - Xu hướng tăng số lượng đơn hàng qua các năm
    - Quý 4 thường có số lượng đơn hàng cao nhất
    - Doanh số trung bình mỗi đơn hàng dao động quanh $450-$550
  - **Tổng Đơn hàng theo Ngày trong Tuần** (Biểu đồ tròn):
    - Wednesday, Saturday, Tuesday, Thursday, Friday: Mỗi ngày chiếm khoảng 17-18%
    - Sunday (9.1%) và Monday: Tỷ lệ thấp hơn đáng kể
  - **Thời gian Giao hàng Trung bình theo Phương thức Vận chuyển** (Biểu đồ cột):
    - Trục X: Ship Mode
    - Trục Y: Average Shipping Duration Days
    - Standard Class: Thời gian dài nhất
    - Same Day: Thời gian ngắn nhất
  - **Tỷ lệ Đơn hàng Giao trễ theo Phương thức Vận chuyển** (Biểu đồ cột):
    - Trục X: Ship Mode
    - Trục Y: Late Shipment Rate %
    - Standard Class: Tỷ lệ giao trễ cao nhất
  - **Bảng Chi tiết Vận chuyển**:
    - Cột: Ship Mode, Orders Count, Average Shipping Duration, Late Shipment Rate %, Total Shipping Cost
    - Sắp xếp theo Orders Count giảm dần

## 5. Insight và Đề xuất Hành động Chi tiết

Dựa trên phân tích dữ liệu, project đã đưa ra các insight và đề xuất hành động sau:

### 5.1. Điểm mạnh

- **Tăng trưởng doanh thu tốt qua các năm**: Doanh thu tăng đều từ 2012 đến 2015, đặc biệt là Quý 4 hàng năm, cho thấy tính mùa vụ mạnh mẽ vào cuối năm (có thể do mua sắm lễ hội, tổng kết cuối năm của doanh nghiệp).
- **Phân khúc Consumer dẫn đầu**: Chiếm 51.48% tổng doanh thu, là phân khúc khách hàng chủ lực.
- **Danh mục Technology hiệu quả nhất**: Vừa có doanh thu cao nhất (37.53%) vừa có tỷ suất lợi nhuận cao nhất (14.00%).
- **Sản phẩm "Copiers" và "Phones" là ngôi sao**: Đóng góp lớn vào cả doanh thu và lợi nhuận.

### 5.2. Điểm cần chú ý

- **Biến động Tỷ suất Lợi nhuận**: Tỷ suất lợi nhuận biến động lớn theo quý, không nhất thiết tương quan với doanh thu cao. Cần phân tích sâu hơn để hiểu rõ nguyên nhân và tìm cách ổn định hoặc cải thiện tỷ suất lợi nhuận, đặc biệt là trong các quý có doanh thu cao.
  - **Nguyên nhân có thể**: Cơ cấu sản phẩm bán ra trong từng quý (bán nhiều sản phẩm có biên lợi nhuận cao/thấp), các chương trình khuyến mãi ảnh hưởng đến giá bán, hoặc chi phí biến đổi.

- **Vấn đề với mặt hàng "Tables"**: Doanh thu khá ($757K) nhưng Lợi nhuận âm (-$64K). Đây là điểm đáng báo động cần xử lý ngay.
  - **Nguyên nhân có thể**: Chi phí sản xuất/vận chuyển cao, chiết khấu quá nhiều, định giá không hợp lý.
  - **Đề xuất cụ thể**: 
    - Phân tích chi tiết cấu trúc chi phí của sản phẩm Tables
    - Xem xét lại chiến lược giá và chiết khấu
    - Đánh giá lại nhà cung cấp hoặc quy trình sản xuất
    - Nếu không thể cải thiện, cân nhắc loại bỏ hoặc thay thế bằng dòng sản phẩm khác

- **Phân bố doanh thu theo địa lý không đồng đều**: United States là thị trường chủ lực, các thị trường khác có doanh thu thấp hơn đáng kể.
  - **Đề xuất cụ thể**: 
    - Đánh giá tiềm năng của từng thị trường ngoài Mỹ
    - Xây dựng chiến lược marketing và bán hàng riêng cho từng khu vực
    - Cân nhắc điều chỉnh danh mục sản phẩm phù hợp với nhu cầu địa phương

### 5.3. Cơ hội

- **Tối ưu hóa chiến lược cho Quý 4**: Không chỉ tăng doanh thu mà còn cả lợi nhuận.
  - **Đề xuất cụ thể**:
    - Tập trung bán các sản phẩm có tỷ suất lợi nhuận cao trong Quý 4
    - Thiết kế chương trình khuyến mãi thông minh không làm giảm nhiều biên lợi nhuận
    - Chuẩn bị kế hoạch logistics tối ưu để giảm chi phí vận chuyển trong mùa cao điểm

- **Tối ưu hóa Chiến lược Chiết khấu**: Phân tích cho thấy phần lớn doanh thu được tạo ra ở mức chiết khấu 0%, và chiết khấu thấp (10-20%) có thể hiệu quả để thúc đẩy doanh số.
  - **Đề xuất cụ thể**:
    - Giảm mức chiết khấu cao (>30%) vì không mang lại doanh thu tương xứng và làm giảm lợi nhuận
    - Áp dụng chiết khấu có mục tiêu: Tập trung vào sản phẩm có biên lợi nhuận cao
    - Thay thế chiết khấu bằng các chương trình khuyến mãi khác: quà tặng, dịch vụ bổ sung, bảo hành mở rộng

- **Khai thác Hành vi Mua sắm theo Ngày**: Sunday và Monday có lượng đơn hàng thấp hơn đáng kể.
  - **Đề xuất cụ thể**:
    - Triển khai các chương trình khuyến mãi đặc biệt vào Chủ Nhật và Thứ Hai
    - Phân tích hành vi mua sắm của từng phân khúc khách hàng theo ngày
    - Tối ưu hóa chiến dịch marketing email/social media vào cuối tuần

- **Tập trung vào các "Ngôi sao"**: "Copiers" và "Phones" trong Technology, "Appliances" và "Storage" trong Office Supplies.
  - **Đề xuất cụ thể**:
    - Đảm bảo nguồn cung ổn định cho các sản phẩm này
    - Đầu tư vào marketing và đào tạo nhân viên bán hàng
    - Phát triển các sản phẩm bổ sung hoặc nâng cấp
    - Xây dựng chương trình khách hàng thân thiết tập trung vào người mua các sản phẩm này

- **Nâng cao tỷ suất lợi nhuận cho Furniture**: Ngoài việc xử lý "Tables", cần xem xét cách tăng tỷ suất lợi nhuận cho các mặt hàng khác trong danh mục Nội thất.
  - **Đề xuất cụ thể**:
    - Đánh giá lại cấu trúc chi phí và định giá
    - Tìm kiếm nhà cung cấp mới hoặc đàm phán lại với nhà cung cấp hiện tại
    - Cân nhắc chuyển hướng sang các sản phẩm nội thất cao cấp có biên lợi nhuận tốt hơn

