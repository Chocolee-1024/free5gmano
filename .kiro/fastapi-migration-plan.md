# Free5GMANO FastAPI 遷移方案

## 📋 概述

本文檔提供了將 Free5GMANO 從 Django 遷移到 FastAPI 的完整方案。

---

## 🎯 為什麼選擇 FastAPI？

### FastAPI 的優勢

| 特性 | Django | FastAPI |
|------|--------|---------|
| 性能 | 中等 | 非常高 |
| 開發速度 | 快 | 非常快 |
| 類型提示 | 可選 | 必需 |
| 自動文檔 | 需要配置 | 自動生成 |
| 異步支持 | 有限 | 完全支持 |
| 學習曲線 | 陡峭 | 平緩 |
| 社區 | 非常大 | 快速增長 |
| 生產就緒 | ✅ | ✅ |

### FastAPI 適合 Free5GMANO 的原因

1. **高性能** - 適合 5G 網路管理的實時性要求
2. **異步支持** - 適合 Kafka 消息處理
3. **自動文檔** - 自動生成 OpenAPI/Swagger 文檔
4. **類型安全** - 更好的代碼質量
5. **現代化** - 使用最新的 Python 特性

---

## 📊 遷移複雜度評估

### 代碼量統計

```
Django 代碼:
  - models.py: ~300 行
  - views.py: ~800 行
  - serializers.py: ~400 行
  - urls.py: ~50 行
  - 總計: ~1550 行

FastAPI 代碼:
  - models.py: ~300 行 (相同)
  - routers.py: ~600 行 (替代 views.py)
  - schemas.py: ~300 行 (替代 serializers.py)
  - main.py: ~100 行 (替代 urls.py)
  - 總計: ~1300 行 (減少 16%)
```

### 遷移難度

| 模塊 | 難度 | 工作量 | 風險 |
|------|------|--------|------|
| NSSMF | 中 | 3-4 天 | 低 |
| MOI | 高 | 4-5 天 | 中 |
| FaultManagement | 中 | 3-4 天 | 低 |
| 資料庫 | 低 | 1-2 天 | 低 |
| 測試 | 中 | 2-3 天 | 中 |
| **總計** | **中** | **13-18 天** | **低-中** |

---

## 🔄 遷移步驟

### 第 1 階段：準備 (1-2 天)

#### 1.1 安裝 FastAPI 依賴

```bash
pip install fastapi uvicorn sqlalchemy pydantic python-multipart
pip install sqlalchemy-utils  # 用於 UUID 等
pip install aiokafka  # 用於異步 Kafka
```

#### 1.2 創建新的項目結構

```
free5gmano-fastapi/
├── app/
│   ├── __init__.py
│   ├── main.py                 # FastAPI 應用
│   ├── config.py               # 配置
│   ├── database.py             # 資料庫連接
│   ├── models/                 # SQLAlchemy 模型
│   │   ├── __init__.py
│   │   ├── nssmf.py
│   │   ├── moi.py
│   │   └── fault_management.py
│   ├── schemas/                # Pydantic 模型
│   │   ├── __init__.py
│   │   ├── nssmf.py
│   │   ├── moi.py
│   │   └── fault_management.py
│   ├── routers/                # API 路由
│   │   ├── __init__.py
│   │   ├── nssmf.py
│   │   ├── moi.py
│   │   └── fault_management.py
│   ├── services/               # 業務邏輯
│   │   ├── __init__.py
│   │   ├── nssmf_service.py
│   │   ├── moi_service.py
│   │   └── fault_management_service.py
│   └── utils/                  # 工具函數
│       ├── __init__.py
│       ├── kafka_producer.py
│       └── logger.py
├── tests/
├── requirements.txt
└── main.py                     # 啟動文件
```

#### 1.3 設置資料庫連接

