# 用反射实现实体类数据导出excel

```java
// 参数解释：
// 1. title, Map<Integer, String[]>
//	1.1 key - Integer, 列数
//	1.2 value - String[] 容量为2的数组，0位置存储实体类属性名，1位置存储excel中该列标题
// 2. data, 需要导出的数据
// 3. clz, 需要导出的实体类的class
// 4. 文件名
public <T> HSSFWorkbook fillExcel(Map<Integer, String[]> title, List<T> data, Class<T> clz, String fileName) {
    HSSFWorkbook workbook = new HSSFWorkbook();
    HSSFSheet sheet = workbook.createSheet(fileName);
    sheet.setDefaultColumnWidth(15);

    HSSFRow titleRow = sheet.createRow(0);
    for (int i = 0; i < title.size(); ++i) {
        titleRow.createCell(i).setCellValue(title.get(i)[1]);
    }

    int dataRow = 1;
    for (int rowNum = 0; rowNum < data.size(); ++rowNum) {
        HSSFRow row = sheet.createRow(rowNum + dataRow);
        for (int colNum = 0; colNum < title.size(); ++colNum) {
            String propertyName = title.get(colNum)[0];
            Object val = null;
            try {
                //反射获取属性值
                Field field = clz.getDeclaredField(propertyName);
                //获取private值
                field.setAccessible(true);
                val = field.get(data.get(rowNum));
            } catch (Exception e) {
                e.printStackTrace();
            }

            if (val == null) {
                row.createCell(colNum).setCellValue("");
            } else if (val instanceof String) {
                row.createCell(colNum).setCellValue((String) val);
            } else if (val instanceof Integer) {
                row.createCell(colNum).setCellValue((Integer) val);
            } else if (val instanceof Double) {
                row.createCell(colNum).setCellValue((Double) val);
            } else {
                row.createCell(colNum).setCellValue(val + "");
            }
        }
    }
    return workbook;
}
```

**调用举例**

实体类

```java
//lombok省略getter、setter
@Data
public class User {
    private String name;
    private Integer age;
    private String city;
    private Integer grade;
    public User() {
        
    }
    public User(String name, Integer age, String city, Integer grade) {
        this.name = name;
        this.age = age;
        this.city = city;
        this.grade = grade;
    }
}
```

调用举例

```java
private void exportUser() {
    List<User> userList = new ArrayList<>();
    userList.add(new User("张三",14,"广州市",8));
    userList.add(new User("李四",18,"北京市",12));
    userList.add(new User("王五",9,"上海市",3));
    userList.add(new User("郝六",16,"武汉市",10));
    userList.add(new User("何七",6,"青岛市",1));
    
    int index = 0;
    Map<Integer, String[]> title = new HashMap<>();
    title.put(index++, new String[]{"name", "姓名"});
    title.put(index++, new String[]{"age", "年龄"});
    title.put(index++, new String[]{"city", "所在城市"});
    title.put(index++, new String[]{"grade", "年级"});

    HSSFWorkbook wb = this.fillExcel(title, userList, User.class, "用户信息");
    this.createExcelFile(wb, "D:\\Download\\", "用户信息");
}

public boolean createExcelFile(HSSFWorkbook book, String path, String name) {
    boolean flag = true;
    try (FileOutputStream out = new FileOutputStream(path + name + ".xls");) {
        book.write(out);
        out.flush();
    } catch (Exception e) {
        e.printStackTrace();
        flag = false;
    }
    return flag;
}
```

![image-20210406141632816](https://raw.githubusercontent.com/mervynlam/Pictures/master/20210406142122.png)