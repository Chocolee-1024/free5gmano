# Free5GMANO 代碼閱讀指南

## 🎯 目標

通過系統化的代碼閱讀，深入理解 Free5GMANO 的架構和實現。

---

## 📖 第一步：理解 Django 代碼結構

### Django 應用的標準結構

```
應用名稱/
├── models.py       # 資料模型 (數據庫表)
├── views.py        # 視圖 (業務邏輯)
├── serializers.py  # 序列化器 (JSON 轉換)
├── urls.py         # URL 路由
├── tests.py        # 測試
└── admin.py        # 管理後台
```

### 代碼流程

```
HTTP 請求
    ↓
URL 路由 (urls.py)
    ↓
視圖 (views.py)
    ↓
序列化器 (serializers.py)
    ↓
模型 (models.py)
    ↓
資料庫
```

---

## 🔍 第二步：NSSMF 模塊代碼閱讀

### 2.1 models.py - 理解資料結構

**關鍵模型**:
1. `ServiceMappingPluginModel` - 插件配置
2. `GenericTemplate` - 通用模板
3. `Content` - 模板內容
4. `SliceTemplate` - 切片模板

**閱讀順序**:
```python
# 1. 查看 ServiceMappingPluginModel
# - 了解插件如何配置
# - 了解 NFVO 主機配置

# 2. 查看 GenericTemplate
# - 了解模板的基本信息
# - 了解模板的狀態

# 3. 查看 Content
# - 了解模板內容的存儲方式
# - 了解 TOSCA 定義

# 4. 查看 SliceTemplate
# - 了解切片模板如何組合
# - 了解模板與 NSSI 的關係
```

### 2.2 serializers.py - 理解序列化邏輯

**關鍵序列化器**:
1. `GenericTemplateSerializer` - 模板序列化
2. `GenericTemplateFileSerializer` - 文件上傳序列化
3. `SliceTemplateSerializer` - 切片模板序列化
4. `ServiceMappingPluginSerializer` - 插件序列化

**閱讀重點**:
- 如何將模型轉換為 JSON
- 如何驗證輸入數據
- 如何處理複雜的序列化邏輯

### 2.3 views.py - 理解業務邏輯

**關鍵視圖**:
1. `GenericTemplateView` - 模板管理
2. `SliceTemplateView` - 切片模板管理
3. `ProvisioningView` - NSSI 分配/回收
4. `ServiceMappingPluginView` - 插件管理

**閱讀重點**:
- 如何處理 HTTP 請求
- 如何調用業務邏輯
- 如何返回響應

### 2.4 urls.py - 理解 URL 路由

**路由配置**:
```python
router.register(r'ObjectManagement/GenericTemplate', GenericTemplateView)
# 生成的 URL:
# GET    /ObjectManagement/GenericTemplate/          - 列表
# POST   /ObjectManagement/GenericTemplate/          - 創建
# GET    /ObjectManagement/GenericTemplate/{id}/     - 詳情
# PATCH  /ObjectManagement/GenericTemplate/{id}/     - 更新
# DELETE /ObjectManagement/GenericTemplate/{id}/     - 刪除
```

---

## 🔍 第三步：MOI 模塊代碼閱讀

### 3.1 models.py - 理解 MOI 資料結構

**關鍵模型**:
1. `NetworkSliceSubnet` - NSSI
2. `AMFFunction` - AMF 功能
3. `SMFFunction` - SMF 功能
4. `UPFFunction` - UPF 功能
5. `Subscription` - 訂閱

**閱讀重點**:
- NSSI 的組成
- 功能實體的配置
- 訂閱的結構

### 3.2 views.py - 理解 MOI 業務邏輯

**關鍵視圖**:
1. `ObjectManagement` - MOI 對象管理
2. `SubscriptionView` - 訂閱管理
3. `NotificationView` - 通知管理
4. `TopologyView` - 拓撲查詢

**閱讀重點**:
- 如何創建/查詢/修改/刪除 MOI 對象
- 如何處理訂閱
- 如何發送通知

### 3.3 TaskThread - 理解後台監控

**功能**:
- 監控 MOI 對象的變化
- 發送通知到 Kafka

**閱讀重點**:
- 如何使用線程
- 如何監控資料庫變化
- 如何發送 Kafka 消息