```python
# app/database.py
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import os

DATABASE_URL = os.getenv(
    "DATABASE_URL",
    "mysql+pymysql://root:password@127.0.0.1:3306/free5gmano"
)

engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

---

### 第 2 階段：遷移模型 (1-2 天)

#### 2.1 遷移 NSSMF 模型

**Django 模型**:
```python
class GenericTemplate(models.Model):
    templateId = models.UUIDField(primary_key=True, default=uuid.uuid4)
    name = models.TextField(null=True, blank=True)
    templateType = models.CharField(max_length=255, choices=TemplateType)
```

**FastAPI 模型**:
```python
# app/models/nssmf.py
from sqlalchemy import Column, String, Text, Enum
from sqlalchemy.dialects.mysql import CHAR
from uuid import uuid4
from app.database import Base

class GenericTemplate(Base):
    __tablename__ = "nssmf_generictemplate"
    
    templateId = Column(CHAR(36), primary_key=True, default=lambda: str(uuid4()))
    name = Column(Text, nullable=True)
    templateType = Column(String(255), nullable=False)
    nfvoType = Column(String(255), nullable=False)
    operationStatus = Column(String(255), default="CREATED")
    description = Column(Text, nullable=True)
    operationTime = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
```

#### 2.2 遷移 MOI 模型

```python
# app/models/moi.py
from sqlalchemy import Column, String, Text, DateTime, ForeignKey
from sqlalchemy.orm import relationship
from app.database import Base

class NetworkSliceSubnet(Base):
    __tablename__ = "moi_networkslicesubnet"
    
    nssiId = Column(CHAR(36), primary_key=True, default=lambda: str(uuid4()))
    administrativeState = Column(String(255), default="LOCKED")
    operationalState = Column(String(255), default="ENABLED")
    nsInfo_id = Column(CHAR(36), ForeignKey("moi_nsinfo.id"))
    
    nsInfo = relationship("NsInfo", back_populates="networkSliceSubnets")
```

#### 2.3 遷移 FaultManagement 模型

```python
# app/models/fault_management.py
from sqlalchemy import Column, String, Text, DateTime
from app.database import Base

class AlarmResource(Base):
    __tablename__ = "faultmanagement_alarmresource"
    
    alarmId = Column(String(255), primary_key=True)
    alarmType = Column(String(255), nullable=True)
    perceivedSeverity = Column(String(255), nullable=True)
    probableCause = Column(String(255), nullable=True)
    specificProblem = Column(Text, nullable=True)
    ackstate = Column(String(255), nullable=True)
    alarmRaisedTime = Column(String(255), nullable=True)
```

---

### 第 3 階段：遷移 Schemas (1 天)

#### 3.1 創建 Pydantic Schemas

**Django Serializer**:
```python
class GenericTemplateSerializer(serializers.ModelSerializer):
    class Meta:
        model = GenericTemplate
        fields = ['templateId', 'name', 'nfvoType', 'templateType', ...]
```

**FastAPI Schema**:
```python
# app/schemas/nssmf.py
from pydantic import BaseModel
from typing import Optional
from datetime import datetime

class GenericTemplateBase(BaseModel):
    name: Optional[str] = None
    templateType: str
    nfvoType: str
    description: Optional[str] = None

class GenericTemplateCreate(GenericTemplateBase):
    pass

class GenericTemplateUpdate(BaseModel):
    name: Optional[str] = None
    description: Optional[str] = None

class GenericTemplate(GenericTemplateBase):
    templateId: str
    operationStatus: str
    operationTime: datetime
    
    class Config:
        from_attributes = True
```

---

### 第 4 階段：遷移 Views 到 Routers (3-4 天)

#### 4.1 遷移 NSSMF Views

**Django View**:
```python
class GenericTemplateView(MultipleSerializerViewSet):
    queryset = GenericTemplate.objects.all()
    
    def list(self, request):
        templates = GenericTemplate.objects.all()
        serializer = GenericTemplateSerializer(templates, many=True)
        return Response(serializer.data)
