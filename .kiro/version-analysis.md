# Free5GMANO 依賴版本分析報告

## 📊 版本對比表

| 套件名稱 | 當前版本 | 最新版本 | 發布日期 | 狀態 | 建議 |
|---------|---------|---------|---------|------|------|
| **Django** | 2.2.8 | 5.0.x | 2024年 | ⚠️ 過時 | 升級到 4.2 LTS |
| **djangorestframework** | 3.10.1 | 3.14.x | 2024年 | ⚠️ 過時 | 升級到 3.14.x |
| **drf-yasg** | 1.17.0 | 1.21.x | 2024年 | ⚠️ 過時 | 升級到 1.21.x |
| **jsonschema** | 3.0.2 | 4.20.x | 2024年 | ⚠️ 過時 | 升級到 4.20.x |
| **requests** | 2.21.0 | 2.31.x | 2024年 | ⚠️ 過時 | 升級到 2.31.x |
| **pytz** | 2019.1 | 2024.1 | 2024年 | ⚠️ 過時 | 升級到 2024.1 |
| **sqlparse** | 0.3.0 | 0.4.x | 2024年 | ⚠️ 過時 | 升級到 0.4.x |
| **PyYAML** | 未指定 | 6.0.x | 2024年 | ⚠️ 無版本 | 指定版本 6.0.x |
| **django-mysql** | 未指定 | 4.0.x | 2024年 | ⚠️ 無版本 | 指定版本 4.0.x |
| **mysqlclient** | 未指定 | 2.2.x | 2024年 | ⚠️ 無版本 | 指定版本 2.2.x |
| **django-cors-headers** | 未指定 | 4.3.x | 2024年 | ⚠️ 無版本 | 指定版本 4.3.x |
| **service-mapping-plugin-framework** | 未指定 | ? | ? | ❓ 未知 | 需要檢查 |

---

## 🚨 主要問題分析

### 1. Django 版本 (2.2.8) - 🔴 嚴重過時

**當前狀態**
- 發布於 2019 年
- 已於 2022 年 4 月停止支持
- 存在多個安全漏洞

**升級路徑**
```
Django 2.2.8 → Django 3.2 LTS → Django 4.2 LTS → Django 5.0
```

**建議**
- 直接升級到 **Django 4.2 LTS** (長期支持版本，支持到 2026 年)
- 或升級到 **Django 5.0** (最新版本)

**升級成本**
- 中等難度
- 需要更新 URL 配置、中間件、序列化器
- 需要測試所有 API 端點

---

### 2. Django REST Framework (3.10.1) - 🔴 嚴重過時

**當前狀態**
- 發布於 2019 年
- 已停止支持
- 缺少新功能和安全修復

**升級建議**
- 升級到 **3.14.x** (最新穩定版本)

**升級成本**
- 低難度
- 主要是 API 相容性改進
- 大多數代碼無需修改

---

### 3. drf-yasg (1.17.0) - 🟡 過時

**當前狀態**
- 發布於 2020 年
- 最新版本是 1.21.x

**升級建議**
- 升級到 **1.21.x**

**升級成本**
- 低難度
- 主要是 Swagger/OpenAPI 文檔改進

---

### 4. jsonschema (3.0.2) - 🔴 嚴重過時

**當前狀態**
- 發布於 2019 年
- 最新版本是 4.20.x
- 版本跨度大

**升級建議**
- 升級到 **4.20.x**

**升級成本**
- 低難度
- 主要是驗證邏輯改進

---

### 5. requests (2.21.0) - 🔴 嚴重過時

**當前狀態**
- 發布於 2018 年
- 最新版本是 2.31.x
- 存在安全漏洞

**升級建議**
- 升級到 **2.31.x**

**升級成本**
- 低難度
- 完全向後相容

---

### 6. pytz (2019.1) - 🔴 嚴重過時

**當前狀態**
- 發布於 2019 年
- 最新版本是 2024.1
- 時區資料已過時

**升級建議**
- 升級到 **2024.1**

**升級成本**
- 低難度
- 完全向後相容

---

### 7. sqlparse (0.3.0) - 🔴 嚴重過時

**當前狀態**
- 發布於 2019 年
- 最新版本是 0.4.x

**升級建議**
- 升級到 **0.4.x**

**升級成本**
- 低難度
- 完全向後相容

---

### 8. 無版本指定的套件 - 🟡 風險

**受影響的套件**
- PyYAML
- django-mysql
- mysqlclient
- django-cors-headers

