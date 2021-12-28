# 大作业算法设计

## 哈夫曼编码问题

Problem description: Open an English article, count the number of occurrences of each character in the article, and then use them as weights to encode each character, and then decode the encoding after the encoding is completed.

   The use of Huffman coding for information communication can greatly improve channel utilization, shorten information transmission time, and reduce transmission costs. However, this requires pre-encoding the data to be transmitted through an encoding system at the transmitting end, and decoding (recovering) the transmitted data at the receiving end. For duplex channels (that is, channels that can transmit information in both directions), a complete encoding and decoding system is required at each end. Try to write a Huffman coding and decoding system for such a messaging station.
A complete system should have the following functions:

(1) I: Initialization (Initialization). Read the character set size n, n characters and n weights from the terminal, build a Huffman tree, and store it in the file hfmTree.

(2) E: Encoding. Use the built Huffman tree (if not in memory, read it from the file htmTree), encode the text in the file ToBeTran, and then save the result in the file CodeFile.

(3) D: Decoding. Use the built Huffman tree to decode the code in the file CodeFile, and store the result in the file TextFile.

(4) P: Print code file (Print). Display the file CodeFile on the terminal in a compact format, with 50 codes per line. At the same time, write the code of this character form into the file CodePrint.

(5) T: Print Huffman tree (TreePrinting). Display the Huffman tree in the memory in an intuitive way (in the form of a tree or recessed table) on the terminal, and write the Huffman tree in the form of characters into the file TreePrint at the same time.

**Involving knowledge points**: Huffman tree

## 问题解答

Most of this topic is implemented by C/C++, and after self-learning python, I found that it is easier and more convenient to use python to deal with some of the problems in this topic, so this article uses python to implement the algorithm:

### 初始化（Initialization）

There are two forms of initialization. One is to generate directly based on the original text, and the other is to directly import characters. On the basis of these two methods, the character set in the list can also be added, deleted, and modified. The character set can also be used to save the file. Finally, when the window is closed, the program will build a tree based on the existing character set.

#### 根据原文生成

```python
def add(self):
        # 加入一空行
        self.tableWidget.insertRow(self.tableWidget.rowCount())
    def generateCharacterSetFromRawtext(self):
        # 根据原文生成字符集
        self.tableWidget.clearContents()
        self.tableWidget.setRowCount(0)
        def getFrequency(text: str) -> dict:
            # 字频(统计)
            cnt = {}
            for i in text:
                if i not in cnt:
                    cnt[i] = 1
                else:
                    cnt[i] += 1
            return cnt
        CharacterSet = getFrequency(rawTextEdit.toPlainText())
        for i, j in CharacterSet.items():
            self.add()
            item1 = QTableWidgetItem(i)
            item2 = QTableWidgetItem(str(j))
            self.tableWidget.setItem(self.tableWidget.rowCount()-1, 0, item1)
            self.tableWidget.setItem(self.tableWidget.rowCount()-1, 1, item2)
```

#### 直接导入字符

```python
    def importWordFrequency(self):
        # 导入字频
        filePath, ok = QFileDialog.getOpenFileName(self, '选择文件')
        if ok:
            self.tableWidget.clearContents()
            self.tableWidget.setRowCount(0)
            with open(filePath, 'r', encoding='utf-8') as file:
                try:
                    frequency = file.read()
                except UnicodeDecodeError:
                    QMessageBox.critical(
                        self, "错误", "请确保打开的是UTF-8编码的文本文件", QMessageBox.OK)
                    return
            global CharacterSet
            CharacterSet = {}
            textlines = re.findall(r'([\s\S])\t(\S+)(\n|$)', frequency)
            if len(textlines) == 0:
                QMessageBox.critical(self, "错误", "字符集生成失败", QMessageBox.Ok)
                return
            for i, j, _ in textlines:
                try:
                    CharacterSet[i] = float(j)
                except ValueError:
                    QMessageBox.critical(
                        self, "错误", "字符集生成失败", QMessageBox.Ok)
                    self.tableWidget.clearContents()
                    self.tableWidget.setRowCount(0)
                    CharacterSet = {}
                    return
                self.add()
                item1 = QTableWidgetItem(i)
                item2 = QTableWidgetItem(j)
                self.tableWidget.setItem(
                    self.tableWidget.rowCount()-1, 0, item1)
                self.tableWidget.setItem(
                    self.tableWidget.rowCount()-1, 1, item2)
```
#### 额外的增删改查

