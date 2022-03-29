# SpringBoot 实现 excel 全自由导入导出，性能强的离谱，用起来还特优雅 - 掘金
### 一、简介

今天我给大家推荐一款性能更好的 Excel 导入导出工具：EasyExcel，希望对大家有所帮助！

easyexcel 是阿里开源的一款 Excel 导入导出工具，具有处理速度快、占用内存小、使用方便的特点，底层逻辑也是基于 apache poi 进行二次开发的，目前的应用也是非常广！

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/16941b181b85409d96840ef19798d49a~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)
 相比 EasyPoi，EasyExcel 的处理数据性能非常高，读取 75M (46W 行 25 列) 的 Excel，仅需使用 64M 内存，耗时 20s，极速模式还可以更快！

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a5dbdde37792450fab808be4dd4e2079~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)
 废话也不多说了，下面直奔主题！

### 二、实践

在 SpringBoot 项目中集成 EasyExcel 其实非常简单，仅需一个依赖即可。

```&lt;!--EasyExcel相关依赖-->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>easyexcel</artifactId>
    <version>3.0.5</version>
</dependency>

```

EasyExcel 的导出导入支持两种方式进行处理

-   第一种是通过实体类注解方式来生成文件和反解析文件数据映射成对象
-   第二种是通过动态参数化生成文件和反解析文件数据

下面我们以用户信息的导出导入为例，分别介绍两种处理方式。

#### 简单导出

首先，我们只需要创建一个`UserEntity`用户实体类，然后添加对应的注解字段即可，示例代码如下

```public

    @ExcelProperty(value = "姓名")
    private String name;
	
    @ExcelProperty(value = "年龄")
    private int age;
	
    @DateTimeFormat("yyyy-MM-dd HH:mm:ss")
    @ExcelProperty(value = "操作时间")
    private Date time;
	
    //set、get...
}

```

然后，使用 EasyExcel 提供的 EasyExcel 工具类，即可实现文件的导出。

```public
    List<UserWriteEntity> dataList = new ArrayList<>();
    for (int i = 0; i < 10; i++) {
        UserWriteEntity userEntity = new UserWriteEntity();
        userEntity.setName("张三" + i);
        userEntity.setAge(20 + i);
        userEntity.setTime(new Date(System.currentTimeMillis() + i));
        dataList.add(userEntity);
    }
    //定义文件输出位置
    FileOutputStream outputStream = new FileOutputStream(new File("/Users/panzhi/Documents/easyexcel-export-user1.xlsx"));
    EasyExcel.write(outputStream, UserWriteEntity.class).sheet("用户信息").doWrite(dataList);
}

```

运行程序，打开文件内容结果！

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/11ded08d4b3e4b4fb0cae662e816314c~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

#### 简单导入

这种简单固定表头的 Excel 文件，如果想要读取文件数据，操作也很简单。

以上面的导出文件为例，使用 EasyExcel 提供的`EasyExcel`工具类，即可来实现文件内容数据的快速读取，示例代码如下：

首先创建读取实体类

```/\*\*
 * 读取实体类
 */
public class UserReadEntity {

    @ExcelProperty(value = "姓名")
    private String name;
    /**
     * 强制读取第三个 这里不建议 index 和 name 同时用，要么一个对象只用index，要么一个对象只用name去匹配
     */
    @ExcelProperty(index = 1)
    private int age;
	
    @DateTimeFormat("yyyy-MM-dd HH:mm:ss")
    @ExcelProperty(value = "操作时间")
    private Date time;
	
    //set、get...
}

```

然后读取文件数据，并封装到对象里面

```public
    //同步读取文件内容
    FileInputStream inputStream = new FileInputStream(new File("/Users/panzhi/Documents/easyexcel-user1.xls"));
    List<UserReadEntity> list = EasyExcel.read(inputStream).head(UserReadEntity.class).sheet().doReadSync();
    System.out.println(JSONArray.toJSONString(list));
}


```

运行程序，输出结果如下：

```\[{"age":20,"name":"张三0","time":1616920360000},{"age":21,"name":"张三1","time":1616920360000},{"age":22,"name":"张三2","time":1616920360000},{"age":23,"name":"张三3","time":1616920360000},{"age":24,"name":"张三4","time":1616920360000},{"age":25,"name":"张三5","time":1616920360000},{"age":26,"name":"张三6","time":1616920360000},{"age":27,"name":"张三7","time":1616920360000},{"age":28,"name":"张三8","time":1616920360000},{"age":29,"name":"张三9","time":1616920360000}]

```

