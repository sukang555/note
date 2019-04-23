http://www.mybatis.org/spring/zh/index.html
http://www.mybatis.org/mybatis-3/zh/index.html

```java
    
    private static final String DRIVER_CLASS = "com.mysql.jdbc.Driver";
    private static final String DATA_URL = "";
    private static final String USER_AME = "";
    private static final String DATA_PASSWORD = "";
    
    
    public static void main(String[] args) {
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        ResultSet resultSet = null;
        try {
            Class.forName(DRIVER_CLASS);
            connection = DriverManager.getConnection(DATA_URL, USER_AME, DATA_PASSWORD);
            preparedStatement = connection
                    .prepareStatement(
                            " SELECT * FROM schedule_job WHERE is_delete = ? ");

            preparedStatement.setInt(1,0);
            resultSet = preparedStatement.executeQuery();
            StringBuilder stringBuilder = new StringBuilder();
            while (resultSet.next()){
                stringBuilder.append(resultSet.getString("bean_name"));
                stringBuilder.append("--");
                stringBuilder.append(resultSet.getString("method_name"));
                stringBuilder.append("\n");
            }
            System.out.println(stringBuilder.toString());
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (resultSet != null){
                try {
                    resultSet.close();
                }catch (Exception e){
                    e.printStackTrace();
                }
            }

            if (preparedStatement != null){
                try {
                    preparedStatement.close();
                }catch (Exception e){
                    e.printStackTrace();
                }
            }
            if (connection != null){
                try {
                    connection.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
        }
```
// mybatis 运行原理以及流程

```java

        
        DefaultTransactionDefinition def = new DefaultTransactionDefinition();
        PlatformTransactionManager txManager = new DataSourceTransactionManager(dataSource);

        TransactionStatus status = txManager.getTransaction(def);
        try {
            //get jdbc connection...
            //execute sql...
        }
        catch (Exception e) {
            txManager.rollback(status);
            throw e;
        }
        txManager.commit(status);
    

```



