```python

    def add(self):
        # 加入一空行
        self.tableWidget.insertRow(self.tableWidget.rowCount())

    def find(self):
        # 对于字符或字频或字符与字频进行查找
        a: str = self.wordFrequencyEdit.text()
        b: str = self.frequencyEdit.text()
        i: int = 0
        if a and b:
            while i < self.tableWidget.rowCount():
                if self.tableWidget.item(i, 0).text() == a and self.tableWidget.item(i, 1).text() == b:
                    self.resultLabel.setText(str(i+1))
                    break
                i += 1
        elif not a and b:
            while i < self.tableWidget.rowCount():
                if self.tableWidget.item(i, 1).text() == b:
                    self.resultLabel.setText(str(i+1))
                    break
                i += 1
        elif a and not b:
            while i < self.tableWidget.rowCount():
                if self.tableWidget.item(i, 0) and self.tableWidget.item(i, 0).text() == a:
                    self.resultLabel.setText(str(i+1))
                    break
                i += 1
        if i == self.tableWidget.rowCount():
            self.resultLabel.setText("未找到")


```

#### 保存字频

```python
    def saveWordFrequency(self):
        # 保存文件
        filePath, ok = QFileDialog.getSaveFileName(self, '选择文件')
        if ok:
            with open(filePath, 'w', encoding='utf-8') as file:
                for i in range(self.tableWidget.rowCount()):
                    m = '\t'.join([self.tableWidget.item(
                        i, 0).text(), self.tableWidget.item(i, 1).text()])
                    file.write(m+'\n')
```

#### 构建二叉树

```python
    def generateCharacterSetFromRawtext(self):
        # 根据原文生成字符集
        self.tableWidget.clearContents()
        self.tableWidget.setRowCount(0)
        def getFrequency(text: str) -> dict:
            # 字频(统计)
            cnt = {}
            for i in text:
                if i not in cnt:
                    cnt[i] = 1
                else:
                    cnt[i] += 1
            return cnt
        CharacterSet = getFrequency(rawTextEdit.toPlainText())
        for i, j in CharacterSet.items():
            self.add()
            item1 = QTableWidgetItem(i)
            item2 = QTableWidgetItem(str(j))
            self.tableWidget.setItem(self.tableWidget.rowCount()-1, 0, item1)
            self.tableWidget.setItem(self.tableWidget.rowCount()-1, 1, item2)

    def closeEvent(self, event):
        # 关闭窗体
        if self.tableWidget.rowCount() == 0:
            return
        global CharacterSet
        CharacterSet = {}
        # 将表格中的字符集存入变量CharacterSet中
        for i in range(self.tableWidget.rowCount()):
            if self.tableWidget.item(i, 0) and self.tableWidget.item(i, 1):
                try:
                    CharacterSet[self.tableWidget.item(i, 0).text()] = float(
                        self.tableWidget.item(i, 1).text())
                except:
                    pass
        global HFTree
        # 将树依据现有的字符集进行更新
        if CharacterSet != {}:
            HFTree = HuffmanTree(CharacterSet)
            global showSVGWidget
            if showSVGWidget:
                HFTree.printTree('tmp')
                showSVGWidget.update()
                paintTreeWindow.printInform()
```

### 编码（Encoding）

Encoding is to encode the original text according to the existing tree in the memory, and there are two ways to read the original text, one is manual input, the other is to read the file, and the original text can also be saved.

#### 编码

```python
    def encode(self, text: str) -> str:
        # 对text中的文本进行编码
        p, q = '', ''  # p是每个字符的编码，q是整篇文章的编码
        for i in text:
            for j in self.nodes:
                if i == j.name:
                    while j.parent:
                        if j.parent.lchild == j:
                            p += '0'
                        elif j.parent.rchild == j:
                            p += '1'
                        j = j.parent
                    q += p[::-1]
                    p = ''
                    break
            else:
                # 若当前字符并不在字符集中，则返回空的密文
                return None
        return q
    def encoding(self):
        if not HFTree:
            QMessageBox.critical(self, "错误", "当前无建好的树", QMessageBox.Ok)
        elif rawTextEdit.toPlainText() == '':
            QMessageBox.critical(self, "错误", "请输入原文", QMessageBox.Ok)
        else:
            t = HFTree.encode(rawTextEdit.toPlainText())
            if not t:
                QMessageBox.critical(self, "错误", "存在无效字符", QMessageBox.Ok)
                return
            self.encodedTextEdit.setText(t)
```

