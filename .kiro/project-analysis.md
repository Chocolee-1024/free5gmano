# Free5GMANO 專案詳細分析

## 📋 專案概述

**Free5GMANO** 是一個 **5G MANO (Management and Network Orchestration)** 管理平台，用於管理和編排 5G 網路切片。這是一個基於 Django 的 REST API 後端系統，遵循 3GPP TS 28.531、TS 28.532 Release 15 標準。

### 核心目的
- 創建和管理 **Network Slice Subnet Instances (NSSIs)** - 網路切片子網實例
- 與 NFV-MANO 編排器（如 Kube5gnfvo、OpenStack Tacker）集成
- 提供故障管理和監控功能
- 支持網路切片模板的上傳和管理

---

## 🏗️ 專案架構

```
free5gmano/
├── free5gmano/          # Django 主專案設定
├── nssmf/               # 網路切片子網管理功能 (Network Slice Subnet Management Function)
├── moi/                 # 管理物件實例 (Management Object Instance)
├── FaultManagement/     # 故障管理模組
├── deploy/              # Kubernetes 部署配置
├── docker/              # Docker 容器配置
└── nssmf/csar/          # CSAR 打包格式範本
```

---

## 📦 主要模組詳解

### 1️⃣ **nssmf 模組** - 網路切片子網管理

#### 用途
負責網路切片子網的生命週期管理，包括模板管理、插件管理、NSSI 的分配和回收。

#### 核心檔案

**models.py** - 資料模型
```python
# ServiceMappingPluginModel - 服務映射插件
- name: 插件名稱 (如 kube5gnfvo)
- allocate_nssi: 分配 NSSI 的腳本路徑
- deallocate_nssi: 回收 NSSI 的腳本路徑
- pluginFile: 插件檔案
- nm_host: 網路管理主機
- nfvo_host: NFV 編排器主機
- subscription_host: 訂閱服務主機

# GenericTemplate - 通用模板
- templateId: UUID 主鍵
- templateType: 模板類型 (VNF/NSD/NRM)
- nfvoType: NFVO 類型
- templateFile: 上傳的模板檔案
- operationStatus: 操作狀態 (CREATED/UPDATED/UPLOAD)

# Content - 模板內容
- contentId: UUID
- templateId: 關聯的模板
- type: 內容類型
- tosca_definitions_version: TOSCA 版本
- topology_template: 拓撲模板 (JSON/YAML)

# SliceTemplate - 切片模板
- templateId: UUID
- nfvoType: 多個 NFVO 類型
- genericTemplates: 多個通用模板 (VNF + NSD + NRM)
- instanceId: 多個 NSSI 實例
```

**views.py** - API 視圖
```python
# CustomAuthToken - 認證
- 用戶登入並獲取 API Token

# GenericTemplateView - 通用模板管理
- list(): 查詢模板列表
- create(): 創建新模板
- retrieve(): 獲取單個模板
- upload(): 上傳模板檔案
- check(): 檢查模板是否已存在

# SliceTemplateView - 切片模板管理
- 管理 VNF + NSD + NRM 的組合模板
```

**serializers.py** - 序列化器
```python
# GenericTemplateSerializer
- 序列化模板資訊，包括關聯的內容

# GenericTemplateFileSerializer
- 專門用於檔案上傳

# SliceTemplateRelationSerializer
- 返回模板之間的關係
- 將 genericTemplates 按 templateType 分組
```

#### 工作流程
1. 用戶上傳 VNF、NSD、NRM 模板
2. 系統檢查模板是否重複
3. 將三個模板組合成 SliceTemplate
4. 通過插件調用 NFVO 分配 NSSI
5. 返回 NSSI ID 給用戶

---

### 2️⃣ **moi 模組** - 管理物件實例

#### 用途
定義和管理 5G 網路的管理物件，包括網路切片、AMF、SMF、UPF 等功能實體。

#### 核心檔案

**models.py** - 資料模型