```

**FastAPI Router**:
```python
# app/routers/nssmf.py
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
from app.database import get_db
from app.models.nssmf import GenericTemplate
from app.schemas.nssmf import GenericTemplate as GenericTemplateSchema
from app.services.nssmf_service import NSSmfService

router = APIRouter(prefix="/ObjectManagement", tags=["NSSMF"])
service = NSSmfService()

@router.get("/GenericTemplate/")
async def list_templates(db: Session = Depends(get_db)):
    """列出所有模板"""
    templates = db.query(GenericTemplate).all()
    return templates

@router.post("/GenericTemplate/")
async def create_template(
    template: GenericTemplateSchema,
    db: Session = Depends(get_db)
):
    """創建新模板"""
    db_template = GenericTemplate(**template.dict())
    db.add(db_template)
    db.commit()
    db.refresh(db_template)
    return db_template

@router.get("/GenericTemplate/{template_id}")
async def get_template(
    template_id: str,
    db: Session = Depends(get_db)
):
    """獲取單個模板"""
    template = db.query(GenericTemplate).filter(
        GenericTemplate.templateId == template_id
    ).first()
    if not template:
        raise HTTPException(status_code=404, detail="Template not found")
    return template

@router.patch("/GenericTemplate/{template_id}")
async def update_template(
    template_id: str,
    template: GenericTemplateUpdate,
    db: Session = Depends(get_db)
):
    """更新模板"""
    db_template = db.query(GenericTemplate).filter(
        GenericTemplate.templateId == template_id
    ).first()
    if not db_template:
        raise HTTPException(status_code=404, detail="Template not found")
    
    for key, value in template.dict(exclude_unset=True).items():
        setattr(db_template, key, value)
    
    db.commit()
    db.refresh(db_template)
    return db_template

@router.delete("/GenericTemplate/{template_id}")
async def delete_template(
    template_id: str,
    db: Session = Depends(get_db)
):
    """刪除模板"""
    db_template = db.query(GenericTemplate).filter(
        GenericTemplate.templateId == template_id
    ).first()
    if not db_template:
        raise HTTPException(status_code=404, detail="Template not found")
    
    db.delete(db_template)
    db.commit()
    return {"message": "Template deleted"}
```

#### 4.2 遷移 MOI Views

```python
# app/routers/moi.py
from fastapi import APIRouter, Depends, HTTPException, Query
from sqlalchemy.orm import Session
from app.database import get_db
from app.models.moi import NetworkSliceSubnet
from app.schemas.moi import NetworkSliceSubnet as NSSubnetSchema

router = APIRouter(prefix="/ObjectManagement", tags=["MOI"])

@router.put("/{class_name}/{object_id}/")
async def create_moi(
    class_name: str,
    object_id: str,
    data: dict,
    db: Session = Depends(get_db)
):
    """創建 MOI 物件"""
    # 根據 class_name 動態創建對象
    model_class = get_model_class(class_name)
    if not model_class:
        raise HTTPException(status_code=400, detail="Invalid class name")
    
    db_object = model_class(**data)
    db.add(db_object)
    db.commit()
    db.refresh(db_object)
    return db_object

@router.get("/{class_name}/{object_id}/")
async def get_moi(
    class_name: str,
    object_id: str,
    scope: str = Query("BASE_ONLY"),
    db: Session = Depends(get_db)
):
    """獲取 MOI 物件"""
    model_class = get_model_class(class_name)
    if not model_class:
        raise HTTPException(status_code=400, detail="Invalid class name")
    
    obj = db.query(model_class).filter(
        model_class.id == object_id
    ).first()
    if not obj:
        raise HTTPException(status_code=404, detail="Object not found")
    
    return obj

@router.patch("/{class_name}/{object_id}/")
async def modify_moi(
    class_name: str,
    object_id: str,
    modifications: dict,
    db: Session = Depends(get_db)
):
    """修改 MOI 物件"""
    model_class = get_model_class(class_name)
    if not model_class:
        raise HTTPException(status_code=400, detail="Invalid class name")
    
    obj = db.query(model_class).filter(
        model_class.id == object_id
    ).first()
    if not obj:
        raise HTTPException(status_code=404, detail="Object not found")
    
    for key, value in modifications.items():
        setattr(obj, key, value)
    
    db.commit()
    db.refresh(obj)
    return obj

