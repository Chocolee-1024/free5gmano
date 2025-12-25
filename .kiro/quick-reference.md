# Free5GMANO 快速參考卡

## 🎯 專案概述

| 項目 | 說明 |
|------|------|
| **名稱** | Free5GMANO |
| **功能** | 5G 網路管理和編排平台 |
| **框架** | Django 4.2.8 + DRF |
| **資料庫** | MySQL |
| **消息隊列** | Kafka |
| **編排器** | Kube5gnfvo / OpenStack Tacker |

---

## 📂 核心模塊

### NSSMF (網路切片子網管理)
```
功能: 模板管理、NSSI 分配/回收
主要類:
  - GenericTemplate: 通用模板 (VNF/NSD/NRM)
  - SliceTemplate: 切片模板 (組合模板)
  - ServiceMappingPluginModel: 插件配置
API 端點:
  - GET/POST /ObjectManagement/GenericTemplate/
  - GET/POST /ObjectManagement/SliceTemplate/
  - POST /ObjectManagement/NSS/SliceProfiles/
```

### MOI (管理物件實例)
```
功能: MOI 對象管理、訂閱、通知
主要類:
  - NetworkSliceSubnet: NSSI
  - AMFFunction: AMF 功能
  - SMFFunction: SMF 功能
  - UPFFunction: UPF 功能
  - Subscription: 訂閱
API 端點:
  - PUT /ObjectManagement/{className}/{id}/
  - GET /ObjectManagement/{className}/{id}/
  - PATCH /ObjectManagement/{className}/{id}/
  - DELETE /ObjectManagement/{className}/{id}/
```

### FaultManagement (故障管理)
```
功能: 告警管理、告警訂閱
主要類:
  - AlarmResource: 告警
  - Header: 通知頭
  - SubscriptionResource: 訂閱
API 端點:
  - GET/POST /alarms/
  - PATCH /alarms/{id}/
  - GET/POST /subscriptions/
  - DELETE /subscriptions/{id}/
```

---

## 🔑 關鍵概念

### NSSI (Network Slice Subnet Instance)
- **定義**: 網路切片子網實例
- **組成**: VNF 模板 + NSD 模板 + NRM 模板
- **狀態**: 分配 → 修改 → 回收
- **存儲**: `moi.models.NetworkSliceSubnet`

### MOI (Management Object Instance)
- **定義**: 管理物件實例
- **類型**: AMF、SMF、UPF、PCF 等
- **功能**: 代表 5G 網路功能
- **特性**: 支持訂閱和通知

### 告警 (Alarm)
- **來源**: NFVO 系統
- **狀態**: 未確認 → 已確認 → 已清除
- **嚴重程度**: Critical、Major、Minor、Warning
- **存儲**: `FaultManagement.models.AlarmResource`

---

## 💻 常用命令

### 開發環境
```bash
# 創建虛擬環境
python -m venv venv
source venv/bin/activate

# 安裝依賴
pip install -r requirements.txt

# 運行遷移
python manage.py makemigrations
python manage.py migrate

# 創建超級用戶
python manage.py createsuperuser

# 運行開發服務器
python manage.py runserver

# 進入 Django shell
python manage.py shell

# 運行測試
python manage.py test
```

### 資料庫
```bash
# 查看 SQL 遷移
python manage.py sqlmigrate nssmf 0001

# 檢查資料庫
python manage.py inspectdb

# 備份資料庫
mysqldump -u root -p free5gmano > backup.sql

# 恢復資料庫
mysql -u root -p free5gmano < backup.sql
```

---

## 🧪 測試代碼

### Django Shell 測試
```python
# 導入模型
from nssmf.models import GenericTemplate
from moi.models import NetworkSliceSubnet
from FaultManagement.models import AlarmResource

# 創建對象
template = GenericTemplate.objects.create(
    name="test",
    templateType="VNF",
    nfvoType="kube5gnfvo"
)

# 查詢對象
templates = GenericTemplate.objects.all()
template = GenericTemplate.objects.get(templateId="...")

# 更新對象
template.name = "updated"
template.save()

# 刪除對象
template.delete()
```

