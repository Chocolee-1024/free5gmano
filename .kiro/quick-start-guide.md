# Free5GMANO 快速入門指南

## 🚀 5 分鐘快速了解

### 專案是什麼？
Free5GMANO 是一個 **5G 網路管理和編排平台**，用於管理和部署 5G 網路切片。

### 核心功能
1. **模板管理** - 上傳和管理 VNF、NSD、NRM 模板
2. **NSSI 管理** - 創建、修改、刪除網路切片
3. **MOI 管理** - 管理 5G 網路功能實體
4. **故障管理** - 監控和管理告警

### 技術棧
- **後端**: Django 4.2.8 + Django REST Framework
- **資料庫**: MySQL
- **消息隊列**: Kafka
- **編排器**: Kube5gnfvo / OpenStack Tacker

---

## 📂 文件結構速查

```
free5gmano/
├── free5gmano/          ← Django 主配置
│   ├── settings.py      ← 全局設定
│   ├── urls.py          ← URL 路由
│   └── wsgi.py          ← WSGI 應用
│
├── nssmf/               ← 網路切片管理 ⭐ 從這裡開始
│   ├── models.py        ← 資料模型
│   ├── views.py         ← API 視圖
│   ├── serializers.py   ← 序列化器
│   └── urls.py          ← URL 路由
│
├── moi/                 ← 管理物件實例
│   ├── models.py        ← 資料模型
│   ├── views.py         ← API 視圖
│   └── urls.py          ← URL 路由
│
├── FaultManagement/     ← 故障管理
│   ├── models.py        ← 資料模型
│   ├── views.py         ← API 視圖
│   └── urls.py          ← URL 路由
│
└── requirements.txt     ← 依賴列表
```

---

## 🎯 推薦學習路徑

### 第 1 天：基礎理解
```
1. 閱讀 README.md (5 分鐘)
   ↓
2. 閱讀 .kiro/project-analysis.md (15 分鐘)
   ↓
3. 理解 Django 基礎 (1 小時)
   ↓
4. 設置本地開發環境 (30 分鐘)
```

### 第 2-3 天：代碼理解
```
1. 閱讀 nssmf/models.py (30 分鐘)
   ↓
2. 閱讀 nssmf/views.py (30 分鐘)
   ↓
3. 在 Django shell 中測試 (30 分鐘)
   ↓
4. 使用 Postman 測試 API (30 分鐘)
```

### 第 4-5 天：深入理解
```
1. 閱讀 moi/models.py 和 views.py (1 小時)
   ↓
2. 閱讀 FaultManagement/models.py 和 views.py (1 小時)
   ↓
3. 理解完整的業務流程 (1 小時)
   ↓
4. 進行小型代碼修改 (1 小時)
```

---

## 💻 本地開發環境設置

### 前置要求
- Python 3.8+
- MySQL 5.7+
- Git

### 快速設置

```bash
# 1. 克隆專案
git clone https://github.com/free5gmano/free5gmano.git
cd free5gmano

# 2. 創建虛擬環境
python -m venv venv
source venv/bin/activate

# 3. 安裝依賴
pip install -r requirements.txt

# 4. 配置環境變數
export FREE5GMANO_MYSQL_USER=root
export FREE5GMANO_MYSQL_PASSWORD=password
export FREE5GMANO_MYSQL_HOST=127.0.0.1
export FREE5GMANO_MYSQL_PORT=3306
export FREE5GMANO_DB_NAME=free5gmano

# 5. 創建資料庫
mysql -u root -p
CREATE DATABASE free5gmano;
EXIT;

# 6. 運行遷移
python manage.py makemigrations
python manage.py migrate

# 7. 創建超級用戶
python manage.py createsuperuser

# 8. 運行開發服務器
python manage.py runserver
```

### 驗證安裝
```bash
# 訪問以下 URL
http://localhost:8000/swagger/  # Swagger 文檔
http://localhost:8000/admin/    # Django 管理後台
```

---

## 🔍 代碼閱讀指南

### 如何閱讀 Django 代碼

