# ini2py

[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![PyPI version](https://badge.fury.io/py/ini2py.svg)](https://badge.fury.io/py/ini2py)

一個能夠從 .ini 檔案生成具有型別提示的 Python 設定類別的 CLI 工具，並支援自動檔案監控和熱重載功能。

## 特色功能

- 🔧 **自動生成型別提示的 Python 類別** - 從 INI 設定檔自動產生
- 🔍 **智慧型別推斷** - 支援 int、float、boolean、string
- 🔄 **熱重載** - 當檔案變更時自動重新載入設定
- 🛡️ **敏感資料保護** - 密碼和 API 金鑰只在使用 print_all() 時才會被遮罩
- 🎯 **智慧路徑偵測** - 自動尋找常見位置的設定檔
- 💡 **IDE 友善** - 完整的自動補全和型別提示支援
- 🏗️ **單例模式** - 確保整個應用程式只有單一設定實例

## 安裝

```bash
pip install ini2py
```

## 快速開始

### 1. 建立 config.ini 檔案

```ini
[system]
mode = development
debug = true
port = 8080
timeout = 30.5

[database]
host = localhost
port = 5432
name = myapp
user = admin
password = secret123

[redis]
host = 127.0.0.1
port = 6379
db = 0

[ai_service]
api_key = sk-1234567890abcdef
model = gpt-4
temperature = 0.7
```

### 2. 生成 Python 設定類別

在專案目錄中執行 CLI 工具：

```bash
ini2py
```

工具將會：
- 自動偵測您的 `config.ini` 檔案
- 建議適當的輸出目錄
- 生成具有型別提示的 Python 類別

範例輸出：
```
Path to your config.ini file [./config/config.ini]: 
Path to the output directory for generated files [./src/config]: 
Reading configuration from: ./config/config.ini
Generating schema.py...
Successfully generated ./src/config/schema.py
Generating manager.py...
Successfully generated ./src/config/manager.py

Configuration generation complete!
```

### 3. 在 Python 程式碼中使用

```python
from src.config.manager import ConfigManager

# 初始化設定管理器（單例模式）
config = ConfigManager()

# 使用完整的型別提示和自動補全存取設定
mode = config.system.mode          # str
debug = config.system.debug        # bool
port = config.system.port          # int
timeout = config.system.timeout    # float

# 正常存取敏感資料
api_key = config.ai_service.api_key  # 返回實際值
password = config.database.password   # 返回實際值

# 使用 print_all() 安全顯示設定，敏感資料會自動遮罩
print("資料庫設定：")
print(config.database.print_all(return_type='list'))
# 輸出：['host: localhost', 'port: 5432', 'password: se****23', ...]

# 或者手動控制是否遮罩（請小心使用！）
print(config.database.return_properties(return_type='list', mask_sensitive=False))
```

## 進階用法

### 手動指定路徑

```bash
# 指定自訂路徑
ini2py --config /path/to/config.ini --output /path/to/output/
```

### 熱重載

生成的設定管理器會自動監控檔案變更：

```python
import time
from src.config.manager import ConfigManager

config = ConfigManager()

# 當 config.ini 被修改時，設定會自動重新載入
while True:
    print(f"目前連接埠：{config.system.port}")
    time.sleep(2)
```

### 敏感資料處理

預設情況下，敏感資料（密碼、API 金鑰等）可以正常存取。使用 `print_all()` 方法可以安全地顯示設定，並自動遮罩敏感資訊。

偵測的敏感關鍵字：
- `password`、`pwd`
- `api_token`、`token`
- `secret`、`key`
- `appkey`

```python
# 正常存取會返回實際值
password = config.database.password      # "secret123"
api_key = config.ai_service.api_key     # "sk-1234567890abcdef"

# 使用 print_all() 安全顯示
config.database.print_all()              # 密碼顯示為 "se****23"
config.ai_service.print_all('dict')      # API 金鑰顯示為 "sk-12**************ef"

# 手動控制 return_properties
raw = config.database.return_properties(mask_sensitive=False)  # 實際值
masked = config.database.return_properties(mask_sensitive=True) # 遮罩值
```

## 設定檔探索

ini2py 會自動在以下位置搜尋設定檔：
1. `./config.ini`
2. `./config/config.ini`
3. `./conf/config.ini`
4. `../config.ini`（最多向上 5 層）

## 生成的檔案結構

```
src/config/
├── schema.py    # 具有型別提示的架構類別
└── manager.py   # 支援熱重載的設定管理器
```

### 生成的架構類別

每個 INI 區段都會成為一個型別化的架構類別：

```python
class SystemSchema(ConfigSchema):
    """[system]"""
    
    @property
    def mode(self) -> str:
        return self._config_section.get('mode')
    
    @property
    def debug(self) -> bool:
        return self._config_section.getboolean('debug')
    
    @property
    def port(self) -> int:
        return self._config_section.getint('port')
```

## CLI 選項

```bash
ini2py --help
```

選項：
- `--config, -c`：輸入的 config.ini 檔案路徑
- `--output, -o`：生成檔案的目錄
- `--help`：顯示說明訊息

## 系統需求

- Python 3.8+
- click >= 8.0
- watchdog >= 2.1.6

## 開發

### 設定開發環境

```bash
git clone https://github.com/joneshong/ini2py.git
cd ini2py
pip install -e .
```

### 執行測試

```bash
python -m pytest tests/
```

### 專案結構

```
ini2py/
├── ini2py/
│   ├── __init__.py
│   ├── cli.py           # 主要 CLI 邏輯
│   └── templates/       # 程式碼生成模板
│       ├── schema.py.tpl
│       └── manager.py.tpl
├── tests/
├── examples/
├── README.md
├── README.zh.md
└── pyproject.toml
```

## 貢獻

1. Fork 此專案
2. 建立功能分支（`git checkout -b feature/amazing-feature`）
3. 提交您的變更（`git commit -m 'Add amazing feature'`）
4. 推送到分支（`git push origin feature/amazing-feature`）
5. 開啟 Pull Request

## 授權條款

本專案採用 MIT 授權條款 - 詳見 [LICENSE](LICENSE) 檔案。

## 更新日誌

### v0.2.2
- 更新 README 文件以釐清敏感資料處理方式
- 新增繁體中文文件（README.zh.md）

### v0.2.1
- 新增 `print_all()` 方法，用於安全顯示並自動遮罩敏感資料
- 修復跨平台相容性的 CI/CD 問題
- 使用 Black 和 isort 改善程式碼格式

### v0.1.0
- 初始版本釋出
- 基本的 INI 到 Python 類別生成
- 型別推斷支援
- 使用 watchdog 的熱重載功能
- 敏感資料保護
- 智慧路徑偵測

## 作者

**JonesHong** - [GitHub](https://github.com/joneshong)

## 支援

如果您遇到任何問題或有疑問：
- [開啟 issue](https://github.com/joneshong/ini2py/issues)
- 查看 [examples](examples/) 目錄
- 檢視生成的程式碼文件