# pyqt5笔记
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