---

## 🔍 第四步：FaultManagement 模塊代碼閱讀

### 4.1 models.py - 理解告警資料結構

**關鍵模型**:
1. `AlarmResource` - 告警
2. `Header` - 通知頭
3. `SubscriptionResource` - 訂閱

**閱讀重點**:
- 告警的字段和狀態
- 告警的嚴重程度
- 告警的確認狀態

### 4.2 views.py - 理解告警業務邏輯

**關鍵視圖**:
1. `FaultSupervisionView` - 告警管理
2. `FaultSupervisionSubscriptionsView` - 告警訂閱

**閱讀重點**:
- 如何從 NFVO 拉取告警
- 如何存儲告警
- 如何確認/清除告警

---

## 💡 代碼閱讀技巧

### 1. 使用 IDE 的代碼導航
- 使用 "Go to Definition" 快速跳轉
- 使用 "Find References" 查找使用位置
- 使用 "Find in Files" 搜索代碼

### 2. 添加註釋和筆記
- 在代碼旁邊添加註釋
- 記錄關鍵概念
- 記錄函數的功能

### 3. 繪製流程圖
- 繪製 API 流程圖
- 繪製資料流圖
- 繪製類關係圖

### 4. 編寫測試代碼
- 編寫簡單的測試
- 驗證理解是否正確
- 發現代碼的邊界情況

### 5. 進行代碼追蹤
- 從 API 端點開始
- 追蹤到視圖
- 追蹤到序列化器
- 追蹤到模型
- 追蹤到資料庫

---

## 🧪 實踐練習

### 練習 1: 追蹤 API 調用

**任務**: 追蹤 `GET /ObjectManagement/GenericTemplate/` 的完整流程

**步驟**:
1. 查看 nssmf/urls.py 中的路由配置
2. 查看 GenericTemplateView.list() 方法
3. 查看 GenericTemplateSerializer
4. 查看 GenericTemplate 模型
5. 查看資料庫查詢

### 練習 2: 理解模板上傳流程

**任務**: 理解模板文件上傳的完整流程

**步驟**:
1. 查看 GenericTemplateView.upload() 方法
2. 理解 ZIP 文件解析
3. 理解 YAML/JSON 解析
4. 理解 Content 模型的創建

### 練習 3: 理解 NSSI 分配流程

**任務**: 理解 NSSI 分配的完整流程

**步驟**:
1. 查看 ProvisioningView.create() 方法
2. 理解插件調用機制
3. 理解 NFVO 交互
4. 理解 NSSI 創建

### 練習 4: 理解訂閱和通知

**任務**: 理解 MOI 訂閱和通知的完整流程

**步驟**:
1. 查看 SubscriptionView.create() 方法
2. 理解 TaskThread 的工作原理
3. 理解 Kafka 消息發送
4. 理解通知機制

---

## 📊 代碼複雜度分析

### 簡單模塊 (適合初學者)
- FaultManagement - 告警管理
- 簡單的 CRUD 操作

### 中等複雜度 (適合中級開發者)
- NSSMF - 模板管理
- 文件上傳和解析

### 複雜模塊 (適合高級開發者)
- MOI - 訂閱和通知
- 後台線程和 Kafka 集成
- NFVO 插件調用

---

## 🎓 學習成果檢驗

完成代碼閱讀後，你應該能夠回答以下問題：

### NSSMF 模塊
- [ ] GenericTemplate 和 SliceTemplate 的區別是什麼？
- [ ] 模板文件上傳的流程是什麼？
- [ ] 如何調用 NFVO 插件？
- [ ] NSSI 分配的流程是什麼？

### MOI 模塊
- [ ] NetworkSliceSubnet 包含哪些信息？
- [ ] 如何創建 MOI 訂閱？
- [ ] TaskThread 的作用是什麼？
- [ ] 如何發送 Kafka 通知？

### FaultManagement 模塊
- [ ] 告警的生命週期是什麼？
- [ ] 如何從 NFVO 拉取告警？
- [ ] 如何確認/清除告警？
- [ ] 告警訂閱的流程是什麼？

---

**下一步**: 按照本指南進行代碼閱讀，然後進行實踐練習。