#### 文件读入

```python
    def encodeFileReadin(self):
        filePath, ok = QFileDialog.getOpenFileName(self, '选择文件')
        if ok:
            with open(filePath, 'r', encoding='utf-8') as file:
                try:
                    text = file.read()
                except UnicodeDecodeError:
                    QMessageBox.critical(
                        self, "错误", "请确保打开的是UTF-8编码的文本文件", QMessageBox.Ok)
                    return
            self.rawTextEdit.setText(text)    
```
#### 保存原文

```python
def saveRawTextContent(self):
    filePath, ok = QFileDialog.getSaveFileName(self, '选择文件')
    if ok:
        with open(filePath, 'w', encoding='utf-8') as file:
            file.write(self.rawTextEdit.toPlainText())
```

### 译码（Decoding）

Decoding is to decode the ciphertext according to the existing tree in the memory, and there are two ways to read the ciphertext, one is manual input, the other is to read the file, and the ciphertext can also be saved

#### 译码（Decoding）

```python
def decode(self, text: str) -> str:
    # 在树中对text中的01串进行解码
    root: TreeNode = self.rootnode
    result = ""
    for i in text:
        if i == '0':
            root = root.lchild
        elif i == '1':
            root = root.rchild
        elif i == '\n':  # 紧凑格式中的'\n'需忽略
            continue
        else:
            return None
        if root.name:
            result += root.name
            root = self.rootnode
    if root != self.rootnode:
        return None
    else:
        return result
def decoding(self):
    if not HFTree:
        QMessageBox.critical(self, "错误", "当前无建好的树", QMessageBox.Ok)
    elif self.encodedTextEdit.toPlainText() == '':
        QMessageBox.critical(self, "错误", "请输入密文", QMessageBox.Ok)
    else:
        t = HFTree.decode(self.encodedTextEdit.toPlainText())
        if not t:
            QMessageBox.critical(self, "错误", "存在无效字符", QMessageBox.Ok)
            return
        self.rawTextEdit.setText(t)
```

#### 文件读入

```python
    def decodeFileReadin(self):
        filePath, ok = QFileDialog.getOpenFileName(self, '选择文件')
        if ok:
            with open(filePath, 'r', encoding='utf-8') as file:
                try:
                    encodedTextEdit = file.read()
                except UnicodeDecodeError:
                    QMessageBox.critical(
                        self, "错误", "请确保打开的是UTF-8编码的文本文件", QMessageBox.Ok)
                    return
            if not checkDecodedText(encodedTextEdit):
                QMessageBox.critical(self, "错误", "存在无效字符", QMessageBox.Ok)
                return
            self.encodedTextEdit.setText(encodedTextEdit)
③	保存密文
    def saveEncodedTextContent(self):
        if not checkDecodedText(self.encodedTextEdit.toPlainText()):
            QMessageBox.critical(self, "错误", "存在无效字符", QMessageBox.Ok)
            return
        filePath, ok = QFileDialog.getSaveFileName(self, '选择文件')
        if ok:
            with open(filePath, 'w', encoding='utf-8') as file:
                file.write(self.encodedTextEdit.toPlainText())
```

### 打印代码文件（Print）

Here we want to realize the output in a compact format, and to store the file

#### 紧凑格式

```python
def compactFormPrint(self):
    Text = self.encodedTextEdit.toPlainText()
    text = ''
    m = 50
    for i in Text.replace('\n', ''):
        text += i
        m -= 1
        if m == 0:
            text += '\n'
            m = 50
    self.encodedTextEdit.setPlainText(text)
```

#### 存储

```python
def saveEncodedTextContent(self):
    if not checkDecodedText(self.encodedTextEdit.toPlainText()):
        QMessageBox.critical(self, "错误", "存在无效字符", QMessageBox.Ok)
        return
    filePath, ok = QFileDialog.getSaveFileName(self, '选择文件')
    if ok:
        with open(filePath, 'w', encoding='utf-8') as file:
            file.write(self.encodedTextEdit.toPlainText())
```