#### 1. 從 URL 開始
```python
# nssmf/urls.py
router.register(r'ObjectManagement/GenericTemplate', GenericTemplateView,
                basename='GenericTemplate')
```
這告訴你：
- URL 路徑是 `/ObjectManagement/GenericTemplate`
- 使用 `GenericTemplateView` 視圖
- 支持 CRUD 操作

#### 2. 查看視圖
```python
# nssmf/views.py
class GenericTemplateView(MultipleSerializerViewSet):
    queryset = GenericTemplate.objects.all()
    serializer_class = GenericTemplateSerializer
    
    def list(self, request, *args, **kwargs):
        """查詢模板列表"""
        return super().list(request, *args, **kwargs)
```
這告訴你：
- 使用 `GenericTemplate` 模型
- 使用 `GenericTemplateSerializer` 序列化器
- `list()` 方法處理 GET 請求

#### 3. 查看模型
```python
# nssmf/models.py
class GenericTemplate(models.Model):
    templateId = models.UUIDField(primary_key=True, default=uuid.uuid4)
    name = models.TextField(null=True, blank=True)
    templateType = models.CharField(max_length=255, choices=TemplateType)
    nfvoType = models.CharField(max_length=255)
```
這告訴你：
- 模型有哪些字段
- 字段的類型和約束
- 主鍵是 `templateId`

#### 4. 查看序列化器
```python
# nssmf/serializers.py
class GenericTemplateSerializer(serializers.ModelSerializer):
    class Meta:
        model = GenericTemplate
        fields = ['templateId', 'name', 'nfvoType', 'templateType', ...]
```
這告訴你：
- 序列化器對應的模型
- 序列化的字段
- 序列化的邏輯

---

## 🧪 測試代碼

### 在 Django Shell 中測試

```bash
# 進入 Django shell
python manage.py shell
```

```python
# 導入模型
from nssmf.models import GenericTemplate

# 創建模板
template = GenericTemplate.objects.create(
    name="test-template",
    templateType="VNF",
    nfvoType="kube5gnfvo"
)
print(f"Created template: {template.templateId}")

# 查詢模板
templates = GenericTemplate.objects.all()
for t in templates:
    print(f"Template: {t.name} ({t.templateType})")

# 更新模板
template.name = "updated-template"
template.save()

# 刪除模板
template.delete()
```

### 使用 Postman 測試 API

#### 1. 獲取 API Token
```
POST http://localhost:8000/api-token-auth/
Body (JSON):
{
    "username": "admin",
    "password": "password"
}
```

#### 2. 查詢模板列表
```
GET http://localhost:8000/ObjectManagement/GenericTemplate/
Headers:
Authorization: Token <your-token>
```

#### 3. 創建模板
```
POST http://localhost:8000/ObjectManagement/GenericTemplate/
Headers:
Authorization: Token <your-token>
Content-Type: application/json

Body (JSON):
{
    "name": "test-template",
    "templateType": "VNF",
    "nfvoType": "kube5gnfvo"
}
```

---

## 📊 核心概念速查

### NSSI (Network Slice Subnet Instance)
- **是什麼**: 網路切片子網實例
- **由什麼組成**: VNF 模板 + NSD 模板 + NRM 模板
- **生命週期**: 分配 → 修改 → 回收
- **存儲位置**: `moi.models.NetworkSliceSubnet`

### MOI (Management Object Instance)
- **是什麼**: 管理物件實例
- **包括**: AMF、SMF、UPF、PCF 等功能實體
- **功能**: 代表 5G 網路中的各種功能
- **存儲位置**: `moi.models.*Function`

### 告警 (Alarm)
- **是什麼**: 系統故障或異常的通知
- **來源**: NFVO 系統
- **狀態**: 未確認 → 已確認 → 已清除
- **存儲位置**: `FaultManagement.models.AlarmResource`

---

## 🔧 常見修改場景

### 場景 1: 添加新的模板字段

