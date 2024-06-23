## 1、需求分析

现在有一个前端项目，需要从一个驱动的接口获取数据，然后在前端显示字段和字段对应的数据。

附加要求如下：

​	1、要求成品必须是.exe应用程序；

​	2、要求每间隔五秒钟刷新数据显示；

​	3、要求每次刷新完，生成一个日志文件存在着本地文件根目录。 

要求使用`openharmony+arkts+UIAbility`来开发，表明每一部分代码在文件目录中的位置。



## 2、工程目录结构

```json
MyOpenHarmonyApp/
├── application/
├── entry/
│   ├── src/
│   │   ├── main/
│   │   │   ├── ets/
│   │   │   │   ├── pages/
│   │   │   │   │   ├── MainAbility/
│   │   │   │   │   │   ├── MainAbility.ts
│   │   │   │   │   │   ├── MainAbilityShell.ets
│   │   │   │   │   │   ├── MainAbilityShell.ts
│   │   │   │   │   │   ├── MainAbilityShell.json
│   │   │   │   ├── app.ts
│   │   │   │   ├── data.ts
│   │   │   │   ├── utils.ts
│   │   │   ├── resources/
│   │   │   │   ├── base/
│   │   │   │   └── default/
│   └── config.json
└── build-profile.json
```



## 3、工程实现

**3.1、配置工程：**

在 `entry/config.json` 中配置应用程序的信息：

```json
{
    "app": {
        "bundleName": "com.example.myopenharmonyapp",
        "version": {
            "code": "1",
            "name": "1.0.0"
        }
    },
    "module": {
        "package": "com.example.myopenharmonyapp",
        "name": "entry",
        "mainAbility": "MainAbility",
        "metadata": {},
        "abilities": [
            {
                "name": "MainAbility",
                "icon": "$media:icon",
                "label": "$string:app_name",
                "type": "page",
                "orientation": "landscape",
                "launchType": "standard"
            }
        ],
        "resources": {
            "base": {},
            "default": {}
        }
    }
}
```

**3.2、主应用程序文件 `app.ts`：**

创建文件 `entry/src/main/ets/app.ts`：

```typescript
import App from '@ohos.application.App';

export default class MyApp extends App {
    onCreate() {
        console.log('MyApp onCreate');
    }

    onDestroy() {
        console.log('MyApp onDestroy');
    }
}
```

**3.3、主界面 `MainAbilityShell.ets`：**

创建文件 `entry/src/main/ets/pages/MainAbility/MainAbilityShell.ets`：

```typescript
import router from '@system.router';
import fetch from '@system.fetch';
import File from '@ohos.file';
import { formatDate } from '../utils';
import { Data } from '../data';

@Entry
export default class MainAbilityShell {
    data: Data = {};

    onInit() {
        this.fetchData();
        setInterval(this.fetchData.bind(this), 5000);
    }

    fetchData() {
        fetch({
            url: 'http://example.com/api/data',
            success: (response) => {
                this.data = JSON.parse(response.data);
                this.logData();
                this.updateUI();
            },
            fail: (error) => {
                console.error('fetch data error:', error);
            }
        });
    }

    logData() {
        const fileName = '/data/local/tmp/log_' + new Date().getTime() + '.txt';
        File.writeFile({
            uri: fileName,
            text: JSON.stringify(this.data),
            success: () => {
                console.log('log data success');
            },
            fail: (error) => {
                console.error('log data error:', error);
            }
        });
    }

    updateUI() {
        // 更新 UI 的逻辑放在这里
        this.build();
    }

    build() {
        Column() {
            Text('Data Viewer')
                .fontSize(40)
                .margin({ top: 20 });
            ForEach(Object.keys(this.data), (key) => {
                Row() {
                    Text(key + ':').fontSize(30).width('30%');
                    Text(this.data[key]).fontSize(30).width('70%');
                }.margin({ top: 10 });
            });
        }
    }
}
```

**3.4、 主能力 `MainAbility.ts`：**

创建文件 `entry/src/main/ets/pages/MainAbility/MainAbility.ts`：

```typescript
import UIAbility from '@ohos.application.UIAbility';
import Window from '@ohos.window.Window';

export default class MainAbility extends UIAbility {
    onCreate(want, launchParam) {
        console.log('MainAbility onCreate');
        this.context = this;
        globalThis.abilityContext = this.context;
        Window.create({
            windowType: Window.WindowType.WINDOW_TYPE_NORMAL,
            windowMode: Window.WindowMode.WINDOW_MODE_FULLSCREEN
        }, (window) => {
            window.loadContent('pages/MainAbilityShell');
        });
    }

    onDestroy() {
        console.log('MainAbility onDestroy');
    }
}
```



## 4、辅助操作

**4.1、数据和实用工具文件 `data.ts` 和 `utils.ts`：**

`data.ts` 用于**定义和管理**<u>*应用程序中的数据结构*</u>：

```typescript
export interface Data {
    [key: string]: any;
}

export const defaultData: Data = {};
```

`utils.ts` 用于定义常用工具函数：

```typescript
export function formatDate(date: Date): string {
    return date.toISOString().replace(/T/, ' ').replace(/\..+/, '');
}
```

**4.2、资源文件：**

在 `entry/src/main/resources/base/` 和 `entry/src/main/resources/default/` 目录中放置应用程序<u>**所需的图片和字符串资源**</u>等。

**4.3、打包和编译：**

在项目根目录下运行以下命令进行编译：

```css
hpm build
```

生成 `.hap`文件后，通过`OpenHarmony`提供的工具将`.hap`文件打包成`.exe`应用程序。