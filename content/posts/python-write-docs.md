---
title: "使用Python写iOS Word文档"
date: 2020-08-24T18:19:05+08:00
draft: false
tags: ["python"]
categories: []
---


背景：公司要求把项目中的每一个方法都编写入技术文档里。基本格式如下：

```text
方法功能描述

方法定义
类名:
UIView
方法名:
layoutSubviews
方法入参:
无
抛出异常:
无
方法返回值:
void
方法逻辑

```

同事从网上找了一份Python代码，代码分析错误比较多，自己改了一份，可以应付大部分情况。  
**llvm或Xcode的语法分析没找到，还望知道的读者不吝赐教。**  

iOS开发语言分`objective-c`和`swift`，所以语法分析也要写两份。  
首先从`objective-c`开始。

OC语言的方法声明:

```text
- (instancetype)initWithFrame:(CGRect)frame
```

+   方法由 `+` 或 `-` 两个符号开头，表示类方法和实例方法。
+   `(instancetype)`表示返回值类型，没有返回值表示为`(void)`。
+   `initWithFrame:`是方法名字。
+   `(CGRect)frame`是方法需要的参数。

由于需要统计所有的方法，因为`.h`文件方法不全，所以直接从`.m`文件分析。

##### 1、解析出`.m`文件中的所有方法名

`.m`文件里的方法无论分几行，开头是由`+(` 或 `-(` ，结尾是 `{` ，也就是方法实现的第一个大括号，这样解析方法名的逻辑就锁定了。***此处极小概率会把运算符号解析。***

```text
def getOCFileMethdLineCountArray(iOSUrlFilePath):
    arr = []
    iOSUrlFile_object = open(iOSUrlFilePath, 'r')
    methodName = ''
  
    try: 
        for lineIdx,line in enumerate(iOSUrlFile_object):

            # 仅判断符号条件的方法
            if line.startswith('+(') or line.startswith('-(') or line.startswith('+ (') or line.startswith('- (') :
                methodName += line
                # 匹配到方法的大括号
                if '{' in methodName:
                    arr.append((os.path.basename(iOSUrlFilePath).split('.')[0] ,methodName))
                    methodName = ''

                    
    finally:
        iOSUrlFile_object.close()
    return arr
```

##### 2、解析出完整方法中的各项说明

按照文档要求，需要解析出`方法名`，`返回值`，`方法参数`3个内容。  
这里罗列了一些常见的方法实现。

```text
-(NSString *)bankCard:(Int)card apiKey:(NSString *)apiKey secretKey:(NSString *)secretKey cardImageUrl:(NSString * _Nonnull)imageUrl block:(cardOCRBlock _Nullable)completion {
-(NSString *)stringUrlEncodeing:(NSString *)str {
-(id)init{
-(void)cancelRequest {
- (id)initWithFrame:(CGRect)frame{
- (void)updateWithConfig: (void(^)(Config *config))block  {
- (NSMutableArray<UIButton *> *)itemBtns{
- (void)layoutSubviews {
- (void(^)(TestModel * ))initWithBlock:(void(^)(NSString*))block{
```

如果从头部解析，复杂度比较高。从尾部开始反而会比较简单，像贪吃蛇一样，一点一点从尾部吃掉内容。

```text
def anyliseOC(text):
    
    methodName = ''
    params = []
    returnValue = ''

    # 去掉关键字
    repArray = ['_Nonnull','_Nullable','__nullable','__nonnull','+','-']
    ss1 = text.replace('{','')
    for rep in repArray:
        ss1 = ss1.replace(rep,'')
    # 反向解析参数
    rfIndex = ss1.rfind(':')
    if rfIndex != -1:
        while rfIndex != -1:
            # 最后的参数部分
            pas = ss1[rfIndex+1:].strip()
            # 去掉参数类型的左右两端小括号
            pas = pas.replace('(','',1)
            rIdx = pas.rfind(')')
            if rIdx != -1:
                pas = pas[:rIdx] + pas[rIdx+1:]
            # 多个空格合并成一个空格,执行2次空格替换。
            pas = pas.replace('  ',' ').replace('  ',' ').strip()
            # 添加参数数组中
            params.insert(0,pas)

            # 截掉解析过的尾部
            ss1 = ss1[:rfIndex+1].strip()
            # 最后方法名。向前解析，是否遇到空格或右括号
            spIndex = ss1.rfind(' ',0,rfIndex)
            spIndex1 = ss1.rfind(')',0, rfIndex)
            # 先找到有括号
            if spIndex1 > spIndex:
                spIndex = spIndex1
            # 解析到方法名
            if spIndex != -1:
                fc = ss1[spIndex+1:rfIndex+1]
                methodName = fc + methodName 

            # 截掉解析过的尾部
            ss1 = ss1[:spIndex+1]

            # 开始下一次解析
            rfIndex = ss1.rfind(':')
            
        
    else:
        # 没有参数的方法
        # 方法名。向前解析，是否遇到空格或右括号
            ss1 = ss1.strip()
            spIndex = ss1.rfind(' ')
            spIndex1 = ss1.rfind(')')
            # 先找到有括号
            if spIndex1 > spIndex:
                spIndex = spIndex1
            # 解析到方法名
            if spIndex != -1:
                fc = ss1[spIndex+1:]
                methodName = fc + methodName 
            
            # 截掉解析过的尾部
            ss1 = ss1[:spIndex]
            
    # 返回值处理两端的括号，最后的* 
    returnValue = ss1.strip()
    if returnValue.startswith('('):
        returnValue = returnValue[1:].strip()
    if returnValue.endswith(')'):
        returnValue = returnValue[:-1].strip()
    if returnValue.endswith('*'):
        returnValue = returnValue[:-1].strip()
    print('方法名：{}'.format(methodName))
    print('参数：{}'.format(params))
    print('返回值：{}'.format(returnValue))

```