@router.delete("/{class_name}/{object_id}/")
async def delete_moi(
    class_name: str,
    object_id: str,
    db: Session = Depends(get_db)
):
    """刪除 MOI 物件"""
    model_class = get_model_class(class_name)
    if not model_class:
        raise HTTPException(status_code=400, detail="Invalid class name")
    
    obj = db.query(model_class).filter(
        model_class.id == object_id
    ).first()
    if not obj:
        raise HTTPException(status_code=404, detail="Object not found")
    
    db.delete(obj)
    db.commit()
    return {"message": "Object deleted"}
```

#### 4.3 遷移 FaultManagement Views

```python
# app/routers/fault_management.py
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
from app.database import get_db
from app.models.fault_management import AlarmResource
from app.schemas.fault_management import AlarmResource as AlarmSchema

router = APIRouter(prefix="/alarms", tags=["Fault Management"])

@router.get("/")
async def list_alarms(db: Session = Depends(get_db)):
    """列出所有告警"""
    alarms = db.query(AlarmResource).all()
    return alarms

@router.post("/")
async def create_alarm(
    alarm: AlarmSchema,
    db: Session = Depends(get_db)
):
    """創建告警"""
    db_alarm = AlarmResource(**alarm.dict())
    db.add(db_alarm)
    db.commit()
    db.refresh(db_alarm)
    return db_alarm

@router.get("/{alarm_id}")
async def get_alarm(
    alarm_id: str,
    db: Session = Depends(get_db)
):
    """獲取單個告警"""
    alarm = db.query(AlarmResource).filter(
        AlarmResource.alarmId == alarm_id
    ).first()
    if not alarm:
        raise HTTPException(status_code=404, detail="Alarm not found")
    return alarm

@router.patch("/{alarm_id}")
async def update_alarm(
    alarm_id: str,
    alarm: AlarmSchema,
    db: Session = Depends(get_db)
):
    """更新告警"""
    db_alarm = db.query(AlarmResource).filter(
        AlarmResource.alarmId == alarm_id
    ).first()
    if not db_alarm:
        raise HTTPException(status_code=404, detail="Alarm not found")
    
    for key, value in alarm.dict(exclude_unset=True).items():
        setattr(db_alarm, key, value)
    
    db.commit()
    db.refresh(db_alarm)
    return db_alarm
```

---

### 第 5 階段：遷移業務邏輯 (2-3 天)

#### 5.1 創建 Service 層

```python
# app/services/nssmf_service.py
from sqlalchemy.orm import Session
from app.models.nssmf import GenericTemplate, SliceTemplate
from app.schemas.nssmf import GenericTemplateCreate
import logging

logger = logging.getLogger(__name__)

class NSSmfService:
    def __init__(self):
        pass
    
    def create_template(self, db: Session, template: GenericTemplateCreate):
        """創建模板"""
        logger.info(f"Creating template: {template.name}")
        db_template = GenericTemplate(**template.dict())
        db.add(db_template)
        db.commit()
        db.refresh(db_template)
        logger.info(f"Template created: {db_template.templateId}")
        return db_template
    
    def get_template(self, db: Session, template_id: str):
        """獲取模板"""
        logger.info(f"Getting template: {template_id}")
        return db.query(GenericTemplate).filter(
            GenericTemplate.templateId == template_id
        ).first()
    
    def allocate_nssi(self, db: Session, slice_template_id: str):
        """分配 NSSI"""
        logger.info(f"Allocating NSSI for template: {slice_template_id}")
        # 業務邏輯
        pass
    
    def deallocate_nssi(self, db: Session, nssi_id: str):
        """回收 NSSI"""
        logger.info(f"Deallocating NSSI: {nssi_id}")
        # 業務邏輯
        pass