#### 动态自由导出导入

在实际使用开发中，我们不可能每来一个 excel 导入导出需求，就编写一个实体类，很多业务需求需要根据不同的字段来动态导入导出，没办法基于实体类注解的方式来读取文件或者写入文件。

因此，基于`EasyExcel`提供的动态参数化生成文件和动态监听器读取文件方法，我们可以单独封装一套动态导出导出工具类，省的我们每次都需要重新编写大量重复工作，以下就是小编我在实际使用过程，封装出来的工具类，在此分享给大家！

-   首先，我们可以编写一个动态导出工具类

```public

    private static final Logger log = LoggerFactory.getLogger(DynamicEasyExcelExportUtils.class);

    private static final String DEFAULT_SHEET_NAME = "sheet1";

    /**
     * 动态生成导出模版(单表头)
     * @param headColumns 列名称
     * @return            excel文件流
     */
    public static byte[] exportTemplateExcelFile(List<String> headColumns){
        List<List<String>> excelHead = Lists.newArrayList();
        headColumns.forEach(columnName -> { excelHead.add(Lists.newArrayList(columnName)); });
        byte[] stream = createExcelFile(excelHead, new ArrayList<>());
        return stream;
    }

    /**
     * 动态生成模版(复杂表头)
     * @param excelHead   列名称
     * @return
     */
    public static byte[] exportTemplateExcelFileCustomHead(List<List<String>> excelHead){
        byte[] stream = createExcelFile(excelHead, new ArrayList<>());
        return stream;
    }

    /**
     * 动态导出文件（通过map方式计算）
     * @param headColumnMap  有序列头部
     * @param dataList       数据体
     * @return
     */
    public static byte[] exportExcelFile(LinkedHashMap<String, String> headColumnMap, List<Map<String, Object>> dataList){
        //获取列名称
        List<List<String>> excelHead = new ArrayList<>();
        if(MapUtils.isNotEmpty(headColumnMap)){
            //key为匹配符，value为列名，如果多级列名用逗号隔开
            headColumnMap.entrySet().forEach(entry -> {
                excelHead.add(Lists.newArrayList(entry.getValue().split(",")));
            });
        }
        List<List<Object>> excelRows = new ArrayList<>();
        if(MapUtils.isNotEmpty(headColumnMap) && CollectionUtils.isNotEmpty(dataList)){
            for (Map<String, Object> dataMap : dataList) {
                List<Object> rows = new ArrayList<>();
                headColumnMap.entrySet().forEach(headColumnEntry -> {
                    if(dataMap.containsKey(headColumnEntry.getKey())){
                        Object data = dataMap.get(headColumnEntry.getKey());
                        rows.add(data);
                    }
                });
                excelRows.add(rows);
            }
        }
        byte[] stream = createExcelFile(excelHead, excelRows);
        return stream;
    }


    /**
     * 生成文件（自定义头部排列）
     * @param rowHeads
     * @param excelRows
     * @return
     */
    public static byte[] customerExportExcelFile(List<List<String>> rowHeads, List<List<Object>> excelRows){
        //将行头部转成easyexcel能识别的部分
        List<List<String>> excelHead = transferHead(rowHeads);
        return createExcelFile(excelHead, excelRows);
    }

    /**
     * 生成文件
     * @param excelHead
     * @param excelRows
     * @return
     */
    private static byte[] createExcelFile(List<List<String>> excelHead, List<List<Object>> excelRows){
        try {
            if(CollectionUtils.isNotEmpty(excelHead)){
                ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
                EasyExcel.write(outputStream).registerWriteHandler(new LongestMatchColumnWidthStyleStrategy())
                        .head(excelHead)
                        .sheet(DEFAULT_SHEET_NAME)
                        .doWrite(excelRows);
                return outputStream.toByteArray();
            }
        } catch (Exception e) {
            log.error("动态生成excel文件失败，headColumns：" + JSONArray.toJSONString(excelHead) + "，excelRows：" + JSONArray.toJSONString(excelRows), e);
        }
        return null;
    }

    /**
     * 将行头部转成easyexcel能识别的部分
     * @param rowHeads
     * @return
     */
    public static List<List<String>> transferHead(List<List<String>> rowHeads){
        //将头部列进行反转
        List<List<String>> realHead = new ArrayList<>();
        if(CollectionUtils.isNotEmpty(rowHeads)){
            Map<Integer, List<String>> cellMap = new LinkedHashMap<>();
            //遍历行
            for (List<String> cells : rowHeads) {
                //遍历列
                for (int i = 0; i < cells.size(); i++) {
                    if(cellMap.containsKey(i)){
                        cellMap.get(i).add(cells.get(i));
                    } else {
                        cellMap.put(i, Lists.newArrayList(cells.get(i)));
                    }
                }
            }
            //将列一行一行加入realHead
            cellMap.entrySet().forEach(item -> realHead.add(item.getValue()));
        }
        return realHead;
    }

    /**
     * 导出文件测试
     * @param args
     * @throws IOException
     */
    public static void main(String[] args) throws IOException {
        //导出包含数据内容的文件（方式一）
        LinkedHashMap<String, String> headColumnMap = Maps.newLinkedHashMap();
        headColumnMap.put("className","班级");
        headColumnMap.put("name","学生信息,姓名");
        headColumnMap.put("sex","学生信息,性别");
        List<Map<String, Object>> dataList = new ArrayList<>();
        for (int i = 0; i < 5; i++) {
            Map<String, Object> dataMap = Maps.newHashMap();
            dataMap.put("className", "一年级");
            dataMap.put("name", "张三" + i);
            dataMap.put("sex", "男");
            dataList.add(dataMap);
        }
        byte[] stream1 = exportExcelFile(headColumnMap, dataList);
        FileOutputStream outputStream1 = new FileOutputStream(new File("/Users/panzhi/Documents/easyexcel-export-user5.xlsx"));
        outputStream1.write(stream1);
        outputStream1.close();


        //导出包含数据内容的文件（方式二）
        //头部，第一层
        List<String> head1 = new ArrayList<>();
        head1.add("第一行头部列1");
        head1.add("第一行头部列1");
        head1.add("第一行头部列1");
        head1.add("第一行头部列1");
        //头部，第二层
        List<String> head2 = new ArrayList<>();
        head2.add("第二行头部列1");
        head2.add("第二行头部列1");
        head2.add("第二行头部列2");
        head2.add("第二行头部列2");
        //头部，第三层
        List<String> head3 = new ArrayList<>();
        head3.add("第三行头部列1");
        head3.add("第三行头部列2");
        head3.add("第三行头部列3");
        head3.add("第三行头部列4");

        //封装头部
        List<List<String>> allHead = new ArrayList<>();
        allHead.add(head1);
        allHead.add(head2);
        allHead.add(head3);

        //封装数据体
        //第一行数据
        List<Object> data1 = Lists.newArrayList(1,1,1,1);
        //第二行数据
        List<Object> data2 = Lists.newArrayList(2,2,2,2);
        List<List<Object>> allData = Lists.newArrayList(data1, data2);

        byte[] stream2 = customerExportExcelFile(allHead, allData);
        FileOutputStream outputStream2 = new FileOutputStream(new File("/Users/panzhi/Documents/easyexcel-export-user6.xlsx"));
        outputStream2.write(stream2);
        outputStream2.close();


    }
}

```

