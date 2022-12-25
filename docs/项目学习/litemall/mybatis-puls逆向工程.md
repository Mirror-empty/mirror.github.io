# mybatis-puls逆向工程



```xml
<dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-generator</artifactId>
            <version>3.5.1</version>
        </dependency>
        3.5.1 版本以下为旧代码生成器
        以上为新代码生成器
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.5.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.velocity</groupId>
            <artifactId>velocity</artifactId>
            <version>1.7</version>
        </dependency>
```
新代码生成器

```java
package com.mall.db;

import com.baomidou.mybatisplus.annotation.FieldFill;
import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.generator.FastAutoGenerator;
import com.baomidou.mybatisplus.generator.config.*;
import com.baomidou.mybatisplus.generator.fill.Column;
import com.baomidou.mybatisplus.generator.fill.Property;

import java.util.Collections;
import java.util.function.Consumer;

/**
 * MyBatisPlus代码生成器（新），基本满足使用了。
 * 官方文档-代码生成器配置新：https://baomidou.com/pages/981406/
 *
 */
public class MyBatisPlusCodeGeneratorUtils {

    public static void main(String[] args) {

        /**
         *
         * TODO 注意修改 TODO处的信息
         */
        runCodeGenerator();

    }

    /**
     * 快速生成
     */
    private static void runCodeGenerator() {


        FastAutoGenerator.create(getDataSourceConfig())
                .globalConfig(getGlobalConfig())
                .packageConfig(getPackageConfig())
                .strategyConfig(getStrategyConfig())
                /**
                 * 默认的是Velocity引擎模板。也可使用其他的。注意引入模板引擎依赖
                 */
                //.templateEngine(new BeetlTemplateEngine())
                //.templateEngine(new FreemarkerTemplateEngine())
                .execute();

    }



    /**
     * 数据库配置(DataSourceConfig)
     * @return
     */
    private static DataSourceConfig.Builder getDataSourceConfig() {
        /**
         * TODO 数据库配置
         */
        String url = "jdbc:mysql://localhost:3306/litemall?useUnicode=true&characterEncoding=utf8&useSSL=true&serverTimezone=GMT";
        String username = "root";
        String password = "123456";
        DataSourceConfig.Builder builder = new DataSourceConfig.Builder(url, username, password);
        return builder;
    }

    /**
     * 全局配置(GlobalConfig)
     * @return
     */
    private static Consumer<GlobalConfig.Builder> getGlobalConfig() {
        Consumer<GlobalConfig.Builder> consumer = new Consumer<GlobalConfig.Builder>() {
            @Override
            public void accept(GlobalConfig.Builder builder) {
                builder.fileOverride() // 覆盖已生成文件
                        // 开启 swagger 模式
                        .enableSwagger()
                        // TODO 设置作者
                        .author("mall")
                        // TODO 指定输出目录
                        .outputDir("F:\\Mall\\mall_demo3\\mall-datab\\src\\main\\java")
                        .build();
            }
        };
        return consumer;
    }

    /**
     * 包配置(PackageConfig)
     */
    private static Consumer<PackageConfig.Builder> getPackageConfig() {
        Consumer<PackageConfig.Builder> consumer = new Consumer<PackageConfig.Builder>() {
            @Override
            public void accept(PackageConfig.Builder builder) {
                builder
                        // TODO 设置父包模块名
                        //.moduleName("sysxxx")
                        // TODO 设置父包名
                        .parent("com.mall.db")
                        // TODO 设置mapperXml生成路径
                        .pathInfo(Collections.singletonMap(OutputFile.mapperXml, "F:\\Mall\\mall_demo3\\mall-datab\\src\\main\\resources\\mapper"))
                        // 设置 Entity 包名
                        .entity("domain")
                        // 设置 Service 包名
                        .service("service")
                        // 设置 Service Impl 包名
                        .serviceImpl("service.impl")
                        // 设置 Controller 包名
                        .controller("controller")
                        .mapper("mapper")
                        .xml("mapper.xml")
                        .build();
            }
        };
        return consumer;
    }

    /**
     * 策略配置(StrategyConfig)
     * @return
     */
    private static Consumer<StrategyConfig.Builder> getStrategyConfig(){

        Consumer<StrategyConfig.Builder> consumer = new Consumer<StrategyConfig.Builder>() {
            @Override
            public void accept(StrategyConfig.Builder builder) {

                /**
                 * 策略配置
                 */
                builder
//                        .addTablePrefix("t_", "c_") // TODO 设置过滤表前缀
                        // TODO 设置需要生成的表
                      .addInclude(
                                "litemall_ad",
                                "litemall_address",
                                "litemall_admin",
                                "litemall_aftersale",
                                "litemall_brand",
                                "litemall_cart",
                                "litemall_category",
                                "litemall_collect",
                                "litemall_comment",
                                "litemall_coupon",
                                "litemall_coupon_user",
                                "litemall_feedback",
                                "litemall_footprint",
                                "litemall_goods",
                                "litemall_goods_attribute",
                                "litemall_goods_product",
                                "litemall_goods_specification",
                                "litemall_groupon",
                                "litemall_groupon_rules",
                                "litemall_issue",
                                "litemall_keyword",
                                "litemall_log",
                                "litemall_notice",
                                "litemall_notice_admin",
                                "litemall_order",
                                "litemall_order_goods",
                                "litemall_permission",
                                "litemall_region",
                                "litemall_role",
                                "litemall_search_history",
                                "litemall_storage",
                                "litemall_system",
                                "litemall_topic",
                                "litemall_user"
                                )
                       .build();

                /**
                 * Entity 策略配置
                 */
                builder.entityBuilder()
                        // 开启 lombok 模型

//                        .enableTableFieldAnnotation()
                        .enableChainModel()
                        .enableLombok()
                        只生产@set@get
                        // 全局主键类型
                        .idType(IdType.AUTO)
                        // TODO 格式化文件名称
                        .formatFileName("%s")
                        .addTableFills(new Column("add_time", FieldFill.INSERT))
                        .addTableFills(new Property("update_time",FieldFill.UPDATE))
                        .logicDeleteColumnName("deleted")
                        .logicDeletePropertyName("deleted")
                        .build();

                /**
                 * Mapper 策略配置
                 */
                builder.mapperBuilder()
                        // 启用 BaseResultMap 生成
                        .enableBaseResultMap()
                        // 启用 BaseColumnList
                        .enableBaseColumnList()
                        .build();

                /**
                 * Service 策略配置
                 */
                builder.serviceBuilder()
                        .formatServiceFileName("%sService")
                        .formatServiceImplFileName("%sServiceImpl")
                        .build();

                /**
                 * Controller 策略配置
                 */
                builder.controllerBuilder()
                        // 开启生成@RestController 控制器
                        .enableRestStyle()
                        // 转换文件名称
                        .formatFileName("%sController")
                        .build();
            }
        };
        return consumer;
    }

}

```
对于表的输入 可以写方法，百度就行，这里是打上去的。