```python
# SST (Slice/Service Type) - 切片服務類型
- value: 整數值 (主鍵)
- type: 類型名稱
- characteristics: 特性描述

# SNSSAIList - S-NSSAI 列表 (Single Network Slice Selection Assistance Information)
- id: UUID
- sST: 多個 SST
- sD: Slice Differentiator (切片區分符)

# PLMNIdList - PLMN ID 列表 (Public Land Mobile Network)
- pLMNId: PLMN ID (主鍵)
- mcc: 行動國家代碼
- mnc: 行動網路代碼
- MobileNetworkOperator: 運營商名稱

# PerfRequirements - 性能要求
- scenario: 場景
- experiencedDataRateDL: 下行數據速率
- experiencedDataRateUL: 上行數據速率
- areaTrafficCapacityDL/UL: 區域流量容量
- overallUserDensity: 用戶密度
- ueSpeed: 用戶設備速度
- coverage: 覆蓋範圍

# SliceProfileList - 切片配置文件
- 包含 S-NSSAI、PLMN、性能要求

# NetworkSliceSubnet - 網路切片子網 (核心模型)
- nssiId: UUID (主鍵)
- mFIdList: 管理功能 ID 列表
- constituentNSSIIdList: 組成 NSSI 列表 (自關聯)
- administrativeState: 管理狀態 (LOCKED/UNLOCKED)
- operationalState: 運行狀態 (ENABLED/DISABLED)
- nsInfo: 關聯的網路服務資訊
- sliceProfileList: 切片配置

# AMFFunction - AMF 功能實體
- aMFIdentifier: UUID
- pLMNIdList: PLMN 列表
- sBIFQDN: 服務基礎設施 FQDN
- sBIServiceList: SBI 服務列表
- weightFactor: 權重因子
- sNSSAIList: S-NSSAI 列表

# SMFFunction - SMF 功能實體
- 類似 AMF，管理會話

# UPFFunction - UPF 功能實體
- 用戶平面功能

# PCFunction - 策略控制功能

# CommonNotification - 通用通知
- notificationId: UUID
- notificationType: 通知類型
- eventTime: 事件時間
- objectClass: 物件類型
- objectInstanceInfos: 物件實例資訊

# Subscription - 訂閱
- subscriptionId: UUID
- timeTick: 時間間隔
- filter: 通知過濾器
- callbackUri: 回調 URI
```

**views.py** - API 視圖

```python
# TaskThread - 後台監控線程
- 監控 MOI 物件的變化
- 支持兩種通知類型：
  1. notifyMOIDeletion: 物件刪除通知
  2. notifyMOIAttributeValueChanges: 屬性變化通知
- 通過 Kafka 發送通知到回調 URI

# ObjectManagement - 物件管理視圖
- 創建、更新、刪除 MOI 物件
- 支持訂閱機制
```

#### 工作流程
1. 用戶創建 NetworkSliceSubnet
2. 系統關聯 AMF、SMF、UPF 等功能實體
3. 用戶訂閱 MOI 變化通知
4. 系統啟動後台線程監控
5. 當物件變化時，通過 Kafka 發送通知

---

### 3️⃣ **FaultManagement 模組** - 故障管理

#### 用途
管理 5G 網路的故障和告警，包括告警的創建、確認、清除等操作。

#### 核心檔案

**models.py** - 資料模型

```python
# Header - 通知頭
- notificationId: UUID (主鍵)
- notificationType: 通知類型
- uri: 資源 URI
- eventTime: 事件時間
- systemDN: 系統 DN

# AlarmResource - 告警資源 (核心模型)
- alarmId: 告警 ID (主鍵)
- alarmType: 告警類型 (ProcessingErrorAlarm 等)
- alarmRaisedTime: 告警產生時間
- alarmChangedTime: 告警變化時間
- alarmClearedTime: 告警清除時間
- probableCause: 可能原因
- perceivedSeverity: 感知嚴重程度 (Critical/Major/Minor/Warning)
- specificProblem: 具體問題
- trendIndication: 趨勢指示 (up/down/steady)
- thresholdinfo: 閾值資訊
- stateChangeDefinition: 狀態變化定義
- monitoredAttributes: 監控屬性
- proposedRepairActions: 建議修復操作
- ackstate: 確認狀態 (acknowledged/unacknowledged)
- ackUserId: 確認用戶 ID
- ackTime: 確認時間
- comments: 評論

# SubscriptionResource - 訂閱資源
- notificationId: 通知 ID (主鍵)
- consumerReference: 消費者參考
- timeTick: 時間間隔
- filter: 過濾條件

# AlarmsCount - 告警計數
- criticalCount: 嚴重告警數
- majorCount: 主要告警數
- minorCount: 次要告警數
- warningCount: 警告數
- indeterminateCount: 不確定告警數
- clearedCount: 已清除告警數
```

