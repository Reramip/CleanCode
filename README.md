# CleanCode

## 命名

### 方法名

方法名应当是动词或动词短语，如save, deletePage。 \
属性访问器、修改器、断言根据其值依JavaBean标准命名为get, set, is。 \

    ```java
    String name = employee.getName();
    customer.setName("Tom");
    if (paycheck.isPosted())...
    ```

重载构造函数，使用描述参数的静态工厂方法更优。

    ```java
    Complex fulcrumPoint = Complex.FromRealNumber(23.0);
    ```

通常好于

    ```java
    Complex fulcrumPoint = new Complex(23.0);
    ```

### 每个概念确定一个词

避免DeviceManager, ProtocolController类似词语混用，应统一为xxManager, xxController或xxDriver

### 使用尽量精确的名称

MAC地址、端口地址、Web地址相区别，使用MAC, PostalAddress, URI这样的精确的名字

## 函数

### 短小

每个函数不应过长，数行为佳，例如：

    ```java
    public static String renderPageWithSetupAndTeardowns(
            PageData pageData, boolean isSuite) throws Exception {
        if (isTestPage(pageData)){
            includeSetipAndTeardownPages(pageData, isSuite);
        }
        return pageData.getHtml();
    }
    ```

if, else, while语句中的代码块尽量只有一行，调用一个拥有较具说明性的函数名称的函数，增加文档上的价值，易于阅读与理解。

### 只做一件事

函数应该只做一件事，只做同一个抽象层级上的步骤。要判断函数是否只做了一件事，看它是否还能在拆分出函数。 \
一个函数中的语句应在一个抽象层级上，基础与细节不能混杂在一起。 \
要让代码具有自顶向下的阅读顺序，每个函数后跟着下一抽象层级的函数。逻辑上，每个函数形如要……就要……如果……就……。例如：

- 要容纳设置与分拆步骤，就先容纳设置步骤，然后纳入测试内容，再纳入分拆步骤
- 要容纳设置步骤，如果是套件，就纳入套件设置步骤，然后再纳入普通设置步骤
- 要容纳套件设置步骤，先搜索"SuiteSetUp"页面的继承关系，再添加包含页面路径的语句
- 要搜索"SuiteSetUp"页面，就要……

### 函数名称

函数命名要保持一致，一个模块内的名称采用一脉相承的描述性短语

### 函数参数

越少越好。如果函数看起来需要很多(3个或3个以上)参数，可能某些参数需要封装成类了。如：

    ```java
    Circle makeCircle(double x, double y, double radius);
    Circle makeCircle(Point center, double radius);
    ```

函数的输出最好通过返回值体现，而不是在参数中输出。习惯上，信息通过函数输入参数，通过返回值从函数中输出。 \
不要向函数传入boolean值标识参数，这等于大声宣布本函数不仅做一件事。 \
对于一元函数，函数名与参数形成良好的动宾形式，如：`write(name)`, `writeField(name)`。 \
也可以在函数名称中体现关键字。`assertExpectedEqualsActual(expected, actual)`优于`assertEqual(expected, actual)`。

### 无副作用

反例：

    ```java
    public class UserValidator {
        private Cryptographer cryptographer;

        public boolean checkPassword(String userName, String password) {
            User user = userGateway.findByName(useeName);
            if (user != user.NULL) {
                String codePhrase = user.getPhraseEncodedByPassword();
                String phrase = cryptographer.decrypt(codedPhrase, password);
                if ("Valid password".equals(phrase)) {
                    Session.initialize();
                    return true;
                }
            }
            return false;
        }
    }
    ```

副作用在于调用了`Session.initialize()`。函数名称并未体现初始化会话的功能，可能导致调用者顾名思义而误操作。这违反了函数“只做一件事”的规则。

### 将指令与查询分开

避免设计使用`if (set("username", "unclebob"))...`这种将判断与指令杂糅的函数，应将它们分开：

    ```java
    if (attributeExists("username")) {
        setAttribute("username", "unclebob");
    }
    ```

这样的代码可读性更高。

### 使用异常

与TIJ中所讲的类似，使用异常可以将函数中的错误处理单独拎出来，减小使用if语句进行错误判断带来的层层嵌套。 \
在CleanCode中，作者鼓励将try/catch代码块抽离出去另外形成函数使得代码结构规整美观。

    ```java
    public void delete(Page page) {
        try {
            deletePageAndAllReferences(page);
        } catch (Exception e) {
            logError(e);
        }
    }

    private void deletePageAndAllReferences(Page page) throws Exception {
        deletePage(page);
        registry.deleteReference(page.name);
        configKeys.deleteKey(page.name.makeKey());
    }

    private void logError(Exception e) {
        logger.log(e.getMessage());
    }
    ```

错误处理本来就是一件事，函数应该专注于一件事，所以`delete`函数只与错误处理相关。 \
而`deletePageAndAllReferences`函数只与删除页面有关。 \
`logError`只与记录异常有关。

### 如何写出这样的函数

写代码如同写文章。先写出粗陋的底稿，在此之上不断打磨成型。将写代码当作讲故事。