好像生成的实体类没有序列化还是没有用@Data注解

旧代码生成器
```java
package com.mall.db;

import com.baomidou.mybatisplus.annotation.FieldFill;
import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.core.exceptions.MybatisPlusException;
import com.baomidou.mybatisplus.core.toolkit.StringUtils;
import com.baomidou.mybatisplus.generator.AutoGenerator;
import com.baomidou.mybatisplus.generator.config.DataSourceConfig;
import com.baomidou.mybatisplus.generator.config.GlobalConfig;
import com.baomidou.mybatisplus.generator.config.PackageConfig;
import com.baomidou.mybatisplus.generator.config.StrategyConfig;
import com.baomidou.mybatisplus.generator.config.rules.DateType;
import com.baomidou.mybatisplus.generator.config.rules.NamingStrategy;

import java.util.ArrayList;
import java.util.Scanner;

/**
 * @author: hu chang
 * Date: 2022/9/28
 * Time: 10:25
 * Description:
 */
public class CodeGenerator {

    public static String scanner(String tip) {
        Scanner scanner = new Scanner(System.in);
        StringBuilder help = new StringBuilder();
        help.append("请输入" + tip + "：");
        System.out.println(help.toString());
        if (scanner.hasNext()) {
            String ipt = scanner.next();
            if (StringUtils.isNotBlank(ipt)) {
                return ipt;
            }
        }
        throw new MybatisPlusException("请输入正确的" + tip + "！");
    }

    public static void main(String[] args) {
        // 代码生成器
        AutoGenerator mpg = new AutoGenerator();

        // 全局配置
        GlobalConfig gc = new GlobalConfig();
        String projectPath = System.getProperty("user.dir");
        gc.setOutputDir(projectPath + "/src/main/java");//设置代码生成路径
        gc.setFileOverride(true);//是否覆盖以前文件
        gc.setOpen(false);//是否打开生成目录
        gc.setAuthor("mall");//设置项目作者名称
        gc.setIdType(IdType.AUTO);//设置主键策略
        gc.setBaseResultMap(true);//生成基本ResultMap
        gc.setBaseColumnList(true);//生成基本ColumnList
        gc.setServiceName("%sService");//去掉服务默认前缀
        gc.setDateType(DateType.ONLY_DATE);//设置时间类型
        mpg.setGlobalConfig(gc);


        // 数据源配置
        DataSourceConfig dsc = new DataSourceConfig();
        dsc.setUrl("jdbc:mysql://localhost:3306/litemall?useUnicode=true&characterEncoding=utf-8&serverTimezone=GMT%2B8");
        dsc.setDriverName("com.mysql.cj.jdbc.Driver");
        dsc.setUsername("root");
        dsc.setPassword("123456");
        mpg.setDataSource(dsc);

        // 包配置
        PackageConfig pc = new PackageConfig();
        pc.setParent("com.mall");
        pc.setMapper("mapper");
        pc.setXml("mapper.xml");
        pc.setEntity("domain");
        pc.setService("service");
        pc.setServiceImpl("service.impl");
        pc.setController("controller");
        mpg.setPackageInfo(pc);

        // 策略配置
        StrategyConfig sc = new StrategyConfig();
        sc.setNaming(NamingStrategy.underline_to_camel);
        sc.setColumnNaming(NamingStrategy.underline_to_camel);
        sc.setEntityLombokModel(true);//自动lombok
        sc.setRestControllerStyle(true);
        sc.setControllerMappingHyphenStyle(true);

        sc.setLogicDeleteFieldName("deleted");//设置逻辑删除

        //设置自动填充配置
        TableFill gmt_create = new TableFill("create_time", FieldFill.INSERT);
        TableFill gmt_modified = new TableFill("update_time", FieldFill.INSERT_UPDATE);
        ArrayList<TableFill> tableFills = new ArrayList<>();
        tableFills.add(gmt_create);
        tableFills.add(gmt_modified);
        sc.setTableFillList(tableFills);



        //乐观锁
        sc.setVersionFieldName("version");
        sc.setRestControllerStyle(true);//驼峰命名


        //  sc.setTablePrefix("tbl_"); 设置表名前缀
        sc.setInclude(scanner("表名，多个英文逗号分割").split(","));
        mpg.setStrategy(sc);

        // 生成代码
        mpg.execute();
    }

}

```


参考：[mybatis官方文档](https://baomidou.com/pages/981406/#entity-%E7%AD%96%E7%95%A5%E9%85%8D%E7%BD%AE) 