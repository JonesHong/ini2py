# PyPI 發布指南

## 🚀 快速發布流程

### 1. 準備工作

首先確保你有 PyPI 和 TestPyPI 帳號：

```bash
# 註冊帳號
# PyPI: https://pypi.org/account/register/
# TestPyPI: https://test.pypi.org/account/register/

# 設置 API Token
# PyPI: https://pypi.org/manage/account/token/
# TestPyPI: https://test.pypi.org/manage/account/token/
```

### 2. 配置認證

創建 `~/.pypirc` 文件：

```ini
[distutils]
index-servers = 
    pypi
    testpypi

[pypi]
username = __token__
password = pypi-AgEIcHlwaS5vcmc...

[testpypi]
repository = https://test.pypi.org/legacy/
username = __token__
password = pypi-AgEIcHlwaS5vcmc...
```

### 3. 設置開發環境

```bash
# 安裝開發依賴
make install-dev

# 或手動安裝
pip install build twine pytest black isort flake8 mypy
```

## 📋 發布檢查清單

### 發布前檢查

- [ ] 所有測試通過：`make test`
- [ ] 代碼格式正確：`make format && make lint`
- [ ] 版本號已更新（在 `pyproject.toml` 和 `ini2py/__init__.py`）
- [ ] CHANGELOG.md 已更新
- [ ] 文檔是最新的
- [ ] 所有更改已提交到 Git

### 發布流程

```bash
# 1. 運行完整檢查
make prepare-release

# 2. 測試發布到 TestPyPI
make upload-test

# 3. 測試安裝
pip install --index-url https://test.pypi.org/simple/ ini2py

# 4. 如果測試通過，發布到 PyPI
make upload

# 5. 創建 Git 標籤
make tag-version
```

## 🤖 自動化發布（推薦）

### 使用發布腳本

```bash
# 自動處理版本更新和檢查
python scripts/release.py 0.2.0
```

### 使用 GitHub Actions

1. **設置 Secrets**：
   - `PYPI_API_TOKEN` - PyPI API token
   - `TEST_PYPI_API_TOKEN` - TestPyPI API token

2. **創建 Release**：
   ```bash
   git tag v0.2.0
   git push origin v0.2.0
   # 在 GitHub 創建 Release
   ```

3. **自動發布**：GitHub Actions 會自動：
   - 運行測試
   - 構建包
   - 發布到 TestPyPI
   - 測試安裝
   - 發布到 PyPI

## 📦 手動發布步驟

如果需要手動發布：

### 1. 更新版本

編輯 `pyproject.toml`：
```toml
version = "0.2.0"
```

編輯 `ini2py/__init__.py`：
```python
__version__ = "0.2.0"
```

### 2. 更新 CHANGELOG

```markdown
## [0.2.0] - 2025-01-XX

### Added
- 新功能描述

### Fixed
- 修復的問題
```

### 3. 運行測試

```bash
make test
make lint
```

### 4. 構建包

```bash
make clean
make build
```

### 5. 檢查包

```bash
make check-dist
```

### 6. 測試發布

```bash
# 發布到 TestPyPI
python -m twine upload --repository testpypi dist/*

# 測試安裝
pip install --index-url https://test.pypi.org/simple/ --extra-index-url https://pypi.org/simple/ ini2py==0.2.0
```

### 7. 正式發布

```bash
# 發布到 PyPI
python -m twine upload dist/*
```

### 8. 創建 Git 標籤

```bash
git add .
git commit -m "Bump version to 0.2.0"
git tag -a v0.2.0 -m "Release version 0.2.0"
git push origin main
git push origin v0.2.0
```

## 🔧 Makefile 命令

| 命令 | 描述 |
|------|------|
| `make help` | 顯示所有可用命令 |
| `make install-dev` | 安裝開發依賴 |
| `make test` | 運行測試 |
| `make lint` | 代碼檢查 |
| `make format` | 格式化代碼 |
| `make clean` | 清理構建文件 |
| `make build` | 構建發布包 |
| `make check-dist` | 檢查發布包 |
| `make upload-test` | 上傳到 TestPyPI |
| `make upload` | 上傳到 PyPI |
| `make prepare-release` | 完整發布準備 |

## 🛠️ 版本管理

### 語義化版本

