# Free5GMANO 升級檢查清單

## ✅ 已完成的升級

### 1. 依賴版本更新
- [x] PyYAML: 未指定 → 6.0
- [x] jsonschema: 3.0.2 → 4.20.0
- [x] Django: 2.2.8 → 4.2.8
- [x] djangorestframework: 3.10.1 → 3.14.0
- [x] pytz: 2019.1 → 2024.1
- [x] sqlparse: 0.3.0 → 0.4.4
- [x] requests: 2.21.0 → 2.31.0
- [x] django-mysql: 未指定 → 4.0.3
- [x] mysqlclient: 未指定 → 2.2.0
- [x] django-cors-headers: 未指定 → 4.3.1
- [x] drf-yasg: 1.17.0 → 1.21.7

### 2. 代碼修改
- [x] requirements.txt - 更新所有依賴版本
- [x] free5gmano/settings.py - 移除 USE_L10N，更新文檔
- [x] free5gmano/urls.py - 用 re_path 替換 url()
- [x] free5gmano/wsgi.py - 更新文檔
- [x] moi/urls.py - 用 re_path 替換 url()

### 3. 棄用 API 移除
- [x] 移除 `from django.conf.urls import url`
- [x] 將所有 `url()` 改為 `re_path()`
- [x] 移除 `USE_L10N` 設定

---

## 📋 後續驗證步驟

### 第一步：環境準備
```bash
# 1. 備份當前環境
pip freeze > requirements-backup.txt

# 2. 備份代碼
git add -A
git commit -m "Backup before dependency upgrade"
```

### 第二步：安裝新版本
```bash
# 1. 清理舊的依賴
pip cache purge

# 2. 安裝新版本
pip install -r requirements.txt

# 3. 驗證 Django 版本
python -c "import django; print(f'Django version: {django.get_version()}')"
```

### 第三步：Django 檢查
```bash
# 1. 運行 Django 檢查
python manage.py check

# 2. 檢查遷移
python manage.py makemigrations --dry-run

# 3. 運行遷移
python manage.py migrate
```

### 第四步：測試
```bash
# 1. 運行單元測試
python manage.py test --verbosity=2

# 2. 測試 API 端點
# 使用 curl 或 Postman 測試所有 API 端點

# 3. 檢查日誌
# 查看是否有棄用警告
```

### 第五步：部署
```bash
# 1. 在預發布環境中測試
# 2. 在生產環境中部署
# 3. 監控應用日誌
```

---

## 🔍 需要測試的功能

### NSSMF 模組
- [ ] 創建通用模板
- [ ] 上傳模板文件
- [ ] 創建切片模板
- [ ] 分配 NSSI
- [ ] 回收 NSSI
- [ ] 查詢模板列表
- [ ] 下載模板

### MOI 模組
- [ ] 創建 MOI 物件
- [ ] 查詢 MOI 物件
- [ ] 修改 MOI 物件
- [ ] 刪除 MOI 物件
- [ ] 創建訂閱
- [ ] 查詢訂閱
- [ ] 刪除訂閱
- [ ] 查詢拓撲

### 故障管理模組
- [ ] 查詢告警
- [ ] 確認告警
- [ ] 清除告警
- [ ] 創建告警訂閱
- [ ] 刪除告警訂閱

### 通用功能
- [ ] API 認證
- [ ] CORS 跨域請求
- [ ] Swagger 文檔
- [ ] 資料庫連接
- [ ] 文件上傳/下載

---

## 🚨 常見問題排查

### 問題 1: ImportError: cannot import name 'url' from 'django.conf.urls'
**解決方案**: 已修復，使用 `re_path` 替換 `url()`

### 問題 2: DeprecationWarning: USE_L10N is deprecated
**解決方案**: 已移除 `USE_L10N` 設定

### 問題 3: 資料庫遷移失敗
**解決方案**: 
```bash
python manage.py migrate --fake-initial
```

### 問題 4: 模板文件上傳失敗
**解決方案**: 檢查 MEDIA_ROOT 權限
```bash
chmod -R 755 /data/nm
```

### 問題 5: Kafka 通知不工作
**解決方案**: 檢查 Kafka 連接配置

---

## 📊 升級前後對比

### 安全性
| 項目 | 升級前 | 升級後 |
|------|--------|--------|
| Django 安全漏洞 | 多個 | 已修復 |
| requests 漏洞 | 存在 | 已修復 |
| PyYAML 漏洞 | 存在 | 已修復 |
| 時區資料 | 2019年 | 2024年 |

### 功能
| 項目 | 升級前 | 升級後 |
|------|--------|--------|
| Django 版本 | 2.2.8 | 4.2.8 |
| DRF 版本 | 3.10.1 | 3.14.0 |
| 支持期限 | 已停止 | 2026年 |

---

## 📞 支持資源

### 官方文檔
- [Django 4.2 文檔](https://docs.djangoproject.com/en/4.2/)
- [Django REST Framework 文檔](https://www.django-rest-framework.org/)
- [drf-yasg 文檔](https://drf-yasg.readthedocs.io/)

### 升級指南
- [Django 2.2 → 3.0 升級指南](https://docs.djangoproject.com/en/3.0/releases/3.0/)
- [Django 3.0 → 3.2 升級指南](https://docs.djangoproject.com/en/3.2/releases/3.2/)
- [Django 3.2 → 4.0 升級指南](https://docs.djangoproject.com/en/4.0/releases/4.0/)
- [Django 4.0 → 4.2 升級指南](https://docs.djangoproject.com/en/4.2/releases/4.2/)

---

## ✨ 升級完成

所有代碼修改已完成，專案已準備好升級到 Django 4.2.8。

**下一步**: 按照上述步驟在測試環境中驗證升級。