```

#### 5.2 遷移 Kafka 集成

```python
# app/utils/kafka_producer.py
from aiokafka import AIOKafkaProducer
import json
import logging

logger = logging.getLogger(__name__)

class KafkaProducer:
    def __init__(self, bootstrap_servers='localhost:9092'):
        self.bootstrap_servers = bootstrap_servers
        self.producer = None
    
    async def start(self):
        """啟動 Kafka 生產者"""
        self.producer = AIOKafkaProducer(
            bootstrap_servers=self.bootstrap_servers
        )
        await self.producer.start()
        logger.info("Kafka producer started")
    
    async def stop(self):
        """停止 Kafka 生產者"""
        await self.producer.stop()
        logger.info("Kafka producer stopped")
    
    async def send_notification(self, topic: str, message: dict):
        """發送通知"""
        try:
            await self.producer.send_and_wait(
                topic,
                json.dumps(message).encode('utf-8')
            )
            logger.info(f"Notification sent to {topic}")
        except Exception as e:
            logger.error(f"Failed to send notification: {e}")
```

---

### 第 6 階段：創建主應用 (1 天)

#### 6.1 創建 FastAPI 應用

```python
# app/main.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from contextlib import asynccontextmanager
from app.routers import nssmf, moi, fault_management
from app.utils.kafka_producer import KafkaProducer
import logging

# 配置日誌
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# Kafka 生產者
kafka_producer = KafkaProducer()

@asynccontextmanager
async def lifespan(app: FastAPI):
    # 啟動
    logger.info("Starting application")
    await kafka_producer.start()
    yield
    # 關閉
    logger.info("Shutting down application")
    await kafka_producer.stop()

# 創建 FastAPI 應用
app = FastAPI(
    title="Free5GMANO API",
    description="5G Network Management and Orchestration",
    version="1.0.0",
    lifespan=lifespan
)

# 添加 CORS 中間件
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# 包含路由
app.include_router(nssmf.router)
app.include_router(moi.router)
app.include_router(fault_management.router)

@app.get("/")
async def root():
    """根端點"""
    return {"message": "Free5GMANO API"}

@app.get("/health")
async def health():
    """健康檢查"""
    return {"status": "healthy"}

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

#### 6.2 啟動文件

```python
# main.py
import uvicorn

if __name__ == "__main__":
    uvicorn.run(
        "app.main:app",
        host="0.0.0.0",
        port=8000,
        reload=True,
        log_level="info"
    )
```

---

### 第 7 階段：測試和驗證 (2-3 天)

#### 7.1 單元測試

```python
# tests/test_nssmf.py
import pytest
from fastapi.testclient import TestClient
from app.main import app
from app.database import get_db

client = TestClient(app)

def test_list_templates():
    """測試列出模板"""
    response = client.get("/ObjectManagement/GenericTemplate/")
    assert response.status_code == 200

def test_create_template():
    """測試創建模板"""
    response = client.post(
        "/ObjectManagement/GenericTemplate/",
        json={
            "name": "test",
            "templateType": "VNF",
            "nfvoType": "kube5gnfvo"
        }
    )
    assert response.status_code == 200
    assert response.json()["name"] == "test"

def test_get_template():
    """測試獲取模板"""
    # 先創建一個模板
    create_response = client.post(
        "/ObjectManagement/GenericTemplate/",
        json={
            "name": "test",
            "templateType": "VNF",
            "nfvoType": "kube5gnfvo"
        }
    )
    template_id = create_response.json()["templateId"]
    
    # 然後獲取它
    response = client.get(f"/ObjectManagement/GenericTemplate/{template_id}")
    assert response.status_code == 200
    assert response.json()["templateId"] == template_id
```

#### 7.2 集成測試

