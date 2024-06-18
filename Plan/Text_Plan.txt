# 需求

先有一个前端项目，需要从一个驱动的接口获取数据，然后在前端显示字段和字段对应的数据。

附加要求如下：

1. 要求成品必须是.exe应用程序；
2. 要求每间隔五秒钟刷新数据显示；
3. 要求每次刷新完，生成一个日志文件存在着本地文件根目录。



# 方案1：

## Electron + Javascript + Vue + Node.js——IDEA

> 添加VUE框架+Node.JS

**前端界面**：可以使用Electron来创建桌面应用程序，它允许我们使用HTML、CSS和JavaScript来构建跨平台的桌面应用，并且可以打包成.exe文件。

**数据获取**：使用JavaScript中的`fetch` API从驱动的接口获取数据。

**数据刷新**：使用JavaScript的`setInterval`函数来每隔五秒刷新一次数据。

**日志记录**：使用Node.js的文件系统模块（`fs`）来生成日志文件。



## 1、IDEA配置

**打开项目**：

- 打开 IntelliJ IDEA，选择 `Open`，并选择你的项目文件夹。

**配置 Node.js**：

- 打开 `File` -> `Settings` -> `Languages & Frameworks` -> `Node.js and NPM`。
- 确保 `Node interpreter` 指向正确的 Node.js 安装路径。

**运行和调试**：

- 在 IntelliJ IDEA 中，你可以创建运行配置来运行和调试你的 Electron 应用：
  - 打开 `Run` -> `Edit Configurations`。
  - 点击 `+` 号，选择 `Node.js`。
  - 配置入口文件为 `main.js`，并保存配置。

## 2、创建Electron应用

**创建新项目**：

- 打开 IntelliJ IDEA，选择 `New Project`。
- 选择 `Node.js` 项目模板，并设置项目名称和路径。

**安装 Electron 和 Vue.js**：

- 打开终端（可以在 IntelliJ IDEA 的底部找到 Terminal 选项卡）。
- 运行以下命令初始化项目并安装必要的依赖：

```nginx
npm init -y
npm install electron vue
```

## 3、设置项目结构

设置项目结构如下：

```nginx
my-electron-vue-app/
├── main.js
├── package.json
├── index.html
├── renderer.js
└── src/
    ├── App.vue
    └── main.js
```

## 4、配置Electron主进程

在`main.js`中配置Electron的主进程：

```javascript
const { app, BrowserWindow } = require('electron');
const path = require('path');

function createWindow () {
  const mainWindow = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      preload: path.join(__dirname, 'renderer.js'),
      contextIsolation: false,
      enableRemoteModule: true
    }
  });

  mainWindow.loadFile('index.html');
}

app.whenReady().then(() => {
  createWindow();

  app.on('activate', () => {
    if (BrowserWindow.getAllWindows().length === 0) createWindow();
  });
});

app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') app.quit();
});

```

## 5、创建Vue.js应用

1. 在`src`文件夹中创建一个Vue组件文件`App.vue`：

   ```vue
   <template>
     <div id="app">
       <h1>Data from API</h1>
       <div>{{ data }}</div>
     </div>
   </template>
   
   <script>
   export default {
     data() {
       return {
         data: null
       };
     },
     created() {
       this.fetchData();
       setInterval(this.fetchData, 5000);
     },
     methods: {
       fetchData() {
         fetch('https://api.example.com/data')
           .then(response => response.json())
           .then(data => {
             this.data = data;
             this.logData(data);
           })
           .catch(error => console.error('Error fetching data:', error));
       },
       logData(data) {
         const fs = require('fs');
         const path = require('path');
         const logContent = `Timestamp: ${new Date().toISOString()}\nData: ${JSON.stringify(data, null, 2)}\n\n`;
         const logFilePath = path.join(__dirname, `log_${Date.now()}.txt`);
   
         fs.writeFile(logFilePath, logContent, err => {
           if (err) {
             console.error('Error writing log file:', err);
           } else {
             console.log('Log file created:', logFilePath);
           }
         });
       }
     }
   };
   </script>
   
   ```

2. 创建Vue应用入口文件`src/main.js`：

   ```javascript
   import Vue from 'vue';
   import App from './App.vue';
   
   new Vue({
     render: h => h(App),
   }).$mount('#app');
   
   ```

3. 修改`index.html`以包含Vue应用挂载点：

   ```html
   <!DOCTYPE html>
   <html>
   <head>
     <title>Data Display</title>
   </head>
   <body>
     <div id="app"></div>
     <script src="dist/build.js"></script>
   </body>
   </html>
   ```

## 6、构建和打包

为了构建和打包Vue应用，我们需要一些额外的工具，如`webpack`。在此简要描述如何设置webpack：

1. 安装所需依赖：

   ```bash
   npm install webpack webpack-cli vue-loader vue-template-compiler css-loader --save-dev
   ```

2. 创建`webpack.config.js`文件：

   ```javascript
   const path = require('path');
   const VueLoaderPlugin = require('vue-loader/lib/plugin');
   
   module.exports = {
     mode: 'development',
     entry: './src/main.js',
     output: {
       path: path.resolve(__dirname, 'dist'),
       filename: 'build.js'
     },
     module: {
       rules: [
         {
           test: /\.vue$/,
           loader: 'vue-loader'
         },
         {
           test: /\.css$/,
           use: [
             'vue-style-loader',
             'css-loader'
           ]
         }
       ]
     },
     plugins: [
       new VueLoaderPlugin()
     ]
   };
   ```

