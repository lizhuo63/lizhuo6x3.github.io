
# 反射

+ 获取Class对象的方法：Class.forName()、class.getClass()、ClassLoader.loadClass()。

1. Class.forName() 会对类解析初始化，而ClassLoader.loadClass()会跳过初始化，直接装载，链接。
2. Class.forName()在加载类时会执行静态代码块，而ClassLoader.loadClass()只有在调用newInstance()时才会执行静态代码块。
3. Class.forName()会使用调用者的类加载器来加载目标类，而ClassLoader.loadClass()需要指定类加载器。

