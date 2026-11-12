# lab2 设计资料

本目录保存活动票务与报名系统的 lab2 设计产物。

- `顶层设计.md`：最终设计说明（模块、聚合、类、端口、交互、状态机、UI 清单）
- `UML/`：14 张 UML 图的 `.puml` 源文件与 200 DPI `.png`
- `UI界面/`：5 屏 UI 的 HTML/CSS 源文件与 `.png` 截图

**实验报告不在本目录**。报告在仓库上级目录的 `我的实验/实验2_report.md`，按课程要求单独提交；本目录只放设计、UML 和 UI。

## 结构

```
design/
├── 顶层设计.md
├── UML/
│   ├── README.md
│   ├── UML类图-核心领域.puml/.png
│   ├── UML类图-支撑领域.puml/.png
│   ├── UML类图-规则对象-状态与编辑.puml/.png
│   ├── UML类图-规则对象-权限与口径.puml/.png
│   ├── UML组件图.puml/.png
│   ├── UML用例图-参与者与组织者.puml/.png
│   ├── UML用例图-检票员管理员与系统.puml/.png
│   ├── UML用例图.puml/.png
│   ├── UML状态图-活动.puml/.png
│   ├── UML状态图-报名.puml/.png
│   ├── UML活动图-报名流程.puml/.png
│   ├── UML序列图-报名与抢名额.puml/.png
│   ├── UML序列图-取消活动.puml/.png
│   └── UML序列图-扫码签到.puml/.png
└── UI界面/
    ├── README.md
    ├── ui.css
    ├── screen1-discover.html … screen5-admin.html
    └── shots/01-discover.png … 05-admin.png
```

## 重新生成

### UML

在本目录准备 PlantUML jar（仓库不提交二进制），然后执行：

```bash
cd lab/design/UML
java -Dfile.encoding=UTF-8 -jar plantuml.jar -tpng -charset UTF-8 -Sdpi=200 *.puml
```

### UI

用 Edge/Chrome headless 截图；`<repo>` 替换为仓库绝对路径：

```bash
"C:/Program Files (x86)/Microsoft/Edge/Application/msedge.exe" \
  --headless=new --disable-gpu --hide-scrollbars --window-size=1440,900 \
  --screenshot="<repo>/lab/design/UI界面/shots/01-discover.png" \
  "file:///<repo>/lab/design/UI界面/screen1-discover.html"
```

五屏分别替换 HTML 与输出文件名。

## 使用约定

- 设计以 `顶层设计.md` 为准；UML 与 UI 是它的可视化表达。
- `UML/` 下的 `.puml` 是源文件，`.png` 是导出图；修改请改 `.puml` 后重新渲染。
- `UI界面/` 下是静态设计稿，不连接后端；示例数据只用于展示。
- lab3 实现时允许根据实际情况调整设计，但应同步更新本目录，保持设计与代码一致。