3. 在`package.json`中添加构建脚本：

   ```json
   "scripts": {
     "start": "electron .",
     "build": "webpack --config webpack.config.js",
     "package-win": "electron-packager . MyElectronVueApp --platform=win32 --arch=x64 --out=dist --overwrite"
   }
   ```

4. 运行构建命令：

   ```bash
   npm run build
   ```

5. 运行Electron应用：

   ```bash
   npm start
   ```

6. 打包成.exe文件：

   ```bash
   npm run package-win
   ```



# 方案2

## Python3.x  + 功能库（PyQt5 + requests + PyInstaller）—— VScode

**前端界面**：使用PyQt5或Tkinter创建桌面应用程序。

**数据获取**：使用Python的`requests`库从驱动的接口获取数据。

**数据刷新**：使用Python的定时器来每隔五秒刷新一次数据。

**日志记录**：使用Python的`logging`模块或直接操作文件系统来生成日志文件。



## 1、开发配置

**1.1、环境和软件：**

**Python**：安装Python 3.x。

**代码编辑器**：VS Code。

**依赖库**：

- `PyQt5`：用于创建桌面应用程序的界面。
- `requests`：用于从API获取数据。
- `PyInstaller`：用于将Python脚本打包成.exe文件。

**1.2、安装和配置步骤**

1.  安装Python

2. VS Code

   - 下载并安装VS Code
   - 安装Python插件。

3. 创建和配置Python项目

   - 打开VS Code，选择 `File` -> `Open Folder`，选择或创建项目文件夹。

   - 打开终端（Terminal），创建和激活虚拟环境：

     ```bash
     # 创建虚拟环境
     python -m venv venv
     
     # 激活虚拟环境 (Windows)
     venv\Scripts\activate
     
     # 激活虚拟环境 (macOS/Linux)
     source venv/bin/activate
     ```

     

## 2、安装所需库

首先，确保已经安装了所需的Python库：

```bash
pip install PyQt5 requests
```

## 3、创建Python项目结构

项目结构如下：

```css
my-python-app/
├── main.py
└── requirements.txt
```

在`requirements.txt`文件中写入：

```nginx
PyQt5
requests
```

## 4、编写主程序 `main.py`

```python
import sys
import requests
import json
import time
import logging
from PyQt5.QtWidgets import QApplication, QWidget, QVBoxLayout, QLabel, QTextEdit, QPushButton
from PyQt5.QtCore import QTimer, QThread, pyqtSignal

# 配置日志记录
logging.basicConfig(filename='app.log', level=logging.INFO, 
                    format='%(asctime)s - %(levelname)s - %(message)s')

class DataFetcher(QThread):
    data_fetched = pyqtSignal(dict)

    def run(self):
        while True:
            try:
                response = requests.get('https://api.example.com/data')
                data = response.json()
                self.data_fetched.emit(data)
                log_data(data)
            except Exception as e:
                logging.error(f"Error fetching data: {e}")
            time.sleep(5)

def log_data(data):
    log_content = f"Timestamp: {time.strftime('%Y-%m-%d %H:%M:%S')}\nData: {json.dumps(data, indent=2)}\n\n"
    with open(f"log_{int(time.time())}.txt", 'w') as log_file:
        log_file.write(log_content)

class App(QWidget):
    def __init__(self):
        super().__init__()
        self.initUI()

        self.data_fetcher = DataFetcher()
        self.data_fetcher.data_fetched.connect(self.update_data)
        self.data_fetcher.start()

    def initUI(self):
        self.setWindowTitle('Data Display')
        self.setGeometry(100, 100, 600, 400)
        
        layout = QVBoxLayout()

        self.label = QLabel('Data from API:')
        layout.addWidget(self.label)

        self.textEdit = QTextEdit()
        self.textEdit.setReadOnly(True)
        layout.addWidget(self.textEdit)

        self.setLayout(layout)

    def update_data(self, data):
        self.textEdit.setText(json.dumps(data, indent=2))

if __name__ == '__main__':
    app = QApplication(sys.argv)
    ex = App()
    ex.show()
    sys.exit(app.exec_())
```

- **DataFetcher** 类继承自 `QThread`，负责定期从API获取数据。每次获取数据后，会发射一个信号 `data_fetched`，将数据传递给主线程。
- **log_data** 函数负责将获取的数据记录到日志文件中。
- **App** 类继承自 `QWidget`，是应用程序的主窗口。初始化时，它创建了一个标签和一个不可编辑的文本编辑框来显示
- **update_data** 方法接收来自 `DataFetcher` 的信号，并更新文本编辑框中的数据。

## 5、运行程序

在终端中运行以下命令以启动应用程序：

```bash
python main.py
```

## 6、打包成 .exe 文件

要将Python脚本打包成 .exe 文件，可以使用 `PyInstaller`：

- 安装 `PyInstaller`：

  ```bash
  pip install pyinstaller
  ```

- 使用 `PyInstaller` 打包：

  ```bash
  pyinstaller --onefile main.py
  ```

打包完成后，会在 `dist` 目录下生成 `main.exe` 文件。

这样，你的Python应用程序就可以作为一个独立的 .exe 文件运行，每隔五秒钟刷新数据显示并生成日志文件。