**views.py** - API 視圖

```python
# FaultSupervisionView - 故障監督視圖
- list(): 檢索告警
  * 從 NFVO 獲取告警
  * 轉換告警格式
  * 存儲到本地資料庫
- create(): 添加評論到告警
- retrieve(): 獲取告警計數
- update(): 清除/確認/取消確認告警

# FaultSupervisionSubscriptionsView - 故障訂閱視圖
- list(): 列出訂閱
- create(): 創建新訂閱
- destroy(): 刪除訂閱
```

#### 工作流程
1. NFVO 產生告警
2. 系統定期從 NFVO 拉取告警
3. 轉換告警格式並存儲
4. 用戶可以查詢、確認、清除告警
5. 支持訂閱機制，告警變化時通知用戶

---

## 🔌 外部集成

### 與 NFVO 的集成
```
free5gmano (NM)
    ↓
Service Mapping Plugin (插件)
    ↓
NFVO (Kube5gnfvo / OpenStack Tacker)
    ↓
Kubernetes / OpenStack
```

### 與 Kafka 的集成
- 用於發送 MOI 變化通知
- 用於發送故障告警通知

---

## 🗄️ 資料庫結構

### 主要表關係
```
GenericTemplate (1) ──→ (N) Content
                    ↓
            SliceTemplate (N) ──→ (N) GenericTemplate
                    ↓
            NetworkSliceSubnet (NSSI)
                    ↓
            AMFFunction / SMFFunction / UPFFunction
                    ↓
            AlarmResource (故障告警)
```

---

## 🔑 關鍵概念

### NSSI (Network Slice Subnet Instance)
- 網路切片子網實例
- 由 VNF、NSD、NRM 模板組合而成
- 可以分配、修改、回收

### MOI (Management Object Instance)
- 管理物件實例
- 代表 5G 網路中的各種功能實體
- 支持訂閱和通知機制

### 告警管理
- 支持多個嚴重程度級別
- 支持確認/清除操作
- 支持評論和追蹤

---

## 📡 API 端點概覽

### NSSMF 模組
- `POST /api/templates/` - 創建模板
- `GET /api/templates/` - 列出模板
- `POST /api/templates/{id}/upload/` - 上傳模板檔案
- `POST /api/slice-templates/` - 創建切片模板
- `POST /api/plugins/` - 註冊插件

### MOI 模組
- `POST /api/network-slice-subnets/` - 創建 NSSI
- `GET /api/network-slice-subnets/` - 列出 NSSI
- `POST /api/subscriptions/` - 創建訂閱

### 故障管理
- `GET /api/alarms/` - 列出告警
- `PATCH /api/alarms/{id}/` - 更新告警狀態
- `POST /api/alarm-subscriptions/` - 創建告警訂閱

---

## 🔐 認證與授權

- 使用 Django REST Framework Token 認證
- 通過 `/api-token-auth/` 端點獲取 Token
- 所有 API 請求需要在 Header 中包含 Token

---

## 📊 部署架構

### Kubernetes 部署
- 支持多個 free5gc 版本 (3.0.4, 3.0.5, 3.0.6, 3.2.1, 3.3.0, 3.4.5)
- 支持 CNI 網路插件
- 支持 Istio 服務網格
- 支持 UERANSIM 模擬器

### Docker 容器
- 控制平面容器 (AMF, SMF, UDR 等)
- 用戶平面容器 (UPF)
- Web UI 容器

---

## 🛠️ 技術棧

- **框架**: Django 2.2.8
- **API**: Django REST Framework 3.10.1
- **資料庫**: MySQL
- **文檔**: drf-yasg (Swagger)
- **跨域**: django-cors-headers
- **配置**: PyYAML
- **驗證**: jsonschema

---

## 📝 總結

Free5GMANO 是一個完整的 5G 網路管理平台，提供：

1. **模板管理** - 上傳和管理 VNF、NSD、NRM 模板
2. **NSSI 生命週期管理** - 分配、修改、回收網路切片
3. **MOI 管理** - 管理 5G 網路功能實體
4. **故障管理** - 告警監控和管理
5. **插件架構** - 支持多個 NFVO 編排器
6. **通知機制** - 支持訂閱和 Kafka 通知

系統設計遵循 3GPP 標準，與開源 NFV 編排器集成，可用於生產環境的 5G 網路管理。