### Postman 測試
```
1. 獲取 Token
   POST /api-token-auth/
   Body: {"username": "admin", "password": "password"}

2. 查詢列表
   GET /ObjectManagement/GenericTemplate/
   Header: Authorization: Token <token>

3. 創建對象
   POST /ObjectManagement/GenericTemplate/
   Header: Authorization: Token <token>
   Body: {"name": "test", "templateType": "VNF", ...}

4. 更新對象
   PATCH /ObjectManagement/GenericTemplate/{id}/
   Header: Authorization: Token <token>
   Body: {"name": "updated"}

5. 刪除對象
   DELETE /ObjectManagement/GenericTemplate/{id}/
   Header: Authorization: Token <token>
```

---

## 📊 資料模型關係

```
GenericTemplate (1) ──→ (N) Content
                    ↓
            SliceTemplate (N) ──→ (N) GenericTemplate
                    ↓
            NetworkSliceSubnet (NSSI)
                    ↓
            AMFFunction / SMFFunction / UPFFunction
                    ↓
            AlarmResource (告警)
```

---

## 🔧 常見修改

### 添加新字段
```python
# 1. 修改 models.py
class GenericTemplate(models.Model):
    new_field = models.CharField(max_length=100)

# 2. 創建遷移
python manage.py makemigrations

# 3. 應用遷移
python manage.py migrate

# 4. 更新 serializers.py
class GenericTemplateSerializer(serializers.ModelSerializer):
    class Meta:
        fields = [..., 'new_field']
```

### 添加新 API 端點
```python
# 在 views.py 中添加方法
@action(detail=True, methods=['post'])
def custom_action(self, request, pk=None):
    obj = self.get_object()
    # 業務邏輯
    return Response({'status': 'success'})

# URL 自動生成: POST /path/{id}/custom_action/
```

### 添加驗證
```python
# 在 serializers.py 中添加驗證
def validate_name(self, value):
    if len(value) < 3:
        raise serializers.ValidationError("Too short")
    return value

def validate(self, data):
    if condition:
        raise serializers.ValidationError("Invalid")
    return data
```

---

## 🐛 調試技巧

### 使用日誌
```python
import logging
logger = logging.getLogger(__name__)

logger.debug("Debug message")
logger.info("Info message")
logger.warning("Warning message")
logger.error("Error message")
```

### 使用 pdb
```python
import pdb; pdb.set_trace()  # 暫停執行
```

### 查看 SQL 查詢
```python
from django.db import connection
from django.test.utils import CaptureQueriesContext

with CaptureQueriesContext(connection) as context:
    # 執行查詢
    pass

for query in context.captured_queries:
    print(query['sql'])
```

---

## 📚 文檔位置

| 文檔 | 位置 | 用途 |
|------|------|------|
| 專案分析 | `.kiro/project-analysis.md` | 了解專案架構 |
| 學習路線 | `.kiro/learning-roadmap.md` | 系統化學習 |
| 快速開始 | `.kiro/quick-start-guide.md` | 快速上手 |
| 代碼閱讀 | `.kiro/code-reading-guide.md` | 代碼理解 |
| 升級報告 | `.kiro/upgrade-summary.md` | 版本升級 |

---

## 🎓 學習資源

### 官方文檔
- Django: https://docs.djangoproject.com/en/4.2/
- DRF: https://www.django-rest-framework.org/
- 3GPP: https://www.3gpp.org/

### 在線教程
- Django for Beginners: https://djangoforbeginners.com/
- Real Python: https://realpython.com/tutorials/django/

### 書籍
- Two Scoops of Django
- Django for APIs
- Mastering Django

---

## ✅ 檢查清單

### 環境設置
- [ ] Python 3.8+ 已安裝
- [ ] MySQL 已安裝
- [ ] 虛擬環境已創建
- [ ] 依賴已安裝
- [ ] 資料庫已配置
- [ ] 遷移已運行
- [ ] 開發服務器可運行

### 代碼理解
- [ ] 理解 NSSMF 模塊
- [ ] 理解 MOI 模塊
- [ ] 理解 FaultManagement 模塊
- [ ] 理解 API 流程
- [ ] 理解資料模型
- [ ] 理解業務邏輯

### 實踐能力
- [ ] 能在 Django shell 中測試
- [ ] 能使用 Postman 測試 API
- [ ] 能添加新字段
- [ ] 能添加新 API 端點
- [ ] 能進行代碼修改
- [ ] 能編寫測試

---

**最後更新**: 2024 年 12 月 25 日
**版本**: Django 4.2.8