```python
# 1. 修改模型
# nssmf/models.py
class GenericTemplate(models.Model):
    # ... 現有字段 ...
    version = models.CharField(max_length=50, null=True, blank=True)  # 新字段

# 2. 創建遷移
python manage.py makemigrations nssmf

# 3. 應用遷移
python manage.py migrate

# 4. 更新序列化器
# nssmf/serializers.py
class GenericTemplateSerializer(serializers.ModelSerializer):
    class Meta:
        model = GenericTemplate
        fields = [..., 'version']  # 添加新字段
```

### 場景 2: 添加新的 API 端點

```python
# 1. 在視圖中添加方法
# nssmf/views.py
class GenericTemplateView(MultipleSerializerViewSet):
    @action(detail=True, methods=['post'])
    def publish(self, request, pk=None):
        """發佈模板"""
        template = self.get_object()
        template.status = 'published'
        template.save()
        return Response({'status': 'published'})

# 2. URL 會自動生成
# POST /ObjectManagement/GenericTemplate/{id}/publish/
```

### 場景 3: 添加新的驗證邏輯

```python
# 1. 在序列化器中添加驗證
# nssmf/serializers.py
class GenericTemplateSerializer(serializers.ModelSerializer):
    def validate_name(self, value):
        if len(value) < 3:
            raise serializers.ValidationError("Name must be at least 3 characters")
        return value
    
    def validate(self, data):
        if data['templateType'] == 'VNF' and not data.get('nfvoType'):
            raise serializers.ValidationError("nfvoType is required for VNF")
        return data
```

---

## 🐛 調試技巧

### 1. 使用日誌記錄
```python
import logging
logger = logging.getLogger(__name__)

logger.debug("Debug message")
logger.info("Info message")
logger.warning("Warning message")
logger.error("Error message")
```

### 2. 使用 pdb 調試
```python
import pdb; pdb.set_trace()  # 在這裡暫停執行
```

### 3. 使用 Django Debug Toolbar
```python
# settings.py
INSTALLED_APPS = [
    'debug_toolbar',
    ...
]

MIDDLEWARE = [
    'debug_toolbar.middleware.DebugToolbarMiddleware',
    ...
]

INTERNAL_IPS = ['127.0.0.1']
```

### 4. 查看 SQL 查詢
```python
from django.db import connection
from django.test.utils import CaptureQueriesContext

with CaptureQueriesContext(connection) as context:
    # 執行查詢
    templates = GenericTemplate.objects.all()
    
# 查看 SQL 查詢
for query in context.captured_queries:
    print(query['sql'])
```

---

## 📚 推薦資源

### 官方文檔
- [Django 4.2 文檔](https://docs.djangoproject.com/en/4.2/)
- [Django REST Framework 文檔](https://www.django-rest-framework.org/)
- [3GPP TS 28.531](https://www.3gpp.org/)

### 在線教程
- [Django for Beginners](https://djangoforbeginners.com/)
- [Real Python Django Tutorials](https://realpython.com/tutorials/django/)
- [YouTube: Django REST Framework](https://www.youtube.com/results?search_query=django+rest+framework)

### 書籍
- "Two Scoops of Django"
- "Django for APIs"
- "Mastering Django"

---

## ✅ 檢查清單

完成以下步驟後，你就準備好開始修改代碼了：

- [ ] 理解 5G 網路管理基礎
- [ ] 理解 Django 框架基礎
- [ ] 設置本地開發環境
- [ ] 能夠運行開發服務器
- [ ] 能夠訪問 Swagger 文檔
- [ ] 能夠在 Django shell 中執行代碼
- [ ] 能夠使用 Postman 測試 API
- [ ] 理解 NSSMF 模塊的代碼
- [ ] 理解 MOI 模塊的代碼
- [ ] 理解 FaultManagement 模塊的代碼
- [ ] 能夠進行小型代碼修改
- [ ] 能夠添加新的 API 端點

---

## 🎓 下一步

1. **完成本指南** - 5-10 分鐘
2. **設置開發環境** - 30 分鐘
3. **閱讀 project-analysis.md** - 15 分鐘
4. **閱讀 learning-roadmap.md** - 10 分鐘
5. **開始代碼閱讀** - 按照 learning-roadmap.md 進行

**祝你學習愉快！** 🚀