### 打印哈夫曼树（Tree printing）

Print the Huffman tree, including the information of the spanning tree, the operation of the image in the control, the display of the tree information, the import and storage of the tree, and the related operations of viewing the character set

#### 生成树的图片

```python
def printTree(self, filename=None):
    # 生成树的图片
    dot = Digraph(comment="生成的树")
    dot.attr('node', fontname="STXinwei", shape='circle', fontsize="20")
    for i, j in enumerate(self.nodes):
        if j.name == '' or not j.name:
            dot.node(str(i), '')
        elif j.name == ' ':
            dot.node(str(i), '[ ]')  # 空格显示为'[ ]'
        elif j.name == '\n':
            dot.node(str(i), '\\\\n')  # 换行符显示为'\n' 转义 此处的还会被调用，因此需要四个斜杠
        elif j.name == '\t':
            dot.node(str(i), '\\\\t')  # 制表符显示为'\t'
        else:
            dot.node(str(i), j.name)
    dot.attr('graph', rankdir='LR')
    for i in self.nodes[::-1]:
        if not (i.rchild or i.lchild):
            break
        if i.lchild:
            dot.edge(str(self.nodes.index(i)), str(
                self.nodes.index(i.lchild)), '0', constraint='true')
        if i.rchild:
            dot.edge(str(self.nodes.index(i)), str(
                self.nodes.index(i.rchild)), '1', constraint='true')
    dot.render(filename, view=False, format='svg')
```

#### 树图像的放大缩小

```python
class ShowSVGWidget(QWidget):
    # 自定义控件，显示svg图片
    leftClick: bool
    svgrender: QSvgRenderer
    defaultSize: QSizeF
    point: QPoint
    scale = 1

    def __init__(self, parent=None):
        super().__init__(parent)
        self.parent = parent
        # 构造一张空白的svg图像
        self.svgrender = QSvgRenderer(
            b'<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 0 0"  width="512pt" height="512pt"></svg>')
        # 获取图片默认大小
        self.defaultSize = QSizeF(self.svgrender.defaultSize())
        self.point = QPoint(0, 0)
        self.scale = 1

    def update(self):
        # 更新图片
        self.svgrender = QSvgRenderer("tmp.svg")
        self.defaultSize = QSizeF(self.svgrender.defaultSize())
        self.point = QPoint(0, 0)
        self.scale = 1
        self.repaint()

    def paintEvent(self, a0: QtGui.QPaintEvent) -> None:
        # 绘画事件(回调函数)
        painter = QPainter()  # 画笔
        painter.begin(self)
        self.svgrender.render(painter, QRectF(
            self.point, self.defaultSize*self.scale))  # svg渲染器来进行绘画，(画笔，QRectF(位置，大小))(F表示float)
        painter.end()

    def mouseMoveEvent(self, a0: QtGui.QMouseEvent) -> None:
        # 鼠标移动事件(回调函数)
        if self.leftClick:
            self.endPos = a0.pos()-self.startPos
            self.point += self.endPos
            self.startPos = a0.pos()
            self.repaint()

    def mousePressEvent(self, a0: QtGui.QMouseEvent) -> None:
        # 鼠标点击事件(回调函数)
        if a0.button() == Qt.LeftButton:
            self.leftClick = True
            self.startPos = a0.pos()

    def mouseReleaseEvent(self, a0: QtGui.QMouseEvent) -> None:
        # 鼠标释放事件(回调函数)
        if a0.button() == Qt.LeftButton:
            self.leftClick = False

    def wheelEvent(self, a0: QtGui.QWheelEvent) -> None:
        # 根据光标所在位置进行图像缩放
        oldScale = self.scale
        if a0.angleDelta().y() > 0:
            # 放大
            if self.scale <= 5.0:
                self.scale *= 1.1
        elif a0.angleDelta().y() < 0:
            # 缩小
            if self.scale >= 0.2:
                self.scale *= 0.9
        self.point = a0.pos()-(self.scale/oldScale*(a0.pos()-self.point))
        self.repaint()
```

#### 树信息的显示