然后，编写一个动态导入工具类

```/\*\*
 * 创建一个文件读取监听器
 */
public class DynamicEasyExcelListener extends AnalysisEventListener<Map<Integer, String>> {
    private static final Logger LOGGER = LoggerFactory.getLogger(UserDataListener.class);
    /**
     * 表头数据（存储所有的表头数据）
     */
    private List<Map<Integer, String>> headList = new ArrayList<>();
    /**
     * 数据体
     */
    private List<Map<Integer, String>> dataList = new ArrayList<>();
    /**
     * 这里会一行行的返回头
     *
     * @param headMap
     * @param context
     */
    @Override
    public void invokeHeadMap(Map<Integer, String> headMap, AnalysisContext context) {
        LOGGER.info("解析到一条头数据:{}", JSON.toJSONString(headMap));
        //存储全部表头数据
        headList.add(headMap);
    }
    /**
     * 这个每一条数据解析都会来调用
     *
     * @param data
     *            one row value. Is is same as {@link AnalysisContext#readRowHolder()}
     * @param context
     */
    @Override
    public void invoke(Map<Integer, String> data, AnalysisContext context) {
        LOGGER.info("解析到一条数据:{}", JSON.toJSONString(data));
        dataList.add(data);
    }
    /**
     * 所有数据解析完成了 都会来调用
     *
     * @param context
     */
    @Override
    public void doAfterAllAnalysed(AnalysisContext context) {
        // 这里也要保存数据，确保最后遗留的数据也存储到数据库
        LOGGER.info("所有数据解析完成！");
    }
    public List<Map<Integer, String>> getHeadList() {
        return headList;
    }
    public List<Map<Integer, String>> getDataList() {
        return dataList;
    }
}

```

