##  Dao 接⼝的⼯作原理

Mapper 接口的工作原理是JDK动态代理，Mybatis运行时会使用JDK动态代理为Mapper接口生成代理对象 MappedProxy，代理对象会拦截接口方法，根据类的全限定名+方法名，唯一定位到一个MapperStatement并调用执行器执行所代表的sql，然后将sql执行结果返回。 Mapper接口里的方法，是不能重载的，因为是使用全限名+方法名 的保存和寻找策略。   