```python
def printInform(self):
    # 更新树的信息
    self.treeHeightlabel.setText(str(self.TreeDepth(HFTree)))
    self.nodeCountlabel.setText(str(len(HFTree.characterset)*2-1))
    self.leafCountlabel.setText(str(len(HFTree.characterset)))
```

#### 树文件的读取

```python
def importtree(self):
    # 将树的信息导入到图片中
    filePath, ok = QFileDialog.getOpenFileName(self, '选择文件')
    if ok:
        with open(filePath, 'r', encoding='utf-8') as file:
            try:
                text = file.read()
            except UnicodeDecodeError:
                QMessageBox.critical(
                    self, "错误", "请确保打开的是UTF-8编码的文本文件", QMessageBox.Ok)
                return
        global CharacterSet
        CharacterSet = self.CharacterSet
        textlines = re.findall(r'([\s\S])\t(\S+)\t\S+(\n|$)', text)
        # 导入后重置字符集信息，并更新内存中的树
        for i, j, _ in textlines:
            CharacterSet[i] = float(j)
        global HFTree
        if CharacterSet != {}:
            HFTree = HuffmanTree(CharacterSet)
            global showSVGWidget
            HFTree.printTree('tmp')
            showSVGWidget.update()
            self.printInform()  # 将树的信息写在面板上
```

#### 树的存储

```python
def savetree(self):
    # 保存树的信息
    filePath, ok = QFileDialog.getSaveFileName(self, '选择文件')
    if ok:
        with open(filePath, 'w', encoding='utf-8') as file:
            for i, j in HFTree.characterset.items():
                m = '\t'.join([i, str(j), HFTree.encode(i)])
                file.write(m+'\n')
```

### 网络通信（Network）

Network communication includes server monitoring interface and waiting for connection, client establishing connection, transmission of tree and ciphertext, waiting for reception and disconnection

#### 服务端监听端口

```python
def buildServerConnection(self):
    # 服务端监听端口
    try:
        # 获取端口号
        port = int(self.lineEdit.text())
    except ValueError:
        QMessageBox.critical(self, "错误", "当前无已输入的端口号", QMessageBox.Ok)
        return
    # 建立一个套接字
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    try:
        # 设置监听端口并监听
        s.bind(("0.0.0.0", port))
        s.listen()
    except OSError:
        QMessageBox.critical(self, "错误", "端口已被占用", QMessageBox.Ok)
        return
    # 等待客户端连接
    self.stateLabel.setText("等待连接")
    # 开启一个新的线程用于等待连接，防止程序阻塞，并利用daemon标记，以便于主线程结束时，自动结束带有此标记的所有线程
    threading.Thread(target=self.handleClient,
                     args=[s], daemon=True).start()
```

#### 服务端等待连接

```python
def handleClient(self, s: socket.socket):
    # 服务器端等待连接
    c = s.accept()[0]
    self.stateLabel.setText("已连接")
    self.s = c
    # 启动等待接收的线程
    threading.Thread(target=self.waitRecv, args=[c], daemon=True).start()
```

#### 客户端建立连接

```python
def buildClientConnection(self):
    # 客户端建立连接
    try:
        # 获取IP地址
        ip = self.connectIpEditText.text()
        if ip == None:
            QMessageBox.critical(
                self, "错误", "当前无已输入的IP地址", QMessageBox.Ok)
            return
        port = int(self.connectPortEditText.text())
    except ValueError:
        QMessageBox.critical(self, "错误", "当前无已输入的端口号", QMessageBox.Ok)
        return
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    try:
        s.connect((ip, port))
    except ConnectionRefusedError:
        QMessageBox.critical(self, "错误", "连接失败", QMessageBox.Ok)
        return
    except OSError:
        QMessageBox.critical(self, "错误", "IP或端口错误", QMessageBox.Ok)
        return
    self.stateLabel.setText("已连接")
    self.s = s
    # 连接成功，启动等待接收的线程
    threading.Thread(target=self.waitRecv, args=[s], daemon=True).start()
```

#### 树和密文的传输

