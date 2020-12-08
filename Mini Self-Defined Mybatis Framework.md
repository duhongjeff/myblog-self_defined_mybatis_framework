<center><b>Readme.md</b></center></br>
Self-Defined Mybatis Framework

Author: Jeff Du Hong

In this article, we are going to implement a self-defined mini mybatis framework by a java spring boot maven project and you will learn the below:

<b>What you can learn?</b>

- Basic understand of proxy design pattern
- Recap complete workflow of JDBC on which mybatis is built in
- Help consolidate your understanding into mybatis framework

<b>Before that, we will</b>

- Use 5 minutes to understand proxy design pattern, you may want to skip it if already handled
- Write a simple query by using JDBC
- Create a new project for self-defined mybatis framework or remove mybatis related maven dependencies in our existing project



1. About Proxy Design Pattern

   - A Proxy object is like intermediary object acting as an interface to another implementation, hiding the underlying complexity of the component

   - Assume there is a JDBC connection object which requires a lot of initial setting process and take up much system resource. We only want to create a new one on demand when there is no in the thread

   - ```java
     public interface IConnection {
         void connect();
     }
     ```

   - ```java
     //Object being proxied soon
     public class ConnectionImpl implements IConnection {
     
         public ConnectionImpl() {
             initialSetupProcess();
         }
             
         private void initialSetupProcess() {
             System.out.println("I have a lot of pre-setup work when bringing up, taking a lot of resources")
         }
       
         @Override
         public void connect() {
             System.out.println("connect successfully")
         }
         
     }
     ```

   - ```java
     //Proxy object. Both proxy and proxied objects implement IConnection interface, usually proxy //implementation is more complex on the top of proxied objects
     public class ConnectionImplProxy implements IConnection {
         private static ConnectionImpl connection;
     
         @Override
         public void connect() {
             if (connection == null) {
                 connection = new ConnectionImpl();
                 //......more stuff here......
             }
             connection.connect();
         }
     }
     ```

   - ```java
     public static void main(String[] args) {
         IConnection conn = new ConnectionImplProxy();
         conn.connect();
         conn.connect();
     }
     ```

2. Quick Recap about JDBC Query Code

   - ```java
     public static void main(String[] args) { 
       
         Connection connection = null;
         PreparedStatement preparedStatement = null;
         ResultSet resultSet = null; 
       
         try {
             /*Load mysql driver, the Class.forName() method requires the JVM to find and load the specified   				class into the memory."com.mysql.jdbc.Driver" is passed in as a parameter, which tells the JVM 					to go to "com.mysql.jdbc". Find the Driver class under this path and load it into memory*/
             Class.forName("com.mysql.jdbc.Driver");
           
             //Get database connection through driver class
             connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/mybatis?    		    	characterEncoding=utf-8","root", "root");
           
             //define sql statement ? is the placeholder
             String sql = "select * from user where username = ?";
         
             //pre handle statement
             preparedStatement = connection.prepareStatement(sql); 
         
             //Set parameters, the first parameter is the serial number of the parameter in the sql statement 				 //(starting from 1), and the second parameter is the parameter value set
             preparedStatement.setString(1, "jeff"); 
         
             //Execute the query, returning resultSet
             resultSet = preparedStatement.executeQuery();
         
             //Interate resultSet
             while(resultSet.next()){
                 System.out.println(resultSet.getString("id")+" "+resultSet.getString("username"));
             }
         } catch (Exception e) { 
           e.printStackTrace();
         }finally{ //Release resource
             if(resultSet!=null){ 
                 try {
                     resultSet.close();
                 } catch (SQLException e) {
                     e.printStackTrace(); 
                 }
             }   
             if(preparedStatement!=null){
                 try { 
                   preparedStatement.close();
                 } catch (SQLException e) {
                   e.printStackTrace();
                 }
             } 
             if(connection!=null){
                 try { 
                   connection.close();
                 } catch (SQLException e) {
                     // TODO Auto-generated catch block e.printStackTrace();
                 } 
             }
         }
     }
     ```