```python
# tests/test_integration.py
import pytest
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

def test_full_workflow():
    """測試完整工作流"""
    # 1. 創建模板
    template_response = client.post(
        "/ObjectManagement/GenericTemplate/",
        json={
            "name": "test",
            "templateType": "VNF",
            "nfvoType": "kube5gnfvo"
        }
    )
    assert template_response.status_code == 200
    template_id = template_response.json()["templateId"]
    
    # 2. 獲取模板
    get_response = client.get(f"/ObjectManagement/GenericTemplate/{template_id}")
    assert get_response.status_code == 200
    
    # 3. 更新模板
    update_response = client.patch(
        f"/ObjectManagement/GenericTemplate/{template_id}",
        json={"name": "updated"}
    )
    assert update_response.status_code == 200
    
    # 4. 刪除模板
    delete_response = client.delete(f"/ObjectManagement/GenericTemplate/{template_id}")
    assert delete_response.status_code == 200
```

---

## 📋 遷移檢查清單

### 準備階段
- [ ] 安裝 FastAPI 依賴
- [ ] 創建新的項目結構
- [ ] 設置資料庫連接

### 模型遷移
- [ ] 遷移 NSSMF 模型
- [ ] 遷移 MOI 模型
- [ ] 遷移 FaultManagement 模型
- [ ] 驗證模型

### Schema 遷移
- [ ] 創建 NSSMF Schemas
- [ ] 創建 MOI Schemas
- [ ] 創建 FaultManagement Schemas
- [ ] 驗證 Schemas

### Router 遷移
- [ ] 遷移 NSSMF Routers
- [ ] 遷移 MOI Routers
- [ ] 遷移 FaultManagement Routers
- [ ] 驗證 Routers

### 業務邏輯遷移
- [ ] 創建 Service 層
- [ ] 遷移 Kafka 集成
- [ ] 遷移其他業務邏輯
- [ ] 驗證業務邏輯

### 應用創建
- [ ] 創建主應用
- [ ] 配置中間件
- [ ] 配置日誌
- [ ] 驗證應用

### 測試
- [ ] 編寫單元測試
- [ ] 編寫集成測試
- [ ] 運行所有測試
- [ ] 性能測試

### 部署
- [ ] 創建 Docker 鏡像
- [ ] 部署到測試環境
- [ ] 部署到生產環境
- [ ] 監控和日誌

---

## 📊 遷移時間表

| 階段 | 任務 | 工作量 | 時間 |
|------|------|--------|------|
| 1 | 準備 | 低 | 1-2 天 |
| 2 | 模型遷移 | 低 | 1-2 天 |
| 3 | Schema 遷移 | 低 | 1 天 |
| 4 | Router 遷移 | 中 | 3-4 天 |
| 5 | 業務邏輯遷移 | 中 | 2-3 天 |
| 6 | 應用創建 | 低 | 1 天 |
| 7 | 測試 | 中 | 2-3 天 |
| **總計** | | **中** | **13-18 天** |

---

## 💡 建議

### 立即行動
1. 完成代碼審查中的 P0 問題 (CSRF 保護)
2. 完成代碼審查中的 P1 問題 (日誌、異常處理)
3. 然後考慮 FastAPI 遷移

### 遷移策略
1. **漸進式遷移**: 一個模塊一個模塊地遷移
2. **並行運行**: 在遷移期間同時運行 Django 和 FastAPI
3. **逐步切換**: 逐步將流量從 Django 切換到 FastAPI

### 風險管理
1. **備份**: 在遷移前備份所有數據
2. **測試**: 編寫全面的測試
3. **監控**: 在生產環境中監控性能
4. **回滾計劃**: 準備回滾計劃

---

## 📚 參考資源

- [FastAPI 官方文檔](https://fastapi.tiangolo.com/)
- [SQLAlchemy 文檔](https://docs.sqlalchemy.org/)
- [Pydantic 文檔](https://docs.pydantic.dev/)
- [Uvicorn 文檔](https://www.uvicorn.org/)

---

**建議**: 先完成代碼審查中的改進，然後再考慮 FastAPI 遷移。