```python
def sendTree(self):
    # 发送树
    if not self.s:
        QMessageBox.critical(self, "错误", "请先建立连接", QMessageBox.Ok)
        return
    global CharacterSet
    if not CharacterSet or not HFTree:
        QMessageBox.critical(self, "错误", "当前树为空", QMessageBox.Ok)
        return
    content = 't'  # 发送树的标志
    for i, j in CharacterSet.items():
        content += i+"\t"+str(j)+'\n'
    # 将其转化为Byte进行发送
    self.s.sendall(content.encode())
    QMessageBox.information(self, "提示", "发送成功", QMessageBox.Ok)

def sendText(self):
    # 发送密文
    if not self.s:
        QMessageBox.critical(self, "错误", "请先建立连接", QMessageBox.Ok)
        return
    global encodedTextEdit
    content = encodedTextEdit.toPlainText()
    if not checkDecodedText(content):
        QMessageBox.critical(self, "错误", "存在无效字符", QMessageBox.Ok)
        return
    self.s.sendall(('c'+content).encode())
    QMessageBox.information(self, "提示", "发送成功", QMessageBox.Ok)
```

#### 等待连接

```python
def waitRecv(self, s: socket.socket):
    # 等待接受线程
    try:
        while True:
            data = s.recv(10000000)
            # 将内容转变为str类型
            data = data.decode()
            if data[0] == 't':
                data = data[1:]
                textlines = re.findall(r'([\s\S])\t(\S+)(\n|$)', data)
                global CharacterSet
                CharacterSet = {}
                for i, j, _ in textlines:
                    try:
                        CharacterSet[i] = float(j)
                    except ValueError:
                        self.stateLabel.setText("接收到无用数据")
                        self.tableWidget.clearContents()
                        self.tableWidget.setRowCount(0)
                        CharacterSet = {}
                        return
                global HFTree
                if CharacterSet != {}:
                    HFTree = HuffmanTree(CharacterSet)
                    self.stateLabel.setText("已收到树")
                else:
                    self.stateLabel.setText("收到空树")
            elif data[0] == 'c':
                self.stateLabel.setText("已收到密文")
                data = data[1:]
                self.setEncodedTextSign.emit(data)
            else:
                self.stateLabel.setText("接收到无用数据")
    except ConnectionResetError:  # 对方断开
        self.stateLabel.setText("连接断开")
        self.s = None
    except ConnectionAbortedError:  # 自己断开
        pass
```

#### 断开连接

```python
def breakConnection(self):
    # 断开连接按钮事件
    try:
        self.s.close()
        self.s = None
        self.stateLabel.setText("未连接")
    except:
        pass   
```
## 项目成果功能展示：

​     This chapter mainly introduces the design details and measured performance of the software in detail, so that readers can have a deeper understanding of the meaning of each step and the principle of implementation.

### 初始化（Initialization）

There are two forms of initialization: one is to generate directly according to the text box input, and the other is to directly import characters. On the basis of these two methods, the character set in the list can be added, deleted, and modified. The existing character set can also be used to save the file. Eventually, when the window is closed, the program will build a tree based on the existing character set.

#### 文本输入

There are two main ways to input text, one is to generate it directly according to the text box input, and the other is to import characters directly. There is no obvious difference between the two input methods. When directly importing text, the final text will still be directly displayed in the text box interface. As shown in Figure 2, text initialization:

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);zoom: 50%;" 
    src="./320190903781_周功海数据结构大作业/关键图片/1.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #000;
    padding: 2px;">图1.文本初始化</div>
</center>


#### 额外的增删改查

After inputting or importing text into the text box of "Original:", the function of modifying the text at any time is also provided. That is, we can adjust the content in the text box at any time before encoding to ensure the accuracy of the content we send.

#### 构建二叉树

After inputting the text, click to edit the word frequency, and the UI box as shown in Figure 3 will pop up. This program provides automatic word frequency generation, word frequency import and word frequency saving functions. It still provides two ways to input word frequency for the convenience of users. Daily use, maximize user efficiency.

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);zoom: 50%;" 
    src="./320190903781_周功海数据结构大作业/关键图片/2.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #000;
    padding: 2px;">图2、编辑字频</div>
</center>
 

​     在字频统计好之后程序会根据每个字字频大小确定其权重大小，为之后的编码操作奠定基础。

#### 保存字频

As shown in Figure 3 above, if the text content is large, in order to avoid the need to automatically generate the character set every time, which may take more time, we provide the function of saving the word frequency to facilitate the improvement of efficiency. We can directly operate the binary tree generated by saving the word frequency in different environments and different computers.