-   动态导入工具类

```/\*\*
 * 编写导入工具类
 */
public class DynamicEasyExcelImportUtils {
    /**
     * 动态获取全部列和数据体，默认从第一行开始解析数据
     * @param stream
     * @return
     */
    public static List<Map<String,String>> parseExcelToView(byte[] stream) {
        return parseExcelToView(stream, 1);
    }
    /**
     * 动态获取全部列和数据体
     * @param stream           excel文件流
     * @param parseRowNumber   指定读取行
     * @return
     */
    public static List<Map<String,String>> parseExcelToView(byte[] stream, Integer parseRowNumber) {
        DynamicEasyExcelListener readListener = new DynamicEasyExcelListener();
        EasyExcelFactory.read(new ByteArrayInputStream(stream)).registerReadListener(readListener).headRowNumber(parseRowNumber).sheet(0).doRead();
        List<Map<Integer, String>> headList = readListener.getHeadList();
        if(CollectionUtils.isEmpty(headList)){
            throw new RuntimeException("Excel未包含表头");
        }
        List<Map<Integer, String>> dataList = readListener.getDataList();
        if(CollectionUtils.isEmpty(dataList)){
            throw new RuntimeException("Excel未包含数据");
        }
        //获取头部,取最后一次解析的列头数据
        Map<Integer, String> excelHeadIdxNameMap = headList.get(headList.size() -1);
        //封装数据体
        List<Map<String,String>> excelDataList = Lists.newArrayList();
        for (Map<Integer, String> dataRow : dataList) {
            Map<String,String> rowData = new LinkedHashMap<>();
            excelHeadIdxNameMap.entrySet().forEach(columnHead -> {
                rowData.put(columnHead.getValue(), dataRow.get(columnHead.getKey()));
            });
            excelDataList.add(rowData);
        }
        return excelDataList;
    }
    /**
     * 文件导入测试
     * @param args
     * @throws IOException
     */
    public static void main(String[] args) throws IOException {
        FileInputStream inputStream = new FileInputStream(new File("/Users/panzhi/Documents/easyexcel-export-user5.xlsx"));
        byte[] stream = IoUtils.toByteArray(inputStream);
        List<Map<String,String>> dataList = parseExcelToView(stream, 2);
        System.out.println(JSONArray.toJSONString(dataList));
        inputStream.close();
    }
}

```

为了方便后续的操作流程，在解析数据的时候，会将列名作为`key`！

### 三、小结

在实际的业务开发过程中，根据参数动态实现 Excel 的导出导入还是非常广的。

当然，EasyExcel 的功能还不只上面介绍的那些内容，还有基于模版进行 excel 的填充，web 端 restful 的导出导出，使用方法大致都差不多，更多的功能，会在后续的文章中再次介绍，如果有描述不对的地方，欢迎网友批评吐槽！

\*\* **更多关于 Java 技术问题，欢迎大家关注我的微信：[docs.qq.com/doc/DQ2Z0eE…](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.qq.com%2Fdoc%2FDQ2Z0eE1aUmlITnNz%25C2%25A0 "https&#x3A;//docs.qq.com/doc/DQ2Z0eE1aUmlITnNz%C2%A0") 备注【666】有我准备的一线程序必备计算机书籍，JAVA 课件，源码，安装包等资料希望可以帮助大家提升技术和能力**\*\* 
 [https://juejin.cn/post/7072275783758118919](https://juejin.cn/post/7072275783758118919)