##### 4、解析swift文件中的所有方法。

`swift`语法中，`func`为关键字。直接搬运同事的代码，稍作修改。这里采用从头到尾读取解析。像`init` ，`deinit`这类没有`func`关键字的方法无法准确识别，略有瑕疵。

```text
def getSwiftFileMethdLineCountArray(iOSUrlFilePath):
    arr = []

    iOSUrlFile_object = open(iOSUrlFilePath, 'r')
    # methodName = ''
    
    fc1 = re.compile(r'[,:=]+')
    try: 
        lines = enumerate(iOSUrlFile_object)
        classLine = ''
        startClass = False
        funcLine = ''
        startFunc = False
        for lineIdx,line in lines:
            # 去除空格,换行符,制表符
            # line = "".join(line.split())
            #判断是否是空行或注释行
            if not len(line)  or line.replace(' ','').startswith('/*') or line.replace(' ','').startswith(' *') or line.replace(' ','').startswith('>>>') or line.replace(' ','').startswith('//') or line.replace(' ','').startswith('*/') or line.replace(' ','').startswith('///'):  
                continue
            else:
                line = line.replace('  ','')
                
                if  line.startswith('init') or \
                    line.startswith('deinit') or \
                    " init" in line or \
                    "init " in line or \
                    "func " in line or \
                    " func" in line or \
                    "func\n" in line or \
                    startFunc :
                    # 如果init前面含有符号,这应该是使用，而不是声明。
                    indexf = line.find(' init')
                    if indexf < 0:
                        indexf = line.find('init ')
                    if indexf >= 0 and re.search(fc1,line[:indexf]) is not None :
                        continue

                    funcLine += line.replace('\n',' ').replace('  ','')
                    if line.find('{') < 0:
                        startFunc = True
                        continue
                    else:
                        startFunc = False
                        arr.append((os.path.basename(iOSUrlFilePath).split('.')[0] ,funcLine))
                        funcLine = ''
    finally:
        iOSUrlFile_object.close()
    return arr

```

##### 5、解析`swift`方法中的各项内容

`swift`方法使用`func`关键字，方法说明和各项参数都是放在`()`之内，返回值是在 `->`到 `{` 符号之间，所以这里从头尾到中间的方式读取解析。

> **这里的方法名分析比较简易，没有去掉参数，有需要的读者还需进一步补充逻辑。**

```text
def anyliseSwift(oriText):
    
    # 去除方法名中的一些关键字
    oriText = oriText.replace('@escaping','').replace('public','')\
        .replace('override','').replace('required','').replace('open','')\
            .replace('private','').replace('convenience','').strip()
    # 获取方法完整名。
    funcName = u''
    beganIndex = oriText.find('func ')
    endIndex = oriText.find('{')
    if beganIndex != -1 :
        # 加上func字符串长度
        funcName = oriText[beganIndex + 5:endIndex].strip()
    else:
        # 没有func开头的方法名不在额外处理
        funcName = oriText[:endIndex].strip()

    # 获取返回值
    retIndex = oriText.rfind('->')
    retValue = u"无"
    if retIndex != -1:
        retValue = oriText[retIndex+2:endIndex].strip()
    # 这里会因为闭包解析出错，小括号会被拆分,造成左右不等量，处理一下。
    if retValue.count('(') != retValue.count(')'):
        retValue = u"无"
        retIndex = endIndex

    # 获取参数
    pStart = oriText.find('(')
    pEnd = oriText.rfind(')',0, retIndex)
    fullParam = oriText[pStart+1:pEnd].strip()
    params = fullParam.split(',')
    if len(params[0]) == 0:
        params[0] = u'无'
    else:
        #格式化每个参数
        for i,p in enumerate(params):
            # 去掉开头的下划线，去掉第一个分号，双空格替换成一个。
            if p.startswith('_'):
                p = p[1:]
            p = p.strip().replace(':',' ',1).replace('  ',' ')
            # 去掉方法中的参数说明。
            if len(p.split(' ')) == 3:
                idx = p.find(' ')
                p = p[idx:].strip()
            params[i] = p
    
    return (funcName,params,retValue)

```

