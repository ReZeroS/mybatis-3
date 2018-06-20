# Dynamic Sql Structure

> http://www.cnblogs.com/fangjian0423/p/mybaits-dynamic-sql-analysis.html

## Introduction of basic classes

1. SqlNode:interface
    Represent every tag in xml like update, trim and so on.

2. SqlSource:interface implement: DynamicSqlSource, StaticSqlSource
    To map xml or annotation to real sql content 

3. BoundSql:class
    To encapsulate sql, parameter, value.
    
4. XNode:class 
    Xml node detail info.
    
5. BaseBuilder:interface
    - XMLConfigBuilder
        Analyze configLocation global xml, and will use inner XMLMapperBuilder to analyze.
    
    - XMLMapperBuilder
        Traversal MapperLocations xml, and will use inner XMLStatementBuilder to analyze.
        
    - XMLStatementBuilder
        Analyze node(select, update ..), it will use inner XMLScriptBuilder to analyze sql part and pass data to mappedStatements.
        
    - XMLScriptBuilder
        Analyze every builder of sql node. 

6. LanguageDriver:interface
   To structure sql.
   
## Source codes
    
SqlSessionFactoryBean[i:InitializingBean(Spring)].

XMLConfigBuilder.parse(configLocationPath). | XMLMapperBuilder.parse(mapperLocationsPath).

XMLLanguageDriver.createSqlSource

## Sample

1. Code

    ```xml
    <update id="update" parameterType="org.format.dynamicproxy.mybatis.bean.User">
        UPDATE users
        <trim prefix="SET" prefixOverrides=",">
            <if test="name != null and name != ''">
                name = #{name}
            </if>
            <if test="age != null and age != ''">
                , age = #{age}
            </if>
            <if test="birthday != null and birthday != ''">
                , birthday = #{birthday}
            </if>
        </trim>
        where id = ${id}
    </update>
    ```

2. Nodes Result

    Text node: UPDATE users
    Trim child node
    Text Node: where id = #{id}

3. Traversal child node
    
    - Text Or CDATA: Create TextSqlNode StaticTextSqlNode

    - Element: which means we meet a dynamic sql, try NodeHandler(XMLScriptHandler's inner interface)'s child class. 
    
        - TrimHandler、WhereHandler、SetHandler、IfHandler、ChooseHandler
    
         - Detail handler code
         
            ```java
            private class IfHandler implements NodeHandler {
                public void handleNode(XNode nodeToHandle, List<SqlNode> targetContents) {
                  List<SqlNode> contents = parseDynamicTags(nodeToHandle);
                  MixedSqlNode mixedSqlNode = new MixedSqlNode(contents);
                  String test = nodeToHandle.getStringAttribute("test");
                  IfSqlNode ifSqlNode = new IfSqlNode(mixedSqlNode, test);
                  targetContents.add(ifSqlNode);
                }
            }
            ```
    
    - Above situations need to be considered recursion function.
    
4. After above steps, you could get BoundSql now.

5. Every SqlNode will apply the content by evaluator or append it directly.


## PostNote

    Try analyze prefixOverrides
    
    