使用 [Semantic Versioning](https://semver.org/)：

- `MAJOR.MINOR.PATCH` (例如：1.2.3)
- **MAJOR**：不兼容的 API 更改
- **MINOR**：向後兼容的新功能
- **PATCH**：向後兼容的錯誤修復

### 預發布版本

```bash
# Alpha 版本
0.2.0a1

# Beta 版本  
0.2.0b1

# Release Candidate
0.2.0rc1
```

## 🚨 常見問題

### 上傳失敗

```bash
# 檢查包格式
twine check dist/*

# 檢查版本衝突
# PyPI 不允許重複上傳相同版本
```

### 認證失敗

```bash
# 檢查 ~/.pypirc 配置
# 確認 API token 正確
# 確認 token 有正確的權限
```

### 測試失敗

```bash
# 運行詳細測試
make test-verbose

# 檢查代碼格式
make format
make lint
```

## 📊 發布後驗證

```bash
# 檢查 PyPI 頁面
# https://pypi.org/project/ini2py/

# 測試安裝
pip install ini2py==0.2.0

# 測試 CLI
ini2py --help

# 檢查 GitHub Release
# https://github.com/joneshong/ini2py/releases

# 檢查下載統計
# https://pypistats.org/packages/ini2py
```

## 🔄 自動化 CI/CD 流程

### GitHub Actions 工作流

我們設置了兩個主要的工作流：

#### 1. CI 工作流 (`.github/workflows/ci.yml`)

**觸發條件：**
- 推送到 main/develop 分支
- 創建 Pull Request

**執行內容：**
- 多版本 Python 測試 (3.8-3.12)
- 跨平台測試 (Ubuntu, Windows, macOS)
- 代碼格式檢查 (black, isort)
- 代碼質量檢查 (flake8, mypy)
- 測試覆蓋率報告
- CLI 功能測試
- 包構建測試

#### 2. Release 工作流 (`.github/workflows/release.yml`)

**觸發條件：**
- 創建 GitHub Release
- 手動觸發

**執行內容：**
- 版本號驗證
- 包構建和檢查
- 自動發布到 TestPyPI
- 測試安裝驗證
- 自動發布到 PyPI
- 創建 Release Assets

### 設置 GitHub Secrets

在 GitHub 倉庫設置中添加以下 Secrets：

1. **PYPI_API_TOKEN**
   ```
   pypi-AgEIcHlwaS5vcmc...
   ```

2. **TEST_PYPI_API_TOKEN**
   ```
   pypi-AgEIcHlwaS5vcmc...
   ```

## 📈 發布策略

### 開發分支策略

```
main     ──●──●──●──●────● (穩定發布)
           │     │        │
develop  ──●──●──●──●──●──● (開發分支)
           │  │     │
feature  ──●──●─────● (功能分支)
```

### 發布流程

1. **開發階段**
   ```bash
   git checkout develop
   git checkout -b feature/new-feature
   # 開發新功能
   git commit -m "Add new feature"
   git push origin feature/new-feature
   # 創建 PR 到 develop
   ```

2. **測試階段**
   ```bash
   git checkout develop
   git merge feature/new-feature
   # CI 自動運行測試
   ```

3. **準備發布**
   ```bash
   git checkout main
   git merge develop
   python scripts/release.py 0.2.0
   git push origin main
   ```

4. **正式發布**
   ```bash
   git tag v0.2.0
   git push origin v0.2.0
   # 在 GitHub 創建 Release
   # GitHub Actions 自動發布
   ```

## 🏷️ 版本標籤管理

### 創建標籤

```bash
# 創建帶註解的標籤
git tag -a v0.2.0 -m "Release version 0.2.0"

# 推送標籤
git push origin v0.2.0

# 推送所有標籤
git push origin --tags
```

### 標籤命名規範

- 正式版本：`v1.0.0`
- 預發布版本：`v1.0.0-alpha.1`
- 修復版本：`v1.0.1`

## 📝 Release Notes 模板

在 GitHub Release 中使用以下模板：

```markdown
## 🚀 What's New

### ✨ New Features
- 新功能描述

### 🐛 Bug Fixes  
- 修復的問題

### 📚 Documentation
- 文檔更新

### 🔧 Internal
- 內部改進

## 📦 Installation

```bash
pip install ini2py==0.2.0
```

## 🔗 Links

- [PyPI Package](https://pypi.org/project/ini2py/0.2.0/)
- [Documentation](https://github.com/joneshong/ini2py)
- [Changelog](https://github.com/joneshong/ini2py/blob/main/CHANGELOG.md)

## 👥 Contributors

Thanks to all contributors who made this release possible!
```

## 🛡️ 安全考慮

### API Token 安全

1. **使用 Scoped Token**
   - 只給予必要的權限
   - 定期輪換 token

2. **環境隔離**
   - 開發/測試使用 TestPyPI
   - 生產只使用 PyPI

3. **審計日誌**
   - 監控發布活動
   - 記錄所有發布操作

### 包安全

```bash
# 掃描安全漏洞
pip install safety bandit

# 檢查依賴安全
safety check

# 檢查代碼安全
bandit -r ini2py/
```

## 📊 發布監控

### 包統計

- **PyPI 統計**：https://pypistats.org/packages/ini2py
- **GitHub 統計**：Insights → Traffic
- **下載趨勢**：PyPI 包頁面

### 監控指標

1. **下載量**
   - 每日/每週/每月下載
   - 版本分布

2. **用戶反饋**
   - GitHub Issues
   - PyPI 評論

3. **性能指標**
   - CI 構建時間
   - 測試執行時間

## 🔧 故障排除

### 常見發布問題

#### 1. 版本衝突
```bash
# 錯誤：File already exists
# 解決：更新版本號
python scripts/release.py 0.2.1
```

#### 2. 測試失敗
```bash
# 檢查測試日誌
make test-verbose

# 修復測試
# 重新運行發布流程
```

#### 3. 包格式錯誤
```bash
# 檢查包結構
twine check dist/*

# 重新構建
make clean && make build
```

#### 4. 權限問題
```bash
# 檢查 API token 權限
# 更新 ~/.pypirc 配置
# 聯系 PyPI 支持
```

### GitHub Actions 問題

#### 1. Workflow 失敗
- 檢查 Actions 日誌
- 驗證 Secrets 設置
- 確認分支保護規則

#### 2. 權限錯誤
- 檢查 GITHUB_TOKEN 權限
- 確認倉庫設置

## 📞 支持和幫助

### 官方資源

- **PyPI 文檔**：https://packaging.python.org/
- **Twine 文檔**：https://twine.readthedocs.io/
- **GitHub Actions**：https://docs.github.com/en/actions

### 社區支持

- **Python Packaging**：https://discuss.python.org/c/packaging
- **GitHub Community**：https://github.community/

---

## 🎉 發布成功！

發布完成後，不要忘記：

1. ✅ 更新項目文檔
2. ✅ 宣布新版本（社交媒體、博客等）
3. ✅ 收集用戶反饋
4. ✅ 規劃下一版本的功能

恭喜你成功發布了 ini2py 包！🚀