##### 6、生成Word文档。

```text
# 方法的固定功能说明。
def generateSumary(oriText,className):
    if oriText.find('init') >= 0:
        return u"类初始化。"
    elif oriText.lower().find('viewdidload') >= 0:
        return u"页面加载完成。"
    elif oriText.lower().find('viewdidappear') >= 0:
        return "页面已经显示。"
    elif oriText.lower().find('viewwillappear') >= 0:
        return "页面将要显示。"
    elif oriText.lower().find('viewwilldisappear') >= 0:
        return u"页面将要消失。"
    elif oriText.lower().find('viewdiddisappear') >= 0:
        return u"页面已经消失。"
    else:
        return u""

# 设置段落格式
def setText(doc,text,size,bold):
    p1 = doc.add_paragraph()
    p1.paragraph_format.first_line_indent = Cm(0.74)  
    p1.paragraph_format.line_spacing = 1.5 # 1.5倍行距 
    l1 = p1.add_run(text)
    l1.font.size = size
    l1.font.name = u"宋体"
    l1.bold = bold
        

# 具体内容添加
def addDetail(doc,text1,textArr):
    setText(doc,text1,Pt(14),True)
    for text in textArr:
        # print(text+'\n')
        setText(doc,text,Pt(14),False)

# 将源文件分析完成后，输出到目标目录下
def writeToWord(sourcePath,outPath):
    # 创建空文档
    document = Document()
    document.styles['Normal'].font.name = u'宋体'
    document.styles['Normal']._element.rPr.rFonts.set(qn('w:eastAsia'), u'宋体')
    # 添加标题，设置级别level，0为Title，1或省略为Heading 1，0<=level<=9
    # document.add_heading('Document Title', 0)

    path = sourcePath+''
    arrT = []

    for dirpath, dirname, filenames in os.walk(path):   
        for filename in filenames:
            if '.m' in filename or '.mm' in filename:
                filePath = os.path.join(dirpath, filename)
                MacroWithCountDicts = getOCFileMethdLineCountArray(filePath)
                print('=====================')
                print('{} 有 {} 个方法'.format(filename,len(MacroWithCountDicts)))
                arrT += MacroWithCountDicts

            elif '.swift' in filename:
                filePath = os.path.join(dirpath, filename)
                MacroWithCountDicts = getSwiftFileMethdLineCountArray(filePath)
                print('=====================')
                print('{} 有 {} 个方法'.format(filename,len(MacroWithCountDicts)))
                arrT += MacroWithCountDicts


    print("********************************")
    print("一共有 {} 个方法".format(len(arrT)))
    print("********************************")
    for result in arrT:
        fullFuncName = result[1]
        allMsg = []
        if fullFuncName.strip().startswith('+') or fullFuncName.strip().startswith('-'):
            allMsg = anyliseOC(fullFuncName)
        else:
            allMsg = anyliseSwift(fullFuncName)

        className = result[0]
        funcName = allMsg[0]
        paras = allMsg[1]
        returnStr = allMsg[2]
        # # 函数名设置为4级标题
        document.add_heading(funcName,level=4)

        # #功能设置为5级标题
        document.add_heading(u"方法功能描述",level=5)

        setText(document,generateSumary(funcName,className),Pt(14),False)

        # 方法定义设置为5级标题
        document.add_heading(u"方法定义",level=5)
        addDetail(document,u"类名:",[className])
        addDetail(document,u"方法名:",[funcName])
        addDetail(document,u"方法入参:",paras)
        addDetail(document,u"抛出异常:",[u"无"])
        addDetail(document,u"方法返回值:",[returnStr])

        # 方法的逻辑设置为5级标题
        document.add_heading(u"方法逻辑",level=5)
        # 一些固定方法填充内容。
        # if 'NSCoder' in funcName:
        #     setText(document,u"调用fatalError方法。",Pt(14),False)
        # elif 'init' in funcName:
        #     setText(document,u"父类初始化，",Pt(14),False)
        # else:
        setText(document,u"",Pt(14),False)

    try:
        document.save(outPath+"生成iOS方法文档.docx")
    except Exception:
        print(u"检查下当前目录是否存在,是否包含/")
    else:
        print(u"完成,请打开"+outPath+u"目录下的文件")
        
  
```

虽然这份Python代码还有诸多缺陷，但已达到预计实现的90%以上，没有在继续完善。如果没有达到你的要求，自己动手补充一下吧！欢迎反馈补充。