### 编码（Encoding）

#### 编码

As shown in Figure 4, the encoding process, clicking the encoding will generate a Huffman tree (Huffmantree) for the nodes of each word determined by the previous word frequency, and then traverse the generated Huffman tree in turn to complete the encoding. Huffman coding the input content to generate ciphertext.

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);zoom: 50%;" 
    src="./320190903781_周功海数据结构大作业/关键图片/3.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #000;
    padding: 2px;">图3、编码过程</div>
</center>

#### 保存原文

The function of saving ciphertext is provided here to facilitate data transmission and recording later.

### 译码（Decoding）

#### 密文导入

As shown in Figure 5, ciphertext import, when we receive the Huffman ciphertext sent by others via network transmission, we can copy the ciphertext into the text box, or choose to import the ciphertext directly

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);zoom: 50%;" 
    src="./320190903781_周功海数据结构大作业/关键图片/4.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #000;
    padding: 2px;">图4、密文导入</div>
</center>


####  译码

When you click Decode, read a series of Huffman sequences in a loop, read "0" from the left child of the root node to continue reading, read "1" to continue from the right child, if you read the left child of a node and Whether the right child is all 0, if it means that a leaf (character) has been read, the translation of a character is successful, the character represented by the leaf node is stored in an array that stores the translated characters, and then continue to read from the root node, Until the Huffman sequence is read, the loop will exit when the terminator is encountered.

Then print the generated characters in the "Original:" text box. Realize the function of text decoding.

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);zoom: 50%;" 
    src="./320190903781_周功海数据结构大作业/关键图片/5.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #000;
    padding: 2px;">图5、译码输出</div>
</center>


#### 文件读入和保存

The decoded text content can be stored locally through "File Save".

### 打印哈夫曼树

####  生成树的图片及存储

Clicking on the operation tree will generate a text image as shown in the figure (the image of the tree displayed is not complete), and you can use the "Save" and "Load" buttons to store or import the tree. This tree has the feature of zooming in and zooming out (it is not convenient to see the details of the tree clearly after zooming out), and it also retains the function of statistically displaying the tree information in the corner of its interface. The content mainly includes the height of the tree, the number of nodes and the number of leaves, etc. In the case of this article, "the height of the tree is 8, the number of nodes of the tree is 213, and the number of leaves of the tree is 107".

In addition, it also provides a search code function, which is mainly to determine the number of character frequencies by querying the content of the character, and then the weight of the character can be determined, and the position of the character in the Huffman tree can be deduced.

If you are not satisfied with the generated Huffman tree, the program also provides the function of deleting or inserting related nodes in the Huffman tree. Just enter the character and its word frequency (ie weight) in the "Edit word frequency" interface, and click "Insert" or "Delete" button can complete the insertion or deletion of the Huffman tree. This function is for us to urgently deal with some emergencies, such as an error occurred during the automatic generation of the word frequency, we can use this function to modify the tree and then the code.

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);zoom: 50%;" 
    src="./320190903781_周功海数据结构大作业/关键图片/6.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #000;
    padding: 2px;">图6、生成图的图片及存储</div>
</center>


### 网络通信

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);zoom: 50%;" 
    src="./320190903781_周功海数据结构大作业/关键图片/7.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #000;
    padding: 2px;">图7、网络通信</div>
</center>

As shown in Figure 8, this program provides the function of querying the local IP address. The local port can be opened as the transmission port of the Huffman code. After connecting the client's IP and receiving port, the local Huffman tree and secret The text is sent to the client together, and the client can import the Huffman tree and run the decoding operation to realize the operation of converting the text.
<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);zoom: 50%;" 
    src="./320190903781_周功海数据结构大作业/关键图片/8.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #000;
    padding: 2px;">图8、网络传输哈夫曼树及密文</div>
</center>

## 4.7 关于界面

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);zoom: 50%;" 
    src="./320190903781_周功海数据结构大作业/关键图片/9.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #000;
    padding: 2px;">图9、关于作者</div>
</center> 

As shown in the figure above, the relevant information about myself is displayed in the about interface, such as: name, school, college, student ID, class, and time to complete the work, etc.
