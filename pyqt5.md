# pyqt5笔记
### QtCore − 其他模块使用的核心非 GUI 类
### QtGui − 图形用户界面组件
### QtMultimedia − 低级多媒体编程类
### QtNetwork − 网络编程类
### QtOpenGL − OpenGL 支持类
### QtScript − 用于评估 Qt 脚本的类
### QtSql − 使用 SQL 进行数据库集成的类
### QtSvg − 显示 SVG 文件内容的类
### QtWebKit − 用于呈现和编辑 HTML 的类
### QtXml − 处理 XML 的类

### QtWidgets − 用于创建经典桌面风格 UI 的类
- QWidget 的子类：QDialog 和 QFrame 类
- QWidget派生出的常用小部件：  
    1. QLabel：显示文本或图像
    2. QLineEdit：单行文本输入框
    3. QTextEdit：多行文本输入框
    4. QPushButton：按钮
    5. QRadioButton：单选或多选按钮
    6. QCheckBox：复选框按钮
    7. QComboBox：下拉列表
    8. QSpinBox：带有向上/向下调整数值按钮的文本框
    9. QScrollBar：滚动条
    10. QSlider：带滑块的标尺
    11. QMenuBar：菜单栏的小部件 可创建子菜单
    12. QStatusBar：底部状态栏
    13. QToolBar：可移动工具栏面板
    14. QListView：在 ListMode 或 IconMode 中提供可选择的项目列表
    15. QPixmap：显示在 QLabel 或 QPushButton 对象上的屏幕外图像表示
    16. QDialog：可以向父窗口返回信息的模态或非模态窗口 弹出对话框
    17. QMessageBox：常用的模式对话框
- QWidget类及子类样式设置方法：
    QWidget.setStyleSheet("width: 200px; height: 20px; font-size: 10px; font-weight: bold")

### QtDesigner − 用于扩展 Qt Designer 的类

### 布局管理
- 整体窗口位于桌面的布局：```QWidget.setGeometry(xpos, ypos, width, height)```
- 三个常用布局类：
    1. QBoxLayout类：垂直或水平排列小部件
        - 子类：QVBoxLayout用于垂直排列小部件
        - 子类：QHBoxLayout用于水平排列小部件  
        #### 方法：  
            addWidget()：将小部件添加到 BoxLayout  
            addStretch()：创建空的可拉伸框  
            addLayout()：添加另一个嵌套布局
    2. QGridLayout类：以行和列排列的单元格网格，可指定单元格的行数和列数
        #### 方法：
            addWidget(QWidget, int r, int c)：指定的行和列添加一个小部件  
            addWidget(QWidget, int r, int c, int rowspan, int columnspan)：指定的行和列添加一个小部件，并具有指定的宽度和/或高度
            addLayout(QLayout, int r, int c)：在指定的行和列添加布局对象
    3. QFormLayout类：创建两列表单的便捷布局方式
        #### 方法：
            addRow(QLabel, QWidget)：添加包含标签和输入字段的行  
            addRow(QLabel, QLayout)：在第二列中添加子布局
            addRow(QWidget)：添加一个跨越两列的小部件

### 信号与槽：对象.信号.connect(槽函数)
1. 简单案例
```
import sys
from Pyqt5.QtWidgets import QApplication,QWidget,QPushButton

class MyWindow(QWidget):
    def __init__(self):
        super().__init__():
        self.init_ui()
    
    def init_ui(self):
        self.resize(500, 300)
        btn = QPushButton("点击触发信号", self)
        btn.setGeometry(200, 200, 100, 30)
        btn.clicked.connect(self.click_my_btn)  #   click信号

    def click_my_btn(self, arg):    #   槽函数
        print("我被点击了", arg)

if __name__ == "__main__":
    app = QApplictaion(sys.argv)
    w = MyWindow()
    w.show()
    app.exec()
```
2. 自定义信号
```
import sys,time
from PyQt5.QtCore import pyqtSignal
from PyQt5.QtWidgets import QApplication,QWidget,QPushButton

class MyWindow(QWidget):
    #   第一步：（创建信号）声明一个信号 能够接受一个字符串参数 注意：只能放在函数外面
    my_signal = pyqtSignal(str)

    def __init__(self):
        super().__init__():
        self.init_ui()
    
    def init_ui(self):
        self.resize(300, 200)
        btn = QPushButton("点击触发信号", self)
        btn.setGeometry(100, 150, 100, 30)

        btn.clicked.connect(self.click_my_btn)  #   普通click信号

        #   第二步：（绑定信号和槽函数）
        self.my_signal.connect(self.my_slot)    #   自定义信号触发方式

    def click_my_btn(self, arg):    #   槽函数
        for i, ip in enumerate(["192.168.1.%d" %x for x in range(1, 255)]):
            print("ip地址是：%s" %ip)
            if i % 5 == 0:
                #   这里发送了一个信号 触发了自定义槽函数
                self.my_signal.emit("我是5的倍数")
            else:
                #   第三步：（触发信号）
                self.my_signal.emit("我不是是5的倍数")
            time.sleep(1)
    
    #   第四步：（执行被绑定槽函数）
    def my_slot(self, arg):  #   执行自定义信号的 槽函数
        print(arg)

if __name__ == "__main__":
    app = QApplication(sys.argv)
    w = MyWindow()
    w.show()
    app.exec()
```
### 多线程：QThread
- 实例代码：
```
import sys
import time
from PyQt5 import uic
from PyQt5.Qt import QApplication, QWidget, QThread

# 定义线程类
class MyThread(QThread):
    def __init__(self) -> None:
        super().__init__()

    def run(self):
        for i in range(10):
            print("是MyThread线程中执行……%d" %(i + 1))
            time.sleep(1)

class MyWin(QWidget):
    def __init__(self) -> None:
        super().__init__()
        self.int_ui()

    def init_ui(self):
        self.ui = uic.loadUi("./thread-1.ui")

        # 从ui中加载控件
        lineedit = self.ui.lineEdit
        btn1 = self.ui.pushButton
        btn2 = self.ui.pushButton_2

        # 给2个按钮绑定槽函数
        btn1.clicked.connect(self.click_1)  # 绑定非线程槽函数
        btn2.clicked.connect(self.click_2)  # 绑定线程槽函数
    
    # 非线程槽函数
    def click_1(self):
        for i in range(10):
            print("是MyThread线程中执行……%d" %(i + 1))
            time.sleep(1)

    # 线程槽函数
    def click_1(self):
        self.my_thread = MyThread() # 创建线程对象 注意：这里一定要加self将线程设置给窗口对象,如果没有加self,槽函数结束后线程将被销毁
        self.my_thread.start()      # 开始执行线程类中的方法

if __name__ == "__main__":
    app = QApplication(sys.argv)
    Myshow = MyWin()
    Myshow.ui.show()
    app.exec()
```