3. Our self-defined mybatis framework for selectList() method

   - The below demo is to display how to configure our own selectList() method

   - Below is our final result, errors prompt because we haven't written our self-defined mybatis implementation classes.

     We will start from step 1 to 6

   ![image-20201206164832691](/Users/jeff/Library/Application Support/typora-user-images/image-20201206164832691.png)

   - Pom.xml

     ```xml
     <?xml version="1.0" encoding="UTF-8"?>
     <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
         <modelVersion>4.0.0</modelVersion>
         <parent>
             <groupId>org.springframework.boot</groupId>
             <artifactId>spring-boot-starter-parent</artifactId>
             <version>2.4.0</version>
             <relativePath/> <!-- lookup parent from repository -->
         </parent>
         <groupId>com.hong.du</groupId>
         <artifactId>selfmybatis</artifactId>
         <version>0.0.1-SNAPSHOT</version>
         <name>selfmybatis</name>
         <description>Add a self defined mybatis</description>
     
         <properties>
             <java.version>11</java.version>
         </properties>
     
         <dependencies>
             <dependency>
                 <groupId>org.springframework.boot</groupId>
                 <artifactId>spring-boot-starter</artifactId>
             </dependency>
     
             <dependency>
                 <groupId>log4j</groupId>
                 <artifactId>log4j</artifactId>
                 <version>1.2.12</version> </dependency>
     
             <dependency>
                 <groupId>dom4j</groupId>
                 <artifactId>dom4j</artifactId>
                 <version>1.6.1</version>
             </dependency>
     
             <dependency>
                 <groupId>mysql</groupId>
                 <artifactId>mysql-connector-java</artifactId>
                 <version>8.0.21</version>
             </dependency>
     
             <!-- https://mvnrepository.com/artifact/jaxen/jaxen -->
             <dependency>
                 <groupId>jaxen</groupId>
                 <artifactId>jaxen</artifactId>
                 <version>1.1.6</version>
             </dependency>
     
             <dependency>
                 <groupId>org.projectlombok</groupId>
                 <artifactId>lombok</artifactId>
                 <optional>true</optional>
             </dependency>
             <dependency>
                 <groupId>org.springframework.boot</groupId>
                 <artifactId>spring-boot-starter-test</artifactId>
                 <scope>test</scope>
             </dependency>
         </dependencies>
     
         <build>
             <plugins>
                 <plugin>
                     <groupId>org.springframework.boot</groupId>
                     <artifactId>spring-boot-maven-plugin</artifactId>
                 </plugin>
             </plugins>
         </build>
     
     </project>
     ```

     

   - Resources

     ```java
     public class Resources {
     //Class.getResourceAsStream() will specify that the resource path to be loaded is consistent with the path of the package where the current class is located.
     
     //For example, if you write a MyTest class under the package com.test.mycode, then MyTest.class.getResourceAsStream("name") will find the corresponding resource under the com.test.mycode package. If the name starts with a'/', then it will be searched from the root path of the classpath.
     
     //ClassLoader.getResourceAsStream() will search from the root path of the classpath regardless of whether the resource to be searched is preceded by a'/'.
      
         public static InputStream getResourceAsStream(String filePath){
             return Resources.class.getClassLoader().getResourceAsStream(filePath);
         }
     }
     ```

   - SqlSession

     - SqlSessionFactory

       ```java
       public interface SqlSessionFactory {
           SqlSession openSession();
       }
       ```

     - DefaultSqlSessionFactory

       ```java
       public class DefaultSqlSessionFactory implements SqlSessionFactory {
       
           private Configuration cfg;
       
           public DefaultSqlSessionFactory(Configuration cfg){
               this.cfg = cfg;
           }
       
           @Override
           public SqlSession openSession() {
               return new DefaultSqlSession(cfg);
           }
       }
       ```

     - SqlSession

       ```java
       public interface SqlSession {
         
           <T> T getMapper(Class<T> daoInterfaceClass);
       
           void close();
       }
       ```

     - DefaultSqlSession

       ```java
       public class DefaultSqlSession implements SqlSession {
       
           private Configuration cfg;
           private Connection connection;
       
           public DefaultSqlSession(Configuration cfg){
               this.cfg = cfg;
               connection = DataSourceUtil.getConnection(cfg);
           }
       
       		/*
       		first param: Which class loader to use to load the proxy object
       
       		second param:动态代理类需要实现的接口
       
       		third param:动态代理方法在执行时，会调用h里面的invoke方法去执行
       		*/
           @Override
           public <T> T getMapper(Class<T> daoInterfaceClass) {
               return (T) Proxy.newProxyInstance(daoInterfaceClass.getClassLoader(),
                       new Class[]{daoInterfaceClass},new MapperProxy(cfg.getMappers(),connection));
           }
       
           @Override
           public void close() {
               if(connection != null) {
                   try {
                       connection.close();
                   } catch (Exception e) {
                       e.printStackTrace();
                   }
               }
           }
       }
       ```

     - MapperProxy

       ```java
       public class MapperProxy implements InvocationHandler {
       
           private Map<String, Mapper> mappers;
           private Connection conn;
       
           public MapperProxy(Map<String,Mapper> mappers,Connection conn){
               this.mappers = mappers;
               this.conn = conn;
           }
       
           @Override
           public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
               String methodName = method.getName();
               String className = method.getDeclaringClass().getName();
               Mapper mapper = mappers.get(key);
               if(mapper == null){
                   throw new IllegalArgumentException("wrong parameter");
               }
               return new Executor().selectList(mapper,conn);
           }
       }
       ```

       

     - SqlSessionFactoryBuilder

       ```java
       public class SqlSessionFactoryBuilder {
           public SqlSessionFactory build(InputStream config){
               Configuration cfg = XMLConfigBuilder.loadConfiguration(config);
               return new DefaultSqlSessionFactory(cfg);
           }
       }
       ```

   - cfg

     - Configuration

       ```java
       @Data
       public class Configuration {
       
           private String driver;
           private String url;
           private String username;
           private String password;
       
           private Map<String,Mapper> mappers = new HashMap<String,Mapper>();
       
           public Map<String, Mapper> getMappers() {
               return mappers;
           }
       
           public void setMappers(Map<String, Mapper> mappers) {
               this.mappers.putAll(mappers);//should cumulately add here
           }
       }
       ```

       

     - Mapper

       ```java
       public class Mapper {
       
           private String queryString;//SQL
           private String resultType;//full name of the resultType, eg com.hong.du.entity.User
       
           public String getQueryString() {
               return queryString;
           }
       
           public void setQueryString(String queryString) {
               this.queryString = queryString;
           }
       
           public String getResultType() {
               return resultType;
           }
       
           public void setResultType(String resultType) {
               this.resultType = resultType;
           }
       }
       ```

       

   - DefaultSqlSessionFactory

     ```java
     public class DefaultSqlSessionFactory implements SqlSessionFactory {
     
         private Configuration cfg;
     
         public DefaultSqlSessionFactory(Configuration cfg){
             this.cfg = cfg;
         }
     
         @Override
         public SqlSession openSession() {
             return new DefaultSqlSession(cfg);
         }
     }
     ```

     

   - Pojo and Dao

     - User

       ```java
       @Data
       public class User implements Serializable{
       
           private Integer id;
           private String username;
           private Date birthday;
           private String sex;
           private String address;
       
           @Override
           public String toString() {
               return "User{" +
                       "id=" + id +
                       ", username='" + username + '\'' +
                       ", birthday=" + birthday +
                       ", sex='" + sex + '\'' +
                       ", address='" + address + '\'' +
                       '}';
           }
       }
       ```

     - IUserDao

       ```java
       public interface IUserDao {
       
           List<User> findAll();
       }
       ```

   - Under Resource Folder

     - SqlConfigXml

       ```xml
    <?xml version="1.0" encoding="UTF-8"?>
       <!-- mybatis main configuration -->
       <configuration>
           <!-- environment setting -->
           <environments default="mysql">
               <!-- mysql environment-->
               <environment id="mysql">
                   <!-- transaction manager type-->
                   <transactionManager type="JDBC"></transactionManager>
                   <!-- datasource -->
                   <dataSource type="POOLED">
                       <property name="driver" value="com.mysql.jdbc.Driver"/>
                       <property name="url" value="jdbc:mysql://localhost:3306/eesy_mybatis"/>
                    <property name="username" value="root"/>
                       <property name="password" value="1234"/>
                </dataSource>
               </environment>
           </environments>
       
           <!-- mapping file location -->
           <mappers>
               <!--<mapper resource="com/itheima/dao/IUserDao.xml"/>-->
               <mapper class="com.itheima.dao.IUserDao"/>
           </mappers>
       </configuration>
       ```
   
     - IUserDao.xml
   
       ```xml
       <?xml version="1.0" encoding="UTF-8"?>
       <mapper namespace="com.itheima.dao.IUserDao">
           <!--find all-->
           <select id="findAll" resultType="com.itheima.domain.User">
               select * from user
           </select>
       </mapper>
       ```
   
   - Utils
   
     - DataSourceUtils
   
       ```java
       public class DataSourceUtil {
           public static Connection getConnection(Configuration cfg){
               try {
                   Class.forName(cfg.getDriver());
                   return DriverManager.getConnection(cfg.getUrl(), cfg.getUsername(), cfg.getPassword());
               }catch(Exception e){
                   throw new RuntimeException(e);
               }
           }
       }
       ```
   
     - XMLConfigBuilder
   
       ```java
       public class XMLConfigBuilder {
           /**
            * Load from xml configuration file and fill into DefaultSqlSession
            * technique：
            *      dom4j+xpath
            */
           public static Configuration loadConfiguration(InputStream config){
               try{
                   //Define the configuration object that encapsulates the connection information
                   Configuration cfg = new Configuration();
       
                   //1.Get SAXReader object
                   SAXReader reader = new SAXReader();
                   //2.Get Document object
                   Document document = reader.read(config);
                   //3.Get root
                   Element root = document.getRootElement();
                   //4.Use the method of selecting the specified node in xpath to get all the property nodes
                   List<Element> propertyElements = root.selectNodes("//property");
                   //5.Loop all properties
                   for(Element propertyElement : propertyElements){
                       //Determine which part of the node is connected to the database
                       //get the value of name property
                       String name = propertyElement.attributeValue("name");
                       if("driver".equals(name)){
                           //driver
                           String driver = propertyElement.attributeValue("value");
                           cfg.setDriver(driver);
                       }
                       if("url".equals(name)){
                           //url
                           String url = propertyElement.attributeValue("value");
                           cfg.setUrl(url);
                       }
                       if("username".equals(name)){
                           //username
                           String username = propertyElement.attributeValue("value");
                           cfg.setUsername(username);
                       }
                       if("password".equals(name)){
                           //password
                           String password = propertyElement.attributeValue("value");
                           cfg.setPassword(password);
                       }
                   }
                   //Load all mapper tages from mappers，check resource or class
                   List<Element> mapperElements = root.selectNodes("//mappers/mapper");
                   //Loop
                   for(Element mapperElement : mapperElements){
                       Attribute attribute = mapperElement.attribute("resource");
                       //get value, com/jeff/dao/IUserDao.xml
                       String mapperPath = attribute.getValue();
                       //load all info from IUserDao.xml and encapsulate them into a map
                       Map<String,Mapper> mappers = loadMapperConfiguration(mapperPath);
                       //assign mappers to configuration
                       cfg.setMappers(mappers);
                   }
                   //return Configuration
                   return cfg;
               }catch(Exception e){
                   throw new RuntimeException(e);
               }finally{
                   try {
                       config.close();
                   }catch(Exception e){
                       e.printStackTrace();
                   }
               }
       
           }
    
       
    		 /*According to the incoming parameters, parse the XML and encapsulate it in the Map
            * @param mapperPath mapping configuration file location
            * @return map contains the unique identifier obtained (key is composed of the fully qualified class 				name and method name of dao)
            * And the necessary information required for execution (value is a Mapper object, which stores the 			 executed SQL statement and the fully qualified class name of the entity class to be encapsulated)
            */
           private static Map<String,Mapper> loadMapperConfiguration(String mapperPath)throws IOException {
               InputStream in = null;
               try{
                   //Define the return object
                   Map<String, Mapper> mappers = new HashMap<String,Mapper>();
                   in = Resources.getResourceAsStream(mapperPath);
                   SAXReader reader = new SAXReader();
                   Document document = reader.read(in);
                   Element root = document.getRootElement();
                   String namespace = root.attributeValue("namespace");
                   List<Element> selectElements = root.selectNodes("//select");
                   for(Element selectElement : selectElements){
                       //get the value of id property (key part of the map)
                       String id = selectElement.attributeValue("id");
                       //get the value of resultType property (value part of the map)
                       String resultType = selectElement.attributeValue("resultType");
                       //get the value of query
                       String queryString = selectElement.getText();
                       //create Key
                       String key = namespace+"."+id;
                       //create Value
                       Mapper mapper = new Mapper();
                       mapper.setQueryString(queryString);
                       mapper.setResultType(resultType);
                       mappers.put(key,mapper);
                   }
                   return mappers;
               }catch(Exception e){
                   throw new RuntimeException(e);
               }finally{
                   in.close();
               }
           }
       }
       ```
   
     - Executor
   
       ```java
           public <E> List<E> selectList(Mapper mapper, Connection conn) {
               PreparedStatement pstm = null;
               ResultSet rs = null;
               try {
                   //1.retrieve relevant data from mapper class
                   String queryString = mapper.getQueryString();
                   String resultType = mapper.getResultType();
                   Class domainClass = Class.forName(resultType);
                   //2.get PreparedStatement object
                   pstm = conn.prepareStatement(queryString);
                   //3.execute SQL statement and get result set
                   rs = pstm.executeQuery();
                   //4.encapsulate result set
                   List<E> list = new ArrayList<E>();
                   while(rs.next()) {
                       //Instanitate target object
                       E obj = (E)domainClass.newInstance();
       
                       //retrieve meta data：ResultSetMetaData
                       ResultSetMetaData rsmd = rs.getMetaData();
                       //total columns count
                       int columnCount = rsmd.getColumnCount();
                       //loop each columns
                       for (int i = 1; i <= columnCount; i++) {
                        //get column name, starting from 1
                           String columnName = rsmd.getColumnName(i);
                        //get current column value
                           Object columnValue = rs.getObject(columnName);
                           //Assign value to obj: use Java introspection mechanism
                           //column name and entity name must be the same
                           PropertyDescriptor pd = new PropertyDescriptor(columnName,domainClass);
                           //retrieve writemethod
                           Method writeMethod = pd.getWriteMethod();
                           //assign value
                           writeMethod.invoke(obj,columnValue);
                       }
                       //put object into list
                       list.add(obj);
                   }
                   return list;
               } catch (Exception e) {
                   throw new RuntimeException(e);
               } finally {
                   release(pstm,rs);
               }
           }
       
       
           private void release(PreparedStatement pstm,ResultSet rs){
               if(rs != null){
                   try {
                       rs.close();
                   }catch(Exception e){
                       e.printStackTrace();
                   }
               }
       
            if(pstm != null){
                   try {
                    pstm.close();
                   }catch(Exception e){
                    e.printStackTrace();
                   }
               }
           }
       ```
   
     4. Finally, put spring boot test class, now everything is fine
   
        ```java
        @SpringBootTest
        class SelfmybatisApplicationTests {
        
            /**
             * beginning tutorial
             * @param args
             */
            public static void main(String[] args)throws Exception {
                //1.Read mybatis configuration file
                InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
                //2.create SqlSessionFactory (Factory class)
                SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
                SqlSessionFactory factory = builder.build(in);
                //3.Get sqlSession from the factory
                SqlSession session = factory.openSession();
                //4.Use SqlSession to create the proxy object of Dao/Mapper class
                IUserDao userDao = session.getMapper(IUserDao.class);
                //5.Execute method by using the Dao/Mapper class
                List<User> users = userDao.findAll();
                for(User user : users){
                    System.out.println(user);
                }
                //6.Release resource
                session.close();
                in.close();
            }
        
        }
        ```
   
        
   
     
   
     