**問題**
- 無版本指定會導致安裝時自動選擇最新版本
- 可能導致不相容的版本被安裝
- 難以重現環境

**建議**
- 為所有套件指定具體版本

---

## 📋 升級計劃

### 第一階段：低風險升級 (推薦先做)

```txt
PyYAML==6.0
jsonschema==4.20.0
pytz==2024.1
sqlparse==0.4.4
requests==2.31.0
django-cors-headers==4.3.1
mysqlclient==2.2.0
django-mysql==4.0.3
djangorestframework==3.14.0
drf-yasg==1.21.7
```

**預期時間**: 1-2 小時
**風險等級**: 低
**測試**: 運行現有測試套件

---

### 第二階段：中等風險升級 (需要測試)

```txt
Django==4.2.8  # LTS 版本，支持到 2026 年
```

**預期時間**: 4-8 小時
**風險等級**: 中
**需要修改的地方**:
- URL 配置 (urls.py)
- 中間件配置
- 序列化器
- 視圖函數
- 資料庫查詢

**測試**: 完整的集成測試

---

### 第三階段：可選升級 (未來考慮)

```txt
Django==5.0.x  # 最新版本
```

**預期時間**: 8-16 小時
**風險等級**: 高
**建議**: 等待 Django 4.2 穩定後再考慮

---

## 🔍 安全漏洞風險

### 已知的安全問題

| 套件 | 版本 | 漏洞 | 嚴重程度 |
|-----|------|------|---------|
| Django | 2.2.8 | CVE-2021-33571 等多個 | 高 |
| requests | 2.21.0 | 多個 HTTP 相關漏洞 | 中 |
| PyYAML | 舊版本 | 任意代碼執行 | 高 |

**建議**: 立即升級以修復安全漏洞

---

## 🛠️ 升級步驟

### 1. 備份當前環境
```bash
pip freeze > requirements-backup.txt
git commit -am "Backup before dependency upgrade"
```

### 2. 創建新的 requirements.txt
```bash
# 使用下面提供的新版本
```

### 3. 在測試環境中升級
```bash
pip install -r requirements-new.txt
python manage.py test
```

### 4. 檢查相容性
```bash
python manage.py check
python manage.py makemigrations
python manage.py migrate
```

### 5. 運行完整測試
```bash
python manage.py test --verbosity=2
```

### 6. 部署到生產環境
```bash
# 在生產環境中執行相同步驟
```

---

## 📝 新的 requirements.txt (推薦)

### 第一階段版本 (低風險)
```txt
PyYAML==6.0
jsonschema==4.20.0
Django==2.2.8
djangorestframework==3.14.0
pytz==2024.1
sqlparse==0.4.4
requests==2.31.0
django-mysql==4.0.3
mysqlclient==2.2.0
django-cors-headers==4.3.1
drf-yasg==1.21.7
service-mapping-plugin-framework
```

### 第二階段版本 (中等風險)
```txt
PyYAML==6.0
jsonschema==4.20.0
Django==4.2.8
djangorestframework==3.14.0
pytz==2024.1
sqlparse==0.4.4
requests==2.31.0
django-mysql==4.0.3
mysqlclient==2.2.0
django-cors-headers==4.3.1
drf-yasg==1.21.7
service-mapping-plugin-framework
```

---

## ✅ 升級檢查清單

- [ ] 備份當前代碼和資料庫
- [ ] 在測試環境中安裝新版本
- [ ] 運行 `python manage.py check`
- [ ] 運行所有單元測試
- [ ] 測試所有 API 端點
- [ ] 檢查日誌中是否有棄用警告
- [ ] 更新文檔
- [ ] 在測試環境中進行完整集成測試
- [ ] 部署到預發布環境
- [ ] 部署到生產環境

---

## 📚 參考資源

- [Django 升級指南](https://docs.djangoproject.com/en/4.2/releases/)
- [Django REST Framework 更新日誌](https://www.django-rest-framework.org/community/release-notes/)
- [Python 安全漏洞資料庫](https://pyup.io/)

---

## 🎯 總結

**當前狀態**: 🔴 非常過時，存在安全風險

**建議優先級**:
1. **立即升級** (安全漏洞): requests, PyYAML, jsonschema
2. **儘快升級** (功能改進): djangorestframework, drf-yasg
3. **計劃升級** (主要版本): Django 4.2 LTS
4. **未來考慮** (最新版本): Django 5.0

**預計工作量**: 
- 第一階段: 1-2 小時
- 第二階段: 4-8 小時
- 總計: 5-10 小時

**風險評估**: 中等 (主要風險在 Django 升級)
