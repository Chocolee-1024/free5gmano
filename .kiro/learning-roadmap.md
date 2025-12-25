# Free5GMANO 專案學習路線圖

## 🎯 學習目標

完全理解 Free5GMANO 專案的架構、功能和代碼，能夠自信地進行修改和擴展。

---

## 📚 第一階段：基礎概念理解 (1-2 天)

### 1.1 理解 5G 網路管理基礎

**學習內容**:
- 什麼是 MANO (Management and Network Orchestration)
- 什麼是 NSSI (Network Slice Subnet Instance)
- 什麼是 MOI (Management Object Instance)
- 5G 網路切片的概念

**推薦資源**:
- [3GPP TS 28.531](https://www.3gpp.org/) - 官方標準
- [ETSI NFV-MAN](https://www.etsi.org/) - 歐洲標準
- YouTube: "5G Network Slicing" 相關視頻

**實踐**:
- 閱讀 README.md 理解專案目的
- 查看 deploy/ 文件夾了解部署方式

### 1.2 理解 Django 框架基礎

**學習內容**:
- Django MVT (Model-View-Template) 架構
- Django ORM 資料庫操作
- Django REST Framework (DRF) API 開發
- URL 路由配置

**推薦資源**:
- [Django 官方文檔](https://docs.djangoproject.com/en/4.2/)
- [Django REST Framework 文檔](https://www.django-rest-framework.org/)
- 書籍: "Two Scoops of Django"

**實踐**:
- 創建一個簡單的 Django 項目練習
- 理解 models.py、views.py、serializers.py 的作用

### 1.3 理解 MySQL 資料庫

**學習內容**:
- 關係型資料庫基礎
- SQL 查詢語言
- Django ORM 與 SQL 的對應關係

**推薦資源**:
- [MySQL 官方文檔](https://dev.mysql.com/doc/)
- [Django ORM 文檔](https://docs.djangoproject.com/en/4.2/topics/db/)

**實踐**:
- 使用 MySQL Workbench 查看資料庫結構
- 編寫簡單的 SQL 查詢

---

## 🏗️ 第二階段：專案架構理解 (2-3 天)

### 2.1 理解專案結構

**學習內容**:
```
free5gmano/
├── free5gmano/          # Django 主配置
│   ├── settings.py      # 全局設定
│   ├── urls.py          # URL 路由
│   └── wsgi.py          # WSGI 應用
├── nssmf/               # 網路切片子網管理
│   ├── models.py        # 資料模型
│   ├── views.py         # API 視圖
│   ├── serializers.py   # 序列化器
│   └── urls.py          # URL 路由
├── moi/                 # 管理物件實例
│   ├── models.py        # 資料模型
│   ├── views.py         # API 視圖
│   └── urls.py          # URL 路由
├── FaultManagement/     # 故障管理
│   ├── models.py        # 資料模型
│   ├── views.py         # API 視圖
│   └── urls.py          # URL 路由
└── requirements.txt     # 依賴列表
```

**實踐**:
- 畫出專案的模塊依賴圖
- 理解各模塊之間的關係

### 2.2 理解資料模型

**學習內容**:
- NSSMF 模塊的資料模型
- MOI 模塊的資料模型
- FaultManagement 模塊的資料模型
- 模型之間的關係 (ForeignKey, ManyToMany)

**實踐**:
```bash
# 查看資料庫結構
python manage.py sqlmigrate nssmf 0001
python manage.py sqlmigrate moi 0001
python manage.py sqlmigrate FaultManagement 0001

# 查看模型
python manage.py inspectdb
```

### 2.3 理解 API 端點

**學習內容**:
- NSSMF API 端點
- MOI API 端點
- FaultManagement API 端點
- 每個端點的功能和參數

**實踐**:
- 使用 Postman 或 curl 測試 API
- 查看 Swagger 文檔 (http://localhost:8000/swagger/)

---

## 💻 第三階段：代碼深入理解 (3-5 天)

### 3.1 NSSMF 模塊深入

**學習內容**:
- GenericTemplate 模型和視圖
- SliceTemplate 模型和視圖
- ServiceMappingPluginModel 模型和視圖
- 模板上傳和管理流程

**代碼閱讀順序**:
1. nssmf/models.py - 理解資料結構
2. nssmf/serializers.py - 理解序列化邏輯
3. nssmf/views.py - 理解業務邏輯
4. nssmf/urls.py - 理解 URL 路由

**實踐**:
```python
# 在 Django shell 中測試
python manage.py shell

# 創建模板
from nssmf.models import GenericTemplate
template = GenericTemplate.objects.create(
    name="test",
    templateType="VNF",
    nfvoType="kube5gnfvo"
)

# 查詢模板
templates = GenericTemplate.objects.all()
for t in templates:
    print(t.name, t.templateType)
```

### 3.2 MOI 模塊深入

**學習內容**:
- NetworkSliceSubnet 模型
- AMFFunction、SMFFunction、UPFFunction 等功能實體
- MOI 物件的創建、查詢、修改、刪除
- 訂閱和通知機制

**代碼閱讀順序**:
1. moi/models.py - 理解 MOI 資料結構
2. moi/serializers.py - 理解序列化邏輯
3. moi/views.py - 理解業務邏輯
4. moi/urls.py - 理解 URL 路由

**實踐**:
```python
# 在 Django shell 中測試
python manage.py shell

# 創建 NSSI
from moi.models import NetworkSliceSubnet
nssi = NetworkSliceSubnet.objects.create(
    administrativeState='LOCKED',
    operationalState='ENABLED'
)

# 查詢 NSSI
nssies = NetworkSliceSubnet.objects.all()
```

### 3.3 FaultManagement 模塊深入

**學習內容**:
- AlarmResource 模型
- 告警的創建、查詢、確認、清除
- 告警訂閱機制
- 與 NFVO 的集成

**代碼閱讀順序**:
1. FaultManagement/models.py - 理解告警資料結構
2. FaultManagement/serializers.py - 理解序列化邏輯
3. FaultManagement/views.py - 理解業務邏輯
4. FaultManagement/urls.py - 理解 URL 路由

**實踐**:
```python
# 在 Django shell 中測試
python manage.py shell

# 創建告警
from FaultManagement.models import AlarmResource
alarm = AlarmResource.objects.create(
    alarmId="alarm-001",
    alarmType="ProcessingErrorAlarm",
    perceivedSeverity="Critical"
)

# 查詢告警
alarms = AlarmResource.objects.all()
```

---

## 🔌 第四階段：集成和流程理解 (2-3 天)

### 4.1 NSSI 生命週期

**學習內容**:
- NSSI 分配流程
- NSSI 修改流程
- NSSI 回收流程
- 與 NFVO 的交互

**代碼追蹤**:
1. 查看 ProvisioningView.create() 方法
2. 理解插件調用機制
3. 理解 NFVO 交互

**實踐**:
- 在測試環境中完整執行一次 NSSI 分配流程
- 查看資料庫中的資料變化

### 4.2 MOI 訂閱和通知

**學習內容**:
- 訂閱創建流程
- 後台監控線程 (TaskThread)
- Kafka 通知機制
- 通知發送流程

**代碼追蹤**:
1. 查看 SubscriptionView.create() 方法
2. 理解 TaskThread 類
3. 理解 Kafka 集成

**實踐**:
- 創建訂閱並觀察通知
- 查看 Kafka 消息

### 4.3 故障告警流程

**學習內容**:
- 告警拉取流程
- 告警存儲流程
- 告警確認/清除流程
- 告警訂閱流程

**代碼追蹤**:
1. 查看 FaultSupervisionView.list() 方法
2. 理解 NFVO 集成
3. 理解告警存儲邏輯

**實踐**:
- 在測試環境中完整執行一次告警流程

---

## 🛠️ 第五階段：實踐項目 (3-5 天)

### 5.1 小型功能擴展

**項目 1: 添加日誌記錄**
- 為所有 API 端點添加詳細的日誌記錄
- 記錄請求參數和響應結果
- 記錄錯誤信息

**項目 2: 添加驗證邏輯**
- 為 NSSI 添加狀態驗證
- 為告警添加嚴重程度驗證
- 為模板添加格式驗證

**項目 3: 添加性能優化**
- 為常用查詢添加緩存
- 優化資料庫查詢
- 添加分頁功能

### 5.2 中型功能開發

**項目 4: 添加新的 MOI 類型**
- 創建新的 MOI 模型
- 創建序列化器
- 創建視圖和 URL 路由
- 編寫測試

**項目 5: 添加新的告警類型**
- 創建新的告警類型
- 添加告警處理邏輯
- 添加告警通知

### 5.3 大型功能開發

**項目 6: 添加新的 NFVO 支持**
- 創建新的插件
- 實現分配/回收邏輯
- 集成到系統中

---

## 📖 推薦閱讀順序

### 第 1 週
1. README.md - 了解專案概況
2. .kiro/project-analysis.md - 了解專案結構
3. Django 官方教程 - 學習 Django 基礎

### 第 2 週
1. nssmf/models.py - 理解資料模型
2. nssmf/views.py - 理解業務邏輯
3. nssmf/serializers.py - 理解序列化邏輯

### 第 3 週
1. moi/models.py - 理解 MOI 資料模型
2. moi/views.py - 理解 MOI 業務邏輯
3. moi/serializers.py - 理解 MOI 序列化邏輯

### 第 4 週
1. FaultManagement/models.py - 理解告警資料模型
2. FaultManagement/views.py - 理解告警業務邏輯
3. FaultManagement/serializers.py - 理解告警序列化邏輯

### 第 5 週
1. 完整的 NSSI 生命週期
2. MOI 訂閱和通知機制
3. 故障告警流程

---

## 🎓 學習資源

### 官方文檔
- [Django 4.2 文檔](https://docs.djangoproject.com/en/4.2/)
- [Django REST Framework 文檔](https://www.django-rest-framework.org/)
- [3GPP TS 28.531](https://www.3gpp.org/)
- [ETSI NFV-MAN](https://www.etsi.org/)

### 在線課程
- Udemy: "Django for Beginners"
- Udemy: "Django REST Framework"
- Coursera: "5G Networks"

### 書籍
- "Two Scoops of Django"
- "Django for APIs"
- "Mastering Django"

### 社區
- Django 官方論壇
- Stack Overflow
- GitHub Issues

---

## 🧪 實踐環境設置

### 本地開發環境

```bash
# 1. 克隆專案
git clone https://github.com/free5gmano/free5gmano.git
cd free5gmano

# 2. 創建虛擬環境
python -m venv venv
source venv/bin/activate  # Linux/Mac
# 或
venv\Scripts\activate  # Windows

# 3. 安裝依賴
pip install -r requirements.txt

# 4. 配置資料庫
export FREE5GMANO_MYSQL_USER=root
export FREE5GMANO_MYSQL_PASSWORD=password
export FREE5GMANO_MYSQL_HOST=127.0.0.1

# 5. 運行遷移
python manage.py makemigrations
python manage.py migrate

# 6. 創建超級用戶
python manage.py createsuperuser

# 7. 運行開發服務器
python manage.py runserver
```

### 測試環境

```bash
# 運行測試
python manage.py test

# 運行特定應用的測試
python manage.py test nssmf
python manage.py test moi
python manage.py test FaultManagement

# 運行特定測試類
python manage.py test nssmf.tests.GenericTemplateTestCase

# 運行特定測試方法
python manage.py test nssmf.tests.GenericTemplateTestCase.test_create_template
```

### 調試工具

```bash
# Django shell
python manage.py shell

# Django debug toolbar
# 在 settings.py 中添加 'debug_toolbar'
# 在 urls.py 中添加 debug toolbar URLs

# 使用 pdb 調試
import pdb; pdb.set_trace()
```

---

## 📊 學習進度追蹤

### 第 1 週
- [ ] 理解 5G 網路管理基礎
- [ ] 理解 Django 框架基礎
- [ ] 理解 MySQL 資料庫
- [ ] 理解專案結構

### 第 2 週
- [ ] 深入理解 NSSMF 模塊
- [ ] 能夠修改 NSSMF 代碼
- [ ] 能夠添加新的模板類型

### 第 3 週
- [ ] 深入理解 MOI 模塊
- [ ] 能夠修改 MOI 代碼
- [ ] 能夠添加新的 MOI 類型

### 第 4 週
- [ ] 深入理解 FaultManagement 模塊
- [ ] 能夠修改 FaultManagement 代碼
- [ ] 能夠添加新的告警類型

### 第 5 週
- [ ] 理解完整的 NSSI 生命週期
- [ ] 理解 MOI 訂閱和通知機制
- [ ] 理解故障告警流程
- [ ] 能夠進行小型功能擴展

---

## 💡 學習技巧

### 1. 代碼追蹤
- 從 API 端點開始
- 追蹤到視圖函數
- 追蹤到序列化器
- 追蹤到模型
- 追蹤到資料庫

### 2. 使用調試工具
- 使用 Django shell 測試代碼
- 使用 pdb 調試
- 使用日誌記錄
- 使用 Postman 測試 API

### 3. 閱讀測試代碼
- 測試代碼展示了如何使用 API
- 測試代碼展示了預期的行為
- 測試代碼可以幫助理解代碼邏輯

### 4. 編寫文檔
- 為每個模塊編寫文檔
- 為每個 API 端點編寫文檔
- 為每個複雜的函數編寫文檔

### 5. 進行實踐項目
- 從小型項目開始
- 逐漸增加複雜性
- 在實踐中學習

---

## 🎯 最終目標

完成本學習路線圖後，你應該能夠：

1. ✅ 理解 Free5GMANO 專案的整體架構
2. ✅ 理解每個模塊的功能和責任
3. ✅ 理解資料流和業務流程
4. ✅ 能夠修改現有代碼
5. ✅ 能夠添加新的功能
6. ✅ 能夠調試和優化代碼
7. ✅ 能夠編寫測試
8. ✅ 能夠部署和維護系統

---

**開始日期**: 2024 年 12 月 25 日
**預計完成時間**: 4-5 週
**難度級別**: 中等
